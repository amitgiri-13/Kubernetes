# Kubernetes Service: ClusterIP - Notes

##  Introduction

* **ClusterIP** is the **default Kubernetes Service type**
* Used for **internal communication** within the cluster
* Enables communication between **different application tiers**

---

##  Typical Application Architecture

A full-stack app may have:

* Frontend Pods → UI (Web server)
* Backend Pods → Business logic
* Redis Pods → Cache (key-value store)
* Database Pods → Persistent storage (e.g., MySQL)

---

##  Problem Without ClusterIP

* Pods have **dynamic IPs**

  * Can change anytime
  * Not reliable for communication

* Example problem:

  * Frontend → Backend communication
  * Which backend pod to connect?
  * Who manages load balancing?

---

##  Solution: ClusterIP Service

* Groups related Pods into a **single logical unit**
* Provides:

  * **Single stable IP**
  * **DNS name**
* Acts as a **single entry point**

---

##  How It Works

* Service is created for a group of Pods (e.g., backend)
* Other Pods connect to:

  * **Service name** (preferred)
  * OR Cluster IP

 Kubernetes:

* Automatically routes traffic to one of the Pods
* Uses **random load balancing**

---

##  Example Flow

```id="c8k9zp"
Frontend Pod → Backend Service → Backend Pods
```

```id="qk3m2a"
Backend Pod → Redis Service → Redis Pods
```

---

##  Benefits

*  Stable communication (no dependency on Pod IPs)
*  Load balancing across Pods
*  Easy scaling (add/remove Pods without impact)
*  Loose coupling of services

---

##  Key Concept

> Services act as a **network abstraction layer** for Pods

---

## ClusterIP Service Definition

```yaml id="2w7mfl"
apiVersion: v1
kind: Service

metadata:
  name: backend-service

spec:
  type: ClusterIP   # optional (default)

  selector:
    app: backend   # matches pod labels

  ports:
    - targetPort: 80
      port: 80
```

---

##  Connecting Service to Pods

* Done using **labels & selectors**

### Pod:

```yaml id="f7slp9"
labels:
  app: backend
```

### Service:

```yaml id="m2v4xa"
selector:
  app: backend
```

---

##  Commands

### Create Service

```bash id="4w9jlp"
kubectl create -f service.yaml
```

### View Services

```bash id="wz2kde"
kubectl get services
```

---

##  Accessing the Service

* Inside cluster:

  * Using **Cluster IP**
  * OR **Service Name (recommended)**

```id="r7pn0v"
http://backend-service
```

---

##  Dynamic Behavior

* Pods added → automatically included
* Pods removed → automatically excluded

 No manual configuration required

---

##  Scaling Advantage

* Each tier (frontend/backend/db):

  * Can scale independently
  * Communication remains unaffected

---

##  Key Takeaways

* ClusterIP is:

  * **Default service type**
  * Used for **internal communication only**
* Provides:

  * Stable endpoint
  * Load balancing
  * Service discovery

---

##  Summary

> ClusterIP allows different parts of your application to **communicate reliably inside the cluster**, without worrying about changing Pod IPs.

---
