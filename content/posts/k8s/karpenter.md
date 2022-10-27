+++ 
date = 2022-10-27T14:30:00+02:00
title = "Let karpenter just-in-time scheduler manage disk pressure"
slug = "" 
tags = "k8s, GitLab, GitOps, karpenter"
categories = []
thumbnail = "/img/karpenter-logo.png"
description = "Let karpenter just-in-time scheduler manage disk pressure"
+++

## Overview
----
Karpenter is used for automatic rightscaling (and rightsizing) kubernetes nodes in a cluster.

For this the ```Provisioner``` kind is used to configure the just-in-time node scheduler.

An issue we have is that GitLab CI jobs fill up node storage. GitLab creates a new pod for each CI job.

This lets karpenter configure the nodes to automatically perform garbage collection, and taint itself to avoid further pod scheduling. As a last resort it will kill greedy CI jobs to keep the cluster as a whole more consistent and performant.


### Karpenter kubelet Configuration
----

The idea here is to let Karpenter set a kubeletConfiguration, so the [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/) starts the [Node-pressure Eviction](https://kubernetes.io/docs/concepts/scheduling-eviction/node-pressure-eviction/) process to proactively terminate pods to reclaim resources on node.

This may fail some GitLab CI jobs when node disks are full, but it also reduce the blast radius when a job fill up disks for other jobs.


Check if the cluster already has a kubeletConfiguration:

```shell
[localhost]$ kubectl get provisioners.karpenter.sh default  -oyaml | yq .spec.kubeletConfiguration
```
----

## Configuration knobs

Setting these values may actually kill some pods and their CI jobs as a last resort. In our setup the most common issue of full disks are when very CI jobs are executed in very big container images.

Before any jobs are evicted because of DiskPressure, the kubelet will try to reclaim resources by culling dead pods and images.

```yaml
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

----

Setting this would also taint the node with ```node.kubernetes.io/disk-pressure```. Given this, the node is tainted so no new CI pods is started, a new instance spins up with enough disk storage to acceppt the new jobs.

```yaml

taints:
    - key: node.kubernetes.io/disk-pressure
      effect: NoSchedule
```
