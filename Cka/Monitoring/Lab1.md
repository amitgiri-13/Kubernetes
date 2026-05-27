# Kubernetes Metrics Server Lab

# Objective

Learn how to:

* Deploy Metrics Server
* Monitor cluster resource usage
* View node metrics
* View pod metrics
* Analyze CPU and memory consumption

---

# Existing Workloads

The cluster already contains three running pods:

```bash id="6qqz9l"
kubectl get pods
```

Example pods:

* elephant
* lion
* rabbit

All pods are in `Running` state.

---

# Goal of the Lab

Monitor resource consumption of:

* Nodes
* Pods

Metrics include:

* CPU usage
* Memory usage

---

# Deploying Metrics Server

## Important Note

The lab uses a preconfigured Metrics Server setup.

This setup is:

* Intended only for lab environments
* NOT recommended for production use

For production:

* Always follow official Metrics Server documentation.

---

# Clone Metrics Server Files

The repository contains deployment files required for:

* Pods
* Services
* Roles
* Permissions

---

# Deploy Metrics Server

Deploy all manifests:

```bash id="b61y4q"
kubectl create -f .
```

This creates all required Kubernetes objects for Metrics Server.

---

# Wait for Metrics Collection

Metrics Server may take a few minutes to:

* Start
* Collect metrics
* Process data

---

# Viewing Node Metrics

Use:

```bash id="5o4g1s"
kubectl top node
```

Displays:

* CPU usage
* Memory usage

Example:

```text id="b83v1f"
NAME            CPU(cores)   MEMORY(bytes)
controlplane    470m         1200Mi
node01          57m          350Mi
```

---

# Lab Findings

## Node Consuming Most CPU

Result:

```text id="b00m0l"
controlplane
```

Reason:

* Control plane runs Kubernetes core components:

  * API Server
  * Scheduler
  * Controller Manager
  * etcd

---

## Node Consuming Most Memory

Result:

```text id="d9g0v2"
controlplane
```

---

# Viewing Pod Metrics

Use:

```bash id="4iv1l2"
kubectl top pod
```

Displays:

* Pod CPU usage
* Pod memory usage

---

# Pod Consuming Most Memory

Example output:

```text id="x4e4f6"
NAME       CPU(cores)   MEMORY(bytes)
elephant   15m          32Mi
lion       1m           18Mi
rabbit     8m           64Mi
```

Result:

```text id="m6n4o2"
rabbit
```

Reason:

* Rabbit pod uses the highest memory.

---

# Pod Consuming Least CPU

Result:

```text id="v2r8j5"
lion
```

Reason:

* Lion pod uses only `1m` CPU.

---

# Key Commands Summary

## Get Pods

```bash id="m6o9xy"
kubectl get pods
```

---

## Deploy Metrics Server

```bash id="o1cx39"
kubectl create -f .
```

---

## View Node Metrics

```bash id="q6p2iy"
kubectl top node
```

---

## View Pod Metrics

```bash id="s1k7lp"
kubectl top pod
```

---

# Important Concepts

## Metrics Server

* Lightweight monitoring solution
* Collects CPU and memory metrics
* Stores metrics in memory only
* No historical data support

---

## `kubectl top`

Used to view live resource consumption.

### Nodes

```bash id="w1h3sd"
kubectl top node
```

### Pods

```bash id="tw9xqs"
kubectl top pod
```

---

# Key Takeaways

* Metrics Server enables Kubernetes resource monitoring.
* `kubectl top node` shows node metrics.
* `kubectl top pod` shows pod metrics.
* Control plane nodes usually consume more resources.
* Metrics Server is suitable for lightweight monitoring.
* Production environments should use official deployment methods and advanced monitoring solutions for long-term metrics.
