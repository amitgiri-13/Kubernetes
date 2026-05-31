# Horizontal Pod Autoscaler (HPA) 

## What is HPA?

**Horizontal Pod Autoscaler (HPA)** automatically increases or decreases the number of Pods in a Deployment, ReplicaSet, or StatefulSet based on resource usage or other metrics.

### Purpose

* Handle increased traffic automatically.
* Reduce manual intervention.
* Optimize resource utilization.
* Scale down when demand decreases.

---

# Manual Horizontal Scaling

Without HPA, an administrator must:

### 1. Monitor Pod Usage

```bash
kubectl top pod
```

Example:

```text
NAME       CPU(cores)   MEMORY(bytes)
my-app     450m         120Mi
```

Requires **Metrics Server**.

---

### 2. Scale Deployment Manually

```bash
kubectl scale deployment my-app --replicas=5
```

### Problems

* Continuous monitoring required.
* Manual scaling commands.
* Slow reaction to traffic spikes.
* Human error possible.

---

# How HPA Works

1. Monitors metrics continuously.
2. Compares current usage against target threshold.
3. Increases Pod count if usage is high.
4. Decreases Pod count if usage is low.

```text
High CPU Usage
      ↓
HPA Detects
      ↓
Creates More Pods
      ↓
Load Distributed
```

```text
Low CPU Usage
      ↓
HPA Detects
      ↓
Removes Extra Pods
      ↓
Resource Savings
```

---

# Example Scenario

Deployment configuration:

```yaml
resources:
  requests:
    cpu: 250m
  limits:
    cpu: 500m
```

Pod maximum CPU capacity:

```text
500m CPU
```

Suppose HPA target:

```text
CPU Utilization = 50%
```

Threshold becomes:

```text
50% of 500m = 250m
```

When CPU usage exceeds target:

```text
250m → 350m → 450m
```

HPA increases replicas.

---

# Imperative HPA Creation

Create HPA from command line:

```bash
kubectl autoscale deployment my-app \
  --cpu-percent=50 \
  --min=1 \
  --max=10
```

### Meaning

| Option           | Description            |
| ---------------- | ---------------------- |
| --cpu-percent=50 | Target CPU utilization |
| --min=1          | Minimum Pods           |
| --max=10         | Maximum Pods           |

---

# View HPA

```bash
kubectl get hpa
```

Example:

```text
NAME      REFERENCE             TARGETS   MINPODS MAXPODS REPLICAS
my-app    Deployment/my-app     35%/50%   1       10      2
```

### Important Columns

| Column   | Meaning               |
| -------- | --------------------- |
| TARGETS  | Current CPU vs Target |
| MINPODS  | Minimum replicas      |
| MAXPODS  | Maximum replicas      |
| REPLICAS | Current Pod count     |

---

# Delete HPA

```bash
kubectl delete hpa my-app
```

---

# Declarative HPA Configuration

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler

metadata:
  name: my-app-hpa

spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app

  minReplicas: 1
  maxReplicas: 10

  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

Apply:

```bash
kubectl apply -f hpa.yaml
```

---

# HPA Prerequisite

### Metrics Server Required

HPA depends on **Metrics Server** for CPU and memory statistics.

Verify:

```bash
kubectl get deployment metrics-server -n kube-system
```

Without Metrics Server:

```bash
kubectl top pod
```

and HPA metrics will not work.

---

# Metrics Supported by HPA

## 1. Resource Metrics (Most Common)

Built-in metrics:

* CPU
* Memory

Source:

```text
Metrics Server
```

---

## 2. Custom Metrics

Metrics from applications running inside cluster.

Examples:

* Requests per second
* Queue length
* Active sessions

Source:

```text
Custom Metrics Adapter
```

---

## 3. External Metrics

Metrics from systems outside Kubernetes.

Examples:

* Datadog
* Dynatrace
* Cloud monitoring systems

Source:

```text
External Metrics Adapter
```

---

# HPA Flow

```text
Application
     ↓
Metrics Server
     ↓
HPA
     ↓
CPU > Target?
     ↓
Yes
     ↓
Increase Replicas
     ↓
Deployment Updates Pod Count
```

---

# CKA Exam Essentials

### Remember

* HPA = Horizontal Pod Autoscaler.
* Scales **Pods**, not Nodes.
* Requires **Metrics Server**.
* Works with:

  * Deployment
  * ReplicaSet
  * StatefulSet
* Supports:

  * CPU metrics
  * Memory metrics
  * Custom metrics
  * External metrics

### Key Commands

```bash
kubectl autoscale deployment my-app --cpu-percent=50 --min=1 --max=10
```

```bash
kubectl get hpa
```

```bash
kubectl describe hpa my-app
```

```bash
kubectl delete hpa my-app
```

### Memory Trick

```text
HPA watches metrics
→ High usage = More Pods
→ Low usage = Fewer Pods
→ Never exceeds MaxReplicas
→ Never goes below MinReplicas
```
