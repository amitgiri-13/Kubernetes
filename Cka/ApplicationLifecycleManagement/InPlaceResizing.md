# In-Place Pod Resource Resizing 

## What Problem Does It Solve?

Traditionally, when you modify CPU or Memory requests/limits of a Pod:

```text
Change Resources
       ↓
Pod Deleted
       ↓
New Pod Created
       ↓
Application Restart
```

Example:

```yaml
resources:
  requests:
    cpu: "500m"
```

↓

```yaml
resources:
  requests:
    cpu: "1"
```

Kubernetes recreates the Pod with the new resource configuration.

### Drawbacks

* Application interruption
* Pod restart required
* Stateful workloads may be affected
* Existing connections may be lost

---

# In-Place Pod Resource Resizing

In-place resizing allows CPU and Memory resources to be updated without recreating the Pod.

```text
Change Resources
       ↓
Pod Updated
       ↓
No Pod Recreation
```

### Benefit

* Less disruption
* Faster updates
* Better for stateful workloads
* Reduced downtime

---

# Feature Status

This feature was introduced as an **Alpha Feature** and requires explicit enablement.

### Feature Gate

```text
InPlacePodVerticalScaling
```

Must be enabled on cluster components.

```text
Disabled by Default
```

---

# Default Behavior vs In-Place Resize

| Action        | Traditional Kubernetes | In-Place Resize                                  |
| ------------- | ---------------------- | ------------------------------------------------ |
| CPU Change    | Pod recreated          | Pod updated                                      |
| Memory Change | Pod recreated          | Pod may be updated/restarted depending on policy |
| Downtime      | Possible               | Reduced                                          |
| Stateful Apps | More disruptive        | Less disruptive                                  |

---

# Resize Policy

After enabling the feature gate, Pods can define a resize policy.

Example:

```yaml
resizePolicy:
- resourceName: cpu
  restartPolicy: NotRequired

- resourceName: memory
  restartPolicy: RestartContainer
```

### Meaning

| Resource | Policy           | Result            |
| -------- | ---------------- | ----------------- |
| CPU      | NotRequired      | No restart needed |
| Memory   | RestartContainer | Restart required  |

---

# Example

Initial configuration:

```yaml
resources:
  requests:
    cpu: "500m"
```

Update:

```yaml
resources:
  requests:
    cpu: "1"
```

With:

```yaml
restartPolicy: NotRequired
```

Result:

```text
500m CPU → 1 CPU
```

Pod remains running.

No deletion.

No recreation.

---

# What Can Be Resized?

CPU Requests

CPU Limits

Memory Requests

Memory Limits

---

# What Cannot Be Resized?

Init Containers

Ephemeral Containers

Pod QoS Class

Windows Pods (currently unsupported)

---

# Important Limitations

### 1. Only CPU and Memory

Only CPU and Memory resources support in-place resizing.

```text
CPU ✔
Memory ✔
Other Resources ✘
```

---

### 2. QoS Class Cannot Change

Pod Quality of Service class remains fixed.

Example:

```text
Guaranteed
Burstable
BestEffort
```

Cannot be modified through resize operations.

---

### 3. Init Containers Cannot Be Resized

```yaml
initContainers:
```

Resources remain unchanged.

---

### 4. Ephemeral Containers Cannot Be Resized

```yaml
ephemeralContainers:
```

Not supported.

---

### 5. Memory Limit Cannot Be Reduced Below Current Usage

Example:

Current usage:

```text
700Mi
```

Attempt:

```text
Memory Limit = 500Mi
```

Result:

```text
Resize remains InProgress
```

until the container's memory consumption becomes lower than the requested limit.

---

# Relation to Vertical Pod Autoscaler (VPA)

### Manual Vertical Scaling

Administrator manually changes:

```yaml
resources:
  requests:
  limits:
```

and Pod resources are adjusted.

---

### VPA

Automates the process.

```text
Monitor Usage
      ↓
Recommend/Apply New CPU & Memory
      ↓
Resize Pod
```

In-place resizing improves how VPA can perform resource updates with less disruption.

---

# CKA Exam Essentials

### Remember

* In-place resize = Update Pod resources without recreating Pod.
* Requires feature gate:

```text
InPlacePodVerticalScaling
```

* Supports only:

  * CPU
  * Memory

* Resize policy controls whether restart is needed.

Example:

```yaml
restartPolicy: NotRequired
```

---

# Quick Comparison

| Feature               | Traditional Update | In-Place Resize   |
| --------------------- | ------------------ | ----------------- |
| Pod Recreation        | Yes                | No                |
| Downtime              | Possible           | Minimal           |
| CPU Resize            | Recreate Pod       | Resize Directly   |
| Memory Resize         | Recreate Pod       | Depends on Policy |
| Feature Gate Required | No                 | Yes               |

### Memory Trick

```text
HPA = More Pods
VPA = Bigger Pods
In-Place Resize = Bigger Pod Without Recreating It
```

**Exam Focus:** Understand the concept, feature gate name (`InPlacePodVerticalScaling`), resize policies, and the difference between traditional pod recreation and in-place resource updates.
