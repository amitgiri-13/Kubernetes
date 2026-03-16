# Kube API Server (Kubernetes)

## Introduction
The **Kube API Server** is the **primary management component** of Kubernetes.  
It acts as the **central control point** through which all cluster operations pass.

Whenever a user interacts with the cluster using `kubectl`, the request is sent to the **Kube API Server**.

---

# How kubectl Works

When you run a command like:

```bash
kubectl get pods
````

The process is:

1. `kubectl` sends a request to **Kube API Server**
2. API Server **authenticates** the request
3. API Server **validates** the request
4. API Server retrieves data from **etcd**
5. API Server returns the response to the user

All cluster information ultimately comes from **etcd**.

---

# Accessing API Server Directly

You can interact with the API Server without `kubectl` by sending HTTP requests.

Example:

```bash
POST /api/v1/pods
```

This allows direct communication with the Kubernetes API.

---

# Pod Creation Workflow

When a pod is created, several components interact with each other.

### Step-by-step process

1. User sends request (`kubectl create pod`)
2. Request reaches **Kube API Server**
3. API Server authenticates and validates the request
4. API Server creates a **Pod object** (without assigning a node)
5. API Server stores pod information in **etcd**

---

### Scheduler Interaction

6. **Scheduler** monitors the API Server
7. Scheduler detects a pod without a node
8. Scheduler selects the **best node**
9. Scheduler informs the **API Server**

---

### Node Execution

10. API Server updates **etcd**
11. API Server informs the **Kubelet** on the selected worker node
12. Kubelet instructs the **container runtime** (Docker/containerd) to run the container
13. Kubelet reports status back to API Server
14. API Server updates the state in **etcd**

---

# Key Responsibilities of Kube API Server

The Kube API Server performs several important functions:

* **Authentication** – verifies user identity
* **Authorization** – checks user permissions
* **Validation** – validates API requests
* **Cluster state management**
* **Communication with etcd**

Important note:

> The **Kube API Server is the only component that directly interacts with etcd.**

Other components use the API server.

---

# Components That Use API Server

The following components communicate with the API server:

* **Scheduler**
* **Kube Controller Manager**
* **Kubelet**
* **kubectl**
* **External clients**

---

# Installing Kube API Server

If setting up Kubernetes manually:

1. Download the **Kube API Server binary** from Kubernetes release page
2. Install it on the **master node**
3. Configure it to run as a **service**

Example command:

```bash
kube-apiserver
```

---

# Important API Server Configuration

The API Server has many configuration parameters.

These include settings for:

* Authentication
* Authorization
* Security
* Networking
* Cluster communication

Many options involve **TLS certificates** used for secure communication between components.

---

# Important Option: etcd Servers

The API Server connects to etcd using:

```
--etcd-servers
```

Example:

```
--etcd-servers=https://127.0.0.1:2379
```

This specifies the **location of the etcd cluster**.

---

# Viewing API Server Configuration

How you view configuration depends on the cluster setup.

---

## kubeadm Setup

If the cluster was created using **kubeadm**:

* API Server runs as a **pod**
* Located in:

```
kube-system namespace
```

View it with:

```bash
kubectl get pods -n kube-system
```

Configuration file location:

```
/etc/kubernetes/manifests/kube-apiserver.yaml
```

---

## Non-kubeadm Setup

If installed manually, it runs as a **system service**.

Service file location:

```
/etc/systemd/system/kube-apiserver.service
```

View running process:

```bash
ps aux | grep kube-apiserver
```

---

# Key Takeaways

* **Kube API Server is the central component of Kubernetes.**
* All operations pass through the API Server.
* It handles **authentication, validation, and cluster management**.
* It is the **only component that communicates directly with etcd**.
* Other components interact with the cluster through the API Server.

---

