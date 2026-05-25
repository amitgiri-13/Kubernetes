# Kubernetes Admission Controllers — Detailed Notes

## What are Admission Controllers?

Admission Controllers are Kubernetes components that intercept requests to the API server **after authentication and authorization**, but **before the object is persisted in etcd**.

They are used to:

* Validate requests
* Modify requests
* Enforce security policies
* Apply defaults automatically
* Restrict cluster usage patterns

---

# Request Flow in Kubernetes

When a user runs a command like:

```bash
kubectl apply -f pod.yaml
```

the request follows this flow:

```text
kubectl
   ↓
API Server
   ↓
Authentication
   ↓
Authorization
   ↓
Admission Controllers
   ↓
etcd
```

---

# Responsibilities

## Authentication

Verifies:

* Who the user is
* Certificates/tokens validity

Example:

* Client certificates
* kubeconfig authentication

---

## Authorization

Checks:

* What the user is allowed to do

Usually implemented using:

* RBAC (Role-Based Access Control)

Example:

* Can user create pods?
* Can user delete deployments?

---

## Admission Controllers

Control:

* How the request should behave
* Whether the request should be modified
* Whether it should be rejected

---

# What Admission Controllers Can Do

Admission Controllers can:

## Validate Requests

Example:

* Reject pods in non-existent namespaces
* Reject privileged containers
* Reject containers using `latest` tag

---

## Mutate Requests

Example:

* Automatically add default storage class
* Add labels/annotations
* Inject sidecars

---

## Perform Extra Operations

Example:

* Automatically create namespaces
* Apply defaults
* Enforce organization policies

---

# Examples of Policies

Admission Controllers can enforce rules such as:

* Only allow images from internal registry
* Prevent containers running as root
* Require labels on all resources
* Prevent use of `latest` image tag
* Restrict dangerous Linux capabilities

---

# Built-in Admission Controllers

Kubernetes ships with many admission controllers.

Some important ones:

| Admission Controller       | Purpose                      |
| -------------------------- | ---------------------------- |
| NamespaceLifecycle         | Validates namespaces         |
| DefaultStorageClass        | Adds default storage class   |
| AlwaysPullImages           | Always pulls images          |
| EventRateLimit             | Limits API event flooding    |
| ResourceQuota              | Enforces resource limits     |
| LimitRanger                | Applies resource defaults    |
| MutatingAdmissionWebhook   | External mutation webhooks   |
| ValidatingAdmissionWebhook | External validation webhooks |

---

# NamespaceLifecycle Admission Controller

Replaces deprecated:

* NamespaceExists
* NamespaceAutoProvision

Functions:

* Reject requests to non-existent namespaces
* Prevent deletion of critical namespaces:

  * `default`
  * `kube-system`
  * `kube-public`

---

# Types of Admission Controllers

There are two major types.

---

# 1. Validating Admission Controllers

These:

* Validate requests
* Allow or reject requests
* Do NOT modify objects

Example:

* NamespaceLifecycle

### Example

User tries:

```bash
kubectl run nginx --image=nginx -n blue
```

If namespace `blue` does not exist:

```text
Error from server (NotFound): namespaces "blue" not found
```

The request is rejected.

---

# 2. Mutating Admission Controllers

These:

* Modify requests before creation

Example:

* DefaultStorageClass

---

# Example: DefaultStorageClass

Suppose a PVC is created without storage class:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
```

The admission controller automatically adds:

```yaml
storageClassName: standard
```

before saving the object.

---

# Order of Execution

Mutating admission controllers run first.

Then validating controllers run.

```text
Mutating Controllers
        ↓
Validating Controllers
```

Reason:

* Validation should happen after all mutations are complete.

---

# Example Flow

## Namespace Auto Provision

Mutating controller:

* Creates missing namespace

Then validating controller:

* Checks namespace existence

If validating ran first:

* Request would always fail.

---

# Enable Admission Controllers

## View Enabled Controllers

If kube-apiserver runs as pod:

```bash
kubectl exec -it kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep enable-admission-plugins
```

---

# kubeadm Clusters

Edit:

```bash
/etc/kubernetes/manifests/kube-apiserver.yaml
```

---

# Enable Plugin

Example:

```yaml
- --enable-admission-plugins=NodeRestriction
```

Add multiple:

```yaml
- --enable-admission-plugins=NodeRestriction,NamespaceAutoProvision
```

---

# Disable Plugin

Example:

```yaml
- --disable-admission-plugins=DefaultStorageClass
```

---

# After Editing

The kube-apiserver pod automatically restarts because it is a static pod.

---

# Lab Example — NamespaceAutoProvision

Initially:

```bash
kubectl run nginx --image=nginx -n blue
```

Fails because namespace does not exist.

---

Enable plugin:

```yaml
- --enable-admission-plugins=NodeRestriction,NamespaceAutoProvision
```

Now:

```bash
kubectl run nginx --image=nginx -n blue
```

works successfully.

Kubernetes automatically creates namespace `blue`.

---

# Webhook Admission Controllers

Built-in admission controllers are compiled into Kubernetes.

But organizations often need custom policies.

Kubernetes supports this using:

| Type                       | Purpose           |
| -------------------------- | ----------------- |
| MutatingAdmissionWebhook   | Modify requests   |
| ValidatingAdmissionWebhook | Validate requests |

---

# Webhook Architecture

```text
kubectl
   ↓
