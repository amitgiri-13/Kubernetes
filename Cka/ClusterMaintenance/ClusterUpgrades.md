# Kubernetes Cluster Upgrade 

## Version Compatibility Rules

The **kube-apiserver** is the central component of Kubernetes. Other components communicate with it, so they must follow version compatibility rules.

### Allowed Version Skew

| Component               | Supported Version   |
| ----------------------- | ------------------- |
| kube-apiserver          | Current version (X) |
| kube-controller-manager | X or X-1            |
| kube-scheduler          | X or X-1            |
| kubelet                 | X, X-1, or X-2      |
| kube-proxy              | X, X-1, or X-2      |
| kubectl                 | X+1, X, or X-1      |

Example if API Server is **v1.10**:

| Component          | Allowed Versions      |
| ------------------ | --------------------- |
| API Server         | v1.10                 |
| Controller Manager | v1.10 or v1.9         |
| Scheduler          | v1.10 or v1.9         |
| Kubelet            | v1.10, v1.9, or v1.8  |
| Kube Proxy         | v1.10, v1.9, or v1.8  |
| kubectl            | v1.11, v1.10, or v1.9 |

### Important Rule

No control-plane component should run a version **higher** than the kube-apiserver.

---

# Kubernetes Support Policy

Kubernetes supports only the latest **three minor versions**.

Example:

| Latest Version | Supported Versions  |
| -------------- | ------------------- |
| v1.12          | v1.12, v1.11, v1.10 |
| v1.13          | v1.13, v1.12, v1.11 |

If your cluster is on v1.10 and v1.13 is released, you should upgrade before v1.10 becomes unsupported.

---

# Upgrade Strategy

### Recommended Approach

Upgrade **one minor version at a time**.

Correct:

```text
v1.10 → v1.11 → v1.12 → v1.13
```

Incorrect:

```text
v1.10 → v1.13
```

---

# Upgrade Methods

## Managed Kubernetes

Examples:

* Google Cloud Kubernetes Engine (GKE)
* Amazon Web Services EKS
* Microsoft AKS

Typically upgraded through the cloud provider console or CLI.

---

## kubeadm Clusters

Use kubeadm commands to plan and perform upgrades.

---

## Manually Built Clusters

Upgrade each Kubernetes component manually.

---

# Upgrade Process Overview

A cluster upgrade has two major phases:

## 1. Upgrade Control Plane

Upgrade:

* kube-apiserver
* kube-controller-manager
* kube-scheduler

During this process:

* Control plane becomes temporarily unavailable.
* Existing applications continue running on worker nodes.
* Users can still access applications.
* Cluster management operations are unavailable.

Unavailable during upgrade:

* kubectl operations
* Deployments
* Scaling
* Resource modifications
* Automatic controller actions

---

## 2. Upgrade Worker Nodes

After the control plane upgrade completes, upgrade worker nodes.

---

# Worker Node Upgrade Strategies

## Strategy 1: Upgrade All Nodes Together

```text
Node1 DOWN
Node2 DOWN
Node3 DOWN
```

Advantages:

* Simple

Disadvantages:

* Application downtime

---

## Strategy 2: Upgrade One Node at a Time

```text
Upgrade Node1
Upgrade Node2
Upgrade Node3
```

Advantages:

* No application downtime
* Preferred production approach

Process:

1. Drain node
2. Upgrade node
3. Uncordon node
4. Repeat for next node

---

## Strategy 3: Replace Nodes

Common in cloud environments.

Process:

1. Create new nodes with newer Kubernetes version.
2. Move workloads.
3. Remove old nodes.

Advantages:

* Safer
* Faster rollback
* Common in auto-scaling environments

---

# kubeadm Upgrade Workflow

## Step 1: Check Upgrade Plan

```bash
kubeadm upgrade plan
```

Provides:

* Current cluster version
* kubeadm version
* Available upgrades
* Component versions
* Required upgrade commands

---

## Step 2: Upgrade kubeadm

Example:

```bash
apt-get install kubeadm=1.12.x
```

### Important

Upgrade **kubeadm first** before upgrading the cluster.

---

## Step 3: Upgrade Control Plane

```bash
kubeadm upgrade apply v1.12.x
```

This:

* Downloads images
* Updates control-plane components
* Applies cluster changes

---

## Step 4: Upgrade kubelet on Control Plane Node

```bash
apt-get install kubelet=1.12.x
systemctl restart kubelet
```

---

# Why `kubectl get nodes` Still Shows Old Version

After upgrading the control plane:

```bash
kubectl get nodes
```

may still display the old version.

Reason:

* The command shows the **kubelet version** running on nodes.
* It does not display the API Server version.

Once kubelet is upgraded and restarted, the node version updates.

---

# Worker Node Upgrade Procedure

## Drain Node

Move workloads safely:

```bash
kubectl drain worker-node-1 --ignore-daemonsets
```

Drain performs:

* Pod eviction
* Workload migration
* Node cordoning

---

## Upgrade kubeadm

```bash
apt-get install kubeadm=1.12.x
```

---

## Upgrade Node Configuration

```bash
kubeadm upgrade node
```

---

## Upgrade kubelet

```bash
apt-get install kubelet=1.12.x
systemctl restart kubelet
```

---

## Uncordon Node

Make node schedulable again:

```bash
kubectl uncordon worker-node-1
```

---

# Important Note About Uncordon

After:

```bash
kubectl uncordon worker-node-1
```

Pods do **not automatically return** to that node.

Uncordon only allows future scheduling.

Pods move there only when:

* New pods are created.
* Existing pods are deleted and recreated.
* Other nodes are drained.

---

# Common Upgrade Commands

## Check Upgrade Availability

```bash
kubeadm upgrade plan
```

## Upgrade Control Plane

```bash
kubeadm upgrade apply <version>
```

## Drain Worker Node

```bash
kubectl drain <node> --ignore-daemonsets
```

## Upgrade Node Configuration

```bash
kubeadm upgrade node
```

## Restart kubelet

```bash
systemctl restart kubelet
```

## Re-enable Scheduling

```bash
kubectl uncordon <node>
```

---

# Exam Tips (CKA)

1. Upgrade one minor version at a time.
2. Upgrade kubeadm before cluster upgrade.
3. Upgrade control plane before worker nodes.
4. Use `kubectl drain` before upgrading worker nodes.
5. Use `kubectl uncordon` after upgrade.
6. `kubectl get nodes` shows kubelet versions.
7. Kubernetes supports only the latest three minor versions.
8. Worker nodes may be up to two versions behind the kube-apiserver.
9. Control-plane components must never be newer than the kube-apiserver.
10. Node upgrades can be performed with zero application downtime by upgrading one node at a time.
