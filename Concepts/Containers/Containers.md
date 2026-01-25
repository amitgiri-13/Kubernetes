# Containers

Containers are a **technology for packaging an application together with its runtime dependencies**. This includes the application code, libraries, and configuration required to run the software consistently across different environments.

> The term **container** is overloaded, so always ensure your audience shares the same definition.

---

## Why Containers?

* **Repeatability**

  * Each container runs the same way everywhere
  * Eliminates “it works on my machine” issues

* **Standardization**

  * Dependencies are bundled with the application
  * Predictable behavior across environments

* **Portability**

  * Applications are decoupled from host OS and infrastructure
  * Run seamlessly across:

    * Local machines
    * On-prem servers
    * Cloud platforms

* **Isolation**

  * Containers share the host kernel but remain isolated
  * Lightweight compared to virtual machines

---

## Containers in Kubernetes

* Kubernetes runs containers **inside Pods**
* A **Pod** can contain:

  * One container (most common)
  * Multiple containers (sidecar pattern)

### Key Characteristics

* Containers in a Pod are:

  * **Co-located** (run on the same node)
  * **Co-scheduled** (scheduled together)
  * Share:

    * Network namespace
    * Storage volumes

---

# Container Images

A **container image** is a **read-only, ready-to-run software package** that includes:

* Application code
* Runtime (for example: JVM, Python)
* Application libraries
* System libraries
* Default configuration values

---

## Image Principles

### Immutable

* Running containers should **not be modified**
* Any change requires:

  1. Build a new image
  2. Deploy new containers from that image

### Stateless

* Containers should not store persistent data
* Persistent data is handled using:

  * Volumes
  * External databases
  * Object storage

---

## Image Lifecycle

1. Build the image
2. Push to an image registry
3. Pull the image onto cluster nodes
4. Create containers from the image

---

# Container Runtimes

A **container runtime** is the component that **runs and manages containers** on a node.

---

## Responsibilities

* Pull container images
* Create and start containers
* Stop and remove containers
* Manage container isolation and resources

---

## Supported Container Runtimes

Kubernetes supports runtimes that implement the **Container Runtime Interface (CRI)**, including:

* `containerd`
* `CRI-O`
* Other CRI-compliant runtimes

---

## Runtime Selection

* By default:

  * Kubernetes uses the **cluster’s default runtime**
* Advanced use cases:

  * Use **RuntimeClass** to:

    * Select a specific container runtime
    * Apply different runtime configurations

### Use Cases for RuntimeClass

* Running workloads with different security requirements
* Using specialized runtimes (for example, sandboxed containers)
* Applying custom runtime settings per workload

---

## Key Takeaways

* Containers package applications with all dependencies
* Container images are **immutable and stateless**
* Kubernetes runs containers inside Pods
* Container runtimes are essential for container execution
* RuntimeClass enables runtime-level customization

---