API Server
   ↓
Built-in Admission Controllers
   ↓
Webhook Admission Controller
   ↓
Custom Webhook Server
```

---

# Custom Webhook Server

You create your own server containing custom logic.

Can be written in:

* Go
* Python
* Java
* Any language

Requirements:

* Accept AdmissionReview requests
* Return AdmissionReview responses

---

# AdmissionReview Object

Kubernetes sends request details as JSON.

Contains:

* User info
* Operation type
* Resource type
* Object data

Example:

```json
{
  "request": {
    "uid": "...",
    "operation": "CREATE",
    "object": {...}
  }
}
```

---

# Webhook Response

Server returns:

```json
{
  "response": {
    "allowed": true
  }
}
```

If:

```json
"allowed": false
```

the request is rejected.

---

# Mutation Example

Webhook can automatically add labels.

Example patch:

```json
[
  {
    "op": "add",
    "path": "/metadata/labels/user",
    "value": "amit"
  }
]
```

Supported patch operations:

* add
* remove
* replace
* move
* copy
* test

---

# Deploying Webhook Server

Usually:

1. Containerize webhook app
2. Deploy as Kubernetes Deployment
3. Expose via Service

Example:

```text
Webhook Server Pod
        ↓
Kubernetes Service
```

---

# Webhook Configuration

Configured using:

## ValidatingWebhookConfiguration

or

## MutatingWebhookConfiguration

---

# Example Configuration

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: pod-policy
```

---

# Client Configuration

Specifies webhook server location.

## External URL

```yaml
clientConfig:
  url: https://webhook.example.com/validate
```

---

## Kubernetes Service

```yaml
clientConfig:
  service:
    namespace: webhook-ns
    name: webhook-service
```

---

# TLS Requirement

Communication must use HTTPS/TLS.

Need:

* Server certificate
* CA bundle

Example:

```yaml
caBundle: <base64-cert>
```

---

# Rules Section

Defines when webhook is triggered.

Example:

```yaml
rules:
- apiGroups: [""]
  apiVersions: ["v1"]
  operations: ["CREATE"]
  resources: ["pods"]
```

Meaning:

* Trigger webhook only when creating pods.

---

# Complete Request Flow with Webhooks

```text
kubectl apply
      ↓
Authentication
      ↓
Authorization
      ↓
Built-in Mutating Controllers
      ↓
Built-in Validating Controllers
      ↓
Mutating Webhook
      ↓
Validating Webhook
      ↓
Persist to etcd
```

---

# Important Exam Points

## Admission Controllers

* Run after authentication and authorization
* Before object persistence

---

## Mutating vs Validating

| Mutating         | Validating     |
| ---------------- | -------------- |
| Modifies request | Only validates |
| Runs first       | Runs second    |

---

## Common Commands

### View enabled plugins

```bash
kube-apiserver -h | grep enable-admission-plugins
```

---

### kubeadm config location

```bash
/etc/kubernetes/manifests/kube-apiserver.yaml
```

---

### Enable plugin

```yaml
--enable-admission-plugins=
```

---

### Disable plugin

```yaml
--disable-admission-plugins=
```

---

# Deprecated Controllers

Deprecated:

* NamespaceExists
* NamespaceAutoProvision

Replacement:

* NamespaceLifecycle

---

# Key Takeaways

* Admission Controllers enforce cluster policies.
* They operate between authorization and persistence.
* Mutating controllers modify requests.
* Validating controllers approve/reject requests.
* Webhooks enable fully custom admission logic.
* Webhooks require TLS and AdmissionReview JSON handling.
* kube-apiserver configuration controls enabled plugins.
