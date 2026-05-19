# Manual Scheduling in Kubernetes

## Overview
- Kubernetes normally uses a scheduler to assign Pods to nodes.
- If no scheduler exists, Pods remain in the `Pending` state.
- Pods can be scheduled manually.

---

# How Scheduling Works

## `nodeName` Field
- Every Pod has a `nodeName` field.
- By default, it is empty when the Pod is created.
- The Kubernetes scheduler automatically sets this field.

## Scheduler Process
1. Finds Pods without a `nodeName`
2. Selects a suitable node using scheduling algorithms
3. Assigns the Pod to the node
4. Creates a binding object internally

---

# Manual Pod Scheduling

## Method 1: Set `nodeName` During Pod Creation

### Example
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeName: worker-node-1
  containers:
    - name: nginx
      image: nginx
````

## Notes

* Pod is directly assigned to the specified node
* Works only during Pod creation
* No scheduler required

---

# Scheduling Existing Pods

## Important

* `nodeName` cannot be modified after Pod creation

## Alternative Approach

Use a Binding object and Kubernetes Binding API.

### Steps

1. Create a Binding object
2. Specify the target node
3. Send a POST request to the Pod Binding API
4. Kubernetes binds the Pod to the node

---

# Key Points

* Pods without a scheduler stay in `Pending`
* `nodeName` allows manual scheduling
* `nodeName` is immutable after creation
* Binding objects can assign existing Pods to nodes
* Manual scheduling mimics scheduler behavior

