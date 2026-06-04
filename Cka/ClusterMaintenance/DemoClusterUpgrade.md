# Kubernetes Cluster Upgrade Using kubeadm (v1.28 → v1.29)

## Overview

Kubernetes clusters should be upgraded one minor version at a time.

Example:

```text
v1.27 → v1.28 → v1.29
```

This demo demonstrates upgrading a kubeadm-managed cluster from **v1.28.x** to **v1.29.x**.

---

# Upgrade Order

Always follow this sequence:

```text
1. Update package repositories
2. Upgrade kubeadm
3. Upgrade Control Plane
4. Upgrade kubelet and kubectl on Control Plane
5. Upgrade Worker Nodes
6. Verify Cluster Health
```

---

# Cluster Information

Check current Kubernetes version:

```bash
kubectl get nodes
```

Example:

```text
NAME           STATUS   VERSION
controlplane   Ready    v1.28.0
node01         Ready    v1.28.0
```

Check OS version:

```bash
cat /etc/*release
```

Example:

```text
Ubuntu 20.04
```

---

# Kubernetes Package Repository Change

Older repositories:

```text
apt.kubernetes.io
yum.kubernetes.io
```

have been deprecated.

Use:

```text
pkgs.k8s.io
```

for all newer Kubernetes releases.

---

# Configure Kubernetes Repository

## Add Repository

For Kubernetes v1.29:

```bash
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

## Add GPG Key

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

## Update Package Index

```bash
sudo apt-get update
```

Perform these steps on:

* Control Plane Node
* All Worker Nodes

---

# Find Available Versions

Check available kubeadm versions:

```bash
apt-cache madison kubeadm
```

Example output:

```text
1.29.3-1.1
1.29.2-1.1
1.29.1-1.1
1.29.0-1.1
```

Choose the latest patch release.

Example:

```text
1.29.3-1.1
```

---

# Upgrade Control Plane

## Step 1: Upgrade kubeadm

```bash
sudo apt-mark unhold kubeadm
sudo apt-get install -y kubeadm=1.29.3-1.1
sudo apt-mark hold kubeadm
```

Verify:

```bash
kubeadm version
```

---

## Step 2: Review Upgrade Plan

```bash
sudo kubeadm upgrade plan
```

Purpose:

* Checks compatibility
* Shows upgrade path
* Lists components upgraded automatically
* Lists components requiring manual upgrades

Example:

```text
Current Cluster: v1.28.0
Target Version: v1.29.3
```

---

## Step 3: Upgrade Control Plane Components

```bash
sudo kubeadm upgrade apply v1.29.3
```

This upgrades:

* kube-apiserver
* kube-controller-manager
* kube-scheduler
* CoreDNS
* kube-proxy

if applicable.

---

# Important Note About Version Output

After upgrade:

```bash
kubectl get nodes
```

may still show:

```text
v1.28.0
```

### Why?

The VERSION column displays the **kubelet version**, not the API server version.

The control plane is already upgraded, but kubelet still needs manual upgrading.

---

# Drain Control Plane Node

Before upgrading kubelet:

```bash
kubectl drain controlplane --ignore-daemonsets
```

Drain performs:

* Eviction of workloads
* Marks node unschedulable
* Prevents disruption during kubelet restart

---

# Upgrade kubelet and kubectl

```bash
sudo apt-mark unhold kubelet kubectl

sudo apt-get install -y \
kubelet=1.29.3-1.1 \
kubectl=1.29.3-1.1

sudo apt-mark hold kubelet kubectl
```

Restart kubelet:

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

---

# Re-enable Scheduling

```bash
kubectl uncordon controlplane
```

Verify:

```bash
kubectl get nodes
```

Expected:

```text
controlplane Ready v1.29.3
```

---

# Additional Control Plane Nodes

For HA clusters:

1. Upgrade kubeadm
2. Run:

```bash
sudo kubeadm upgrade node
```

3. Upgrade kubelet
4. Restart kubelet
5. Uncordon node

Repeat for every additional control-plane node.

---

# Upgrade Worker Nodes

Perform worker node upgrades one node at a time.

---

## Step 1: Upgrade kubeadm

On worker node:

```bash
sudo apt-mark unhold kubeadm
sudo apt-get install -y kubeadm=1.29.3-1.1
sudo apt-mark hold kubeadm
```

---

## Step 2: Upgrade Node Configuration

```bash
sudo kubeadm upgrade node
```

Unlike control planes:

```bash
kubeadm upgrade apply
```

is NOT used on worker nodes.

---

## Step 3: Drain Worker Node

From control plane:

```bash
kubectl drain node01 --ignore-daemonsets
```

---

## Step 4: Upgrade kubelet and kubectl

On worker node:

```bash
sudo apt-mark unhold kubelet kubectl

sudo apt-get install -y \
kubelet=1.29.3-1.1 \
kubectl=1.29.3-1.1

sudo apt-mark hold kubelet kubectl
```

Restart kubelet:

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

---

## Step 5: Uncordon Worker Node

From control plane:

```bash
kubectl uncordon node01
```

Verify:

```bash
kubectl get nodes
```

Expected:

```text
NAME           STATUS   VERSION
controlplane   Ready    v1.29.3
node01         Ready    v1.29.3
```

---

# Multiple Worker Nodes

Repeat for each worker node:

```text
1. Upgrade kubeadm
2. kubeadm upgrade node
3. Drain node
4. Upgrade kubelet
5. Restart kubelet
6. Uncordon node
```

Upgrade nodes one at a time to avoid application downtime.

---

# Commands Summary

## Check Nodes

```bash
kubectl get nodes
```

## View Upgrade Plan

```bash
kubeadm upgrade plan
```

## Upgrade Control Plane

```bash
kubeadm upgrade apply <version>
```

## Upgrade Worker Configuration

```bash
kubeadm upgrade node
```

## Drain Node

```bash
kubectl drain <node> --ignore-daemonsets
```

## Restart kubelet

```bash
systemctl daemon-reload
systemctl restart kubelet
```

## Re-enable Scheduling

```bash
kubectl uncordon <node>
```

---

# Key Exam Points (CKA)

* Upgrade one minor version at a time.
* Upgrade **kubeadm first**.
* Upgrade **control plane before workers**.
* Run `kubeadm upgrade plan` before upgrading.
* Use `kubeadm upgrade apply` only on the first control-plane node.
* Use `kubeadm upgrade node` on worker nodes and additional control-plane nodes.
* Drain nodes before upgrading kubelet.
* `kubectl get nodes` displays the **kubelet version**.
* Restart kubelet after upgrading it.
* Uncordon nodes after maintenance.
* Upgrade worker nodes one by one to minimize downtime.
