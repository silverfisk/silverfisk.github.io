


+++ 
date = 2022-10-27T14:30:00+02:00
title = "Let karpenters just-in-time scheduler manage disk pressure"
slug = "" 
tags = "k8s, GitLab, GitOps, karpenter"
categories = []
thumbnail = "/img/karpenter-logo.png"
description = "Kapacity planning in kubernetes"
+++



## Overview
----

Capacity planning in Kubernetes can be challenging.
Karpenter is useful for automatically rightscaling and rightsizing Kubernetes nodes in an EKS cluster, but it usually only considers CPU and memory usage. We experienced difficulties troubleshooting this issue because the node storage was not full at the time we checked it. The disk usage would spike to almost capacity (98-99%) during the execution of GitLab CI jobs, then decrease after the job was completed. This made it difficult to identify the cause of the issue.

To address this issue, we used the```Provisioner``` kind to configure the just-in-time node scheduler to place bids for new (or larger) spot instance nodes. In addition to providing a better ROI on the cluster, this approach also helps to improve Cloud Sustainability targets, such as [SUS05-BP01](https://docs.aws.amazon.com/wellarchitected/latest/sustainability-pillar/sus_sus_hardware_a2.html), by using the minimum amount of hardware needed to meet your needs if you follow the new [AWS Well Architected Sustaninability Pillar](https://docs.aws.amazon.com/wellarchitected/latest/sustainability-pillar/sustainability-pillar.html).

This lets karpenter configure nodes to automatically perform garbage collection, and taint itself to avoid further pod scheduling. As a last resort it will kill greedy CI jobs to keep the cluster as a whole more consistent, performant and stable.
{{%portfolio image=/img/k8s_rightsizing.png %}}

### Karpenter kubelet Configuration
----


One way to address the issue of full disks on Kubernetes nodes is to use Karpenter to set a kubeletConfiguration. This allows the [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/) to start the [Node-pressure Eviction](https://kubernetes.io/docs/concepts/scheduling-eviction/node-pressure-eviction/) process, which proactively terminates pods to reclaim resources on the affected node.

While this approach may cause some GitLab CI jobs to fail when node disks are full, it also reduces the impact on other jobs by limiting the "blast radius" of a job that fills up disks. In general, it can help to keep the cluster more consistent, performant, and stable.


Check if the cluster already has a kubeletConfiguration:
{{% /portfolio %}}

```shell
[localhost]$ kubectl get provisioners.karpenter.sh default  -oyaml | yq .spec.kubeletConfiguration
```
----

## Configuration knobs

Setting these values may cause some pods and their CI jobs to be killed as a last resort. In our setup, the most common issue with full disks was CI jobs executing in very large container images which caused the image storage to fill up. It is important to investigate whether adding more node disk space would be a more effective preventive solution, as this is more of a way to handle node scaling if the disks fill up despite preventive actions already being taken.

Before any jobs are evicted due to DiskPressure, the kubelet will attempt to reclaim resources by culling dead pods and images.

```yaml
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
metadata:
  name: default
<...>
spec:
  kubeletConfiguration:
        ephemeral-storage: 15Gi
      kubeReserved:
        cpu: 200m
        memory: 100Mi
        ephemeral-storage: 3Gi
      evictionHard:
        memory.available: 5%
        nodefs.available: 10%
        nodefs.inodesFree: 10%
      evictionSoft:
        memory.available: 500Mi
        nodefs.available: 15%
        nodefs.inodesFree: 15%
      evictionSoftGracePeriod:
        memory.available: 3m
        nodefs.available: 1m30s
        nodefs.inodesFree: 2m
```


