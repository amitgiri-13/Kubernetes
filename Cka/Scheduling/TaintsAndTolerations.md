# Kubernetes Taints and Tolerations

# Overview

- Taints and tolerations control Pod placement on nodes
- Used to restrict which Pods can run on specific nodes
- Commonly used for dedicated workloads

---

# Core Concept

## Taints
- Applied to nodes
- Repel Pods that do not tolerate them

## Tolerations
- Applied to Pods
- Allow Pods to run on tainted nodes

---

# Default Scheduling

Without taints:
- Scheduler distributes Pods across all nodes
- No restrictions exist

Example:
```text id="9p3n0n"
Nodes: node1, node2, node3
Pods: A, B, C, D
````

Pods are spread evenly.

---

# Dedicated Node Example

## Goal

Allow only Pod D on `node1`.

---

# Step 1: Add Taint to Node

## Command

```bash id="1n2fsl"
kubectl taint nodes node1 app=blue:NoSchedule
```

## Structure

```text id="1zz0w9"
key=value:effect
```

### Components

* Key: `app`
* Value: `blue`
* Effect: `NoSchedule`

---

# Result of Taint

* Pods without matching tolerations cannot run on `node1`
* Existing Pods without toleration may stay depending on effect

---

# Step 2: Add Toleration to Pod

## Pod Definition

```yaml id="0v9k4w"
apiVersion: v1
kind: Pod
metadata:
  name: pod-d

spec:
  tolerations:
    - key: "app"
      operator: "Equal"
      value: "blue"
      effect: "NoSchedule"

  containers:
    - name: nginx
      image: nginx
```

---

# Important Rule

```text id="gh3rr9"
Taints are applied on nodes.
Tolerations are applied on Pods.
```

---

# Taint Effects

## 1. NoSchedule

### Behavior

* New Pods without toleration are not scheduled

### Existing Pods

* Continue running

---

# 2. PreferNoSchedule

### Behavior

* Scheduler tries to avoid scheduling Pods
* Not guaranteed

---

# 3. NoExecute

### Behavior

* New Pods without toleration are blocked
* Existing Pods without toleration are evicted

---

# NoExecute Example

## Scenario

* Node gets tainted
* Existing Pod lacks toleration

## Result

```text id="gyrzkl"
Pod is evicted from the node.
```

---

# Remove Taint

## Command

```bash id="ehz22k"
kubectl taint nodes node1 app=blue:NoSchedule-
```

* Trailing `-` removes taint

---

# View Node Taints

## Describe Node

```bash id="cnhwop"
kubectl describe node node1
```

Look for:

```text id="2s7f8d"
Taints:
```

---

# Important Limitation

## Taints and Tolerations Do NOT Guarantee Placement

They only:

* Prevent unwanted Pods from entering nodes

They do NOT:

* Force Pods onto specific nodes

Example:

* Pod D tolerates `node1`
* Scheduler may still place it on another node

---

# Node Affinity

## Purpose

Used to force or prefer Pods onto specific nodes.

```text id="9p6vms"
Taints/Tolerations = Node protection
Node Affinity = Pod placement preference
```

---

# Master Node Taint

## Default Behavior

Control plane nodes are automatically tainted.

### Effect

```text id="7yq90u"
Application Pods are not scheduled on control plane nodes.
```

---

# Check Master Node Taint

## Command

```bash id="0b8xqm"
kubectl describe node controlplane
```

---

# Key Points

* Taints repel Pods
* Tolerations allow Pods onto tainted nodes
* `NoSchedule` blocks scheduling
* `NoExecute` also evicts running Pods
* Taints do not force Pod placement
* Control plane nodes are tainted by default


