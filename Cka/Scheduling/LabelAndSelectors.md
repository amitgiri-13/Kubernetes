# Kubernetes Labels, Selectors, and Annotations

# Labels and Selectors

## Purpose
- Labels are used to organize and categorize Kubernetes objects
- Selectors are used to filter and identify objects using labels

---

# Kubernetes Objects

Kubernetes manages many object types:

* Pods
* Services
* ReplicaSets
* Deployments

In large clusters, labels help group and manage these resources.

---

# Adding Labels to a Pod

## Pod Definition

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: app1
    tier: frontend
spec:
  containers:
    - name: nginx
      image: nginx
```

---

# Using Selectors

## Filter Pods by Label

```bash
kubectl get pods --selector app=app1
```

## Multiple Conditions

```bash
kubectl get pods --selector app=app1,tier=frontend
```

---

# Labels and ReplicaSets

## Purpose

ReplicaSets use selectors to identify and manage Pods.

---

# ReplicaSet Example

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: app-rs
  labels:
    app: app1

spec:
  replicas: 3

  selector:
    matchLabels:
      app: app1
      tier: frontend

  template:
    metadata:
      labels:
        app: app1
        tier: frontend

    spec:
      containers:
        - name: nginx
          image: nginx
```

---

# Important Concept

## Labels in Two Places

### ReplicaSet Labels

```yaml
metadata:
  labels:
```

* Labels for the ReplicaSet object itself

### Pod Labels

```yaml
template:
  metadata:
    labels:
```

* Labels applied to Pods created by the ReplicaSet

---

# How ReplicaSet Selects Pods

## Selector Matching

```yaml
selector:
  matchLabels:
```

* Must match Pod labels exactly
* ReplicaSet manages only matching Pods

---

# Services and Selectors

## Service Example

A Service uses selectors to find matching Pods.

```yaml
selector:
  app: app1
```

* Service forwards traffic to Pods with matching labels

---

# Annotations

## Purpose

Annotations store additional metadata.

## Examples

* Build version
* Tool information
* Contact details
* Email IDs
* Phone numbers

---

# Difference Between Labels and Annotations

| Feature                | Labels  | Annotations |
| ---------------------- | ------- | ----------- |
| Used for Selection     | Yes     | No          |
| Used for Grouping      | Yes     | No          |
| Informational Metadata | Limited | Yes         |
| Queried by Selectors   | Yes     | No          |

---

# Key Points

* Labels organize Kubernetes objects
* Selectors filter objects using labels
* ReplicaSets and Services rely on selectors
* Pod labels must match selector labels
* Annotations store extra metadata

