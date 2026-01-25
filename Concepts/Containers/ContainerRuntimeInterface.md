# Container Runtime Interface (CRI) in Kubernetes

**Feature State:** v1.23 [stable]

The **Container Runtime Interface (CRI)** is a plugin interface that allows the kubelet to use a variety of container runtimes **without recompiling cluster components**.

---

## 1. Overview

* Each Node must have a **working container runtime** to launch Pods and containers.
* CRI defines the **gRPC protocol** for communication between:

  * `kubelet` (client)
  * Container runtime (server)
* Enables Kubernetes to support multiple runtimes such as:

  * containerd
  * CRI-O
  * Docker (via dockershim, deprecated)

---

## 2. Kubelet and CRI

* The **kubelet acts as a gRPC client** connecting to the container runtime.
* Container runtime must expose:

  * **Runtime service endpoint**
  * **Image service endpoint**
* Configure kubelet with:

  ```bash
  --container-runtime-endpoint=<runtime-endpoint>
  ```

---

## 3. CRI v1 API Requirement

* From **Kubernetes v1.26+**:

  * The kubelet requires the container runtime to support **v1 CRI API**
  * If runtime does **not support v1**, kubelet **fails to register the node**

---

## 4. Upgrading Considerations

* During **Kubernetes upgrades**:

  * kubelet restarts
  * If container runtime does not support v1 CRI → kubelet cannot register
* If container runtime is upgraded and gRPC re-dial is required:

  * Runtime **must support v1 CRI API** for kubelet connection
  * May require **manual kubelet restart** after runtime configuration

---

## 5. Summary

* CRI allows **runtime-agnostic kubelet operations**
* Uses **gRPC protocol** for communication between kubelet and container runtime
* v1 CRI API support is **mandatory** from Kubernetes v1.26+
* Proper configuration and runtime upgrades are essential to ensure kubelet registration and Pod creation

---
