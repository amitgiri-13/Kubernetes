# Container Images

A **container image** represents executable binary data that encapsulates an application and **all its software dependencies**. Container images are standalone software bundles that make well-defined assumptions about their runtime environment.

In Kubernetes, you usually **build a container image**, **push it to a registry**, and then **reference it in a Pod**.

> **Note:** If you are looking for container images used by Kubernetes itself (for example, for a specific Kubernetes release like `v1.35`), refer to *Download Kubernetes*.

---

## Image Names

Container images are typically identified by names such as:

- `pause`
- `example/mycontainer`
- `kube-apiserver`

Images may also include:
- A **registry hostname**
- An optional **port number**

### Examples of Registry Formats

- `fictional.registry.example/imagename`
- `fictional.registry.example:10443/imagename`

If **no registry hostname is specified**, Kubernetes assumes the **Docker public registry (`docker.io`)**.  
This behavior can be changed by configuring a **default image registry** in the container runtime.

---

## Image Name Structure

[registry-host[:port]/]image-name[:tag][@digest]


After the image name, you can specify either a **tag** or a **digest** (or both).

---

## Image Tags

- Tags identify different versions of the same image
- If no tag is specified, Kubernetes assumes `:latest`

### Tag Characteristics

- Tags are **mutable**
- A tag can be updated to point to a different image version

### Tag Rules

- Allowed characters:
  - Uppercase and lowercase letters
  - Digits
  - `_` (underscore), `.` (period), `-` (dash)
- Maximum length: **128 characters**
- Validation regex:
[a-zA-Z0-9_][a-zA-Z0-9._-]{0,127}


---

## Image Digests

- Digests uniquely identify a **specific image version**
- Based on a cryptographic hash of the image content
- **Immutable** by design

### Digest Format

sha256:[hash]

### Example 

sha256:1ff6c18fbef2045af6b9c16bf034cc421a29027b800e4f9b68ae9b1cb3e9ae07


Digests ensure that Kubernetes always runs **exactly the same image**, regardless of tag changes.

---

## Image Name Examples

- `busybox`  
  Uses Docker public registry and `latest` tag  
  → `docker.io/library/busybox:latest`

- `busybox:1.32.0`  
  Image with a specific tag  
  → `docker.io/library/busybox:1.32.0`

- `registry.k8s.io/pause:latest`  
  Custom registry with `latest` tag

- `registry.k8s.io/pause:3.5`  
  Custom registry with a non-latest tag

- `registry.k8s.io/pause@sha256:1ff6c18f...`  
  Image referenced by digest

- `registry.k8s.io/pause:3.5@sha256:1ff6c18f...`  
  Image with both tag and digest  
  *(Only the digest is used for pulling the image)*

---

## Key Takeaways

- Container images package applications and dependencies into a single executable unit
- Image names can include registry, tag, and digest
- Tags are mutable; digests are immutable
- Using digests ensures consistent and reproducible deployments

---

# Updating Images in Kubernetes

## Overview

When you first create a **Deployment, StatefulSet, Pod**, or any object that includes a **PodTemplate**, if `imagePullPolicy` is **not explicitly specified**, Kubernetes automatically sets it **at creation time**.

By default:

* The pull policy is decided based on the image **tag or digest**
* The value is **not re-evaluated later**, even if the image reference changes

---

## Image Pull Policy

The `imagePullPolicy` and the **image tag/digest** together determine **when kubelet pulls the container image**.

### Supported Values

#### `IfNotPresent`

* Image is pulled **only if it does not exist locally**
* Cached image is reused if present
* Default for versioned tags

#### `Always`

* Kubelet queries the registry **every time a container starts**
* Image name is resolved to a **digest**
* Cached image is used if digest matches
* Otherwise, image is pulled

#### `Never`

* Kubelet **never pulls** the image
* Container starts only if image exists locally
* Otherwise, container startup fails
* Used mainly with **pre-pulled images**

---

