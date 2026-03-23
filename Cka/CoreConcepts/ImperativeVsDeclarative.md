# Imperative vs Declarative in Kubernetes

## 1. Core Idea

### Imperative Approach (HOW to do)

* You give **step-by-step instructions**
* Example (real life): Giving directions turn-by-turn to a driver
* You control:

  * What to do
  * How to do it

### Declarative Approach (WHAT you want)

* You define the **final desired state**
* System decides **how to achieve it**
* Example: Enter destination in Uber → system handles routing

---

## 2. In Infrastructure (DevOps View)

### Imperative Example

You write steps like:

1. Create VM
2. Install Nginx
3. Configure port
4. Download code
5. Start service

Problem:

* Needs manual logic (checks, retries)
* Hard to maintain
* Not idempotent by default

---

### Declarative Example

You just define:

* VM name
* Nginx installed
* Port = 8080
* App path

System handles:

* Creation
* Updates
* Fixing drift

Tools:

* Ansible
* Terraform
* Puppet
* Chef

---

## 3. In Kubernetes

### Imperative Commands

Used for quick/manual actions:

```
kubectl run
kubectl create
kubectl expose
kubectl edit
kubectl scale
kubectl set image
kubectl delete
kubectl replace
```

Characteristics:

* Fast
* No YAML needed
* Not tracked properly
* Hard to reproduce

---

### Declarative Approach

Use YAML files + one command:

```
kubectl apply -f file.yaml
```

You define:

* Pods
* Deployments
* Services

Kubernetes:

* Creates if not exists
* Updates if changed
* Maintains desired state

---

## 4. Key Difference

| Feature         | Imperative  | Declarative      |
| --------------- | ----------- | ---------------- |
| Focus           | How         | What             |
| Commands        | Multiple    | Single (`apply`) |
| State tracking  | Manual      | Automatic        |
| Reproducibility | Low         | High             |
| Best for        | Quick tasks | Production       |

---

## 5. Important Concepts (Very Exam Relevant)

### Problem with Imperative

* If resource exists → error
* If partially done → inconsistent state
* You must handle logic manually

---

### Declarative Advantage

* Idempotent (safe to run multiple times)
* Handles:

  * Create
  * Update
  * Delete
* Works well with Git (GitOps)

---

## 6. kubectl apply (Important)

What it does:

* Compares:

  * Current state (cluster)
  * Desired state (YAML)
* Applies only required changes

---

## 7. kubectl edit vs YAML file

### kubectl edit

* Edits **live object**
* Not saved in your file
* Changes can be lost later

### Best Practice

* Edit YAML locally
* Then:

```
kubectl apply -f file.yaml
```

---

## 8. Exam Tips (Very Important)

### Use Imperative when:

* Simple tasks
* Example:

  * Create pod
  * Create deployment

Fast command:

```
kubectl run nginx --image=nginx
```

---

### Use Declarative when:

* Complex configs
* Multi-container pods
* Environment variables
* Volumes

---

### Smart Strategy

* Small task → Imperative
* Complex task → YAML + apply

---

## 9. One-Line Summary

* Imperative = “Do this step-by-step”
* Declarative = “Make it look like this”

---

