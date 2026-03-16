# Kubelet (Kubernetes)

## Introduction
The **Kubelet** is an agent that runs on every **worker node** in a Kubernetes cluster.

It is responsible for **managing containers and pods on the node** and communicating with the **Kube API Server**.


---

# Responsibilities of Kubelet

The Kubelet performs several important tasks on the worker node:

- Registers the node with the Kubernetes cluster
- Receives instructions from the control plane
- Creates and manages pods and containers
- Monitors pod and container health
- Reports node and pod status to the API Server

---

# Kubelet Workflow

### 1. Node Registration

When Kubelet starts:

- It registers the **worker node** with the Kubernetes cluster.

---

### 2. Pod Deployment

When a pod is scheduled to the node:

1. Scheduler selects the node
2. API Server notifies the **Kubelet**
3. Kubelet instructs the **container runtime**

Example container runtimes:

- Docker
- containerd
- CRI-O

The runtime then:

- Pulls the container image
- Starts the container

---

### 3. Pod Monitoring

The Kubelet continuously:

- Monitors the status of pods
- Checks container health
- Restarts containers if needed

---

### 4. Status Reporting

The Kubelet regularly sends updates to the **Kube API Server** about:

- Node status
- Pod status
- Container health

---

# Interaction with Container Runtime

Kubelet interacts with the **container runtime engine** to manage containers.

Example workflow:

1. Kubelet receives pod instructions
2. Kubelet asks runtime to pull image
3. Runtime creates container
4. Kubelet monitors container lifecycle

---

# Installing Kubelet

Unlike some control plane components, **Kubelet must be installed manually on worker nodes**.

Installation steps:

1. Download the **Kubelet binary**
2. Extract the package
3. Run it as a **system service**

Example command:

```bash
kubelet
````

---

# Kubelet in kubeadm Clusters

When using **kubeadm**:

* Kubelet **is not automatically deployed**
* It must be **installed manually on each worker node**

After installation, kubeadm configures it to join the cluster.

---

# Viewing Kubelet Process

You can check if Kubelet is running on a worker node using:

```bash
ps aux | grep kubelet
```

This displays the running process and configuration options.

---

# Future Configuration Topics

Kubelet configuration may involve:

* TLS certificates
* Secure communication with API Server
* TLS bootstrapping
* Node authentication


---

# Key Takeaways

* **Kubelet runs on every worker node.**
* It manages **pods and containers on the node**.
* Communicates with the **Kube API Server**.
* Uses the **container runtime** to run containers.
* Reports node and pod status regularly.
* Must be **installed manually on worker nodes**.

---
