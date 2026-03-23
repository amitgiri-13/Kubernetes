# Kubernetes Deployments - Notes

##  Introduction

* A **Deployment** in Kubernetes is used to manage application deployment in a **production environment**.
* It sits **above ReplicaSet** in the Kubernetes object hierarchy.

---

##  Why Deployments?

In real-world production:

### 1. Multiple Instances

* You need **multiple instances** of an application (e.g., web server) for:

  * High availability
  * Load balancing

---

### 2. Rolling Updates

* When a new version is available (e.g., from Docker registry):

  * You **should not update all instances at once**
  * Instead, update **gradually (one by one)**

 This is called **Rolling Update**

---

### 3. Rollback

* If something breaks after an update:

  * You can **revert to the previous version**

 This is called **Rollback**

---

### 4. Pause & Resume Changes

* You may want to:

  * Update version
  * Scale replicas
  * Modify resources

* Instead of applying changes immediately:

  * **Pause deployment**
  * Make changes
  * **Resume deployment**

---

## Kubernetes Object Hierarchy

```
Deployment
   ↓
ReplicaSet
   ↓
Pods
   ↓
Containers
```

* **Pod** → Runs a single instance
* **ReplicaSet** → Maintains multiple Pods
* **Deployment** → Manages ReplicaSets and updates

---

##  Deployment Definition File

Similar to ReplicaSet, but with:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  labels:
    app: my-app

spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app

  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-container
          image: my-image
```

---

##  Commands

### Create Deployment

```bash
kubectl create -f deployment.yaml
```

### View Deployments

```bash
kubectl get deployments
```

### View ReplicaSets

```bash
kubectl get replicaset
```

### View Pods

```bash
kubectl get pods
```

### View All Resources

```bash
kubectl get all
```

---

##  How Deployment Works Internally

1. Deployment is created
2. It automatically creates a **ReplicaSet**
3. ReplicaSet creates **Pods**
4. Pods run your application containers

---

##  Key Features of Deployments

* Rolling Updates
* Rollbacks
* Scaling
* Pause & Resume
* Declarative updates

---
