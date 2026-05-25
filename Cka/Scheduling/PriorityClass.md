# Kubernetes Priority Classes

## What are Priority Classes?

In Kubernetes, different workloads may have different importance levels.

Examples:

* Kubernetes control plane components → extremely critical
* Databases and production apps → high priority
* Batch jobs or background tasks → lower priority

Kubernetes uses **Priority Classes** to decide:

* Which Pods should be scheduled first
* Which Pods can evict lower-priority Pods when resources are limited

---

# Why Priority Classes are Needed

Suppose a cluster is almost full.

A new critical Pod arrives, but there are no free resources.

Without priorities:

* The critical Pod stays pending.

With priorities:

* Kubernetes can remove lower-priority Pods
* The critical Pod gets scheduled immediately

This mechanism is called **preemption**.

---

# Key Concepts

## 1. PriorityClass Object

A `PriorityClass` defines a numeric priority value.

* Higher number = higher priority
* Lower number = lower priority

Priority classes are:

* **Cluster-wide objects**
* **Non-namespaced**
* Can be used by Pods in any namespace

---

# Priority Value Range

## Application Workloads

Range approximately:

* Minimum: `-2,000,000,000`
* Maximum: `1,000,000,000`

Higher value means higher priority.

Example:

| Workload    | Priority |
| ----------- | -------- |
| Critical DB | 100000   |
| Backend API | 50000    |
| Batch Job   | 1000     |

---

## System Critical Pods

Kubernetes reserves higher ranges for internal system Pods.

Examples:

* `system-cluster-critical`
* `system-node-critical`

These have values close to `2,000,000,000`.

Check existing classes:

```bash
kubectl get priorityclass
```

Example output:

```bash
NAME                      VALUE
system-cluster-critical   2000000000
system-node-critical      2000001000
```

---

# Creating a Priority Class

Example YAML:

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 100000
description: "High priority workloads"
```

Apply it:

```bash
kubectl apply -f priorityclass.yaml
```

---

# Using PriorityClass in a Pod

Attach the priority class using `priorityClassName`.

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  priorityClassName: high-priority
  containers:
  - name: nginx
    image: nginx
```

The Pod inherits the priority value from the PriorityClass.

---

# Default Priority

If no `priorityClassName` is specified:

* Pod priority defaults to `0`

---

# Global Default Priority

You can define a custom default priority using:

```yaml
globalDefault: true
```

Example:

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: default-priority
value: 1000
globalDefault: true
description: "Default priority for all pods"
```

Important:

* Only ONE PriorityClass can have:

```yaml
globalDefault: true
```

Otherwise Kubernetes won't know which default to use.

---

# Scheduling Behavior

## Example Scenario

Cluster has limited resources.

Two Pods arrive:

| Pod            | Priority |
| -------------- | -------- |
| Critical App   | 7        |
| Background Job | 5        |

Scheduler behavior:

1. Critical app gets scheduled first
2. Remaining resources are used for lower-priority Pods

---

# Pod Preemption

Now suppose:

* Cluster is full
* New Pod with priority `6` arrives

What happens?

Kubernetes checks the **preemption policy**.

---

# Preemption Policy

## Default Behavior

Default:

```yaml
preemptionPolicy: PreemptLowerPriority
```

Meaning:

* Kubernetes can evict lower-priority Pods
* Higher-priority Pod gets scheduled

Example:

| Existing Pod | Priority |
| ------------ | -------- |
| Batch Job    | 5        |

Incoming Pod:

| Incoming Pod | Priority |
| ------------ | -------- |
| API Server   | 6        |

Result:

* Priority 5 Pod gets terminated
* Priority 6 Pod gets scheduled

---

# Disable Preemption

If you do NOT want eviction:

```yaml
preemptionPolicy: Never
```

Example:

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority-nonpreempting
value: 100000
preemptionPolicy: Never
description: "High priority but non-preempting"
```

Behavior:

* Pod waits in scheduling queue
* Does NOT evict existing Pods
* Still gets scheduling preference over lower-priority waiting Pods

---

# Important Difference

| Feature    | Meaning                                    |
| ---------- | ------------------------------------------ |
| Priority   | Scheduling order                           |
| Preemption | Whether lower-priority Pods can be evicted |

---

# Real-World Use Cases

| Workload Type          | Recommended Priority |
| ---------------------- | -------------------- |
| Kubernetes system Pods | Highest              |
| Databases              | High                 |
| APIs                   | Medium-High          |
| Monitoring             | Medium               |
| CI/CD jobs             | Medium-Low           |
| Batch processing       | Low                  |
| Experimental workloads | Lowest               |

---

# Commands Summary

## List Priority Classes

```bash
kubectl get priorityclass
```

## Describe Priority Class

```bash
kubectl describe priorityclass high-priority
```

## Create Priority Class

```bash
kubectl apply -f priorityclass.yaml
```

---

# Interview Questions

## What is a PriorityClass?

A cluster-wide Kubernetes object used to assign scheduling priority to Pods.

---

## What happens if no PriorityClass is assigned?

Pod gets default priority value `0`.

---

## What is pod preemption?

Kubernetes evicts lower-priority Pods to schedule higher-priority Pods when resources are insufficient.

---

## Difference between priority and preemption?

* Priority determines scheduling order
* Preemption determines whether lower-priority Pods can be evicted

---

# Quick Revision

* PriorityClasses define Pod importance
* Higher number = higher priority
* PriorityClasses are cluster-wide objects
* Default Pod priority = `0`
* `globalDefault: true` sets cluster default priority
* Preemption evicts lower-priority Pods
* `preemptionPolicy: Never` disables eviction
* System-critical Pods use reserved high values
