# Kubernetes Resource Requests and Limits

## Resource Scheduling

* Every node in a Kubernetes cluster has CPU and memory resources.
* Every pod requires CPU and memory to run.
* The Kubernetes scheduler decides which node a pod should run on.
* Scheduler checks:

  * Resource requests of the pod
  * Available resources on nodes
* If sufficient resources are unavailable:

  * Pod remains in `Pending` state
  * `kubectl describe pod <pod-name>` shows errors like:

```bash
Insufficient cpu
```

---

# Resource Requests

Resource requests define the **minimum guaranteed resources** for a container.

Example:

```yaml
resources:
  requests:
    memory: "4Gi"
    cpu: "2"
```

Meaning:

* 4 GiB memory guaranteed
* 2 CPU guaranteed

Scheduler places the pod only on nodes that can provide these resources.

---

# CPU Units

## CPU Examples

| Value  | Meaning                  |
| ------ | ------------------------ |
| `1`    | 1 vCPU / 1 core          |
| `0.1`  | 100 milliCPU             |
| `100m` | 0.1 CPU                  |
| `1m`   | Minimum allowed CPU unit |

## Cloud Mapping

| Platform | Equivalent |
| -------- | ---------- |
| AWS      | 1 vCPU     |
| GCP      | 1 Core     |
| Azure    | 1 Core     |

Example:

```yaml
cpu: "500m"
```

Means:

* Half CPU core

---

# Memory Units

## Common Units

| Unit | Meaning               |
| ---- | --------------------- |
| `Mi` | Mebibyte (1024-based) |
| `Gi` | Gibibyte              |
| `M`  | Megabyte (1000-based) |
| `G`  | Gigabyte              |

Examples:

```yaml
memory: "256Mi"
memory: "1Gi"
memory: "1G"
```

## Difference Between `G` and `Gi`

| Unit  | Value    |
| ----- | -------- |
| `1G`  | 1000 MB  |
| `1Gi` | 1024 MiB |

---

# Resource Limits

Limits define the **maximum resources** a container can use.

Example:

```yaml
resources:
  limits:
    memory: "512Mi"
    cpu: "1"
```

Meaning:

* Container cannot use more than:

  * 1 CPU
  * 512Mi memory

---

# Complete Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        memory: "256Mi"
        cpu: "500m"
      limits:
        memory: "512Mi"
        cpu: "1"
```

---

# CPU vs Memory Behavior

## CPU Limit Behavior

If container exceeds CPU limit:

* CPU is throttled
* Container continues running

## Memory Limit Behavior

If container exceeds memory limit:

* Pod may be terminated
* Kubernetes triggers OOMKill

OOM = Out Of Memory

Check using:

```bash
kubectl describe pod <pod-name>
```

Possible error:

```bash
OOMKilled
```

---

# Default Kubernetes Behavior

By default:

* No requests
* No limits

This means:

* Pods can consume all node resources
* One pod can starve others

---

# Common Resource Configurations

## 1. No Requests, No Limits

Problem:

* One pod may consume everything

Not recommended.

---

## 2. Limits Only

Example:

```yaml
limits:
  cpu: "3"
```

Behavior:

* Kubernetes automatically sets request = limit

So:

* Request = 3 CPU
* Limit = 3 CPU

---

## 3. Requests and Limits

Example:

```yaml
requests:
  cpu: "1"

limits:
  cpu: "3"
```

Behavior:

* Guaranteed 1 CPU
* Can burst up to 3 CPU

Common production setup.

---

## 4. Requests Only

Example:

```yaml
requests:
  cpu: "1"
```

Behavior:

* Guaranteed 1 CPU
* No upper limit
* Can use extra CPU if available

Often considered best for CPU-intensive workloads.

---

# Important CPU Observation

CPU can be throttled.

Memory cannot be throttled.

If memory pressure occurs:

* Kubernetes kills pods to free memory.

---

# Best Practices

## CPU

Recommended:

* Set requests
* Avoid unnecessary limits unless needed

## Memory

Recommended:

* Set both requests and limits
* Prevent OOM issues

---

# LimitRange

`LimitRange` sets default requests and limits for pods in a namespace.

Example:

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
  - default:
      cpu: "500m"
    defaultRequest:
      cpu: "500m"
    max:
      cpu: "1"
    min:
      cpu: "100m"
    type: Container
```

## Purpose

* Set default values
* Enforce min/max constraints
* Applied at namespace level

Important:

* Affects only newly created pods
* Existing pods are unchanged

---

# ResourceQuota

`ResourceQuota` limits total namespace resource consumption.

Example:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 4Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

## Purpose

Restricts:

* Total requested CPU/memory
* Total limits across namespace

Applied at:

* Namespace level

---

# Useful Commands

## Describe Pod

```bash
kubectl describe pod <pod-name>
```

## Check Resource Usage

```bash
kubectl top pod
kubectl top node
```

---

# Key Takeaways

* Requests = minimum guaranteed resources
* Limits = maximum allowed resources
* Scheduler uses requests for placement
* CPU can be throttled
* Memory overuse causes OOMKill
* Default Kubernetes has no limits or requests
* Use `LimitRange` for namespace defaults
* Use `ResourceQuota` for namespace-wide restrictions
* Always define resource requests in production clusters
