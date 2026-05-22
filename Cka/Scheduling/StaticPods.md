# Static Pods in Kubernetes

# Introduction

Normally in Kubernetes:

* User submits Pod definition to the API Server
* Scheduler decides which node runs the pod
* Kubelet receives instructions from API Server
* Container runtime runs the containers

Flow:

```text id="p8j0j8"
User -> API Server -> Scheduler -> Kubelet -> Container Runtime
```

---

# What are Static Pods?

Static Pods are pods managed directly by the **Kubelet** without involvement of:

* API Server
* Scheduler
* Controller Manager
* ETCD

The Kubelet reads pod definition files directly from a directory on the node and creates pods automatically.

---

# Key Idea

The Kubelet can work independently even if:

* No Kubernetes cluster exists
* No master/control plane exists
* No API Server exists

As long as:

* Kubelet is installed
* Container runtime exists

the node can still run pods.

---

# How Static Pods Work

## Step 1: Create Pod Definition File

Example:

```yaml id="y33i71"
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
```

---

## Step 2: Place File in Manifest Directory

Example directory:

```bash id="k5w7pf"
/etc/kubernetes/manifests
```

---

## Step 3: Kubelet Watches Directory

The Kubelet continuously checks the directory for:

* New files
* Modified files
* Deleted files

---

# Automatic Behavior

## File Added

```text id="f6s9bl"
New YAML file -> Pod created
```

---

## File Modified

```text id="y7x3wx"
File updated -> Pod recreated
```

---

## File Deleted

```text id="w8d7pu"
File removed -> Pod deleted
```

---

# Important Characteristics

* Managed directly by Kubelet
* No API Server required
* Only Pod objects supported
* Automatically restarted if crashed

---

# Static Pods Support Only Pods

You cannot create:

* Deployments
* ReplicaSets
* Services
* DaemonSets

using static pod manifests.

Reason:

* Kubelet understands only Pod objects.

---

# Configuring Static Pod Path

The Kubelet needs a directory path where manifest files are stored.

---

# Method 1: pod-manifest-path

Configured directly in Kubelet service.

Example:

```bash id="skt8r6"
--pod-manifest-path=/etc/kubernetes/manifests
```

---

# Method 2: Kubelet Config File

Kubelet may use a config file.

Example:

```bash id="7yw56w"
--config=/var/lib/kubelet/config.yaml
```

Inside config file:

```yaml id="s60hkp"
staticPodPath: /etc/kubernetes/manifests
```

---

# How to Identify Static Pod Directory

## Step 1

Check Kubelet service:

```bash id="vkggot"
ps -ef | grep kubelet
```

or

```bash id="bbceyy"
systemctl status kubelet
```

---

## Step 2

Look for:

```text id="1lg96r"
--pod-manifest-path
```

or

```text id="kpw4bl"
--config
```

---

## Step 3

If config file is used:

* Open config file
* Find:

```yaml id="6vn0gj"
staticPodPath:
```

---

# Viewing Static Pods

## Without API Server

Use container runtime commands:

```bash id="z9yzka"
docker ps
```

or

```bash id="ytz9jw"
crictl ps
```

Reason:

* No API Server exists
* `kubectl` cannot work

---

# Static Pods in a Cluster

When node belongs to a Kubernetes cluster:

* Kubelet still creates static pods locally
* API Server becomes aware of them

---

# Mirror Pods

When static pods run inside a cluster:

* Kubelet creates a mirror object in API Server

This allows:

```bash id="t4lspl"
kubectl get pods
```

to display static pods.

---

# Important Note About Mirror Pods

Mirror pods are:

* Read-only
* Cannot be edited
* Cannot be deleted using kubectl

To modify them:

* Edit manifest file on node

To delete them:

* Remove manifest file

---

# Static Pod Naming

Static pod names automatically include node name.

Example:

```text id="wzglpw"
kube-apiserver-node01
```

---

# Major Use Case of Static Pods

## Running Kubernetes Control Plane Components

Kubeadm uses static pods to deploy:

* kube-apiserver
* kube-controller-manager
* kube-scheduler
* etcd

---

# How kubeadm Uses Static Pods

## Process

1. Install Kubelet
2. Place control plane pod manifests in:

   ```bash
   /etc/kubernetes/manifests
   ```
3. Kubelet automatically creates pods

---

# Benefits

* Easy deployment
* Automatic restart on failure
* No manual service management
* Control plane becomes self-hosted

---

# Common Static Pod Manifest Location

```bash id="0w4w8v"
/etc/kubernetes/manifests
```

Typical files:

```bash id="2ob0j8"
etcd.yaml
kube-apiserver.yaml
kube-controller-manager.yaml
kube-scheduler.yaml
```

---

# Static Pods vs DaemonSets

| Feature               | Static Pods               | DaemonSets           |
| --------------------- | ------------------------- | -------------------- |
| Created By            | Kubelet                   | DaemonSet Controller |
| API Server Required   | No                        | Yes                  |
| Scheduler Used        | No                        | No                   |
| Managed Through       | Local manifest files      | Kubernetes API       |
| Purpose               | System/control plane pods | One pod per node     |
| Editable via kubectl  | No                        | Yes                  |
| Works Without Cluster | Yes                       | No                   |

---

# Scheduler Behavior

Both:

* Static Pods
* DaemonSet Pods

are ignored by the Scheduler.

Reason:

* Their node assignment is already determined.

---

# Example Static Pod Manifest

```yaml id="z0pl7v"
apiVersion: v1
kind: Pod

metadata:
  name: static-nginx

spec:
  containers:
  - name: nginx
    image: nginx
```

Place it in:

```bash id="45d9u2"
/etc/kubernetes/manifests
```

Kubelet automatically creates the pod.

---

# Important Commands

## Check Static Pod Directory

```bash id="j9gr44"
ps -ef | grep kubelet
```

---

## View Running Containers

```bash id="q78mta"
docker ps
```

or

```bash id="hjdkh8"
crictl ps
```

---

## View Static Pods in Cluster

```bash id="tx9zcf"
kubectl get pods -n kube-system
```

---

# Summary

Static Pods are pods directly managed by the Kubelet without API Server involvement.


Key points:

* Created from manifest files
* Stored in static pod directory
* Automatically restarted
* Used heavily for Kubernetes control plane components
* kubeadm deploys control plane as static pods
* Only Pod objects are supported
* Static Pods differ significantly from DaemonSets
