# Kubernetes Services - Notes

##  Introduction

* **Kubernetes Service** enables communication:

  * Between **components inside the cluster**
  * Between **applications and external users**

---

##  Why Services?

* Connect **frontend ↔ backend**
* Connect **applications ↔ external data sources**
* Expose applications to **end users**
* Enable **loose coupling** in microservices architecture

---

## Example Architecture

* Frontend Pods → Serve UI
* Backend Pods → Handle logic
* Database / External Service → Data source

 **Services connect all these components**

---

##  Problem Without Services

* Pods have **internal IPs (e.g., 10.x.x.x)**
* These are:

  * Not accessible externally
  * Not stable (can change)

 Need a stable and accessible layer → **Service**

---

##  What is a Service?

* A **Kubernetes object**
* Acts as a **bridge / proxy**
* Routes traffic to Pods

---

##  Types of Kubernetes Services

### 1. NodePort

* Exposes app on a **port of the node**
* Accessible externally using:

  ```
  NodeIP:NodePort
  ```

---

### 2. ClusterIP

* Default type
* Creates a **virtual IP inside cluster**
* Used for **internal communication**

---

### 3. LoadBalancer

* Creates an **external load balancer**
* Used in cloud environments (AWS, GCP, etc.)

---

##  NodePort Deep Dive

###  Three Ports Involved

| Port Type      | Description                    |
| -------------- | ------------------------------ |
| **TargetPort** | Port on Pod (container port)   |
| **Port**       | Port on Service                |
| **NodePort**   | Port on Node (external access) |

---

###  Example

* Pod (container) → `80`
* Service → `80`
* Node → `30008`

 Access app via:

```
http://<NodeIP>:30008
```

---

###  NodePort Range

* Default:

```
30000 – 32767
```

---

##  Service Definition (NodePort)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service

spec:
  type: NodePort

  selector:
    app: my-app   # matches pod labels

  ports:
    - targetPort: 80
      port: 80
      nodePort: 30008
```

---

##  Connecting Service to Pods

* Done using **labels & selectors**

### Example:

```yaml
selector:
  app: my-app
```

 Matches Pods with:

```yaml
labels:
  app: my-app
```

---

##  Commands

### Create Service

```bash
kubectl create -f service.yaml
```

### View Services

```bash
kubectl get services
```

---

##  Load Balancing Behavior

* If multiple Pods match selector:

  * Service automatically:

    * Selects all Pods
    * Distributes traffic

 Uses **random load balancing**

---

##  Multi-Node Behavior

* Service spans across **all nodes**
* Same NodePort is exposed on every node

 You can access app using:

```
<AnyNodeIP>:NodePort
```

---

##  Dynamic Nature

* If Pods:

  * Added → automatically included
  * Removed → automatically excluded

 No manual updates needed

---

##  Key Takeaways

* Service = **network abstraction layer**
* Provides:

  * Stable IP
  * Load balancing
  * Service discovery
* Works seamlessly across:

  * Single pod
  * Multiple pods
  * Multiple nodes

---

##  Summary

> Kubernetes Services make your application **accessible, scalable, and connected** without worrying about Pod IP changes.

---
