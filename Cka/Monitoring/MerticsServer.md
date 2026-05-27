# Kubernetes Monitoring — Metrics Server Notes

## Overview

Monitoring in Kubernetes helps track:

* Cluster health
* Resource consumption
* Application performance

Kubernetes does not include a fully featured built-in monitoring solution by default, but several monitoring tools are available.

---

# What Should Be Monitored?

## Node-Level Metrics

Monitor cluster infrastructure metrics such as:

* Number of nodes
* Healthy/unhealthy nodes
* CPU utilization
* Memory utilization
* Network usage
* Disk utilization

---

## Pod-Level Metrics

Monitor application workload metrics such as:

* Number of pods
* CPU usage per pod
* Memory consumption per pod

---

# Monitoring Requirements

A monitoring solution should:

1. Collect metrics
2. Store metrics
3. Provide analytics and visualization

---

# Monitoring Solutions in Kubernetes

## Open Source Solutions

### Metrics Server

* Lightweight monitoring solution
* Collects resource metrics from nodes and pods
* Used for basic monitoring

### Prometheus

* Advanced monitoring system
* Time-series database
* Alerting support

### Elastic Stack (ELK)

Used for:

* Monitoring
* Logging
* Analytics

Components:

* Elasticsearch
* Logstash
* Kibana

---

## Proprietary Solutions

Examples:

* Datadog
* Dynatrace

---

# Heapster (Deprecated)

## What was Heapster?

Heapster was an older Kubernetes monitoring project.

Features:

* Monitoring
* Metrics aggregation
* Analytics

## Current Status

* Deprecated
* Replaced by Metrics Server

You may still find Heapster mentioned in older Kubernetes documentation and architectures.

---

# Metrics Server

## Key Features

* One Metrics Server per cluster
* Collects metrics from nodes and pods
* Aggregates metrics
* Stores metrics in memory only

---

## Important Limitation

Metrics Server:

* Does **NOT** store historical data
* Does **NOT** save metrics to disk

Because of this:

* Historical analysis is unavailable
* Long-term monitoring requires tools like Prometheus

---

# How Metrics Collection Works

## Kubelet

Each Kubernetes node runs an agent called:

```text id="bll09n"
Kubelet
```

Responsibilities:

* Receives instructions from API Server
* Manages pods on the node

---

## cAdvisor (Container Advisor)

Kubelet contains a sub-component called:

```text id="tr54wp"
cAdvisor
```

Responsibilities:

* Collects container and pod performance metrics
* Exposes metrics through Kubelet API

Metrics Server retrieves metrics from this API.

---

# Installing Metrics Server

## In Minikube

Enable Metrics Server using:

```bash id="t4w6ye"
minikube addons enable metrics-server
```

---

## In Other Kubernetes Environments

### Steps

1. Clone Metrics Server deployment files
2. Deploy using kubectl

Example:

```bash id="k1d73k"
kubectl apply -f metrics-server-components.yaml
```

Deployment creates:

* Pods
* Services
* Roles
* RBAC resources

---

# Viewing Metrics

After deployment, wait a few moments for metrics collection.

---

## View Node Metrics

```bash id="v50jjy"
kubectl top node
```

Displays:

* CPU usage
* Memory usage

Example output:

```text id="rjz1pf"
NAME       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
master     166m         8%     1200Mi          60%
```

---

## View Pod Metrics

```bash id="bfq1zn"
kubectl top pod
```

Displays:

* Pod CPU usage
* Pod memory usage

---

# Architecture Flow

```text id="qf0hcl"
Pods/Containers
       ↓
   cAdvisor
       ↓
    Kubelet
       ↓
 Metrics Server
       ↓
 kubectl top
```

---

# Key Takeaways

* Kubernetes monitoring tracks node and pod performance.
* Metrics Server is the default lightweight monitoring solution.
* Metrics Server stores metrics only in memory.
* Historical monitoring requires advanced tools like Prometheus.
* Kubelet and cAdvisor work together to expose metrics.
* `kubectl top node` shows node metrics.
* `kubectl top pod` shows pod metrics.
