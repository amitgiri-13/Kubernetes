# Kubernetes: Taints & Tolerations vs Node Affinity

# Goal

We have:

- 3 nodes
  - blue node
  - red node
  - green node

- 3 pods
  - blue pod
  - red pod
  - green pod

Desired result:

| Pod | Node |
|---|---|
| blue pod | blue node |
| red pod | red node |
| green pod | green node |

Also:

- other teams share the cluster
- we do NOT want:
  - other pods on our nodes
  - our pods on other nodes

---

# Using Only Taints and Tolerations

## Step 1: Taint Nodes

Example:

```bash id="s5o2zr"
kubectl taint nodes blue-node color=blue:NoSchedule
````

---

## Step 2: Add Toleration to Pod

```yaml id="u4d9yb"
tolerations:
  - key: "color"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"
```

---

# Result

* blue pod can run on blue node
* green pod can run on green node

BUT:

Pods may still run on other untainted nodes.

Example:

* red pod may run on another normal node

---

# Important Point

Taints and tolerations:

* prevent unwanted pods from entering a node
* DO NOT force pods onto specific nodes

---

# Using Only Node Affinity

## Step 1: Label Nodes

```bash id="r7kq1m"
kubectl label nodes blue-node color=blue
```

---

## Step 2: Add Node Affinity

```yaml id="n6hv3x"
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: color
              operator: In
              values:
                - blue
```

---

# Result

* blue pod goes to blue node
* red pod goes to red node

BUT:

Other pods can still run on these nodes.

---

# Important Point

Node affinity:

* controls where OUR pods go
* DOES NOT block other pods

---

# Best Solution: Use Both Together

Combine:

* taints & tolerations
* node affinity

---

# Why Combine Them?

| Feature              | Purpose                        |
| -------------------- | ------------------------------ |
| Taints & Tolerations | Block unwanted pods from nodes |
| Node Affinity        | Force pods onto desired nodes  |

Together they create dedicated nodes.

---

# Complete Workflow

## 1. Taint Node

```bash id="z1ow4e"
kubectl taint nodes blue-node color=blue:NoSchedule
```

Prevents normal pods from entering.

---

## 2. Label Node

```bash id="t2jq9f"
kubectl label nodes blue-node color=blue
```

Adds scheduling identity.

---

## 3. Add Toleration

```yaml id="m3bx8p"
tolerations:
  - key: "color"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"
```

Allows blue pod onto blue node.

---

## 4. Add Node Affinity

```yaml id="v5pc2r"
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: color
              operator: In
              values:
                - blue
```

Ensures pod prefers only blue node.

---

# Final Result

| Requirement                          | Solved By     |
| ------------------------------------ | ------------- |
| Prevent other pods on our nodes      | Taints        |
| Allow our pods on tainted nodes      | Tolerations   |
| Ensure our pods run on correct nodes | Node Affinity |

---

# Key Difference

| Feature            | Controls        |
| ------------------ | --------------- |
| Taints/Tolerations | Node acceptance |
| Node Affinity      | Pod preference  |

---

# Summary

* Taints repel unwanted pods
* Tolerations allow specific pods
* Node affinity attracts pods to specific nodes
* Using both together creates isolated dedicated nodes

