# Vertical Pod Autoscaler (VPA) 

## What is VPA?

**Vertical Pod Autoscaler (VPA)** automatically adjusts the CPU and Memory resources assigned to Pods based on actual resource consumption.

Instead of creating more Pods like HPA, VPA makes existing Pods larger or smaller.

```text
HPA → More Pods
VPA → Bigger/Smaller Pods
```

---

# Manual Vertical Scaling

Without VPA, an administrator must:

### Step 1: Monitor Resource Usage

```bash
kubectl top pod
```

Example:

```text
NAME      CPU(cores)   MEMORY
my-app    450m         800Mi
```

Requires Metrics Server.

---

### Step 2: Edit Deployment

```bash
kubectl edit deployment my-app
```

Change resources:

```yaml
resources:
  requests:
    cpu: 250m
    memory: 512Mi
  limits:
    cpu: 500m
    memory: 1Gi
```

to

```yaml
resources:
  requests:
    cpu: 1
    memory: 2Gi
  limits:
    cpu: 2
    memory: 4Gi
```

---

### Step 3: Pod Recreation

Traditional Kubernetes behavior:

```text
Resource Change
      ↓
Old Pod Deleted
      ↓
New Pod Created
      ↓
New Resources Applied
```

Problems:

* Manual monitoring
* Manual updates
* Pod restart
* Possible downtime

---

# How VPA Works

VPA continuously monitors resource consumption.

```text
Metrics Server
       ↓
VPA Recommender
       ↓
Resource Recommendation
       ↓
VPA Updater
       ↓
Pod Recreation (or future in-place update)
       ↓
Admission Controller
       ↓
New Resource Values Applied
```

---

# Installing VPA

Unlike HPA:

```text
HPA → Built into Kubernetes
VPA → Must be Installed Separately
```

Deploy VPA components from the VPA manifests.

Verify:

```bash
kubectl get pods -n kube-system | grep vpa
```

Expected components:

```text
vpa-admission-controller
vpa-recommender
vpa-updater
```

---

# VPA Components

## 1. Recommender

Responsible for:

* Collecting metrics
* Monitoring historical usage
* Monitoring live usage
* Generating CPU and memory recommendations

```text
Metrics
    ↓
Recommender
    ↓
Suggested Resources
```

Example:

```text
Current CPU Request: 500m
Recommended CPU: 1.5 CPU
```

### Important

Recommender only suggests changes.

It does NOT modify Pods.

---

## 2. Updater

Responsible for:

* Comparing current resources with recommendations
* Detecting under-sized or over-sized Pods
* Evicting Pods when update is required

```text
Recommendation
      ↓
Updater
      ↓
Pod Evicted
```

Eviction means:

```text
Pod Terminated
```

Deployment automatically creates a replacement Pod.

---

## 3. Admission Controller

Responsible for:

* Intercepting new Pod creation
* Injecting recommended resources

```text
New Pod Creation
        ↓
Admission Controller
        ↓
Recommended Resources Applied
```

Result:

```text
New Pod Starts With Correct CPU/Memory
```

---

# VPA Resource Flow

```text
Metrics Server
       ↓
Recommender
       ↓
Recommendation
       ↓
Updater
       ↓
Evict Existing Pod
       ↓
Deployment Creates New Pod
       ↓
Admission Controller
       ↓
Apply New Resources
```

---

# VPA Configuration

Example:

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

  resourcePolicy:
    containerPolicies:
    - containerName: "*"
      minAllowed:
        cpu: 250m
        memory: 256Mi
      maxAllowed:
        cpu: 2
        memory: 4Gi
```

Apply:

```bash
kubectl apply -f vpa.yaml
```

---

# VPA Update Modes

## 1. Off Mode

```yaml
updateMode: Off
```

Behavior:

```text
Recommender ✔
Updater ✘
Admission Controller ✘
```

Only recommendations are generated.

No automatic updates.

### Use Case

Observe recommendations before enabling automation.

---

## 2. Initial Mode

```yaml
updateMode: Initial
```

Behavior:

```text
Recommender ✔
Admission Controller ✔
Updater ✘
```

Recommendations applied only during Pod creation.

Existing Pods remain unchanged.

### Example

```text
Deployment Scale-Up
        ↓
New Pod Created
        ↓
Recommended Resources Applied
```

---

## 3. Recreate Mode

```yaml
updateMode: Recreate
```

Behavior:

```text
Recommender ✔
Updater ✔
Admission Controller ✔
```

Flow:

```text
Resource Recommendation
        ↓
