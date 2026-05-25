# Kubernetes Admission Controllers — Complete Summary

## What are Admission Controllers?

Admission Controllers are plugins in Kubernetes that intercept requests to the API server **after authentication and authorization**, but **before the object is persisted in etcd**.

They are used to:

* Validate requests
* Reject invalid configurations
* Mutate/modify objects
* Enforce security policies
* Apply defaults automatically

---

# Request Flow in Kubernetes

```text
kubectl request
       ↓
Authentication
       ↓
Authorization (RBAC)
       ↓
Admission Controllers
       ↓
Object stored in etcd
```

---

# What Admission Controllers Can Do

## Validation

Reject invalid requests.

Example:

* Reject pod creation in non-existent namespace
* Reject containers running as root
* Reject images using `:latest` tag

---

## Mutation

Modify requests before creation.

Example:

* Automatically add default storage class
* Add labels/annotations
* Set default security context

---

# Types of Admission Controllers

| Type                            | Purpose                              |
| ------------------------------- | ------------------------------------ |
| Validating Admission Controller | Validates and allows/rejects request |
| Mutating Admission Controller   | Modifies request before creation     |

---

# Validating Admission Controller Example

## NamespaceLifecycle / NamespaceExists

Checks whether namespace exists.

Example:

```bash
kubectl run nginx --image=nginx -n blue
```

If namespace `blue` doesn't exist:

```text
Error from server (NotFound): namespaces "blue" not found
```

This is validation.

---

# Mutating Admission Controller Example

## DefaultStorageClass

If PVC has no storageClass:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
spec:
  accessModes:
    - ReadWriteOnce
```

Admission controller automatically adds:

```yaml
storageClassName: default
```

This is mutation.

---

# Order of Execution

## Important

Mutating admission controllers run first.

Then validating controllers run.

```text
Mutating → Validating
```

Reason:

* Validation should happen after all modifications are complete.

---

# Built-in Admission Controllers

Some common built-in admission controllers:

| Admission Controller       | Purpose                       |
| -------------------------- | ----------------------------- |
| NamespaceLifecycle         | Validates namespaces          |
| DefaultStorageClass        | Adds default storage class    |
| AlwaysPullImages           | Always pulls image            |
| EventRateLimit             | Limits API request rate       |
| ResourceQuota              | Enforces quotas               |
| LimitRanger                | Applies resource limits       |
| NodeRestriction            | Restricts kubelet permissions |
| MutatingAdmissionWebhook   | External mutation webhooks    |
| ValidatingAdmissionWebhook | External validation webhooks  |

---

# Check Enabled Admission Controllers

## kube-apiserver Help

```bash
kube-apiserver -h | grep enable-admission-plugins
```

---

## In kubeadm Clusters

API server runs as Pod.

First exec into pod:

```bash
kubectl exec -it kube-apiserver-controlplane -n kube-system -- sh
```

Then:

```bash
kube-apiserver -h | grep enable-admission-plugins
```

---

# kube-apiserver Manifest Location

```bash
/etc/kubernetes/manifests/kube-apiserver.yaml
```

---

# Enable Admission Controller

Example:

```yaml
--enable-admission-plugins=NodeRestriction,NamespaceAutoProvision
```

---

# Disable Admission Controller

Example:

```yaml
--disable-admission-plugins=DefaultStorageClass
```

---

# NamespaceAutoProvision Example

Normally:

```bash
kubectl run nginx --image=nginx -n blue
```

Fails if namespace doesn't exist.

After enabling NamespaceAutoProvision:

* Namespace automatically created
* Pod successfully created

---

# Deprecated Controllers

These are deprecated:

* NamespaceExists
* NamespaceAutoProvision

Replaced by:

## NamespaceLifecycle

Functions:

* Rejects requests to non-existent namespaces
* Prevents deletion of important namespaces like:

  * default
  * kube-system
  * kube-public

---

# External Admission Controllers

Built-in controllers are compiled into Kubernetes.

For custom logic, Kubernetes supports:

| External Admission Controller |
| ----------------------------- |
| MutatingAdmissionWebhook      |
| ValidatingAdmissionWebhook    |

---

# Admission Webhook Architecture

```text
API Server
    ↓
