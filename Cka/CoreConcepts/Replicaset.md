# Kubernetes Controllers: ReplicationController & ReplicaSet

## What are Controllers?
- Controllers are the **brain of Kubernetes**.
- They continuously monitor the state of objects and take action to maintain the desired state.

---

## What is a Replica?
- A replica = **copy of a pod**.
- Multiple replicas ensure:
  - High availability
  - Load balancing
  - Fault tolerance

---

## Why ReplicationController?
### Problem:
- Single pod → if it crashes → app becomes unavailable.

### Solution:
- Run **multiple pods (replicas)**.
- If one fails → others continue serving users.

### Key Responsibilities:
- Ensures a **fixed number of pods** are always running.
- Automatically creates a new pod if one fails.
- Helps with **load distribution** across pods.
- Supports scaling across multiple nodes.

---

## ReplicationController (RC)
- Older Kubernetes technology.
- API Version: `v1`
- Still works, but **replaced by ReplicaSet**.

---

## ReplicaSet (RS)
- New and recommended approach.
- API Version: `apps/v1`
- Performs same role as RC but with **more advanced features**.

---

## Key Difference: RC vs ReplicaSet

| Feature                | ReplicationController | ReplicaSet |
|----------------------|----------------------|------------|
| Status               | Deprecated (older)   | Recommended |
| API Version          | `v1`                 | `apps/v1` |
| Selector             | Optional             | **Required** |
| Label Matching       | Basic                | Advanced (matchLabels, expressions) |

---

## ReplicaSet YAML Structure

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-app-rs
  labels:
    app: my-app

spec:
  replicas: 3

  selector:
    matchLabels:
      app: my-app

  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: nginx
          image: nginx
````

---

## Understanding the Spec Section

### 1. replicas

* Number of pod instances to run.
* Example:

  ```yaml
  replicas: 3
  ```

### 2. selector

* Defines **which pods belong to this ReplicaSet**.
* Must match pod labels.
* Example:

  ```yaml
  selector:
    matchLabels:
      app: my-app
  ```

### 3. template

* Blueprint for creating pods.
* Same as pod definition (excluding `apiVersion` and `kind`).
* Required even if pods already exist.

---

## Labels & Selectors

### Why Labels?

* Used to **identify and group pods**.

### Why Selectors?

* Used by ReplicaSet to **find and monitor pods**.

### Example:

```yaml
labels:
  app: frontend
```

```yaml
selector:
  matchLabels:
    app: frontend
```

### Key Idea:

* ReplicaSet manages all pods **matching the selector**, even if it didn’t create them.

---

## Important Concept

* ReplicaSet can:

  * Manage existing pods
  * Create new pods if needed
* Template is still required to:

  * Recreate pods if they fail in the future

---

## Scaling ReplicaSet

### Method 1: Update YAML

```yaml
replicas: 6
```

```bash
kubectl replace -f rs.yaml
```

### Method 2: Command Line

```bash
kubectl scale rs my-app-rs --replicas=6
```

 Note:

* Scaling via CLI does **not update YAML file automatically**.

---

## Kubernetes Commands

### Create ReplicaSet

```bash
kubectl create -f rs.yaml
```

### View ReplicaSets

```bash
kubectl get rs
```

### View Pods

```bash
kubectl get pods
```

### Delete ReplicaSet

```bash
kubectl delete rs my-app-rs
```

### Update ReplicaSet

```bash
kubectl replace -f rs.yaml
```

### Scale ReplicaSet

```bash
kubectl scale rs my-app-rs --replicas=6
```

---

## Summary

* Controllers maintain desired state in Kubernetes.
* ReplicationController ensures pod availability (older).
* ReplicaSet is the modern replacement with better selector support.
* Labels + selectors are critical for pod management.
* ReplicaSet enables:

  * High availability
  * Auto-healing
  * Scalability



## Kubernetes ReplicaSet – All Commands 

```bash
# get all pods
kubectl get pods

# get all replicasets
kubectl get rs
kubectl get replicaset

# describe rs
kubectl describe rs <rs-name>

# create rs
kubectl create rs -f <file>

# delete rs
kubectl delete rs <rs-name>

# edit
kubectl edit rs <rs-name>

# explain
kubectl explain rs

# scaling
kubectl scale rs <rs-name> --replicas=<number>
# or edit
kubectl edit rs <rs-name>

# replace resource
kubectl replace -f <file>
```

---
