# Kubernetes Node Affinity Practice Notes

# 1. View Labels on a Node

Describe a node:

```bash id="u1xq9f"
kubectl describe node node01
````

Check the `Labels` section.

---

# 2. Count Labels

Example labels:

```text id="6m7abn"
Labels:
  beta.kubernetes.io/arch=amd64
  beta.kubernetes.io/os=linux
  kubernetes.io/hostname=node01
  ...
```

Count total labels manually.

---

# 3. Get Value of a Label

Example:

```text id="y4sp8h"
beta.kubernetes.io/arch=amd64
```

Value:

```text id="v5rj2p"
amd64
```

---

# 4. Add Label to Node

Add label:

```bash id="g6l1ke"
kubectl label nodes node01 color=blue
```

Verify:

```bash id="t9zmq3"
kubectl describe node node01
```

---

# 5. Create Deployment

```bash id="v0cn2w"
kubectl create deployment blue \
  --image=nginx \
  --replicas=3
```

---

# 6. Check Taints on Nodes

```bash id="g1xj7o"
kubectl describe node node01
```

Look for:

```text id="p4m2dh"
Taints:
```

If no taints exist:

```text id="eq3srm"
Taints: <none>
```

Pods can run on that node.

---

# 7. Edit Deployment

```bash id="m6af0u"
kubectl edit deployment blue
```

---

# 8. Add Node Affinity

Place pods only on nodes with:

```text id="ovp8d4"
color=blue
```

Example:

```yaml id="m2g5tf"
spec:
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

# 9. Verify Pod Placement

```bash id="i8az7n"
kubectl get pods -o wide
```

Check `NODE` column.

---

# 10. Create Deployment YAML Without Creating Resource

```bash id="k5q2lx"
kubectl create deployment red \
  --image=nginx \
  --replicas=2 \
  --dry-run=client \
  -o yaml > red.yaml
```

---

# 11. Use Existing Node Label

Example control-plane label:

```text id="g7rw2s"
node-role.kubernetes.io/control-plane
```

This label has no value.

Use `Exists` operator.

---

# 12. Node Affinity Using Exists

```yaml id="r4tb6v"
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: node-role.kubernetes.io/control-plane
              operator: Exists
```

Meaning:

* schedule pods only on nodes having this label

---

# 13. Create Deployment from YAML

```bash id="y9ej6d"
kubectl apply -f red.yaml
```

---

# 14. Common YAML Error

Error:

```text id="h3tq8u"
did not find expected key
```

Cause:

* incorrect indentation

YAML is space-sensitive.

---

# 15. Verify Final Pod Placement

```bash id="l0vp7n"
kubectl get pods -o wide
```

Example:

```text id="h6kx4c"
NAME      NODE
red-*     control-plane
blue-*    node01
```

---

# Important Commands Summary

| Purpose           | Command                                |
| ----------------- | -------------------------------------- |
| Describe node     | `kubectl describe node <node>`         |
| Add label         | `kubectl label nodes <node> key=value` |
| Create deployment | `kubectl create deployment`            |
| Edit deployment   | `kubectl edit deployment <name>`       |
| Generate YAML     | `--dry-run=client -o yaml`             |
| Apply YAML        | `kubectl apply -f file.yaml`           |
| Check pod node    | `kubectl get pods -o wide`             |

---

# Key Learning Points

* Node affinity controls pod placement
* Labels are required on nodes
* `In` operator checks values
* `Exists` checks only label existence
* `requiredDuringSchedulingIgnoredDuringExecution`

  * strict scheduling
  * running pods continue after label changes
* YAML indentation is critical

