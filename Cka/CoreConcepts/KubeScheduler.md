# Kube Scheduler (Kubernetes)

## Introduction
The **Kube Scheduler** is responsible for **deciding which node a pod should run on**.

Important:
> The scheduler only **selects the node**.  
> The **Kubelet actually creates and runs the pod** on that node.

---

# Role of the Scheduler

In a Kubernetes cluster with many nodes and pods, the scheduler ensures that:

- Pods are placed on suitable nodes
- Resource requirements are satisfied
- Workloads are distributed efficiently

It determines the **best node for each pod**.

---

# Why Scheduling is Important

Clusters may contain:

- Nodes with different CPU and memory capacities
- Pods with different resource requirements
- Nodes dedicated to specific workloads

The scheduler ensures the **right pod runs on the right node**.

---

# Scheduler Workflow

When a new pod is created:

1. Pod is created without a node assignment
2. Scheduler detects the unscheduled pod
3. Scheduler selects the best node
4. Scheduler informs the **API Server**
5. API Server updates **etcd**
6. **Kubelet on the selected node** creates the pod

---

# Scheduler Decision Process

The scheduler selects a node in **two main phases**.

---

## 1. Filtering Phase

The scheduler removes nodes that **do not meet the pod requirements**.

Example filters:

- Insufficient CPU
- Insufficient memory
- Node restrictions
- Policy constraints

Only nodes that satisfy the requirements move to the next phase.

---

## 2. Scoring (Ranking) Phase

The remaining nodes are **scored and ranked**.

The scheduler assigns scores based on **priority functions**.

Score range:

```

0 – 10

````

Nodes with better resource availability receive higher scores.

Example criteria:

- Remaining CPU
- Remaining memory
- Balanced resource usage

The node with the **highest score is selected**.

---

# Example Scheduling Scenario

Pod requires:

- 2 CPU
- 2 GB RAM

Cluster nodes:

| Node | CPU Available | Result |
|-----|---------------|------|
| Node1 | 1 CPU | Filtered out |
| Node2 | 1 CPU | Filtered out |
| Node3 | 6 CPU | Candidate |
| Node4 | 10 CPU | Candidate |

Nodes 3 and 4 pass the filtering phase.

During scoring:

- Node4 leaves more free resources
- Node4 gets a higher score

Final decision → **Node4 selected**

---

# Scheduler Customization

Kubernetes scheduling behavior can be customized using:

- Resource requirements
- Node selectors
- Taints and tolerations
- Node affinity rules
- Custom schedulers

It is also possible to **create your own scheduler**.

---

# Installing Kube Scheduler

If installing Kubernetes manually:

1. Download the **kube-scheduler binary**
2. Extract the binary
3. Run it as a service

Example command:

```bash
kube-scheduler
````

A **scheduler configuration file** can be specified during startup.

---

# Viewing Scheduler in a Cluster

How you view it depends on cluster setup.

---

## kubeadm Setup

When Kubernetes is installed using **kubeadm**:

* Scheduler runs as a **pod**
* Located in the **kube-system namespace**

Check with:

```bash
kubectl get pods -n kube-system
```

Configuration file location:

```
/etc/kubernetes/manifests/kube-scheduler.yaml
```

---

## Manual Setup

If installed manually, it runs as a **system service**.

View running process:

```bash
ps aux | grep kube-scheduler
```

---

# Key Takeaways

* **Kube Scheduler decides where pods run.**
* It does **not create pods**; Kubelet does.
* Scheduling happens in two phases:

  * Filtering
  * Scoring
* Scheduler selects the **best node based on resource availability and policies**.
* Runs as:

  * a **pod (kubeadm setup)**
  * a **service (manual setup)**.

---
