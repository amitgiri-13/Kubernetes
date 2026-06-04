# Kubernetes Software Release Notes

## Kubernetes Versioning

Kubernetes follows a versioning scheme with three parts:

```text
Major.Minor.Patch
```

Example:

```text
v1.33.0
│ │  │
│ │  └── Patch Version
│ └───── Minor Version
└─────── Major Version
```

### Components of a Version

* **Major Version**: Significant architectural changes (rarely changes).
* **Minor Version**: Introduces new features and enhancements.
* **Patch Version**: Contains bug fixes, security updates, and minor improvements.

Example:

```text
v1.33.0
```

* Major = 1
* Minor = 33
* Patch = 0

---

## Kubernetes Release Cycle

### Minor Releases

* Released every few months.
* Include new features and functionality.
* Example:

```text
v1.32 → v1.33
```

### Patch Releases

* Released more frequently.
* Include:

  * Bug fixes
  * Security fixes
  * Stability improvements

Example:

```text
v1.33.0 → v1.33.1 → v1.33.2
```

---

## Kubernetes Release Stages

New features move through three stages before becoming stable.

### 1. Alpha

Characteristics:

* Experimental features
* Disabled by default
* May contain bugs
* Not recommended for production

Example:

```text
v1.34.0-alpha.1
```

---

### 2. Beta

Characteristics:

* More stable than Alpha
* Well tested
* Usually enabled by default
* Suitable for testing environments

Example:

```text
v1.34.0-beta.1
```

---

### 3. Stable (GA - General Availability)

Characteristics:

* Fully tested
* Production-ready
* Officially supported

Example:

```text
v1.34.0
```

---

## Kubernetes Release History

| Version | Year |
| ------- | ---- |
| v1.0    | 2015 |
| v1.20   | 2020 |
| v1.33   | 2025 |

Kubernetes continuously releases new versions with improvements and new capabilities.

---

## Kubernetes Installation Packages

When downloading Kubernetes binaries, the package contains executables for:

### Control Plane Components

* kube-apiserver
* kube-controller-manager
* kube-scheduler
* kubectl
* kubelet
* kube-proxy

These components typically share the same Kubernetes version.

Example:

```text
Kubernetes v1.33.0
```

All core Kubernetes components are version 1.33.0.

---

## External Components Have Independent Versions

Some components bundled with Kubernetes are separate projects and maintain their own version numbers.

### etcd

* Distributed key-value database
* Stores cluster state

Example:

```text
Kubernetes v1.33
etcd v3.x
```

---

### CoreDNS

* Cluster DNS service
* Handles service discovery

Example:

```text
Kubernetes v1.33
CoreDNS v1.x
```

---

## Checking Kubernetes Version

### View Node Versions

```bash
kubectl get nodes
```

Output:

```text
NAME           STATUS   VERSION
controlplane   Ready    v1.33.0
worker01       Ready    v1.33.0
```

### View Client and Server Versions

```bash
kubectl version
```

### Detailed Version Information

```bash
kubectl version --output=yaml
```

---

## Release Notes

Each Kubernetes release provides:

* New features
* Deprecations
* Bug fixes
* Security fixes
* Supported etcd versions
* Supported CoreDNS versions
* Upgrade considerations

Always review release notes before upgrading a cluster.

---

## Key Points

* Kubernetes uses **Major.Minor.Patch** versioning.
* Minor releases introduce new features.
* Patch releases provide bug and security fixes.
* Features progress through **Alpha → Beta → Stable** stages.
* Core Kubernetes components usually share the same version.
* **etcd** and **CoreDNS** maintain their own version numbers.
* Review release notes before performing upgrades.
* Stable releases are recommended for production environments.
