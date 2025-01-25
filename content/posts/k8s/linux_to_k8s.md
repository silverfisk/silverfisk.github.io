
---
title: "Kubernetes: The API-Driven Platform for Linux Engineers"
date: 2025-01-25
tags:
  - "Linux"
  - "k8s"
  - "API"
  - "GPU"
categories:
  - "kubernetes"
---


# From Linux to Kubernetes: From Files to APIs

## 1. Introduction: Bridging the Gap

For veteran Linux engineers, a core principle that drives much of Linux’s simplicity is [**“everything is a file.”**](https://en.wikipedia.org/wiki/Everything_is_a_file) Devices, processes, network interfaces, and more all appear as files or directories, making them easy to inspect and manipulate. In contrast, Kubernetes takes a similarly unifying approach but with an **API-driven** model, enabling declarative management of infrastructure at scale.

This article is for you — **a senior Linux/DevOps engineer** — to show how your Linux expertise can translate to Kubernetes, and understand where Kubernetes still depends on Linux foundations.

---

## 2. Linux’s Bedrock: “Everything is a File”

Linux typically handle resources through standard file operations such as:

- [**`/proc`**](https://www.kernel.org/doc/html/latest/filesystems/proc.html): Exposes process and kernel information (e.g., `/proc/cpuinfo`, `/proc/<PID>`).  
- [**`/sys`**](https://www.kernel.org/doc/html/latest/admin-guide/sysctl/): Provides a window into kernel subsystems (devices, power states, networking at [`/sys/class/net`](https://www.kernel.org/doc/html/latest/admin-guide/sysctl/net.html), etc.).  
- **`/dev`**: Houses device nodes for block and character devices (e.g., `/dev/sda`, `/dev/null`).  
- **`Sockets/Pipes`**: Used for IPC and network communication; they appear as special files (e.g., `/tmp/mysocket`).

This unifying file concept ensures that standard utilities like `cat`, `echo`, and `ls` can read or modify system state without specialized tooling.

---

## 3. Kubernetes: “Everything is an API”

Kubernetes extends simplicity to cluster management by presenting resources through APIs. Every object—Pods, Deployments, Services, ConfigMaps, and beyond—exists in the system as its own object but can be represented as a **YAML/JSON resource** that you query and modify via the **Kubernetes API Server**.

Kubernetes provides an abstraction for cluster management by presenting resources through APIs. Every object — Pods, Deployments, Services, ConfigMaps, and more — exists as an **API object** within the cluster's control plane. As a Kubernetes administrator, you interact with these objects using **YAML or JSON resource specifications**, which can be queried and modified via the **Kubernetes API Server**.

The `kubectl` command-line tool serves as the primary interface for interacting with the API Server, enabling you to manage resources declaratively. For example:

```bash
# Get a YAML representation of a pod
kubectl get pod <pod-name> -o yaml
```

This API-centric design is what makes so Kubernetes powerful and extensible. Every API object defined in Kubernetes is eventually translated into operations that the underlying Linux OS can understand — whether it's creating network namespaces for Pods, writing iptables rules for Services, or mounting volumes for PersistentVolumeClaims. By breaking down its core principles, we can better understand how Kubernetes bridges its API abstractions to the Linux systems it orchestrates.


**Key Aspects**

Kubernetes relies on a few core principles to manage and orchestrate resources effectively. One of the most fundamental is its **declarative configuration model**, which allows you to define the desired state of your infrastructure in files.

1. **Declarative Configuration**  
   You describe your desired state in a file:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx-deployment
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: nginx
     template:
       metadata:
         labels:
           app: nginx
       spec:
         containers:
         - name: nginx
           image: nginx:latest

   A simple `kubectl apply -f deployment.yaml` makes Kubernetes reconcile the current state to match it.

2. **Central API Server**  
   - The **control plane** tracks resources in `etcd` and manages cluster state.  
   - Tools like `kubectl` or custom controllers interact with the API to create, read, update, or delete objects.

3. **CRDs and Operators**  
   - **CRDs (Custom Resource Definitions):** These allow you to extend Kubernetes by defining new types of resources beyond the built-in ones (e.g., Pods, Deployments). A CRD acts as a blueprint for creating custom API objects that Kubernetes can manage just like native resources.

   - **Operators:** Operators are specialized controllers that manage the lifecycle of custom resources defined by CRDs. They monitor the state of these resources and take actions to ensure the desired state is maintained. This brings automation and intelligence to Kubernetes, effectively turning operational tasks into code.
     - Example: The [CloudNativePG](https://github.com/cloudnative-pg/cloudnative-pg) Operator automates tasks like provisioning PostgreSQL instances, setting up replication, and performing backups or failover operations based on the `PostgresCluster` CRD.
     - Key Components:
       1. **Custom Resources**: Created using the CRD.
       2. **Reconciliation Logic**: The Operator’s code continuously ensures the desired state of the resource is achieved, even if failures occur.

3. **CRDs and Operators**  
   - **CRDs (Custom Resource Definitions):** These allow you to extend Kubernetes by defining new types of resources beyond the built-in ones (e.g., Pods, Deployments). A CRD acts as a blueprint for creating custom API objects that Kubernetes can manage just like native resources.
     - Example: The `Cluster` CRD used by the CloudNativePG Operator to manage PostgreSQL clusters.

   - **Operators:** Operators are specialized controllers that manage the lifecycle of custom resources defined by CRDs. They monitor the state of these resources and take actions to ensure the desired state is maintained. This brings automation and intelligence to Kubernetes, effectively turning operational tasks into code.
     - Example: The CloudNativePG Operator automates tasks like provisioning PostgreSQL instances, initializing the database, and ensuring high availability or backups based on the `Cluster` CRD.
     - Key Components:
       1. **Custom Resources**: Created using the CRD.
       2. **Reconciliation Logic**: The Operator’s code continuously ensures the desired state of the resource is achieved, even if failures occur.

    **How It Works**:

    1. Define the `Cluster` CRD:
      The CloudNativePG Operator introduces a CRD that allows users to define their desired PostgreSQL cluster configuration. Here’s an example:
        ```yaml
        apiVersion: postgresql.cnpg.io/v1
        kind: Cluster
        metadata:
          name: cluster-example-initdb
        spec:
          instances: 3

          bootstrap:
            initdb:
              database: app
              owner: app
              secret:
                name: app-secret

          storage:
            size: 1Gi
        ```

    2. Deploy the CloudNativePG Operator:
      The Operator monitors the `Cluster` resource and translates it into Kubernetes-native components such as Pods, StatefulSets, PersistentVolumeClaims, and Services.

    3. The Operator’s Lifecycle Management:
      When the `Cluster` resource is applied using `kubectl apply -f cluster-example-initdb.yaml`, the Operator:
      - Creates the specified number of PostgreSQL instances (three in this case).
      - Initializes the database (`app`) with the specified owner (`app`) and secrets for authentication.
      - Allocates persistent storage of `1Gi` per instance for database files.

    4. Automated Reconciliation:
      The Operator continuously ensures the PostgreSQL cluster matches the specified configuration. For example:
      - If a Pod crashes, the Operator recreates it.
      - If you update the configuration, the Operator applies the changes to the cluster.

## 4. Where Kubernetes Defers to Linux

Even though Kubernetes is heavily API-driven, certain tasks rely on more directly on the underlying operating system. Here are the main areas:

### 4.1 CNI (Container Network Interface)

- **What it is**: A specification that network plugins use to configure pod networking — assigning IP addresses, setting up routes, etc.  
- **Why it’s not purely an API**:  
  - The **CNI plugin** runs on each node host, modifying Linux networking constructs (iptables, routes, [veth pairs](https://man7.org/linux/man-pages/man4/veth.4.html)).  
  - Kubernetes itself just invokes the plugin; the heavy lifting happens at the Linux level.  

**Troubleshooting Example**:
```bash
# Check iptables rules on a node
iptables -L -t nat

# List network namespaces
ip netns list
```
If you see unexpected routing or NAT rules, the issue often lies in the CNI plugin configuration, not in the Kubernetes API definitions.

---

### 4.2 CSI (Container Storage Interface)

- **What it is**: A standardized interface for storage plugins to provision and attach volumes dynamically.  
- **Why it’s not purely an API**:  
  - While you create **PersistentVolumeClaims (PVCs)** via the Kubernetes API, the actual block devices and mount points live on the node’s filesystem.  
  - The **CSI plugin** is responsible for orchestrating filesystem operations (e.g., `mount`, `umount`) on the host.

**Troubleshooting**:
```bash
# Inspect where volumes are mounted
mount | grep pvc

# Check PVC binding status
kubectl describe pvc my-volume-claim
```
If the PVC is stuck in `Pending`, the plugin or underlying storage backend might be misconfigured.

---

### 4.3 CRI (Container Runtime Interface)

- **What it is**: The interface between kubelet and container runtimes (e.g., containerd, CRI-O).  
- **Why it’s not purely an API**:  
  - Container logs, filesystems, and runtime states live on the host.  
  - Although kubelet interacts with the runtime using gRPC calls (a form of API), the underlying container management relies on Linux cgroups, namespaces, and file paths (e.g., `/var/lib/containerd`).

**Troubleshooting**:
```bash
# Logs from containerd runtime
journalctl -u containerd

# Check container logs at node level (locations may vary)
ls /var/log/containers/
```

---

### 4.4 Other Nuances

- **kube-proxy**: Implements Kubernetes `Service` networking by writing iptables or IPVS rules directly on the node.  
- **CoreDNS**: Configured via Kubernetes objects, but ultimately relies on Linux’s DNS resolution stack.

---

## 5. Putting It Together: Debugging with Linux & Kubernetes Tools

When a service misbehaves in Kubernetes, you’ll combine **API-based** and **file-based** diagnostics:

1. **Check the Kubernetes API**:
   ```bash
   kubectl get pods
   kubectl describe pod <pod>
   kubectl logs <pod>
   ```
   This reveals what the control plane knows.

2. **Inspect the Host (Linux)**:
   ```bash
   # Network debugging
   iptables -L -t nat
   ip route

   # Storage debugging
   mount | grep pvc

   # GPU debugging (on a node)
   dmesg | grep nvidia
   ```
Understanding which layer you’re debugging—API or Linux—will guide you to the right tools.

---

## 6. Best Practices for Linux Engineers in Kubernetes

1. **Know the Boundaries**  
   - Use Kubernetes commands (`kubectl`) for API-level insights.  
   - Use Linux commands (`ip`, `mount`, `ls /dev`) for host-level checks, including GPU nodes.

2. **Avoid Over-Reliance on `kubectl`**  
   - Some failures manifest at the OS level; a quick `ip a` or `mount` can unearth root causes faster than scanning YAML files.

3. **Use Official Docs**  
   - **Kubernetes**: [Kubernetes Documentation](https://kubernetes.io/docs/home/)  
   - **Linux**: [Kernel Docs](https://www.kernel.org/doc/html/latest/)  
   - **NVIDIA Device Plugin**: [NVIDIA Documentation](https://docs.nvidia.com/datacenter/cloud-native/)

---

## 7. Conclusion

Kubernetes’ *“everything is an API”* parallels Linux’s *“everything is a file”*, just at a higher abstraction. For the seasoned Linux engineer, this means:

- You already have the foundational skillset to diagnose Kubernetes issues, from networking & performance to GPUs.  
- Kubernetes simply introduces another layer of APIs to handle container orchestration at scale.  
- By blending **API-level** knowledge with your **Linux-level** knowledge, you can tackle complex issues with confidence.
