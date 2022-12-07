---
title: "Fluent Bit: Isolating pod logs in a multitenant EKS cluster"
date: 2022-12-07T22:00:07+02:00
tags: ["observability", "fluent bit","EKS", "k8s", "logging"]
categories: ["Observability"]
description: "Fluent Bit: Isolating pod logs in a multitenant EKS cluster"
---

# Overview
----

How to isolate log access in a multitenant AWS EKS cluster between tenants. I'm writing a small post here in case some one else is searching for a similar solution since I could not find any working examples or documentation on this.


## Situation

We have a multi-tenant EKS Kubernetes cluster that is used as an internal development platform. Development teams use the platform to deploy their microservices. While the developers do not have access to the AWS account hosting the cluster, they do need access to their own log files. These log files are stored in AWS CloudWatch Logs in the EKS AWS account, to which the developers do not have access. However, the customer team does have their own AWS account.

By default, the cluster uses a Fluent Bit daemonset to forward all pod logs to the same Log Group in AWS CloudWatch Logs. This configuration allows the customer team to access their logs, even though the developers cannot log in to the EKS AWS account.


## Task

Give customer team members access to their own microservice log output, without replicating the log storage out to other logging solutions, and/or out of the AWS account to minimise risk of data leakage. It is also important that other teams using the same multitenant cluster cannot access each others logs.
System pods and similar should log to a common Log Group so the Platform Engineering team have easy access to logs not belonging to development teams. This also ensures separation of concerns and least privilege, so that platform engineers cannot read application output unless given access to those log groups.

## Action

**First short overview of how Fluent Bit works in kubernetes:**
A Fluent Bit daemonset is configured to ingest logs on each node, each log source is defined as an ```[INPUT]``` section in the fluent configuration and every event always have to consist of a _timestamp_ and a _message_, and then it is _tagged_ as it comes in to its input.

If any processing of such an event needs to be done, this is done with a ```[FILTER]``` block. This can be used to reshape, drop or even annotate the event with metadata.

There are other features such as parsers, storage and routers, but they will not be mentioned here. However the last part to configure is an ```[OUTPUT]```

#### Configuration

Say that you have a standard input. It is standard, but I will show it here just to have a working example. Se here that we ingest container logs and give the events the ```kube.*``` tag:

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

We then let the [cloudwatch_logs](https://docs.fluentbit.io/manual/pipeline/outputs/cloudwatch) output use the ```log_group_template``` key to define a unique log group based on _namespace_ and other labels seen below. For us, this combination of labels would correspond so that it only resolves for customer pods. All other logs will fall back to the ```log_group_name``` and end up in a cluster application log.

```
[OUTPUT]
    Name                cloudwatch_logs
    Match               kube.*
    region              $${AWS_REGION}
    log_group_name      /aws/containerinsights/$${CLUSTER_NAME}/application
    log_group_template  application-logs/$${CLUSTER_NAME}.$kubernetes['namespace_name'].$kubernetes['labels']['app.kubernetes.io/name']
    log_stream_prefix   $${HOST_NAME}-
    auto_create_group   true
    extra_user_agent    container-insights
    log_retention_days  90
```

__But there's one last catch!__ If you read the documentation for _log_group_template_ carefully, a warning is logged each time a fallback is made. You need to make sure to configure Log_Level to only log Fluent Bit errors, or your logs will be filled with unwanted warning messages:
```
[SERVICE]
    Flush                     5
    Log_Level                 error
```

## Result

To summarize, log files has now been isolated so that access to information can limited according to _least privilege_. We let Flutent Bit query the Kubernetes kontrol plane to enrich the message so it is diverted to the correct log storage, and lowering the logging costs by only logging errors from the Fluent daemon itself.
Additionally, we avoid any potential issues with single source of truth. And since log data is not duplicated, environmental and cost footprint is kept at a minimum while lowering the risk of data leakage.
