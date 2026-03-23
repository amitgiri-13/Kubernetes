# Kubernetes Namespaces — Clean Explanation

## 1. Core Idea (Very Important)

A **Namespace = logical isolation inside a Kubernetes cluster**

Think of it like:

* Separate environments inside the same cluster
* Resources are grouped and isolated

---

## 2. Analogy 

* Two people named **Mark**
* Same name → confusion
* Solution → use full name (Mark Smith vs Mark Williams)

In Kubernetes:

* Same resource names can exist in different namespaces

Example:

```bash
dev/db-service
prod/db-service
```

---

## 3. Default Namespaces

Kubernetes automatically creates:

### 1. `default`

* Where your apps run (by default)

### 2. `kube-system`

* Internal Kubernetes components
* Example:

  * DNS
  * Networking
* Do NOT modify

### 3. `kube-public`

* Publicly accessible resources

---

## 4. Why Namespaces Matter

### Use cases:

* Separate environments:

  * dev
  * staging
  * production
* Avoid accidental changes
* Apply security policies
* Control resource usage

---

## 5. Communication Inside Namespace

Inside same namespace:

```bash
db-service
```

No need for full DNS.

---

## 6. Communication Across Namespaces

Use full DNS:

```bash
db-service.dev.svc.cluster.local
```

### Structure:

```bash
<service>.<namespace>.svc.cluster.local
```

### Breakdown:

* service → db-service
* namespace → dev
* svc → service type
* cluster.local → cluster domain

---

## 7. Important kubectl Commands

### List pods (default namespace)

```bash
kubectl get pods
```

### List pods in specific namespace

```bash
kubectl get pods -n dev
```

### List all namespaces

```bash
kubectl get namespaces
```

### List pods in all namespaces

```bash
kubectl get pods --all-namespaces
```

---

## 8. Creating Namespace

### Method 1: CLI

```bash
kubectl create namespace dev
```

### Method 2: YAML

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

---

## 9. Creating Resources in Namespace

### CLI:

```bash
kubectl apply -f pod.yaml -n dev
```

### YAML:

```yaml
metadata:
  name: mypod
  namespace: dev
```

---

## 10. Switch Default Namespace

Instead of writing `-n dev` every time:

```bash
kubectl config set-context --current --namespace=dev
```

Now:

```bash
kubectl get pods
```

→ shows pods in `dev`

---

## 11. Resource Quotas (Very Important in real-world)

Namespaces can limit resources:

Example:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: "8Gi"
```

### Why?

* Prevent one team from using all resources

---

## 12. Key Concepts to Remember

### 1. Isolation

* Namespaces separate environments

### 2. Same Name Allowed

* `db-service` can exist in multiple namespaces

### 3. DNS Changes Across Namespace

* Same namespace → short name
* Different namespace → full DNS

### 4. Default Behavior

* Everything runs in `default` unless specified

---

## 13. Real DevOps Use Case

Single cluster:

* `dev` namespace → testing
* `prod` namespace → live app

Benefits:

* No need for multiple clusters
* Cost-efficient
* Safe isolation

---

## 14. Common Mistakes (Important)

### 1. Forgetting namespace

```bash
kubectl get pods
```

→ might show nothing (wrong namespace)

---

### 2. Service not reachable

* Cause: wrong namespace in DNS

---

### 3. Resource created in wrong namespace

* Fix: always define namespace in YAML

---

## Final Summary (One Line)

> Namespace = **logical boundary that isolates and organizes resources inside a Kubernetes cluster**

---

