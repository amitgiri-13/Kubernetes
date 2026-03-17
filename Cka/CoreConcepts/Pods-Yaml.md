# Kubernetes: Creating a Pod with YAML

## Introduction
- Kubernetes objects (Pods, Deployments, Services, etc.) are defined using **YAML files**.
- YAML provides configuration input for Kubernetes to create these objects.

## Kubernetes YAML Structure
A Kubernetes YAML file has **four top-level fields**:
1. **apiVersion** – Version of the Kubernetes API to use.
   - For pods: `v1`
   - Others: `apps/v1`, `extensions/v1beta1`, etc.
2. **kind** – Type of object being created.
   - Example: `Pod`, `Deployment`, `Service`
3. **metadata** – Information about the object.
   - Includes `name` (string) and `labels` (dictionary)
   - Labels help group and filter objects later.
   - Correct indentation is crucial:
     ```yaml
     metadata:
       name: my-app-pod
       labels:
         app: my-app
     ```
   - Only allowed properties under metadata (name, labels, etc.)
   - Labels can have arbitrary key-value pairs.
4. **spec** – Specification of the object.
   - Provides details specific to the object type.
   - For pods with containers:
     - `containers` is a **list**, allowing multiple containers in one pod (multi-container pods are rare)
     - Each list item is a dictionary with at least `name` and `image`:
       ```yaml
       spec:
         containers:
           - name: my-app-container
             image: nginx
       ```

## Creating the Pod
- Save the YAML file (e.g., `pod-definition.yaml`).
- Run:
```bash
  kubectl create -f pod-definition.yaml
```

## Viewing Pods

* List all pods:

  ```bash
  kubectl get pods
  ```
* Get detailed pod info:

  ```bash
  kubectl describe pod <pod-name>
  ```

  * Shows creation info, labels, container details, and events.

## Notes

* YAML indentation is critical; children must be indented relative to parents.
* For a single container pod, only one item is added under `containers`.
* Kubernetes automatically handles pod lifecycle: creation, network, storage, and helper container management.


Here’s a structured Markdown note from your demo transcript about creating a Kubernetes pod with a YAML file:

````md id="14i6kx"
# Kubernetes Demo: Creating a Pod Using YAML

## Objective
- Create a Kubernetes pod using a **YAML definition file** instead of `kubectl run`.

## Tools
- **Linux:** Use `vim`, `vi`, or any YAML-supporting editor.
- **Windows:** Use Notepad++ or any editor that supports YAML syntax.
- Proper syntax and indentation are crucial for YAML.

## YAML File Creation
1. **File Name:** `pod.yaml` (example)
2. **Root-level Fields:**
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: nginx
     labels:
       app: nginx
       tier: frontend
   spec:
     containers:
       - name: nginx
         image: nginx
````

3. **Notes on YAML Syntax:**

   * `metadata` and `spec` are dictionaries.
   * `labels` is a dictionary under `metadata`.
   * `containers` is a **list**; each item is a dictionary with `name` and `image`.
   * Indentation: **2 spaces recommended**, avoid tabs.
   * `kind` is case-sensitive (`Pod`).

## Adding Multiple Containers

* To add a second container in the pod:

  ```yaml
  spec:
    containers:
      - name: nginx
        image: nginx
      - name: busybox
        image: busybox
  ```
* Each item in `containers` list is independent.

## Creating the Pod

* Use either:

  ```bash
  kubectl create -f pod.yaml
  ```

  or

  ```bash
  kubectl apply -f pod.yaml
  ```
* Both commands work similarly for creating new objects.

## Verifying Pod

* **Check pod status:**

  ```bash
  kubectl get pods
  ```

  * Shows `ContainerCreating` → `Running` states.
* **Detailed information:**

  ```bash
  kubectl describe pod nginx
  ```

  * Displays creation info, labels, containers, and events.

## Tips

* Stick to consistent indentation.
* Use meaningful container names for clarity.
* YAML tools and editors can reduce syntax errors.