## Image Caching Behavior

Even with `imagePullPolicy: Always`:

* Container runtimes reuse existing image layers
* Layers are not re-downloaded if already present
* Only metadata resolution occurs

This makes `Always` efficient as long as the registry is reachable.

---

## Image Tag Best Practices

⚠️ **Avoid using `:latest` in production**

Problems with `:latest`:

* Hard to know which version is running
* Rollbacks are unreliable
* Can cause mixed versions across Pods

### Recommended Alternatives

* Use meaningful version tags:

  ```text
  my-app:v1.42.0
  ```
* Or use image digests:

  ```text
  my-app@sha256:45b23dee08af5e43a7fea6c4cf9c25ccf269ee113168c19722f87876677c5cb2
  ```

---

## Using Image Digests

### Why Digests?

* Digests uniquely identify an image
* Guarantees the **same code runs every time**
* Prevents registry tag mutation issues

### Benefits

* No version drift
* No mixed image versions
* Predictable rollouts and rollbacks

---

## Admission Controllers and Digests

Some third-party admission controllers:

* Mutate Pods or PodTemplates at creation time
* Replace image tags with **image digests**
* Ensure the entire workload runs the same immutable image

Useful when:

* Registry tags might change
* Strong consistency is required across Pods

---

## Default Image Pull Policy Rules

If `imagePullPolicy` is **omitted**, Kubernetes sets it automatically:

| Image Reference               | Default imagePullPolicy |
| ----------------------------- | ----------------------- |
| Image digest specified        | `IfNotPresent`          |
| Tag is `:latest`              | `Always`                |
| No tag specified              | `Always`                |
| Versioned tag (not `:latest`) | `IfNotPresent`          |

---

## Important Note ⚠️

> `imagePullPolicy` is set **only once**, when the object is created.

Example:

* Deployment created with `my-app:v1.0.0`
* Policy set to `IfNotPresent`
* Image later updated to `my-app:latest`
* ❌ Policy does **not** change automatically
* ✅ Must be updated manually

---

## Required Image Pull (Force Pulling Images)

To **always force image pulls**, choose one:

1. Explicitly set:

   ```yaml
   imagePullPolicy: Always
   ```

2. Omit `imagePullPolicy` and use `:latest`

   ```yaml
   image: my-app:latest
   ```

3. Omit both `imagePullPolicy` and image tag:

   ```yaml
   image: my-app
   ```

4. Enable the **AlwaysPullImages** admission controller

   * Forces image pull for all Pods
   * Useful in security-sensitive environments

---

## ImagePullBackOff

### What is ImagePullBackOff?

`ImagePullBackOff` indicates that:

* Kubernetes failed to pull a container image
* The container is stuck in the **Waiting** state
* Kubelet retries pulling the image with increasing delay

### Common Causes

* Invalid image name or tag
* Image does not exist in registry
* Missing or incorrect `imagePullSecrets`
* Private registry access issues
* Network or DNS problems

### Back-off Behavior

* Retry delay increases after each failure
* Maximum delay is **300 seconds (5 minutes)**
* Kubelet continues retrying indefinitely

---

## Image Pull Per RuntimeClass

**FEATURE STATE:**

* Kubernetes v1.29
* **Alpha (disabled by default)**

### Description

Kubernetes supports pulling images based on the **RuntimeClass** of a Pod.

When the `RuntimeClassInImageCriApi` feature gate is enabled:

* Images are referenced using:

  ```
  (image name, runtime handler)
  ```
* Instead of just image name or digest

### Why This Matters

* Container runtimes can change behavior based on runtime handler
* Useful for **VM-based containers**

  * Example: Windows Hyper-V containers

### Use Case

* Same image name may need different handling
* Runtime-specific image preparation or isolation

---

## Quick Interview Summary 🚀

