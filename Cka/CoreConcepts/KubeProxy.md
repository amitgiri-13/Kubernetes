# Kube Proxy (Kubernetes)

## Introduction
**Kube Proxy** is a network component that runs on every node in a Kubernetes cluster.

Its main job is to **manage network communication for Kubernetes Services** by forwarding traffic to the appropriate backend pods.

---

# Pod Networking in Kubernetes

Inside a Kubernetes cluster:

- Every **pod can communicate with every other pod**
- This is achieved using a **Pod Network**

A **Pod Network** is a virtual network that:

- Spans across all nodes
- Connects all pods
- Enables pod-to-pod communication

Examples of pod networking solutions:

- Flannel
- Calico
- Weave
- Cilium

---

# Problem with Pod IPs

Pods communicate using **IP addresses**.

Example:

```

Web App Pod → Database Pod

```

However:

- Pod IPs are **not permanent**
- If a pod restarts, it may receive a **new IP**

This makes direct pod communication unreliable.

---

# Kubernetes Service

To solve this issue, Kubernetes provides **Services**.

A **Service**:

- Provides a **stable name**
- Provides a **stable IP address**
- Routes traffic to backend pods

Example:

```

Service Name: db
Service IP: 10.96.0.12

```

Now the web application connects using:

```

db

```

instead of a pod IP.

---

# What is a Kubernetes Service Internally?

Important concept:

> A **Service is not a container or a pod**.

It is a **virtual object** in Kubernetes.

Characteristics:

- No container
- No network interface
- No running process
- Exists only inside **Kubernetes control plane**

---

# How Services Actually Work

Even though services are virtual, they must still route traffic to pods.

This routing is handled by **Kube Proxy**.

---

# Role of Kube Proxy

**Kube Proxy** runs on every node and:

1. Watches the API Server for **new services**
2. Creates network rules
3. Routes traffic from service IP → backend pod

---

# Traffic Flow Example

Example:

```

Service IP: 10.96.0.12
Backend Pod IP: 10.32.0.15

```

When a pod sends traffic to:

```

10.96.0.12

```

Kube Proxy redirects it to:

```

10.32.0.15

```

---

# How Kube Proxy Implements Routing

One common method used by Kube Proxy is:

**iptables rules**

Example rule logic:

```

Destination: Service IP
Forward to: Pod IP

````

These rules are created **on every node in the cluster**.

---

# Installing Kube Proxy

If installing manually:

1. Download **kube-proxy binary**
2. Extract it
3. Run it as a service

Example:

```bash
kube-proxy
````

---

# Kube Proxy in kubeadm Clusters

When using **kubeadm**:

* Kube Proxy is deployed automatically
* Runs as a **DaemonSet**

A **DaemonSet** ensures:

> One pod runs on every node in the cluster.

Check Kube Proxy pods:

```bash id="8gntq1"
kubectl get pods -n kube-system
```

---

# Kube Proxy Summary

| Feature | Description             |
| ------- | ----------------------- |
| Runs on | Every node              |
| Purpose | Service networking      |
| Watches | API Server for services |
| Creates | Routing rules           |
| Method  | iptables or IPVS        |

---

# Key Takeaways

* **Kube Proxy enables Kubernetes service networking.**
* Runs on **every node** in the cluster.
* Creates rules to route traffic from **service IP → pod IP**.
* Uses **iptables or IPVS** for packet forwarding.
* In **kubeadm clusters**, it runs as a **DaemonSet**.

---
