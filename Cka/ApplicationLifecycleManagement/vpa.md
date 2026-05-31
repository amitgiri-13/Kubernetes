# Vertical Pod Autoscaler (VPA) — CKA Notes

## What is VPA?

**Vertical Pod Autoscaler (VPA)** automatically adjusts CPU and Memory requests/limits for Pods based on actual resource usage.

### Purpose

* Prevent over-provisioning.
* Prevent under-provisioning.
* Optimize resource utilization.
* Automatically recommend or apply resource changes.

---

# Manual Vertical Scaling

Without VPA, an administrator must:

### 1. Monitor Resource Usage

```bash
kubectl top pod
```

Example:

```text
NAME      CPU(cores)   MEMORY(bytes)
my-app    800m         900Mi
```

---

### 2. Edit Resource Requests/Limits

```bash
kubectl edit deployment my-app
```

Change:

```yaml
resources:
  requests:
    cpu: 500m
    memory: 512Mi
```

to:

```yaml
resources:
  requests:
    cpu: 1
    memory: 1Gi
```

### Problems

* Requires continuous monitoring.
* Manual intervention.
* Resource settings may become outdated.
* Difficult in dynamic workloads.

---

# How VPA Works

VPA continuously monitors Pod resource consumption.

```text
Monitor CPU & Memory
          ↓
Analyze Usage Trends
          ↓
Calculate Optimal Resources
          ↓
Recommend or Apply Changes
```

Instead of adding more Pods, VPA makes existing Pods bigger or smaller.

---

# HPA vs VPA

| Feature      | HPA                   | VPA                  |
| ------------ | --------------------- | -------------------- |
| Scaling Type | Horizontal            | Vertical             |
| Changes      | Number of Pods        | CPU/Memory of Pods   |
| Example      | 2 Pods → 10 Pods      | 500m CPU → 1 CPU     |
| Primary Goal | Handle traffic spikes | Right-size resources |

### Memory Trick

```text
HPA = More Pods
VPA = Bigger Pods
```

---

# VPA Components

## 1. Recommender

Collects metrics and calculates optimal CPU and Memory values.

```text
Usage Data
      ↓
Recommendation
```

Example:

```text
Current CPU Request = 500m
Recommended CPU = 1 CPU
```

---

## 2. Updater

Checks whether Pods should be updated with new recommendations.

```text
Recommendation
      ↓
Updater
      ↓
Evict Pod (if needed)
```

---

## 3. Admission Controller

Applies recommended values when new Pods are created.

```text
New Pod Creation
        ↓
Admission Controller
        ↓
Apply Recommended Resources
```

---

# VPA Modes

## 1. Off Mode

Only generates recommendations.

No automatic changes.

```yaml
updatePolicy:
  updateMode: "Off"
```

### Use Case

Safely observe recommendations before applying them.

---

## 2. Initial Mode

Applies recommendations only when Pods are first created.

Existing Pods are not modified.

```yaml
updatePolicy:
  updateMode: "Initial"
```

---

## 3. Auto Mode

Automatically updates Pods using recommendations.

```yaml
updatePolicy:
  updateMode: "Auto"
```

Behavior:

```text
High Resource Usage
        ↓
VPA Recommends Increase
        ↓
Pod Recreated
        ↓
New Resources Applied
```

---

## 4. Recreate Mode

Explicitly recreates Pods when updates are needed.

```yaml
updatePolicy:
  updateMode: "Recreate"
```

---

# Example VPA Configuration

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler

metadata:
  name: my-app-vpa

spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app

  updatePolicy:
    updateMode: Auto
```

Apply:

```bash
kubectl apply -f vpa.yaml
```

---

# View VPA Recommendations

```bash
kubectl describe vpa my-app-vpa
```

Example:

```text
Recommendation:
  Container Recommendations:
    Target:
      cpu: 1
      memory: 1Gi
```

---

# Resource Policies

You can limit how much VPA can scale.

Example:

```yaml
resourcePolicy:
  containerPolicies:
  - containerName: '*'
    minAllowed:
      cpu: 250m
      memory: 256Mi
    maxAllowed:
      cpu: 2
      memory: 4Gi
```

---

# VPA and Pod Restart

Traditionally:

```text
Resource Change
       ↓
Pod Recreated
       ↓
New Resources Applied
```

Because CPU/Memory changes usually require Pod recreation.

---

# VPA + In-Place Resize

With In-Place Pod Resizing:

```text
Resource Change
       ↓
Pod Updated
       ↓
No Recreation Needed
```

This reduces disruption and improves VPA efficiency.

---

# VPA Prerequisites

VPA relies on resource metrics.

### Metrics Server Required

```bash
kubectl top pod
```

must work properly.

Without Metrics Server:

```text
No CPU/Memory Metrics
        ↓
VPA Cannot Calculate Recommendations
```

---

# When to Use VPA

### Good For

Databases

Stateful Applications

Long-running workloads

Applications with predictable scaling needs

---

### Less Suitable For

Rapid traffic spikes

Applications needing immediate scale-out

For those workloads:

```text
Use HPA
```

---

# HPA + VPA Together

Generally avoid using both on the same CPU/Memory metrics because they can conflict.

Example:

```text
VPA increases CPU request
           ↓
HPA sees lower utilization
           ↓
HPA scales down
```

This can create unstable behavior.

---

# CKA Exam Essentials

### Remember

* VPA = Vertical Pod Autoscaler.
* Scales CPU and Memory resources.
* Does NOT add Pods.
* Uses historical usage data.
* Depends on Metrics Server.
* Main components:

  * Recommender
  * Updater
  * Admission Controller

### Common Modes

| Mode     | Action                    |
| -------- | ------------------------- |
| Off      | Recommendation only       |
| Initial  | Apply on Pod creation     |
| Auto     | Automatic updates         |
| Recreate | Recreate Pods for updates |

### Key Flow

```text
Metrics Server
      ↓
VPA Recommender
      ↓
Resource Recommendation
      ↓
Updater / Admission Controller
      ↓
Pod Resources Adjusted
```

## Quick Revision

```text
HPA = More Pods
VPA = Bigger Pods
Cluster Autoscaler = More Nodes
In-Place Resize = Bigger Pod Without Recreating It
```

### Exam Tip

For CKA, focus on:

* Purpose of VPA
* VPA components
* VPA modes (Off, Initial, Auto, Recreate)
* Difference between HPA and VPA
* Relationship between VPA and In-Place Pod Resizing
* Metrics Server dependency
