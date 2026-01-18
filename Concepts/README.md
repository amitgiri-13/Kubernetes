# Kubernetes – Complete Conceptual Overview

## 1. Overview

**Kubernetes (K8s)** is an open-source container orchestration platform designed to deploy, scale, and manage containerized applications automatically.

### Key Features
- **Portable** – Works across public cloud, private cloud, and on-premise
- **Extensible** – Supports custom APIs, operators, and plugins
- **Declarative** – Desired state is defined in YAML/JSON
- **Automated** – Self-healing, scaling, and rolling updates

### What Kubernetes Provides
- Container scheduling
- Service discovery and load balancing
- Storage orchestration
- Configuration and secrets management
- Automated rollouts and rollbacks

---

## 2. Cluster Architecture

A Kubernetes cluster is composed of a **Control Plane** and **Worker Nodes**.

### Control Plane Components

| Component | Description |
|--------|------------|
| kube-apiserver | Central management interface |
| etcd | Distributed key-value store |
| kube-scheduler | Assigns Pods to Nodes |
| kube-controller-manager | Maintains desired state |
| cloud-controller-manager | Cloud provider integration |

### Worker Node Components

| Component | Description |
|--------|------------|
| kubelet | Node agent |
| container runtime | Runs containers (containerd, CRI-O) |
| kube-proxy | Network traffic handling |

---

## 3. Containers

Containers package an application with all its dependencies, ensuring consistency across environments.

### Benefits
- Lightweight
- Fast startup
- Isolated execution
- Platform-independent

Kubernetes orchestrates containers but does not create them.

---

## 4. Workloads

### Pods
- Smallest deployable unit
- Contains one or more containers
- Shares networking and storage

### Workload Controllers

| Type | Use Case |
|----|---------|
| Deployment | Stateless applications |
| ReplicaSet | Ensures replica count |
| StatefulSet | Stateful workloads |
| DaemonSet | One Pod per Node |
| Job | One-time tasks |
| CronJob | Scheduled tasks |

---

## 5. Services, Load Balancing, and Networking

### Services
Provide stable networking endpoints for Pods.

#### Service Types

| Type | Purpose |
|----|--------|
| ClusterIP | Internal access |
| NodePort | External access via Node |
| LoadBalancer | Cloud-based LB |
| ExternalName | External service mapping |

### Networking Model
- Each Pod has its own IP
- Flat networking (no NAT)
- Implemented using CNI plugins

---

## 6. Storage

Kubernetes supports both ephemeral and persistent storage.

### Storage Types

| Type | Description |
|----|------------|
| emptyDir | Temporary storage |
| hostPath | Node filesystem |
| PersistentVolume (PV) | Physical storage |
| PersistentVolumeClaim (PVC) | Storage request |
| StorageClass | Dynamic provisioning |

---

## 7. Configuration

### ConfigMaps
Used for non-sensitive configuration data.

### Secrets
Used for sensitive data such as passwords and API keys.

⚠ Secrets are base64-encoded by default.

---

## 8. Security

### Security Layers
- Authentication and Authorization (RBAC)
- Pod Security Standards
- Network Policies
- Secrets management

---

## 9. Policies

Policies enforce best practices and security rules.

| Policy | Purpose |
|----|-------|
| RBAC | Access control |
| Pod Security Admission | Pod restrictions |
| NetworkPolicy | Traffic control |
| ResourceQuota | Namespace limits |
| LimitRange | Resource defaults |

---

## 10. Scheduling, Preemption, and Eviction

### Scheduling
Assigns Pods to Nodes based on resource availability and constraints.

### Preemption
High-priority Pods can evict lower-priority Pods.

### Eviction
Pods are removed due to resource pressure based on QoS:
- BestEffort
- Burstable
- Guaranteed

---

## 11. Cluster Administration

### Admin Tasks
- Cluster setup and upgrades
- Node management
- Backup and restore
- Monitoring and logging

---

## 12. Windows in Kubernetes

Kubernetes supports Windows worker nodes for running Windows-based applications.

### Limitations
- Linux-only control plane
- Limited volume and networking support

---

## 13. Extending Kubernetes

### Extension Methods

| Method | Description |
|----|------------|
| CRDs | Custom resources |
| Operators | Automated application management |
| Admission Controllers | Request validation |
| Plugins | Scheduler, CNI, CSI |

---

## 14. Feedback and Ecosystem

Kubernetes has a strong open-source ecosystem supported by CNCF, cloud providers, and the global community.

---

## Summary

Kubernetes is the foundation of cloud-native infrastructure, providing scalability, reliability, security, and extensibility for modern applications.

---