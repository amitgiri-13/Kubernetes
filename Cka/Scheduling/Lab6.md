# Kubernetes DaemonSet Lab Notes

# Viewing DaemonSets

## List DaemonSets in Default Namespace

```bash id="wl6d7z"
kubectl get daemonsets
```

Result:

* No DaemonSets found in the default namespace.

---

## List DaemonSets in All Namespaces

```bash id="m1v6v8"
kubectl get daemonsets -A
```

Example Output:

```bash id="i8p8wj"
NAMESPACE     NAME          DESIRED   CURRENT   READY
kube-system   kube-flannel  1         1         1
kube-system   kube-proxy    1         1         1
```

Observation:

* Total DaemonSets = `2`
* Both are in the `kube-system` namespace.

---

# DaemonSets in kube-system Namespace

DaemonSets found:

* `kube-flannel-ds`
* `kube-proxy`

Namespace:

* `kube-system`

---

# Checking DaemonSet Scheduling

## Describe kube-proxy DaemonSet

```bash id="5t0x8h"
kubectl describe daemonset kube-proxy -n kube-system
```

Important Fields:

```text id="9k55ot"
Desired Number of Nodes Scheduled: 1
Current Number of Nodes Scheduled: 1
```

Meaning:

* `kube-proxy` pod is scheduled on 1 node.

Reason:

* The cluster currently contains only 1 node.

---

# Checking Cluster Nodes

```bash id="tt9d0g"
kubectl get nodes
```

Example:

```bash id="i0jlwm"
NAME       STATUS   ROLES    AGE   VERSION
controlplane   Ready    master   ...
```

Observation:

* Only one node exists in the cluster.

---

# Finding Image Used by kube-flannel DaemonSet

## Describe DaemonSet

```bash id="6tp52k"
kubectl describe daemonset kube-flannel-ds -n kube-system
```

Look under:

```text id="zzkp80"
Containers:
```

Example Image:

```text id="b09jfy"
Image: quay.io/coreos/flannel:vX.X.X
```

---

# Creating a DaemonSet

## Problem Statement

Create a DaemonSet for:

* Name: `elasticsearch`
* Namespace: `kube-system`
* Image: `k8s.gcr.io/fluentd-elasticsearch:1.20`

---

# Generating YAML Using Dry Run

Since there is no direct:

```bash id="c0y5ia"
kubectl create daemonset
```

command, generate a Deployment YAML first and modify it.

---

## Generate Deployment YAML

```bash id="2d3y3r"
kubectl create deployment elasticsearch \
  --image=k8s.gcr.io/fluentd-elasticsearch:1.20 \
  --namespace=kube-system \
  --dry-run=client -o yaml > daemonset.yaml
```

---

# Edit the YAML

## Change Resource Type

From:

```yaml id="9rm7vx"
kind: Deployment
```

To:

```yaml id="v3dd5l"
kind: DaemonSet
```

---

# Remove Unnecessary Fields

Delete:

```yaml id="z69m5m"
replicas:
strategy:
status:
```

DaemonSets do not use replica count like Deployments.

---

# Final DaemonSet YAML

```yaml id="r3ik57"
apiVersion: apps/v1
kind: DaemonSet

metadata:
  name: elasticsearch
  namespace: kube-system

spec:
  selector:
    matchLabels:
      app: elasticsearch

  template:
    metadata:
      labels:
        app: elasticsearch

    spec:
      containers:
      - name: elasticsearch
        image: k8s.gcr.io/fluentd-elasticsearch:1.20
```

---

# Create the DaemonSet

```bash id="nsnls7"
kubectl apply -f daemonset.yaml
```

---

# Verify DaemonSet

```bash id="zpxu6j"
kubectl get daemonset -n kube-system
```

Example:

```bash id="g5zntj"
NAME            DESIRED   CURRENT   READY
elasticsearch   1         1         0
```

---

# Key Learnings

## Important Commands

### List DaemonSets

```bash id="jlwmk5"
kubectl get daemonsets -A
```

---

### Describe DaemonSet

```bash id="g16j9j"
kubectl describe daemonset <daemonset-name> -n kube-system
```

---

### Generate YAML

```bash id="pdq3j6"
kubectl create deployment <name> \
  --image=<image> \
  --dry-run=client -o yaml
```

---

### Create DaemonSet

```bash id="5n48e7"
kubectl apply -f daemonset.yaml
```

---

# Summary

* DaemonSets run one pod per node.
* Commonly used for:

  * Logging
  * Monitoring
  * Networking
* `kube-proxy` and `kube-flannel` are common DaemonSets.
* DaemonSet YAML is very similar to Deployment YAML.
* A Deployment YAML can be converted into a DaemonSet by:

  * Changing `kind`
  * Removing `replicas`
  * Removing Deployment-specific fields