* `IfNotPresent` → pull only if missing
* `Always` → check registry every start
* `Never` → local images only
* Avoid `:latest` in production
* Prefer **image digests**
* `imagePullPolicy` is immutable after creation
* `ImagePullBackOff` = pull failed, retrying with delay
* RuntimeClass-based image pulls are **alpha in v1.29**

---


# Serial and Parallel Image Pulls in Kubernetes

## Default Behavior (Serial Image Pulls)

By default, the **kubelet pulls images serially**:

* Only **one image pull request** is sent to the container runtime at a time
* Other image pull requests wait until the current pull completes
* This behavior applies **per node**

> Even with serial image pulls enabled, **different nodes operate independently**, so multiple nodes can still pull the same image in parallel.

---

## Enabling Parallel Image Pulls

You can enable parallel image pulls by updating the kubelet configuration:

```yaml
serializeImagePulls: false
```

### Behavior When Enabled

* Image pull requests are sent immediately
* Multiple images can be pulled **at the same time**
* Improves Pod startup time in image-heavy workloads

### Important Considerations

* Ensure your container runtime can handle **parallel image pulls**
* Excessive parallel pulls may increase:

  * Network bandwidth usage
  * Disk I/O pressure

---

## Per-Pod Image Pull Behavior

The kubelet **never pulls multiple images in parallel for the same Pod**.

Example:

* A Pod with:

  * 1 init container
  * 1 application container
* Image pulls happen **sequentially**, even if parallel pulls are enabled

Parallel image pulling applies only:

* Across **different Pods**
* Using **different images**

---

## Maximum Parallel Image Pulls

**FEATURE STATE:**

* Kubernetes v1.35
* **Stable**

### Default Behavior

* When `serializeImagePulls: false`
* There is **no default limit** on parallel image pulls

---

## Limiting Parallel Image Pulls

To limit how many images can be pulled concurrently:

```yaml
serializeImagePulls: false
maxParallelImagePulls: 4
```

### Behavior

* At most **n images** are pulled at the same time
* Additional image pull requests wait
* Prevents excessive:

  * Network usage
  * Disk I/O contention

---

## Configuration Rules

* `maxParallelImagePulls` must be:

  * A positive integer ≥ 1
* If `maxParallelImagePulls >= 2`:

  * `serializeImagePulls` **must** be set to `false`
* Invalid configuration:

  * Kubelet will **fail to start**

---

## Summary Table

| Setting                      | Effect                       |
| ---------------------------- | ---------------------------- |
| `serializeImagePulls: true`  | Serial image pulls (default) |
| `serializeImagePulls: false` | Enable parallel image pulls  |
| `maxParallelImagePulls: n`   | Limit concurrent image pulls |
| Same Pod images              | Always pulled serially       |
| Different Pods               | Can be pulled in parallel    |

---

# Multi-Architecture Images and Image Indexes

## What Is a Multi-Architecture Image?

A container registry can serve an **image index (manifest list)** that references:

* Multiple architecture-specific image manifests
* Each image built for a different CPU architecture

Kubernetes and the container runtime:

* Automatically select the correct image
* Based on the node’s architecture (amd64, arm64, etc.)

---

## Why Image Indexes Matter

* Same image name works across architectures
* No need to maintain separate manifests
* Supports heterogeneous clusters

Example image names:

* `pause`
* `example/mycontainer`
* `kube-apiserver`

---

## Kubernetes Image Naming Convention

Kubernetes typically publishes images in two forms:

### Multi-Architecture Image

```text
pause
```

* Contains manifests for all supported architectures
* Preferred for modern setups

### Architecture-Specific Image (Backward Compatibility)

```text
pause-amd64
```

* Used by older configurations
* Used when image names are hardcoded in YAML files

---

## Backward Compatibility

* Older Kubernetes versions and manifests may reference:

  ```text
  image: pause-amd64
  ```
* Newer clusters can safely use:

  ```text
  image: pause
  ```
* Container runtime selects the correct binary automatically

---

## Quick Interview Summary 🚀

