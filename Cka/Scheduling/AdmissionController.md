# Kubernetes Admission Controllers

## What are Admission Controllers?

Admission Controllers are components in Kubernetes that intercept requests to the API server after:

1. Authentication
2. Authorization

and before the object is persisted in etcd.

They help enforce security policies, governance, and cluster standards.

---

# Kubernetes Request Flow

```text id="h9w8np"
kubectl command
        ↓
Authentication
        ↓
Authorization
        ↓
Admission Controllers
        ↓
etcd Database
````

---

# Authentication

Verifies identity of the user.

Usually done using:

* Certificates
* kubeconfig credentials
* Tokens

Example:

```bash id="1wp0vw"
kubectl get pods
```

Kubernetes checks whether the user is valid.

---

# Authorization

Checks whether user has permission to perform action.

Usually implemented using:

* RBAC (Role Based Access Control)

Example Role:

```yaml id="l4rj8q"
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get","list","create","update","delete"]
```

This allows user to:

* Get pods
* List pods
* Create pods
* Update pods
* Delete pods

---

# Limitation of RBAC

RBAC controls:

* WHO can access
* WHAT operations are allowed

RBAC does NOT validate:

* Image registry
* Image tags
* Security context
* Labels
* Capabilities
* Root users

Example policies RBAC cannot enforce:

* Block Docker Hub images
* Disallow `latest` tag
* Prevent root containers
* Require labels
* Restrict Linux capabilities

---

# Admission Controllers Solve This

Admission Controllers can:

* Validate requests
* Reject requests
* Modify requests
* Add defaults
* Perform additional operations

---

# Examples of Admission Controllers

| Admission Controller | Purpose                      |
| -------------------- | ---------------------------- |
| AlwaysPullImages     | Always pull images           |
| DefaultStorageClass  | Add default storage class    |
| EventRateLimit       | Limit API requests           |
| NamespaceLifecycle   | Validate namespace lifecycle |
| ResourceQuota        | Enforce quotas               |
| LimitRanger          | Enforce resource limits      |

---

# Example: NamespaceExists

Suppose user creates pod in non-existing namespace:

```bash id="jlwmvq"
kubectl run nginx --image=nginx -n blue
```

If namespace `blue` does not exist:

```text id="r3jlwm"
Error from server (NotFound): namespaces "blue" not found
```

---

# Request Processing Flow

```text id="ljch98"
Request
  ↓
Authentication
  ↓
Authorization
  ↓
Admission Controller
  ↓
Accept or Reject
```

`NamespaceExists` admission controller checks:

* Does namespace exist?

If not:

* Request rejected

---

# NamespaceAutoProvision

Another admission controller:

```text id="mohm4u"
NamespaceAutoProvision
```

Purpose:

* Automatically create namespace if missing

This admission controller is NOT enabled by default.

---

# Built-in Admission Controllers

Kubernetes ships with many built-in admission controllers.

To see enabled admission controllers:

```bash id="p6w2k1"
kube-apiserver -h | grep enable-admission-plugins
```

---

# kubeadm Based Cluster

In kubeadm clusters, API server runs as static pod.

Execute inside API server pod:

```bash id="ayl5pj"
kubectl exec -it kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep enable-admission-plugins
```

---

# Enable Admission Controller

Modify kube-apiserver configuration.

Example:

```yaml id="7v0o3g"
--enable-admission-plugins=NodeRestriction,NamespaceAutoProvision
```

In kubeadm:

Edit manifest:

```bash id="hmbtvx"
/etc/kubernetes/manifests/kube-apiserver.yaml
```

---

# Disable Admission Controller

Use:

```yaml id="h0jz6n"
--disable-admission-plugins=NamespaceLifecycle
```

---

# Example Workflow with NamespaceAutoProvision

Command:

```bash id="o4pc2h"
kubectl run nginx --image=nginx -n blue
```

Flow:

1. Authenticate user
2. Authorize request
3. Admission controller checks namespace
4. Namespace created automatically
5. Pod created successfully

---

# Deprecated Admission Controllers

Deprecated:

* NamespaceExists
* NamespaceAutoProvision

Replaced by:

```text id="og2ek1"
NamespaceLifecycle
```

---

# NamespaceLifecycle Responsibilities

* Reject requests to non-existing namespaces
* Protect system namespaces

Protected namespaces:

* default
* kube-system
* kube-public

---

# Types of Admission Controllers

## Validating Admission Controllers

Validate requests.

Can:

* Allow
* Reject

Example:

* Block root containers

---

## Mutating Admission Controllers

Modify requests before object creation.

Example:

* Add default storage class
* Add labels
* Inject sidecars

---

# Important Use Cases

## Security

* Block privileged containers
* Prevent root users
* Restrict capabilities

---

## Governance

* Enforce labels
* Naming standards
* Resource limits

---

## Automation

* Add default values
* Inject configurations
* Auto create namespaces

---

# Key Commands

## Check Admission Plugins

```bash id="g8p4m9"
kube-apiserver -h | grep admission
```

---

## Edit API Server Manifest

```bash id="m3f7wx"
vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

---

# Summary

Admission Controllers operate:

```text id="5q4r2z"
After Authentication
After Authorization
Before Object Persistence
```

They help:

* Validate requests
* Modify requests
* Enforce policies
* Improve cluster security
* Automate cluster behavior

They are essential for Kubernetes governance and security.

