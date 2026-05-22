# Kubernetes Static Pods Lab Notes

# Identifying Static Pods

## List All Pods in All Namespaces

```bash id="m2dx5q"
kubectl get pods -A
```

---

# How to Identify a Static Pod

## Method 1: Pod Name Pattern

Static pods usually have:

```text id="5zpbqy"
<pod-name>-<node-name>
```

Example:

```text id="5uq8zi"
kube-apiserver-controlplane
etcd-controlplane
kube-scheduler-controlplane
```

Observation:

* Node name is appended to pod name.

---

# Total Static Pods

Static pods found:

* kube-apiserver-controlplane
* etcd-controlplane
* kube-controller-manager-controlplane
* kube-scheduler-controlplane

Total:

```text id="hbgcjp"
4
```

---

# Method 2: Check Owner References

## View Pod YAML

```bash id="1fj02m"
kubectl get pod kube-apiserver-controlplane \
  -n kube-system -o yaml
```

---

## Look for ownerReferences

Example:

```yaml id="a5pd4m"
ownerReferences:
- apiVersion: v1
  kind: Node
  name: controlplane
```

Meaning:

* Pod is owned directly by a node
* Therefore it is a static pod

---

# Comparing with Normal Pods

## Example

```bash id="wz9nsf"
kubectl get pod coredns-xxxxx \
  -n kube-system -o yaml
```

Owner reference:

```yaml id="2f09qy"
ownerReferences:
- apiVersion: apps/v1
  kind: ReplicaSet
```

Meaning:

* Managed by ReplicaSet
* Not a static pod

---

# Components Deployed as Static Pods

## Static Pods

* kube-apiserver
* etcd
* kube-controller-manager
* kube-scheduler

---

# Components NOT Running as Static Pods

## CoreDNS

Reason:

* Managed through Deployment/ReplicaSet

---

## kube-proxy

Reason:

* Managed by DaemonSet

---

# Node Hosting Static Pods

Current node:

```text id="v81s4r"
controlplane
```

All static pods are running on:

* controlplane node

---

# Finding Static Pod Manifest Path

## Check Kubelet Configuration

```bash id="hmrwdg"
cat /var/lib/kubelet/config.yaml
```

Look for:

```yaml id="nwm4zh"
staticPodPath: /etc/kubernetes/manifests
```

---

# Static Pod Manifest Directory

```bash id="mjlwmq"
/etc/kubernetes/manifests
```

---

# List Manifest Files

```bash id="0h6n6u"
ls /etc/kubernetes/manifests
```

Example:

```text id="g3a4eo"
etcd.yaml
kube-apiserver.yaml
kube-controller-manager.yaml
kube-scheduler.yaml
```

Total manifest files:

```text id="kwnhy5"
4
```

---

# Finding Image Used by kube-apiserver

## Open Manifest File

```bash id="4n0wtl"
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

Look for:

```yaml id="a33qri"
image: k8s.gcr.io/kube-apiserver:v1.20.0
```

---

# Creating a Static Pod

## Requirement

Create:

* Pod name: `static-busybox`
* Image: `busybox`
* Command: `sleep 1000`

---

# Generate Pod YAML

```bash id="4k9f2t"
kubectl run static-busybox \
  --image=busybox \
  --dry-run=client \
  -o yaml \
  -- sleep 1000 > static-busybox.yaml
```

---

# Move YAML to Static Pod Directory

```bash id="vyz1m8"
mv static-busybox.yaml /etc/kubernetes/manifests/
```

---

# Verify Static Pod

```bash id="a1o1k0"
kubectl get pods
```

Example:

```text id="upjzhf"
static-busybox-controlplane
```

---

# Editing Static Pod Image

## Edit Manifest File

```bash id="4nyntx"
vi /etc/kubernetes/manifests/static-busybox.yaml
```

---

# Change Image

From:

```yaml id="s0a5hz"
image: busybox
```

To:

```yaml id="m3u2u5"
image: busybox:1.28.4
```

---

# Important Note

Always verify after editing.

Example:

```bash id="3gbwbf"
kubectl get pods -w
```

Watch pod recreation status.

---

# Deleting a Static Pod

## Problem

Deleting with kubectl does not permanently remove static pods.

Example:

```bash id="k0s1hk"
kubectl delete pod static-greenbox-node01
```

Pod gets recreated automatically.

Reason:

* Kubelet recreates pod from manifest file.

---

# Correct Way to Delete Static Pod

## Step 1: Find Node Hosting Pod

Example:

```text id="kquzgn"
static-greenbox-node01
```

Hosted on:

```text id="r78bxr"
node01
```

---

# Step 2: Get Node IP

```bash id="8ztv0j"
kubectl get nodes -o wide
```

---

# Step 3: SSH into Node

```bash id="p6mmto"
ssh <node-ip>
```

Example:

```bash id="jhz4aw"
ssh 10.30.1.2
```

---

# Step 4: Find Static Pod Path

Check kubelet config:

```bash id="i9v3yw"
cat /var/lib/kubelet/config.yaml
```

Example:

```yaml id="dkv17v"
staticPodPath: /etc/just-to-mess-with-you
```

---

# Step 5: Delete Manifest File

```bash id="x0g8jh"
rm /etc/just-to-mess-with-you/greenbox.yaml
```

---

# Step 6: Verify Pod Removal

```bash id="3u8ik0"
kubectl get pods -w
```

Pod enters:

```text id="7mb9f9"
Terminating
```

Then disappears.

---

# Important Concepts

## Static Pods are Managed by Kubelet

* Kubelet watches manifest directory
* Recreates deleted pods automatically
* Manifest file controls pod lifecycle

---

# Static Pod Lifecycle

| Action           | Result        |
| ---------------- | ------------- |
| Add YAML file    | Pod created   |
| Edit YAML file   | Pod recreated |
| Delete YAML file | Pod removed   |

---

# Important Commands

## List All Pods

```bash id="zv86rk"
kubectl get pods -A
```

---

## View Pod YAML

```bash id="v2t1r0"
kubectl get pod <pod-name> -o yaml
```

---

## Check Kubelet Config

```bash id="fgql9h"
cat /var/lib/kubelet/config.yaml
```

---

## Generate Pod YAML

```bash id="fx6mmt"
kubectl run <pod-name> \
  --image=<image> \
  --dry-run=client \
  -o yaml
```

---

## Watch Pod Changes

```bash id="a0egkp"
kubectl get pods -w
```

---

# Exam Tips

## Identify Static Pods Quickly

Look for:

* Pod names ending with node name
* Owner reference = Node

---

## Static Pods Cannot Be Permanently Deleted with kubectl

Always:

* Find manifest file
* Delete manifest file from node

---

## Always Verify Changes

After:

* Editing image
* Creating pod
* Deleting pod

run:

```bash id="4k8ubv"
kubectl get pods -w
```

---

# Summary

Static pods:

* Are managed directly by Kubelet
* Use manifest files stored on node
* Automatically restart/recreate
* Are heavily used for Kubernetes control plane components

Key operations:

* Create → add manifest file
* Modify → edit manifest file
* Delete → remove manifest file
