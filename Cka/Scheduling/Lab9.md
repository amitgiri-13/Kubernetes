# Kubernetes Admission Controllers Lab Notes

# Key Concept

Admission Controllers work:

```text id="9jqf2m"
After Authentication
After Authorization
Before Object Creation/Persistence
```

They:

* Validate requests
* Modify requests
* Enforce policies
* Automate operations

---

# Question 1

## What is NOT a function of Admission Controllers?

### Correct Answer

```text id="m4xlc0"
Authenticate Users
```

Reason:

* Authentication happens before admission controllers.

Flow:

```text id="p3b9kw"
Authentication
→ Authorization
→ Admission Controllers
```

---

# Question 2

## Which Admission Controller is NOT Enabled by Default?

Check enabled plugins:

```bash id="x2z4ae"
kubectl get pods -n kube-system
```

Find kube-apiserver pod:

```text id="6eh6tz"
kube-apiserver-controlplane
```

Execute command inside pod:

```bash id="5s2n1h"
kubectl exec -it kube-apiserver-controlplane \
-n kube-system -- kube-apiserver -h | \
grep enable-admission-plugins
```

Check default enabled plugins list.

### Correct Answer

```text id="kt8e7m"
NamespaceAutoProvision
```

Reason:

* Not present in default enabled plugins.

---

# Question 3

## Which Admission Controller is Enabled Manually?

Check kube-apiserver manifest:

```bash id="mbv4q7"
vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

Search:

```text id="3bdv5r"
/enable-admission-plugins
```

Alternative:

```bash id="5czn0x"
grep enable-admission-plugins \
/etc/kubernetes/manifests/kube-apiserver.yaml
```

### Correct Answer

```text id="y7r6tb"
NodeRestriction
```

Reason:

* Enabled manually
* Not enabled by default

---

# Question 4

## Create NGINX Pod in Non-Existing Namespace

Command:

```bash id="1j9t4p"
kubectl run nginx --image=nginx -n blue
```

Result:

```text id="0ujpx9"
Error from server (NotFound):
namespaces "blue" not found
```

Reason:

* Namespace does not exist
* NamespaceExists admission controller rejects request

---

# Question 5

## Enable NamespaceAutoProvision Admission Controller

Edit kube-apiserver manifest:

```bash id="n7e0yu"
vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

Locate:

```yaml id="y4k4dc"
--enable-admission-plugins=
```

Add plugin:

```yaml id="y5pgo1"
--enable-admission-plugins=NodeRestriction,NamespaceAutoProvision
```

Save file.

API server automatically restarts.

Wait a few moments.

---

# Test Again

Run same command:

```bash id="0jlwm4"
kubectl run nginx --image=nginx -n blue
```

Now succeeds.

Check namespaces:

```bash id="7qjybm"
kubectl get ns
```

Output includes:

```text id="v3snm2"
blue
```

Reason:

* Namespace created automatically by admission controller

---

# Deprecated Admission Controllers

Deprecated:

* NamespaceExists
* NamespaceAutoProvision

Replaced by:

```text id="9z7h0l"
NamespaceLifecycle
```

---

# NamespaceLifecycle Responsibilities

* Reject invalid namespaces
* Protect system namespaces

Protected namespaces:

* default
* kube-system
* kube-public

---

# Question 6

## Disable DefaultStorageClass Admission Controller

Edit kube-apiserver manifest:

```bash id="0laz7w"
vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

Add:

```yaml id="m6kp0j"
--disable-admission-plugins=DefaultStorageClass
```

Save file.

API server restarts automatically.

---

# Verify Enabled/Disabled Plugins

Example command:

```bash id="6j3vka"
ps -ef | grep kube-apiserver
```

This shows:

* Enabled admission plugins
* Disabled admission plugins

---

# Important kube-apiserver File

```text id="skwjlwm"
/etc/kubernetes/manifests/kube-apiserver.yaml
```

Used in:

* kubeadm clusters
* Static pod configuration

---

# Common Admission Controllers

| Admission Controller | Purpose                      |
| -------------------- | ---------------------------- |
| NamespaceLifecycle   | Namespace validation         |
| DefaultStorageClass  | Add default storage class    |
| NodeRestriction      | Restrict kubelet permissions |
| AlwaysPullImages     | Always pull container image  |
| ResourceQuota        | Enforce resource quotas      |
| LimitRanger          | Apply resource limits        |

---

# Important Commands

## List kube-system Pods

```bash id="f3s9b1"
kubectl get pods -n kube-system
```

---

## Check Default Admission Plugins

```bash id="p7j4yw"
kube-apiserver -h | grep enable-admission-plugins
```

---

## Edit kube-apiserver Manifest

```bash id="jl6y8q"
vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

---

## Check Namespaces

```bash id="h1e8mr"
kubectl get ns
```

---

# Summary

Admission Controllers:

* Operate after authentication and authorization
* Can validate requests
* Can mutate requests
* Improve security and governance

Common operations:

* Enable plugins
* Disable plugins
* Modify kube-apiserver manifest
* Restart API server automatically

Key file:

```text id="4a0zrx"
/etc/kubernetes/manifests/kube-apiserver.yaml
```

