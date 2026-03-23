# kubectl explain – Complete Guide

## 1. What is `kubectl explain`?

`kubectl explain` helps you:

* Understand **Kubernetes resources**
* Explore **fields and structure**
* Build **YAML files directly from terminal**

You don’t need to open documentation every time.

---

## 2. Step 1: Discover Resources

Before using `explain`, find available resources:

```bash
kubectl api-resources
```

This shows:

* Resource names (pods, deployments, svc, etc.)
* Short names (po, deploy, svc)
* API versions (v1, apps/v1)

Useful when:

* You forget resource names
* Unsure about API version
* Writing YAML from scratch

---

## 3. Basic Usage

```bash
kubectl explain pod
```

Output includes:

* `apiVersion`
* `kind`
* `metadata`
* `spec`
* `status`

Also shows:

* Data types (string, object, etc.)
* Description of each field

---

## 4. Drill Down into Fields

To explore deeper:

```bash
kubectl explain pod.spec
```

This shows:

* All fields inside `spec`
* Example:

  * containers
  * volumes
  * restartPolicy

---

### Go Even Deeper

```bash
kubectl explain pod.spec.containers
```

You can keep chaining:

```bash
kubectl explain pod.spec.containers.image
```

---

## 5. Get Full YAML Structure (Very Important)

```bash
kubectl explain pod --recursive
```

This gives:

* Complete field hierarchy
* All nested fields
* YAML-like structure

Perfect for:

* Writing YAML files
* Learning object structure
* Exam preparation

---

## 6. Why This is Powerful

Instead of:

* Googling docs
* Switching tabs

You can:

* Explore everything **inside terminal**
* Work faster in exams
* Avoid syntax mistakes

---

## 7. Real DevOps Use Case

When you forget:

* Field name (e.g., `imagePullPolicy`)
* Structure of containers
* Placement of volumes

Just run:

```bash
kubectl explain deployment.spec.template.spec.containers
```

---

## 8. Exam Tips

### Use when:

* You forget YAML structure
* You’re stuck in exam
* You need exact field names

### Fast workflow:

1. `kubectl api-resources`
2. `kubectl explain <resource>`
3. `kubectl explain <resource>.<field> --recursive`

---

## 9. One-Line Summary

`kubectl explain` = **Kubernetes documentation inside your terminal**

---
