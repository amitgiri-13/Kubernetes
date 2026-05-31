# Kubernetes Autoscaling 

## 1. What is Scaling?

Scaling means increasing or decreasing resources to handle application demand.

### Vertical Scaling (Scale Up)

Increase resources of an existing server/pod.

**Example:**

* 2 CPU → 4 CPU
* 4 GB RAM → 8 GB RAM

**Characteristics**

* Same instance/server
* May require restart or downtime
* Limited by maximum hardware capacity

### Horizontal Scaling (Scale Out)

Add more instances.

**Example:**

* 1 server → 3 servers
* 2 pods → 10 pods

**Characteristics**

* Better availability
* Better fault tolerance
* Preferred in Kubernetes

---

# 2. Scaling in Kubernetes

Kubernetes supports scaling at two levels:

## A. Workload Scaling

Scaling applications (Pods).

### Horizontal Workload Scaling

Increase/decrease number of Pods.

```text
2 Pods → 5 Pods
```

### Vertical Workload Scaling

Increase Pod CPU/Memory resources.

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"
```

↓

```yaml
resources:
  requests:
    cpu: "1"
    memory: "1Gi"
```

---

## B. Cluster Infrastructure Scaling

Scaling Kubernetes nodes.

### Horizontal Cluster Scaling

Add/remove worker nodes.

```text
3 Nodes → 5 Nodes
```

### Vertical Cluster Scaling

Increase resources of existing nodes.

```text
4 CPU Node → 8 CPU Node
```

Less common because it may require node replacement or downtime.

---

# 3. Manual Scaling Methods

## Scale Workloads Horizontally

Increase replicas manually:

```bash
kubectl scale deployment nginx --replicas=5
```

---

## Scale Workloads Vertically

Edit CPU/Memory requests and limits:

```bash
kubectl edit deployment nginx
```

Modify:

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"
  limits:
    cpu: "1"
    memory: "1Gi"
```

---

## Scale Cluster Infrastructure

Add a new node:

```bash
kubeadm join ...
```

New node joins cluster and workloads can be scheduled on it.

---

# 4. Automated Scaling Components

## Cluster Autoscaler

**Purpose:**
Automatically adds or removes worker nodes.

### Trigger

* Pods cannot be scheduled because of insufficient resources.
* Nodes remain underutilized.

### Scales

Cluster Infrastructure (Horizontal)

```text
3 Nodes → 5 Nodes
```

---

## Horizontal Pod Autoscaler (HPA)

**Purpose:**
Automatically adjusts Pod count.

### Metrics Used

* CPU utilization
* Memory utilization
* Custom metrics

### Scales

Workloads (Horizontal)

```text
2 Pods → 10 Pods
```

**Most commonly used autoscaler.**

---

## Vertical Pod Autoscaler (VPA)

**Purpose:**
Automatically adjusts CPU and Memory requests/limits for Pods.

### Scales

Workloads (Vertical)

```text
500m CPU → 1 CPU
512Mi → 1Gi
```

---

# 5. Quick Exam Mapping

| Scaling Type                | What Changes?   | Kubernetes Component |
| --------------------------- | --------------- | -------------------- |
| Horizontal Workload Scaling | Number of Pods  | HPA                  |
| Vertical Workload Scaling   | Pod CPU/Memory  | VPA                  |
| Horizontal Cluster Scaling  | Number of Nodes | Cluster Autoscaler   |
| Vertical Cluster Scaling    | Node CPU/Memory | Usually Manual       |

---

# CKA Exam Essentials

### Horizontal Scaling

* Add more Pods or Nodes.
* Preferred approach in Kubernetes.
* No application downtime.

### Vertical Scaling

* Increase CPU/Memory of existing Pod or Node.
* May require recreation/restart.
* Less common than horizontal scaling.

### Autoscaling Tools

| Tool               | Scales                   |
| ------------------ | ------------------------ |
| HPA                | Pods horizontally        |
| VPA                | Pod resources vertically |
| Cluster Autoscaler | Nodes horizontally       |

### One-Line Memory Trick

```text
HPA = More Pods
VPA = Bigger Pods
Cluster Autoscaler = More Nodes
```

