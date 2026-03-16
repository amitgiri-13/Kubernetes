# ETCD

A distributed, reliable key-value store used by Kubernetes to persist cluster configuration, state, and metadata. It ensures that all control plane components have a consistent view of the cluster.

## Setup etcd

```bash
# install (fedora)
sudo dnf install etcd

# enable and start
sudo systemctl enable etcd
sudo systemctl start etcd

# checking status
sudo systemctl status etcd

# set environment variables for client
export ETCDCTL_API=3
export ETCDCTL_ENDPOINTS=http://127.0.0.1:2379

```

Basic Key-Value Operations

```bash
# put a key
etcdctl put mykey "myvalue"

# get a key
etcdctl get mykey

# get all keys
etcdctl get "" --prefix

# delete a key
etcdctl del mykey

# delete key with prefix
etcdctl del "" --prefix
```

Advanced Operation

```bash
# check cluster health
etcdctl endpoint health

# list members 
etcdctl member list

# add member
etcdctl member add <name> --peer-urls=<url>

# remove member
etcdctl member remove <ID>

# snapshot
etcdctl snapshot save snapshot.db

# restore snapshot
etcdctl snapshot restore snapshot.db
```

---

# etcd Role in Kubernetes

## Introduction
**etcd** is a distributed key-value data store used by Kubernetes to store all cluster data.  
It acts as the **primary database for Kubernetes**.

---

# What etcd Stores

The etcd data store keeps information about the entire cluster, including:

- Nodes
- Pods
- ConfigMaps
- Secrets
- Service Accounts
- Roles
- RoleBindings
- ReplicaSets
- Deployments

Any information retrieved using:

```bash
kubectl get
````

comes from the **etcd server**.

---

# Cluster State Changes

Whenever changes occur in the cluster, they are recorded in **etcd**.

Examples of changes:

* Adding new nodes
* Deploying pods
* Creating ReplicaSets
* Updating configurations

A change is considered **complete only after it is stored in etcd**.

---

# etcd Deployment Methods

etcd deployment depends on how the Kubernetes cluster is set up.

Two common methods:

1. **Manual setup (from scratch)**
2. **Using kubeadm**

---

# etcd Deployment (Manual Setup)

If Kubernetes is installed manually:

1. Download etcd binaries
2. Install them on the **master node**
3. Configure etcd as a **service**

Example service execution:

```bash
etcd
```

The configuration includes:

* TLS certificate settings
* Cluster configuration options

These options are important when setting up **secure communication and high availability**.

---

# Important etcd Configuration Option

### Advertised Client URL

This specifies the address where etcd listens for client requests.

Example:

```
--advertise-client-urls=https://<IP_ADDRESS>:2379
```

Key points:

* Default etcd port → **2379**
* Used by **Kube API Server** to connect to etcd.

---

# etcd Deployment with kubeadm

When using **kubeadm**, etcd is automatically deployed as a **pod**.

Location:

```
Namespace: kube-system
```

View etcd pod:

```bash
kubectl get pods -n kube-system
```

Example etcd pod name:

```
etcd-master
```

---

# Exploring etcd Database

You can explore etcd data using **etcdctl** inside the etcd pod.

Example command:

```bash
etcdctl get "" --prefix
```

This lists **all keys stored in etcd**.

---

# etcd Data Structure in Kubernetes

Kubernetes stores data in a hierarchical structure.

Root directory:

```
/registry
```

Under this directory:

```
/registry/nodes
/registry/pods
/registry/replicasets
/registry/deployments
/registry/services
```

Each resource type has its own directory.

---

# High Availability etcd

In **High Availability (HA) Kubernetes clusters**:

* Multiple **master nodes** exist.
* Each master node runs an **etcd instance**.

These etcd instances form an **etcd cluster**.

---

# etcd Cluster Configuration

To configure multiple etcd nodes, use:

```
--initial-cluster
```

This option lists all etcd members in the cluster.

Example:

```
--initial-cluster=node1=https://10.0.0.1:2380,node2=https://10.0.0.2:2380
```

This ensures all etcd instances know about each other.

---

# Key Takeaways

* **etcd is the Kubernetes cluster database.**
* Stores all cluster state and configuration data.
* Kubernetes reads and writes cluster data through **etcd**.
* Runs as:

  * a **service (manual setup)** or
  * a **pod (kubeadm setup)**.
* Default etcd client port → **2379**.
* Supports clustering for **high availability**.

---