* Image pulls are **serial by default**
* Parallel pulls are enabled with `serializeImagePulls: false`
* Same Pod images are **never pulled in parallel**
* `maxParallelImagePulls` limits concurrent pulls (v1.35 stable)
* Multi-arch images use **image indexes**
* One image name → correct architecture automatically

---

# Using Private Registries in Kubernetes

Private registries often require **authentication** to pull images. Kubernetes provides multiple ways to supply credentials.

---

## 1. Methods for Providing Credentials

### a. `imagePullSecrets` on a Pod (Recommended)

* Specify secrets in the Pod specification.
* Secrets must exist in the **same Namespace** as the Pod.
* Secret types supported:

  * `kubernetes.io/dockercfg`
  * `kubernetes.io/dockerconfigjson`
* Only Pods providing the secrets can access the private registry.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mycontainer
    image: my-private-registry.com/my-image:tag
  imagePullSecrets:
  - name: my-registry-secret
```

---

### b. Configuring Nodes to Authenticate

* Credentials configured at the **node level** allow all Pods to use private images.
* Requires **cluster administrator privileges**.
* Implementation depends on container runtime and registry.

---

### c. Kubelet Credential Provider Plugin

* Kubelet can dynamically fetch credentials via an **exec plugin**.
* Useful for:

  * **Static Pods** (cannot reference Secrets or ServiceAccounts)
  * Dynamic credential rotation
* Requires **kubelet-level configuration**.

---

### d. Pre-Pulled Images

* Images cached locally on nodes can be used without registry access.
* Requires **root access** to all nodes to preload images.
* Works well with static Pods.
* Must ensure **all nodes have the same pre-pulled images**.

---

### e. Vendor-Specific / Local Extensions

* Custom node configuration or cloud provider mechanisms.
* Can implement node authentication to private registries.

---

## 2. `config.json` Interpretation

* Kubernetes supports **glob patterns** and prefix matching.
* Examples:

```json
{
  "auths": {
    "my-registry.example/images": { "auth": "…" },
    "*.my-registry.example/images": { "auth": "…" }
  }
}
```

### Matching Rules

| Pattern                | Matches                     |
| ---------------------- | --------------------------- |
| `*.kubernetes.io`      | `abc.kubernetes.io`         |
| `*.*.kubernetes.io`    | `abc.def.kubernetes.io`     |
| `prefix.*.io`          | `prefix.kubernetes.io`      |
| `*-good.kubernetes.io` | `prefix-good.kubernetes.io` |

* Kubelet tries credentials **sequentially** for each pattern.
* Multiple entries are allowed for subpaths:

```json
{
  "auths": {
    "my-registry.example/images": { "auth": "…" },
    "my-registry.example/images/subpath": { "auth": "…" }
  }
}
```

---

## 3. Pre-Pulled Images & Policies

* If `imagePullPolicy` is:

  * `IfNotPresent`: local image used preferentially
  * `Never`: only local image is used
* Pre-pulled images are suitable for:

  * Speed optimizations
  * Static Pods

**Feature State:** Kubernetes v1.35 [beta]

---

## 4. `KubeletEnsureSecretPulledImages` Feature

* Ensures credentials are **validated** even for pre-pulled images.
* Controlled by:

  * `imagePullCredentialsVerificationPolicy` in kubelet config.

### Policy Options

| Policy                         | Description                                       |
| ------------------------------ | ------------------------------------------------- |
| `NeverVerify`                  | Do not verify credentials if image exists locally |
| `NeverVerifyPreloadedImages`   | Preloaded images not verified; others are         |
| `NeverVerifyAllowListedImages` | Preloaded images in allowlist not verified        |
| `AlwaysVerify`                 | Verify credentials for all images before use      |

### Notes

* Credential rotation:

  * Previously used credentials continue to verify
  * New/rotated credentials require **re-pull**
* On first enablement:

  * All current images considered **pre-pulled**
  * Recommended: clean nodes before enabling to avoid stale records
* Removing image pull records directory resets all images to **pre-pulled** on kubelet restart

---

## 5. Key Recommendations

1. Use **`imagePullSecrets`** at Pod level whenever possible.
2. Use **pre-pulled images** only when node configuration is controllable.
3. Enable **KubeletEnsureSecretPulledImages** for secure validation.
4. Use **kubelet credential provider** for dynamic/static Pods requiring private registry access.
5. Ensure **config.json** patterns are correctly matched for multi-path registries.

---

# Private Registry Access in Kubernetes

## 1. Enabling `KubeletEnsureSecretPulledImages` for the First Time

* When enabled (via **kubelet upgrade** or **explicit feature toggle**):

  * Any images the kubelet **can already access** are considered **pre-pulled**
  * This occurs because the kubelet has **no previous pull records**
  * Only **new image pulls** will start creating pull records

**Recommendations:**

* Clean up nodes of images **not meant to be pre-pulled** before enabling
* Removing the **image pull records directory** has the same effect on kubelet restart:

  * All cached images are treated as pre-pulled

---

## 2. Creating a Secret with Docker Credentials

### Using `kubectl create secret docker-registry`

```bash
kubectl create secret docker-registry <name> \
  --docker-server=<docker-registry-server> \
  --docker-username=<docker-user> \
  --docker-password=<docker-password> \
  --docker-email=<docker-email>