Updater Evicts Pod
        ↓
Deployment Recreates Pod
        ↓
Admission Controller Applies New Resources
```

---

## 4. Auto Mode

```yaml
updateMode: Auto
```

Current behavior:

```text
Auto ≈ Recreate
```

As of today:

* Existing Pods are evicted.
* New Pods are created with updated resources.

Future behavior:

```text
Auto
   ↓
In-Place Pod Resize
   ↓
No Restart Needed
```

when in-place resizing becomes fully stable.

---

# Resource Policies

Restrict VPA recommendations.

Example:

```yaml
resourcePolicy:
  containerPolicies:
  - containerName: "*"
    minAllowed:
      cpu: 250m
      memory: 256Mi
    maxAllowed:
      cpu: 4
      memory: 8Gi
```

Prevents VPA from assigning values outside the defined range.

---

# View Recommendations

```bash
kubectl describe vpa my-app-vpa
```

Example:

```text
Recommendations:
  Target:
    CPU: 1500m
    Memory: 2Gi
```

---

# VPA vs HPA

| Feature                | VPA           | HPA            |
| ---------------------- | ------------- | -------------- |
| Scaling Type           | Vertical      | Horizontal     |
| Changes                | CPU/Memory    | Number of Pods |
| Handles Traffic Spikes | No            | Yes            |
| Pod Restart Needed     | Usually Yes   | No             |
| Best For               | Stateful apps | Stateless apps |
| Scaling Speed          | Slower        | Faster         |

---

# When to Use VPA

## Good Candidates

### Databases

```text
MySQL
PostgreSQL
MongoDB
```

Need larger resources rather than more replicas.

---

### JVM Applications

```text
Java Applications
Spring Boot Services
```

Often require memory tuning.

---

### AI / ML Workloads

```text
Inference Services
Training Jobs
```

Require precise CPU and memory allocation.

---

### Stateful Workloads

```text
StatefulSets
Database Pods
```

Benefit from optimized resources.

---

# When to Use HPA

## Good Candidates

### Web Applications

```text
Nginx
Apache
```

---

### APIs

```text
REST APIs
Microservices
```

---

### Message Processing

```text
RabbitMQ Consumers
Kafka Consumers
```

Need fast response to traffic spikes.

---

# Traffic Spike Comparison

## HPA

```text
Traffic Spike
      ↓
Create More Pods
      ↓
Immediate Capacity Increase
```

---

## VPA

```text
Traffic Spike
      ↓
Recommend More CPU
      ↓
Pod Restart
      ↓
Slower Response
```

HPA is the preferred solution for sudden load increases.

---

# Cost Optimization

## VPA

Prevents:

```text
Over-Provisioning
```

Example:

```text
Allocated: 4 CPU
Actual Usage: 500m
```

VPA recommends smaller resources.

---

## HPA

Prevents:

```text
Idle Pods Running Unnecessarily
```

Scales down unused replicas.

---

# VPA and In-Place Pod Resize

Traditional VPA:

```text
Update Resources
       ↓
Restart Pod
```

Future VPA:

```text
Update Resources
       ↓
Resize Pod In Place
       ↓
No Restart
```

This will make VPA significantly more efficient.

---

# CKA Exam Essentials

### Remember

* VPA is NOT built into Kubernetes.
* Must be installed separately.
* Depends on Metrics Server.
* Adjusts CPU and Memory only.
* Main Components:

  * Recommender
  * Updater
  * Admission Controller

### Important Commands

```bash
kubectl apply -f vpa.yaml
```

```bash
kubectl describe vpa my-app-vpa
```

```bash
kubectl top pod
```

---

# Quick Revision Table

| Component            | Responsibility                      |
| -------------------- | ----------------------------------- |
| Recommender          | Generates recommendations           |
| Updater              | Evicts Pods needing updates         |
| Admission Controller | Applies recommendations to new Pods |

| Mode     | Action                     |
| -------- | -------------------------- |
| Off      | Recommend only             |
| Initial  | Apply on new Pods only     |
| Recreate | Evict and recreate Pods    |
| Auto     | Currently same as Recreate |

## Memory Trick

```text
HPA = More Pods
VPA = Bigger Pods
Cluster Autoscaler = More Nodes
In-Place Resize = Bigger Pod Without Restart
```

### Exam Focus

Know:

* VPA architecture (Recommender, Updater, Admission Controller)
* VPA modes (Off, Initial, Recreate, Auto)
* Difference between HPA and VPA
* Metrics Server dependency
* VPA is not built into Kubernetes and must be installed separately.
