# Kubernetes Node Selectors

## Problem

In a Kubernetes cluster, pods can run on any available node by default.

Example:

- 2 small nodes
- 1 large node

Some workloads (like data processing) require more CPU and memory, so they should run only on the large node.

---

# Node Selector

Node Selector allows Kubernetes to schedule pods only on nodes with specific labels.

---

# Step 1: Label the Node

Add a label to the desired node.

```bash
kubectl label nodes node1 size=large
```

Example:

* Node: `node1`
* Label:

  * Key → `size`
  * Value → `large`

---

# Step 2: Use nodeSelector in Pod

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: data-processor

spec:
  containers:
    - name: app
      image: my-data-image

  nodeSelector:
    size: large
```

---

# How It Works

Kubernetes scheduler checks:

```yaml
nodeSelector:
  size: large
```

It searches for nodes having:

```text
size=large
```

Then schedules the pod on that node.

---

# Verify Labels

```bash
kubectl get nodes --show-labels
```

---

# Limitations of Node Selectors

Node selectors support only exact matching.

Supported:

```yaml
size: large
```

Not supported:

* large OR medium
* NOT small
* complex conditions

---

# For Complex Scheduling

Use:

* Node Affinity
* Node Anti-Affinity

These provide advanced scheduling rules.

---

# Summary

| Feature         | Description                    |
| --------------- | ------------------------------ |
| nodeSelector    | Schedules pod to labeled nodes |
| Labels          | Key-value pairs on nodes       |
| Matching        | Exact match only               |
| Limitation      | No complex conditions          |
| Advanced Option | Node Affinity                  |

