# Understanding `kubectl apply` Internals

## 1. Core Idea

`kubectl apply` is used to **manage objects declaratively**. Unlike imperative commands (`create`, `replace`), it **tracks changes over time** to safely update Kubernetes objects.

It compares **three sources** when making changes:

1. **Local configuration file** – the YAML/manifest on your system
2. **Live object configuration** – the object currently running in Kubernetes
3. **Last applied configuration** – the snapshot of the object from the previous `apply`

---

## 2. Workflow When Running `kubectl apply`

1. **Object does not exist:**

   * Creates the object in Kubernetes based on your local YAML.
   * Live configuration now includes your defined fields **plus additional status fields**.

2. **Object exists:**

   * Converts your local YAML to **JSON** and stores it as the **last applied configuration**.
   * Compares:

     * Local YAML
     * Live configuration
     * Last applied configuration
   * Only changes necessary fields in the live object are updated.

---

## 3. Handling Updates and Deletions

* **Field update example:**

  * Local YAML updates Nginx image from `1.18` → `1.19`
  * `kubectl apply` compares with live object → updates only the image field

* **Field deletion example:**

  * A label is removed from local YAML
  * `kubectl apply` sees that the label exists in **last applied configuration** but not in local YAML → removes it from live object

* **Field present only in live object (not in last applied/local):**

  * Left unchanged → prevents accidental deletion

---

## 4. Storage of Last Applied Configuration

* Stored as an **annotation** in the live object:

  ```
  kubectl.kubernetes.io/last-applied-configuration
  ```
* **Only** `apply` stores this annotation
* Commands like `create` or `replace` do **not** store last applied configuration
* Mixing imperative and declarative commands on the same object can lead to inconsistencies

---

## 5. Summary Table

| Concept                         | Notes                                                                     |
| ------------------------------- | ------------------------------------------------------------------------- |
| Local file                      | Your YAML/manifest file                                                   |
| Live configuration              | Current object in Kubernetes, includes status fields                      |
| Last applied configuration      | JSON snapshot stored as annotation, used to detect removed/changed fields |
| Apply command vs create/replace | Apply tracks history; create/replace does not                             |
| Safety                          | Only updates differences, prevents accidental deletion                    |

---

## 6. Key Takeaways

* Always use `kubectl apply` for **declarative management**
* Avoid mixing imperative (`create`, `replace`) with declarative (`apply`)
* `kubectl apply` ensures:

  * **Idempotency**
  * **Safe updates**
  * **Accurate deletions**

---