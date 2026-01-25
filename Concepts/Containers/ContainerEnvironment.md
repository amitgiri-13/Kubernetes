

# Container Environment in Kubernetes

Kubernetes provides several resources to containers running inside Pods.

---

## 1. Container Environment Overview

Containers in Kubernetes have access to:

1. **Filesystem**

   * Combination of:

     * The **container image**
     * One or more **volumes** mounted into the Pod

2. **Container Information**

   * Metadata about the container itself
   * Hostname, Pod name, namespace, and environment variables

3. **Cluster Information**

   * Metadata about other objects in the cluster
   * Services and network information

---

## 2. Container Information

### Hostname

* The **hostname** of the container is set to the **Pod name**
* Accessible via:

  * `hostname` command
  * `gethostname()` in libc

### Pod Metadata

* **Pod name** and **namespace** are available via the **Downward API** as environment variables:

  ```yaml
  env:
  - name: POD_NAME
    valueFrom:
      fieldRef:
        fieldPath: metadata.name
  - name: POD_NAMESPACE
    valueFrom:
      fieldRef:
        fieldPath: metadata.namespace
  ```

### User-defined Environment Variables

* Environment variables defined in the Pod spec are accessible in the container
* Variables defined in the container image are also available

---

## 3. Cluster Information

### Service Environment Variables

* A container gets environment variables for all services **existing in the same namespace** at Pod creation
* Example for a service named `foo`:

  ```
  FOO_SERVICE_HOST=<service host>
  FOO_SERVICE_PORT=<service port>
  ```

### Service Access via DNS

* Services have **dedicated IP addresses**
* Containers can reach services via **DNS names** if the DNS addon is enabled
* Example:

  ```
  http://foo.<namespace>.svc.cluster.local:<port>
  ```

---

## 4. Summary

* Containers have access to:

  * **Filesystem** (image + volumes)
  * **Container info** (hostname, Pod name, namespace, env vars)
  * **Cluster info** (service host/port, DNS)
* Use **Downward API** to expose Pod metadata
* Services can be accessed via **environment variables** or **DNS**

---