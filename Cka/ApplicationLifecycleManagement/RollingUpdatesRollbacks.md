# Kubernetes Deployments: Updates and Rollbacks

## 1. Deployment Rollout and Revision

* When a Deployment is created, Kubernetes triggers a **rollout**.
* Each rollout creates a new **Deployment Revision**.
* Example:

  * Initial deployment → `Revision 1`
  * Updated application version → `Revision 2`

### Why revisions matter

* Track deployment changes
* Enable rollback to previous working versions

---

## 2. Check Rollout Status and History

### Check rollout status

```bash
kubectl rollout status deployment/<deployment-name>
```

Example:

```bash
kubectl rollout status deployment/nginx-deployment
```

### View rollout history

```bash
kubectl rollout history deployment/<deployment-name>
```

Example:

```bash
kubectl rollout history deployment/nginx-deployment
```

---

# Deployment Strategies

Kubernetes supports two deployment strategies:

## 3. Recreate Strategy

### Process

1. Destroy all old Pods
2. Create new Pods with updated version

### Problem

* Application downtime occurs
* Users cannot access application during update

### Flow

```text
Old Pods → Terminated → New Pods Created
```

### Characteristics

* Causes downtime
* Simpler approach
* Not default strategy

---

## 4. Rolling Update Strategy (Default)

### Process

* Replace Pods gradually
* Old Pods are terminated one by one
* New Pods are created simultaneously

### Benefits

* Zero downtime
* Seamless upgrade experience

### Flow

```text
Old Pod 1 → New Pod 1
Old Pod 2 → New Pod 2
...
```

### Important

If no strategy is specified:

```yaml
strategy:
  type: RollingUpdate
```

is assumed automatically.

---

# Updating Deployments

## 5. Method 1: Update YAML File + Apply

### Steps

1. Modify deployment YAML
2. Apply changes

```bash
kubectl apply -f deployment.yaml
```

### Result

* New rollout triggered
* New deployment revision created

---

## 6. Method 2: Set Image Command

Update container image directly:

```bash
kubectl set image deployment/<deployment-name> <container-name>=<image-name>
```

Example:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.25
```

### Warning

* Deployment YAML file will not automatically reflect this change
* Can create configuration mismatch later

---

# Deployment Inspection

## 7. Describe Deployment

View detailed deployment information:

```bash
kubectl describe deployment <deployment-name>
```

Example:

```bash
kubectl describe deployment nginx-deployment
```

### What you can observe

### Recreate Strategy

```text
Old ReplicaSet scaled down to 0
New ReplicaSet scaled up to desired replicas
```

### Rolling Update Strategy

```text
Old ReplicaSet scaled down gradually
New ReplicaSet scaled up gradually
```

---

# How Deployment Works Internally

## 8. ReplicaSets and Pods

### Initial Deployment

Deployment creates:

* ReplicaSet
* ReplicaSet creates Pods

Architecture:

```text
Deployment
   ↓
ReplicaSet
   ↓
Pods
```

---

## 9. During Upgrade

When application is updated:

1. Deployment creates a **new ReplicaSet**
2. New Pods are created
3. Old Pods are gradually terminated

### Check ReplicaSets

```bash
kubectl get rs
```

Example output:

```text
NAME               DESIRED   CURRENT   READY
nginx-rs-old       0         0         0
nginx-rs-new       5         5         5
```

---

# Rollback in Kubernetes

## 10. Rollback Deployment

If new version fails:

```bash
kubectl rollout undo deployment/<deployment-name>
```

Example:

```bash
kubectl rollout undo deployment/nginx-deployment
```

### What happens internally

* New ReplicaSet Pods are terminated
* Old ReplicaSet Pods are restored

---

## 11. ReplicaSet Before and After Rollback

### Before Rollback

```text
Old ReplicaSet → 0 Pods
New ReplicaSet → 5 Pods
```

### After Rollback

```text
Old ReplicaSet → 5 Pods
New ReplicaSet → 0 Pods
```

---

# Important Commands Summary

| Purpose              | Command                                  |
| -------------------- | ---------------------------------------- |
| Create deployment    | `kubectl create -f deployment.yaml`      |
| List deployments     | `kubectl get deployments`                |
| Update deployment    | `kubectl apply -f deployment.yaml`       |
| Update image         | `kubectl set image deployment/...`       |
| Check rollout status | `kubectl rollout status deployment/...`  |
| View rollout history | `kubectl rollout history deployment/...` |
| Rollback deployment  | `kubectl rollout undo deployment/...`    |
| View ReplicaSets     | `kubectl get rs`                         |
| Describe deployment  | `kubectl describe deployment ...`        |

---

# Key Interview Points

## Rolling Update

* Default deployment strategy
* Zero downtime deployment
* Gradual Pod replacement

## Recreate Strategy

* Causes downtime
* Deletes all old Pods first

## ReplicaSet Role

* Maintains desired number of Pods
* New ReplicaSet created during upgrades

## Rollback

* Restores previous stable revision
* Uses deployment revision history

---

# Quick Architecture Flow

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
```

During update:

```text
Deployment
   ├── Old ReplicaSet → Old Pods
   └── New ReplicaSet → New Pods
```

After rollback:

```text
Deployment
   ├── Old ReplicaSet → Active
   └── New ReplicaSet → Scaled Down
```
