---
title: "Fluent Bit: Isolating pod logs in a multitenant EKS cluster"
date: 2022-12-07T22:00:07+02:00
tags: ["observability", "fluent bit","EKS", "k8s", "logging"]
categories: ["Observability"]
description: "Fluent Bit: Isolating pod logs in a multitenant EKS cluster"
---

# Overview
----

In this post, we discuss how to isolate log access in a multitenant AWS EKS cluster between tenants. This solution is intended for those who need to give customer team members access to their own microservice log output without replicating the log storage out to other logging solutions or out of the AWS account to minimize the risk of data leakage.


## Situation

We have a multi-tenant EKS Kubernetes cluster that is used as an internal development platform. Development teams use the platform to deploy their microservices. While the developers do not have access to the AWS account hosting the cluster, they do need access to their own log files. These log files are stored in AWS CloudWatch Logs in the EKS AWS account, to which the developers do not have access. However, the customer team does have their own AWS account.
![FluentBit](/img/fluent-bit.png)
By default, the cluster uses a Fluent Bit daemonset to forward all pod logs to the same Log Group in AWS CloudWatch Logs. This configuration allows the customer team to access their logs, even though the developers cannot log in to the EKS AWS account.

## Task

The goal is to give customer team members access to their own microservice log output, without replicating the log storage out to other logging solutions, and/or out of the AWS account to minimize the risk of data leakage. It is also important that other teams using the same multitenant cluster cannot access each other's logs. System pods and similar should log to a common Log Group so the Platform Engineering team has easy access to logs not belonging to development teams. This also ensures separation of concerns and least privilege, so that platform engineers cannot read application output unless given access to those log groups.

## Action

**Fluent Bit Overview**

A Fluent Bit daemonset is configured to ingest logs on each node. Each log source is defined as an [INPUT] section in the fluent configuration, and every event always has to consist of a timestamp and a message, and then it is tagged as it comes in to its input.

If any processing of such an event needs to be done, this is done with a [FILTER] block. This can be used to reshape, drop, or even annotate the event with metadata.

There are other features such as parsers, storage, and routers, but they will not be mentioned here. The last part to configure is an [OUTPUT].

#### Configuration

Here is a standard input configuration for Fluent Bit. It ingests container logs and gives the events the kube.* tag:

```
[INPUT]
    Name                tail
    Tag                 kube.*
    Path                /var/log/containers/*.log
    DB                  /var/fluent-bit/state/flb_kube.db
    Parser              docker
    Docker_Mode         On
    Mem_Buf_Limit       5MB
    Skip_Long_Lines     On
    Refresh_Interval    10
```

Next step is crucial, letting the Fluent Bit [kubernetes](https://docs.fluentbit.io/manual/pipeline/filters/kubernetes) filter query the Kubernetes control plane to enrich each event with metadata:

```
[FILTER]
    Name                kubernetes
    Match               kube.*
    Kube_URL            https://kubernetes.default.svc.cluster.local:443
    Merge_Log           On
    Merge_Log_Key       data
    K8S-Logging.Parser  On
    K8S-Logging.Exclude On
```

To isolate logs and limit access to information based on least privilege, we use the cloudwatch_logs output and the log_group_template key to define a unique log group based on namespace and other labels. For our purposes, this combination of labels is only relevant for customer pods. All other logs will fall back to the log_group_name and be stored in the cluster application log.
```
[OUTPUT]
    Name                cloudwatch_logs
    Match               kube.*
    region              ${AWS_REGION}
    log_group_name      /aws/containerinsights/${CLUSTER_NAME}/application
    log_group_template  application-logs/${CLUSTER_NAME}.$kubernetes['namespace_name'].$kubernetes['labels']['app.kubernetes.io/name']
    log_stream_prefix   ${HOST_NAME}-
    auto_create_group   true
    extra_user_agent    container-insights
    log_retention_days  90
```

__But there's one last catch!__ If you read the documentation for log_group_template carefully, you will see a warning is logged each time a fallback is made. To avoid unwanted warning messages in your logs, make sure to configure the Log_Level to only log Fluent Bit errors:
```
[SERVICE]
    Flush                     5
    Log_Level                 error
```

## Result

![FluentBit](/img/fluent-bit2.png)
By querying the Kubernetes control plane to enrich log messages and diverting them to the correct log storage, we can lower logging costs and reduce the risk of data leakage. Additionally, we avoid any potential issues with a single source of truth, and keep our environmental and cost footprint to a minimum by not duplicating log data.
