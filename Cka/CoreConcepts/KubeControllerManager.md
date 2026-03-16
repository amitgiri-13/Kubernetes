# Kube Controller Manager

## Introduction
The **Kube Controller Manager** is a core component of the Kubernetes control plane.  
It runs multiple **controllers** that continuously monitor the cluster and ensure the system reaches the **desired state**.

---

# What is a Controller?

A **controller** is a process that:

1. Continuously monitors the state of cluster resources.
2. Takes corrective actions to bring the system to the desired state.

Controllers work by communicating with the **Kube API Server**.

---

# Node Controller

The **Node Controller** monitors the health of nodes.

### Responsibilities
- Checks node status
- Detects node failures
- Reschedules workloads

### How it works

| Parameter | Description |
|---|---|
| Node Monitor Period | Checks node health every **5 seconds** |
| Grace Period | Waits **40 seconds** before marking node unreachable |
| Pod Eviction Timeout | Waits **5 minutes** before rescheduling pods |

### Failure Handling

1. Node stops sending heartbeat
2. Node marked **Unreachable**
3. Wait **5 minutes**
4. Pods are moved to healthy nodes (if part of ReplicaSet)

---

# Replication Controller

The **Replication Controller** ensures the desired number of pods are running.

### Responsibilities

- Monitors pods in a **ReplicaSet**
- Creates new pods if any pod dies
- Maintains the desired pod count

Example:

Desired Pods = 3

If 1 pod crashes → Controller automatically creates a new one.

---

# Other Controllers in Kubernetes

Kubernetes contains many controllers responsible for different resources.

Examples include:

- Deployment Controller
- Service Controller
- Namespace Controller
- Persistent Volume Controller
- Job Controller

These controllers implement the **logic behind Kubernetes features**.

---

# Kubernetes Controller Manager

All controllers are packaged into a single component called:

**Kube Controller Manager**

It runs as a **single process** that manages all controllers.

---

# Installing Kube Controller Manager

Steps:

1. Download from Kubernetes release page
2. Extract the binaries
3. Run as a service

Example:

```bash
kube-controller-manager
````

---

# Controller Manager Configuration

Controller Manager supports many configuration options.

Example parameters:

| Option                        | Description                        |
| ----------------------------- | ---------------------------------- |
| `--node-monitor-period`       | Node health check interval         |
| `--node-monitor-grace-period` | Time before marking node unhealthy |
| `--pod-eviction-timeout`      | Time before rescheduling pods      |

---

# Enabling or Disabling Controllers

You can control which controllers run using:

```bash
--controllers
```

Example:

```bash
--controllers=*,bootstrapsigner,tokencleaner
```

By default, **all controllers are enabled**.

---

# Viewing Controller Manager in a Cluster

How you view it depends on how Kubernetes was installed.

---

## Kubeadm Cluster

In **kubeadm setups**, the controller manager runs as a **pod** in the `kube-system` namespace.

Check it with:

```bash
kubectl get pods -n kube-system
```

Configuration file location:

```
/etc/kubernetes/manifests/kube-controller-manager.yaml
```

---

## Non-Kubeadm Setup

Controller manager runs as a **system service**.

You can inspect it here:

```
/etc/systemd/system/
```

or view the running process:

```bash
ps aux | grep kube-controller-manager
```

---

# Key Takeaways

* Controllers maintain the **desired state** of the cluster.
* They continuously **monitor and take corrective actions**.
* Examples: **Node Controller, Replication Controller**.
* All controllers run inside the **Kube Controller Manager**.
* Configuration depends on cluster setup (**kubeadm or manual**).

---