```

* Requires: **username, password, email, and registry hostname**
* Creates a Secret usable by Pods in **the same namespace**
* Only works with a **single registry**

### Using Existing Docker Credentials File

* You can import an existing `.docker/config.json` as a Secret
* Useful when managing **multiple private registries**

**Note:**

* Secrets must exist in the **same namespace** as the Pod referencing them
* Must create one Secret **per namespace**

---

## 3. Referencing `imagePullSecrets` in a Pod

Add `imagePullSecrets` to the Pod specification:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: foo
  namespace: awesomeapps
spec:
  containers:
    - name: foo
      image: janedoe/awesomeapp:v1
  imagePullSecrets:
    - name: myregistrykey
```

* Each item in `imagePullSecrets` references **one Secret**
* Can be automated using a **ServiceAccount**:

  * All Pods using that ServiceAccount inherit the secrets
  * Works alongside per-node `.docker/config.json`, credentials are merged

---

## 4. Use Cases and Suggested Solutions

| Cluster Type                                                 | Suggested Approach                                                                                                                                                                                                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Non-proprietary (public) images**                          | Use public registry; no configuration required. Cloud providers may cache/mirror images automatically.                                                                                                                                                 |
| **Proprietary images, visible to all cluster users**         | - Hosted private registry (manual node configuration) <br> - Internal private registry with open read access (no Kubernetes config) <br> - Hosted registry service with controlled access <br> - Use `imagePullSecrets` if node config is inconvenient |
| **Proprietary images with strict access control**            | - Enable `AlwaysPullImages` admission controller <br> - Store sensitive data in Secrets instead of image                                                                                                                                               |
| **Multi-tenant cluster with separate registries per tenant** | - Enable `AlwaysPullImages` <br> - Private registry with authorization <br> - Generate credentials per tenant and store in a Secret <br> - Propagate Secret to each tenant namespace <br> - Each Pod references Secret via `imagePullSecrets`          |

---

## 5. Multiple Registries

* Create **one Secret per registry**
* Reference the appropriate Secret in Pods that require access
* Works in conjunction with node-level `.docker/config.json`

---

## 6. Summary Notes

* `KubeletEnsureSecretPulledImages` ensures **credential verification for pre-pulled images**
* `imagePullSecrets` is the **recommended approach** for Pod-level private registry access
* Node-level configuration allows **cluster-wide private registry access**
* For static Pods, use **kubelet credential provider** or **pre-pulled images**
* Use **Secrets per namespace**; can automate via **ServiceAccounts**
* Multi-tenant or multi-registry setups require **separate Secrets per registry and namespace**

---

