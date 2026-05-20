# Kubernetes Node Affinity

## Purpose

Node Affinity is used to control which nodes a pod can run on.

It is an advanced version of `nodeSelector`.

---

# Why Node Affinity?

`nodeSelector` only supports exact matching.

Example:

```yaml
nodeSelector:
  size: large
````

It cannot handle:

* OR conditions
* NOT conditions
* complex rules

Node Affinity solves this problem.

---

# Basic Node Affinity Example

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: data-processor

spec:
  containers:
    - name: app
      image: nginx

  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: size
                operator: In
                values:
                  - large
```

---

# Structure Explanation

| Field                                          | Purpose                     |
| ---------------------------------------------- | --------------------------- |
| affinity                                       | Affinity rules              |
| nodeAffinity                                   | Node-based scheduling       |
| requiredDuringSchedulingIgnoredDuringExecution | Affinity type               |
| nodeSelectorTerms                              | List of matching conditions |
| matchExpressions                               | Matching expressions        |
| key                                            | Label key                   |
| operator                                       | Matching operator           |
| values                                         | Accepted values             |

---

# Operators

## 1. In

Matches nodes whose label value exists in the list.

```yaml
operator: In
values:
  - large
  - medium
```

Meaning:

* schedule on `large` OR `medium`

---

## 2. NotIn

Exclude specific values.

```yaml
operator: NotIn
values:
  - small
```

Meaning:

* any node except `small`

---

## 3. Exists

Checks whether label exists.

```yaml
- key: size
  operator: Exists
```

Meaning:

* any node having label `size`

No `values` needed.

---

# Affinity Types

## 1. requiredDuringSchedulingIgnoredDuringExecution

Strict rule.

If matching node does not exist:

* pod will NOT be scheduled

Example use case:

* database pods
* GPU workloads
* critical applications

---

## 2. preferredDuringSchedulingIgnoredDuringExecution

Soft rule.

Scheduler tries to match node affinity.

If no matching node exists:

* pod is scheduled on another available node

Example use case:

* non-critical workloads
* optional optimization

---

# During Scheduling vs During Execution

## During Scheduling

Pod is being created for the first time.

Scheduler checks affinity rules before placing pod.

---

## During Execution

Pod is already running.

Example:

* node label changes later

Current affinity types use:

```text
IgnoredDuringExecution
```

Meaning:

* running pods are NOT removed
* pod continues running even if labels change

---

# Example: Preferred Affinity

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: web-app

spec:
  containers:
    - name: nginx
      image: nginx

  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 1
          preference:
            matchExpressions:
              - key: size
                operator: In
                values:
                  - large
```

---

# Label a Node

```bash
kubectl label nodes node1 size=large
```

---

# View Node Labels

```bash
kubectl get nodes --show-labels
```

---

# Node Selector vs Node Affinity

| Feature        | nodeSelector | nodeAffinity |
| -------------- | ------------ | ------------ |
| Exact Match    | Yes          | Yes          |
| OR Conditions  | No           | Yes          |
| NOT Conditions | No           | Yes          |
| Complex Rules  | No           | Yes          |
| Simplicity     | Simple       | More complex |

---

# Summary

* Node Affinity controls pod placement on nodes
* Uses node labels
* Supports advanced matching rules
* `required` = strict scheduling
* `preferred` = best effort scheduling
* `IgnoredDuringExecution` means running pods continue even after label changes

