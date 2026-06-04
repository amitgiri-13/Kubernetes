# Kubernetes Cluster Maintenance 

## Check Cluster Status

### View Nodes

```bash
kubectl get nodes
```

### View Deployments

```bash
kubectl get deploy
```

### View Pods and Their Assigned Nodes

```bash
kubectl get pods -o wide
```

---

# Drain a Node for Maintenance

To safely remove a node from service and move workloads to other nodes:

```bash
kubectl drain node01 --ignore-daemonsets
```

### What Drain Does

1. Marks the node as unschedulable.
2. Evicts running pods.
3. Reschedules managed workloads on other available nodes.

### Note

DaemonSet pods are not removed unless explicitly handled, so `--ignore-daemonsets` is commonly used.

---

# Verify Node Status

Check node status:

```bash
kubectl get nodes
```

A drained node appears as:

```text
Ready,SchedulingDisabled
```

---

# Bring Node Back Into Service

After maintenance is complete:

```bash
kubectl uncordon node01
```

This allows new pods to be scheduled on the node again.

### Important

Existing pods do not automatically move back to the node after uncordoning. Only newly created pods may be scheduled there.

---

# Why Pods May Stay on Other Nodes

When a node is drained:

* Pods are recreated on available nodes.
* After uncordon, Kubernetes does not rebalance existing pods automatically.
* Pods remain where they are unless recreated or rescheduled.

---

# Control Plane Scheduling

Normally, control plane nodes have taints that prevent application workloads from running on them.

Check taints:

```bash
kubectl describe node controlplane
```

If no taints exist, workloads can be scheduled on the control plane node.

---

# Drain Failure Scenario

Running:

```bash
kubectl drain node01 --ignore-daemonsets
```

may fail with an error similar to:

```text
cannot delete Pods not managed by a ReplicationController,
ReplicaSet, Job, DaemonSet, or StatefulSet
```

### Reason

A standalone pod exists on the node that is not managed by a controller.

Example:

```text
hr-app
```

Kubernetes cannot safely delete such pods because they will not be recreated automatically.

---

# Force Drain

```bash
kubectl drain node01 --ignore-daemonsets --force
```

### Warning

For standalone pods:

* The pod is deleted permanently.
* It is not recreated elsewhere.
* Any local data may be lost.

Use force only when you understand the impact.

---

# Converting Standalone Pods to Deployments

Instead of running critical applications as standalone pods:

* Create a Deployment.
* Deployment manages ReplicaSets.
* Pods are recreated automatically if removed.

Benefits:

* High availability
* Self-healing
* Safe node maintenance

---

# Cordon a Node

To prevent new pods from being scheduled while keeping existing pods running:

```bash
kubectl cordon node01
```

### What Cordon Does

* Marks node as unschedulable.
* Existing pods continue running.
* No pod eviction occurs.

### Use Case

When a critical application is running on the node and should not be disrupted.

---

# Node Maintenance Commands Summary

| Command                                    | Purpose                                                    |
| ------------------------------------------ | ---------------------------------------------------------- |
| `kubectl drain <node> --ignore-daemonsets` | Evict workloads and prepare node for maintenance           |
| `kubectl uncordon <node>`                  | Allow scheduling again                                     |
| `kubectl cordon <node>`                    | Prevent new scheduling while keeping existing pods running |
| `kubectl get nodes`                        | View node status                                           |
| `kubectl get pods -o wide`                 | View pod placement                                         |
| `kubectl describe node <node>`             | Inspect node details and taints                            |

---

# Key Takeaways

* **Drain** = Evict pods + mark node unschedulable.
* **Cordon** = Mark node unschedulable without evicting pods.
* **Uncordon** = Re-enable scheduling on a node.
* Standalone pods can block drain operations.
* Critical workloads should be managed by Deployments.
* Pods do not automatically move back after uncordoning a node.
* Control plane nodes usually have taints to prevent workload scheduling.
