## Kubernetes Taints and Tolerations — Key Notes

### Core Idea

* **Taints** are applied to **nodes**
* **Tolerations** are applied to **pods**

They work together to control **which pods are allowed on which nodes**.

---



# Important Clarification

Taints and tolerations:

Restrict nodes from accepting certain pods
Do NOT force a pod onto a specific node

For guaranteed placement, use:

* **Node Affinity**
* **Node Selectors**

---

# Example Cluster

Suppose:

* Nodes: node1, node2, node3
* Pods: A, B, C, D

You taint `node1`:

```bash
kubectl taint nodes node1 app=blue:NoSchedule
```

Now:

* Pods A/B/C cannot run on node1
* Pod D can run only if it has matching toleration

---

# Taint Syntax

```bash
kubectl taint nodes <node-name> key=value:effect
```

Example:

```bash
kubectl taint nodes node1 app=blue:NoSchedule
```

Meaning:

* key = `app`
* value = `blue`
* effect = `NoSchedule`

---

# Taint Effects

## 1. NoSchedule

Pods without toleration:

Cannot be scheduled

Existing pods:

Stay running

---

## 2. PreferNoSchedule

Scheduler tries to avoid placing pods there.

Not guaranteed.

---

## 3. NoExecute

Pods without toleration:

Cannot be scheduled
Existing pods are evicted

---

# Pod Toleration YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: bee
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

Important:

* Values should be in quotes
* `operator` commonly:

  * `Equal`
  * `Exists`

---

# Lab Flow Summary

## Step 1 — Check Nodes

```bash
kubectl get nodes
```

---

## Step 2 — Describe Node

```bash
kubectl describe node node01
```

Look for:

```text
Taints:
```

---

## Step 3 — Add Taint

```bash
kubectl taint nodes node01 spray=mortein:NoSchedule
```

---

## Step 4 — Create Pod Without Toleration

```bash
kubectl run mosquito --image=nginx
```

Result:

```text
STATUS = Pending
```

Why?

Because:

```text
pod didn't tolerate node taint
```

---

## Step 5 — Check Pod Events

```bash
kubectl describe pod mosquito
```

Important error:

```text
had taint that the pod didn't tolerate
```

---

## Step 6 — Create Pod With Toleration

Generate YAML:

```bash
kubectl run bee --image=nginx --dry-run=client -o yaml > bee.yaml
```

Edit YAML:

```yaml
spec:
  tolerations:
  - key: "spray"
    operator: "Equal"
    value: "mortein"
    effect: "NoSchedule"
```

Apply:

```bash
kubectl apply -f bee.yaml
```

Now pod runs successfully.

---

# Removing a Taint

Syntax:

```bash
kubectl taint nodes <node-name> key=value:effect-
```

Example:

```bash
kubectl taint nodes controlplane node-role.kubernetes.io/control-plane:NoSchedule-
```

Notice the trailing `-`

That removes the taint.

---

# Why Pods Usually Don't Run on Control Plane

Kubernetes automatically taints the control-plane node:

```text
node-role.kubernetes.io/control-plane:NoSchedule
```

This prevents workloads from running there.

Best practice:

Keep workloads on worker nodes

---

# Useful Commands

## View Nodes

```bash
kubectl get nodes
```

---

## Describe Node

```bash
kubectl describe node <node-name>
```

---

## View Pods

```bash
kubectl get pods -o wide
```

---

## Describe Pod

```bash
kubectl describe pod <pod-name>
```

---

## Add Taint

```bash
kubectl taint nodes node1 app=blue:NoSchedule
```

---

## Remove Taint

```bash
kubectl taint nodes node1 app=blue:NoSchedule-
```

---

# Most Important Exam Points

## Remember:

### Taints are on nodes

### Tolerations are on pods

---

## Toleration does NOT guarantee placement

It only allows placement.

---

## NoExecute evicts existing pods

This is commonly tested.

---

## Control plane node already has taint

Usually:

```text
NoSchedule
```

---

# Labels/Selectors vs Taints/Tolerations

| Feature              | Purpose                 |
| -------------------- | ----------------------- |
| Labels & Selectors   | Group/filter objects    |
| Taints & Tolerations | Restrict pod scheduling |
| Node Affinity        | Force/prefer placement  |
