


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

It can be challenging to do capacity planning in kubernetes.
Karpenter can be used for automatic rightscaling (and rightsizing) of kubernetes nodes in an EKS cluster, but it usually only handle CPU and memory. We had an issue where GitLab CI jobs quickly filled up node storage. The node disk usage was big enough to almost fill the disk to 98-99%. The CI job itself did not fail, but parallell jobs sheduled on the same node sometimes would.

GitLab creates a new pod for each CI job.

For this the ```Provisioner``` kind is used to configure the just-in-time node scheduler to place bids for new (or bigger) spot instance nodes. In addition to the better ROI on the cluster, it also improve Cloud Sustainability targets such as [SUS05-BP01](https://docs.aws.amazon.com/wellarchitected/latest/sustainability-pillar/sus_sus_hardware_a2.html) to use the minimum amount of hardware to meet your needs if you follow the new AWS Well Architected Sustaninability Pillar.

This lets karpenter configure the nodes to automatically perform garbage collection, and taint itself to avoid further pod scheduling. As a last resort it will kill greedy CI jobs to keep the cluster as a whole more consistent, performant and stable.
{{%portfolio image=/img/k8s_rightsizing.png %}}

### Karpenter kubelet Configuration
----


The idea here is to let Karpenter set a kubeletConfiguration, so the [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/) starts the [Node-pressure Eviction](https://kubernetes.io/docs/concepts/scheduling-eviction/node-pressure-eviction/) process to proactively terminate pods to reclaim resources on node.

This may fail some GitLab CI jobs when node disks are full, but it also reduce the blast radius when a job fill up disks for other jobs.


Check if the cluster already has a kubeletConfiguration:
{{% /portfolio %}}

```shell
[localhost]$ kubectl get provisioners.karpenter.sh default  -oyaml | yq .spec.kubeletConfiguration
```
----

## Configuration knobs

Setting these values may actually kill some pods and their CI jobs as a last resort. In our setup the most common issue of full disks are when very CI jobs are executed in very big container images. Naturally it should be investigated if more node disk space should be added as a preventive solution, since this is more a way to handle node scaling *if the disks fill up in spite of the preventive actions already taken*.

Before any jobs are evicted because of DiskPressure, the kubelet will try to reclaim resources by culling dead pods and images.

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


