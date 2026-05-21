# Kubernetes Resource Requests & Limits — Lab Notes

## Overview

This lab focuses on:

* CPU and memory requests
* Resource limits
* Pod failures due to resource exhaustion
* Debugging using `kubectl describe`
* Updating pod resource configuration

---

## 1. Identify CPU Request (Rabbit Pod)

### Command

```bash id="k8r1a1"
kubectl get pod
kubectl describe pod rabbit
```

### Key Observation

* CPU request found under:

  * `resources.requests.cpu`

### Result

* CPU request = `1`

---

## 2. Delete Rabbit Pod

```bash id="k8r1a2"
kubectl delete pod rabbit
```

---

## 3. Inspect Elephant Pod Failure

### Command

```bash id="k8r1a3"
kubectl describe pod elephant
```

### Key Finding

* Pod status: `OOMKilled`

### Meaning

* Container exceeded memory limit
* Kubernetes terminated the process

---

## 4. Memory Limit of Elephant Pod

From `describe` output:

* Memory limit = `10Mi`

---

## 5. Root Cause

### Application behavior:

* Process requires ~15Mi memory

### Configuration:

* Limit set to `10Mi`

### Result:

* Memory exceeded → OOMKill → pod restart/failure

---

## 6. Fix Requirement

### Task

Increase memory limit:

* From `10Mi` → `20Mi`

---

## 7. Attempted Fix (Edit)

```bash id="k8r1a4"
kubectl edit pod elephant
```

### Result

* Failed: cannot modify resource limits on running pod

---

## 8. Correct Approach

### Option A (Recreate Pod)

* Delete and recreate

```bash id="k8r1a5"
kubectl delete pod elephant
kubectl apply -f elephant.yaml
```

---

### Option B (Recommended in Lab)

Force replace:

```bash id="k8r1a6"
kubectl replace --force -f elephant.yaml
```

### Effect

* Deletes existing pod
* Recreates pod with updated spec

---

## 9. Verification

### Check pod status

```bash id="k8r1a7"
kubectl get pods
```

### Confirm resources

```bash id="k8r1a8"
kubectl describe pod elephant
```

---

## Outcome

* Elephant pod moves to `Running` state
* Memory limit updated to `20Mi`
* OOMKilled issue resolved

---

## Key Takeaways

* CPU request determines scheduling eligibility
* Memory exhaustion causes `OOMKilled`
* Pod spec changes require recreation (not in-place edit)
* `kubectl replace --force` is a quick fix in labs
* Always align limits with application memory needs

---