Admission Webhook
    ↓
Webhook Server
```

Webhook server can run:

* Inside cluster
* Outside cluster

---

# AdmissionReview Object

Kubernetes sends request details as JSON.

Contains:

* User info
* Operation type
* Resource
* Object specification

Webhook responds with:

```json
{
  "allowed": true
}
```

or

```json
{
  "allowed": false
}
```

---

# Mutating Webhook Example

Webhook adds label automatically:

```yaml
metadata:
  labels:
    user: amit
```

This is done using JSON Patch operations.

---

# JSON Patch Operations

Common operations:

| Operation | Purpose        |
| --------- | -------------- |
| add       | Add field      |
| remove    | Remove field   |
| replace   | Replace field  |
| move      | Move value     |
| copy      | Copy value     |
| test      | Validate field |

---

# Webhook Server Deployment

Usually deployed as:

```text
Deployment + Service
```

Inside Kubernetes cluster.

---

# ValidatingWebhookConfiguration

Example structure:

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
```

---

# Important Sections

## Webhook Name

```yaml
name: pod-policy.example.com
```

---

## Client Config

### External Server

```yaml
url: https://webhook-server.example.com
```

### Internal Service

```yaml
service:
  namespace: webhook-demo
  name: webhook-service
```

---

# TLS Requirement

Communication between API server and webhook must use TLS.

Required:

* Server certificate
* Key
* CA bundle

Example:

```yaml
caBundle: <base64-cert>
```

---

# Rules Section

Defines when webhook should trigger.

Example:

```yaml
rules:
- operations: ["CREATE"]
  apiGroups: [""]
  apiVersions: ["v1"]
  resources: ["pods"]
```

Meaning:

* Trigger only during pod creation

---

# Lab Commands Summary

## Create Namespace

```bash
kubectl create ns webhook-demo
```

---

# Create TLS Secret

```bash
kubectl -n webhook-demo create secret tls webhook-server-tls \
  --cert=/root/keys/webhook-server-tls.crt \
  --key=/root/keys/webhook-server-tls.key
```

---

# Deploy Webhook

```bash
kubectl apply -f webhook-deployment.yaml
```

---

# Deploy Service

```bash
kubectl apply -f webhook-service.yaml
```

---

# Deploy Webhook Configuration

```bash
kubectl apply -f webhook-config.yaml
```

---

# Security Context Webhook Lab

Webhook behavior:

## If no securityContext

Automatically sets:

```yaml
runAsNonRoot: true
runAsUser: 1234
```

This is mutation.

---

## If explicitly allowed

```yaml
runAsNonRoot: false
```

Pod allowed to run as root.

---

## Invalid Configuration

```yaml
runAsNonRoot: true
runAsUser: 0
```

Rejected because:

* User 0 = root
* Contradicts runAsNonRoot=true

This is validation.

---

# Core Exam Concepts

## Must Remember

### Admission Controllers operate:

* AFTER authentication
* AFTER authorization
* BEFORE object persistence

---

### Mutating controllers:

* Modify requests

---

### Validating controllers:

* Allow or deny requests

---

### Execution order:

```text
Mutating → Validating
```

---

### Important built-ins:

* NamespaceLifecycle
* DefaultStorageClass
* ResourceQuota
* LimitRanger
* NodeRestriction

---

### Webhook Types

* MutatingAdmissionWebhook
* ValidatingAdmissionWebhook

---

# Real-World Uses

Admission controllers are heavily used for:

* Security enforcement
* Pod security policies
* Image validation
* Automatic sidecar injection
* Label enforcement
* Policy management

Popular tools using admission webhooks:

* Istio
* Kyverno
* OPA Gatekeeper
* Linkerd
