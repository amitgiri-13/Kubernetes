# Kubernetes Service: LoadBalancer - Notes

##  Introduction

* **LoadBalancer** is a Kubernetes Service type used to:

  * Expose applications **externally**
  * Provide a **single access point (URL/IP)**
* Commonly used for **frontend applications**

---

##  Recap: NodePort Problem

With **NodePort**:

* App is accessible via:

  ```
  NodeIP:NodePort
  ```
* In a cluster with multiple nodes:

  * Multiple IP + port combinations

###  Problems:

* Not user-friendly
* Hard to manage
* No single domain (e.g., `app.com`)

---

##  Solution: LoadBalancer Service

* Provides:

  * **Single external IP / URL**
  * Built-in **load balancing**
* Automatically distributes traffic across nodes/pods

---

##  Real-World Requirement

Users expect:

```id="f2z1xy"
https://voting-app.com
https://result-app.com
```

 Not:

```id="h7k9lp"
192.168.1.10:30008
192.168.1.11:30008
```

---

##  How It Works

### Without LoadBalancer

* You manually:

  * Create VM
  * Install load balancer (e.g., Nginx, HAProxy)
  * Configure routing

 Complex & time-consuming

---

### With LoadBalancer (Kubernetes)

* Set:

```yaml id="k8s7dl"
spec:
  type: LoadBalancer
```

 Kubernetes:

* Automatically provisions:

  * External load balancer
  * Public IP
  * Routing rules

---

##  Cloud Requirement

 Works only on supported cloud providers:

* AWS
* GCP
* Azure

 Uses **native cloud load balancers**

---

##  Behavior in Local Environment

* On environments like:

  * VirtualBox
  * Minikube (without addons)

 LoadBalancer behaves like:

* **NodePort only**
* No real external load balancer created

---

##  Architecture Flow

```id="q9m3pt"
User → LoadBalancer → Node → Pods
```

---

##  LoadBalancer Service Definition

```yaml id="x8c4vd"
apiVersion: v1
kind: Service

metadata:
  name: frontend-service

spec:
  type: LoadBalancer

  selector:
    app: frontend

  ports:
    - port: 80
      targetPort: 80
```

---

##  Load Balancing Behavior

* Distributes traffic:

  * Across nodes
  * Across pods

 No manual setup required

---

##  Multi-Node Scenario

* Even if pods run on only some nodes:

  * LoadBalancer routes traffic correctly
* Works across entire cluster

---

##  Key Benefits

*  Single external endpoint
*  Automatic load balancing
*  No manual LB setup
*  Cloud-native integration

---

##  Limitations

* Requires cloud provider support
* Not fully functional in local setups

---

##  Key Takeaways

* LoadBalancer:

  * Best for **production external access**
  * Simplifies exposure of applications
* Internally:

  * Still uses NodePort + ClusterIP

---

##  Summary

> LoadBalancer Service provides a **simple, scalable, and production-ready way** to expose applications externally with a single endpoint.

---
