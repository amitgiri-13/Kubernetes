# Kubernetes Manual Scheduling Lab

# Cluster Information

## Check Nodes
```bash
kubectl get nodes
```
---

# Create the Pod

## Pod Definition File

`nginx.yaml`

### Create Pod

```bash
kubectl apply -f nginx.yaml
```

---

# Check Pod Status

## View Pods

```bash
kubectl get pods
```

### Result

```text
STATUS: Pending
```

---

# Why Pod Is Pending

## Inspect Pod

```bash
kubectl describe pod nginx
```

### Observation

* `Node: <none>`
* Pod is not assigned to any node

## Check Control Plane Components

```bash
kubectl get pods -n kube-system
```

### Observation

* API Server exists
* Controller Manager exists
* Proxy exists
* Scheduler is missing

## Reason

* Without a scheduler, Pods remain in `Pending` state

---

# Manually Schedule the Pod

## Edit Pod Definition

Add `nodeName` inside `spec`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeName: node01
  containers:
    - name: nginx
      image: nginx
```

---

# Recreate the Pod

## Replace Existing Pod

```bash
kubectl replace --force -f nginx.yaml
```

### What Happens

* Existing Pod is deleted
* New Pod is created with `nodeName`

---

# Monitor Pod Status

## Watch Pod

```bash
kubectl get pods -w
```

### Status Flow

```text
Pending -> ContainerCreating -> Running
```

---

# Schedule Pod on Control Plane Node

## Change `nodeName`

```yaml
spec:
  nodeName: controlplane
```

## Recreate Pod

```bash
kubectl replace --force -f nginx.yaml
```

---

# Verify Node Assignment

## Wide Output

```bash
kubectl get pods -o wide
```

### Result

* Pod runs on the specified node

---

# Important Notes

## Pods Cannot Be Moved

* Running Pods cannot be transferred between nodes
* Pods must be deleted and recreated on another node

## Pod Deletion Delay

* Kubernetes sends termination signals to container processes
* Graceful shutdown may take a few seconds
* Delay is normal behavior


