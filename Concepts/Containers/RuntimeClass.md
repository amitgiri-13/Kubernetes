
# RuntimeClass in Kubernetes

**Feature State:** v1.20 [stable]

RuntimeClass allows you to **select the container runtime configuration** for running a Pod's containers.

---

## 1. Motivation

* Different Pods may require different runtime configurations for:

  * **Security**: e.g., hardware virtualization for isolation
  * **Performance**: e.g., lightweight runtimes for high throughput
* RuntimeClass can also differentiate Pods using the **same runtime** but with **different settings**

---

## 2. Setup

### Step 1: Configure CRI on Nodes

* RuntimeClass configurations are **CRI-specific**
* Handlers must be valid **DNS label names**
* Assumes **homogeneous node configuration** by default

  * Heterogeneous nodes require node labels & scheduling (see below)

---

### Step 2: Create RuntimeClass Resources

* Each handler in the CRI configuration must have a corresponding RuntimeClass object:

```yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: myclass   # Reference name for the RuntimeClass
handler: myconfiguration  # Corresponding CRI handler
```

* **Notes:**

  * RuntimeClass is **non-namespaced**
  * Recommended: restrict write operations to **cluster administrators**
  * Name must be a **valid DNS subdomain**

---

## 3. Usage

* Specify the RuntimeClass in a Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  runtimeClassName: myclass
  containers:
    - name: mycontainer
      image: myimage:v1
```

* **Behavior:**

  * If the RuntimeClass does not exist or CRI cannot run the handler → Pod enters **Failed** state
  * If `runtimeClassName` is omitted → default runtime handler is used

---

## 4. CRI Configuration Examples

### containerd

* Configure handlers in `/etc/containerd/config.toml`:

```toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.${HANDLER_NAME}]
# Configuration for this runtime handler
```

### CRI-O

* Configure handlers in `/etc/crio/crio.conf`:

```toml
[crio.runtime.runtimes.${HANDLER_NAME}]
  runtime_path = "${PATH_TO_BINARY}"
```

* Refer to CRI-specific documentation for details

---

## 5. Scheduling RuntimeClass Pods

**Feature State:** v1.16 [beta]

* Use the `scheduling` field in RuntimeClass to constrain Pods to nodes that support it
* Node constraints:

  * `nodeSelector`: selects nodes with specific labels
  * Merged with Pod's own `nodeSelector` (intersection)
* Taints & tolerations:

  * `tolerations` in RuntimeClass are merged with Pod tolerations (union)
  * Ensures Pods can run on nodes with runtime-specific taints

> See [Assigning Pods to Nodes](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/) for more info

---

## 6. Pod Overhead

**Feature State:** v1.24 [stable]

* RuntimeClass can declare **overhead resources** associated with running Pods
* Scheduler accounts for this **Pod overhead** when allocating resources
* Defined using the `overhead` field in the RuntimeClass object

```yaml
overhead:
  podFixed:
    cpu: "50m"
    memory: "64Mi"
```

* Helps the cluster make **accurate scheduling and resource allocation decisions**

---

## 7. Summary

* **RuntimeClass**:

  * Selects a **container runtime configuration** per Pod
  * Supports security/performance isolation
  * Works with CRI-specific runtime handlers
* **Scheduling**:

  * NodeSelector + Tolerations can restrict Pods to compatible nodes
* **Pod Overhead**:

  * Define extra resources required for runtime
* Default behavior applies if `runtimeClassName` is omitted

---