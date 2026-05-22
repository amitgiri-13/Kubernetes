# DaemonSets in Kubernetes

## What is a DaemonSet?

A **DaemonSet** in Kubernetes ensures that **one copy of a pod runs on every node** in the cluster.

It automatically:

* Deploys a pod on every node
* Adds the pod when a new node joins the cluster
* Removes the pod when a node is removed

DaemonSets are similar to ReplicaSets, but instead of maintaining a fixed number of replicas, they maintain **one pod per node**.

---

# Why DaemonSets are Needed

In many cases, certain services must run on **all worker nodes**.

Examples:

* Monitoring agents
* Log collectors
* Networking agents
* Kubernetes system components

DaemonSets automate this deployment process.

---

# How DaemonSet Works

## Behavior

* Existing nodes → pod created automatically
* New node added → new pod scheduled automatically
* Node removed → pod deleted automatically

This guarantees:

> Every node always has exactly one pod instance.

---

# Common Use Cases

## 1. Monitoring Agents

Used to run monitoring software on every node.

Examples:

* Node Exporter
* Datadog Agent
* Prometheus Agent

Purpose:

* Collect CPU, memory, disk, and node metrics

---

## 2. Log Collection

Log collectors run on every node to gather container logs.

Examples:

* Fluentd
* Fluent Bit
* Filebeat

Purpose:

* Send logs to centralized logging systems

---

## 3. kube-proxy

`kube-proxy` is commonly deployed as a DaemonSet.

Purpose:

* Handles Kubernetes networking rules on each node

---

## 4. Networking Plugins

CNI plugins require agents on all nodes.

Examples:

* Calico
* Weave
* Flannel

Purpose:

* Configure pod networking across cluster nodes

---

# DaemonSet YAML Structure

A DaemonSet definition is very similar to a ReplicaSet.

## Basic Structure

```yaml
apiVersion: apps/v1
kind: DaemonSet

metadata:
  name: monitoring-daemon

spec:
  selector:
    matchLabels:
      app: monitoring-agent

  template:
    metadata:
      labels:
        app: monitoring-agent

    spec:
      containers:
      - name: monitoring-agent
        image: monitoring-agent:latest
```

---

# Important Components

## 1. apiVersion

```yaml
apiVersion: apps/v1
```

Defines the API version.

---

## 2. kind

```yaml
kind: DaemonSet
```

Specifies Kubernetes resource type.

---

## 3. selector

```yaml
selector:
  matchLabels:
    app: monitoring-agent
```

Used to identify pods managed by the DaemonSet.

---

## 4. template

Contains pod specification:

* Labels
* Containers
* Images
* Ports
* Volumes

---

# Label Matching

The labels in:

```yaml
selector.matchLabels
```

must match:

```yaml
template.metadata.labels
```

Otherwise, the DaemonSet will not manage the pods correctly.

---

# Creating a DaemonSet

## Apply YAML

```bash
kubectl apply -f daemonset.yaml
```

---

# Viewing DaemonSets

## List DaemonSets

```bash
kubectl get daemonset
```

or

```bash
kubectl get ds
```

---

## Describe DaemonSet

```bash
kubectl describe daemonset monitoring-daemon
```

Shows:

* Events
* Scheduled pods
* Node information
* Labels and selectors

---

# DaemonSet Scheduling

## Older Kubernetes Versions (< 1.12)

Earlier, DaemonSets bypassed the scheduler using:

```yaml
nodeName:
```

Pods were directly assigned to nodes.

---

## Modern Kubernetes Versions (>= 1.12)

Modern DaemonSets use:

* Default Kubernetes scheduler
* Node affinity rules

This provides:

* Better scheduling control
* Improved integration with Kubernetes scheduling system

---

# DaemonSet vs ReplicaSet

| Feature       | DaemonSet                         | ReplicaSet               |
| ------------- | --------------------------------- | ------------------------ |
| Purpose       | One pod per node                  | Fixed number of replicas |
| Scaling       | Automatically based on node count | Manual replica count     |
| Node Coverage | All nodes                         | Selected nodes           |
| Common Usage  | Monitoring, logging, networking   | Application scaling      |

---

# Key Characteristics

* Runs one pod per node
* Automatically handles node additions/removals
* Commonly used for infrastructure services
* Similar structure to ReplicaSet
* Uses Kubernetes scheduler in modern versions

---

# Important Commands

```bash
kubectl apply -f daemonset.yaml
kubectl get daemonset
kubectl describe daemonset <daemonset-name>
kubectl delete daemonset <daemonset-name>
```

---

# Summary

DaemonSets are used when a pod must run on every node in a Kubernetes cluster.

Typical examples include:

* Monitoring agents
* Log collectors
* kube-proxy
* Networking plugins

They automatically maintain one pod instance per node and dynamically adapt when cluster nodes change.
