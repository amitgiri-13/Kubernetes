# Understanding Kubernetes Pods

## Prerequisites
- The application is already developed and built into Docker images.
- Docker images are available in a repository (e.g., Docker Hub) for Kubernetes to pull.
- A Kubernetes cluster is already set up and running (single-node or multi-node).

---

## Kubernetes Deployment Basics
- Kubernetes deploys containers **inside Pods**, not directly on worker nodes.
- **Pod**:
  - Smallest deployable object in Kubernetes.
  - Encapsulates one or more containers.
  - Typically, a one-to-one relationship with a container running the application.

---

## Scaling Applications
1. **Single Node Example**:
   - One instance of an application runs inside a single pod.
2. **Scaling Up**:
   - Add additional pods to share load.
   - Each pod has its own container instance.
3. **Scaling Across Nodes**:
   - Add new nodes to the cluster if the current node is at capacity.
   - Deploy pods to new nodes to expand cluster capacity.

**Important**:
- Do **not** add more containers to an existing pod to scale.
- To scale down, delete existing pods.

---

## Multi-container Pods
- Rare use case: Pods can have multiple containers.
- Example: Helper container for tasks like processing uploaded files.
- Benefits:
  - Helper container lifecycle tied to main container.
  - Direct communication via `localhost`.
  - Shared network and storage space.
- For simplicity in most cases, **one container per pod** is used.

---

## Why Pods Exist (vs. plain Docker)
- Without Kubernetes:
  - You’d manually manage multiple Docker containers and helper containers.
  - Tasks include networking, storage sharing, and monitoring container lifecycle.
- With Kubernetes:
  - Pods automate container creation, networking, storage, and lifecycle management.
  - Containers in a pod share network namespace, storage, and are created/destroyed together.

---

## Deploying Pods
- `kubectl run`:
  - Automatically creates a pod.
  - Deploys the specified container image (e.g., Nginx).
  - Image can come from Docker Hub (public) or a private repository.
- `kubectl get pods`:
  - Lists all pods in the cluster.
  - Pod states: `ContainerCreating` → `Running`.

**Note**:
- Initially, pods may not be accessible externally.
- Accessing services externally requires networking configuration and services (covered later).

---

## Summary
- Pods are the fundamental building block for deploying containers in Kubernetes.
- Each pod typically contains a single container.
- Kubernetes handles lifecycle, networking, and storage management automatically.
- Multi-container pods exist but are uncommon.