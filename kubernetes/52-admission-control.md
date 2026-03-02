# Admission Control in Kubernetes

## What is Admission Control?

**Admission Control** is a stage in the Kubernetes API request lifecycle that runs **after authentication and authorization** but **before an object is persisted in etcd**.

Admission controllers can:

* **Validate** requests (allow or deny)
* **Mutate** requests (modify objects before they are stored)

They act as policy enforcement points for the cluster.

---

## Where Admission Control Fits in the API Request Flow

When a request like `kubectl apply -f pod.yaml` is sent:

1. Request reaches **kube-apiserver**
2. **Authentication** – Who are you?
3. **Authorization** – Are you allowed to do this?
4. **Admission Control** – Should this request be allowed or modified?
5. Object is stored in **etcd**
6. Controllers and schedulers react

Admission Control decides **whether the request is safe, compliant, and well-formed**.

---

## Full Admission Control Flow (Plain Diagram)

```
kubectl / client
      |
      v
+-------------------+
| kube-apiserver    |
+-------------------+
      |
      v
+-------------------+
| Authentication    |
| (certs, tokens)  |
+-------------------+
      |
      v
+-------------------+
| Authorization     |
| (RBAC / ABAC)    |
+-------------------+
      |
      v
+-----------------------------+
| Admission Controllers       |
|                             |
| 1. Mutating Controllers     |
|    - Modify request object  |
|                             |
| 2. Validating Controllers   |
|    - Accept / Reject        |
+-----------------------------+
      |
      v
+-------------------+
| Persist to etcd   |
+-------------------+
      |
      v
Controllers / Scheduler
```

---

## Types of Admission Controllers

### 1. Mutating Admission Controllers

They **modify** objects before they are saved.

Examples:

* Inject default values
* Add labels or annotations
* Inject sidecars

Runs **first**.

---

### 2. Validating Admission Controllers

They **validate** objects and can reject them.

Examples:

* Enforce resource limits
* Block privileged containers
* Require labels

Runs **after mutation**.

---

## Built-in Admission Controllers (Common Ones)

| Controller          | Purpose                                |
| ------------------- | -------------------------------------- |
| NamespaceLifecycle  | Prevents deletion of system namespaces |
| LimitRanger         | Applies default CPU/memory limits      |
| ResourceQuota       | Enforces namespace quotas              |
| PodSecurity         | Enforces pod security standards        |
| NodeRestriction     | Restricts kubelet permissions          |
| ServiceAccount      | Auto-injects service account tokens    |
| DefaultStorageClass | Assigns default storage class          |

---

## Example 1: Mutating Admission Controller (LimitRanger)

### Pod YAML (No Limits Defined)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
spec:
  containers:
  - name: app
    image: nginx
```

### What Happens

* `LimitRanger` checks namespace limits
* Automatically injects CPU/memory defaults

### Pod Stored in etcd (After Mutation)

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "200m"
    memory: "256Mi"
```

---

## Example 2: Validating Admission Controller (ResourceQuota)

### Namespace Quota

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: pod-limit
spec:
  hard:
    pods: "2"
```

### Attempt to Create 3rd Pod

```bash
kubectl apply -f pod.yaml
```

### Result

```
Error from server (Forbidden):
pods "demo-pod" is forbidden: exceeded quota
```

The request is **rejected before etcd write**.

---

## Dynamic Admission Controllers (Webhooks)

Kubernetes also supports **custom admission controllers** using **webhooks**.

### Types

* **MutatingAdmissionWebhook**
* **ValidatingAdmissionWebhook**

### Flow with Webhook

```
kube-apiserver
      |
      v
Webhook Configuration
      |
      v
External HTTPS Service
      |
      v
Allow / Modify / Deny
```

---

## Example: Validating Admission Webhook Use Case

Policy:

* All Pods must have label `env`

### Pod Without Label

```yaml
metadata:
  name: test-pod
```

### Webhook Response

```
Denied: missing required label 'env'
```

---

## Why Admission Control Is Important

* Enforces **cluster-wide policies**
* Prevents misconfigurations
* Improves security posture
* Enables automation (sidecars, defaults)
* Acts as a final guardrail before etcd

---

## Key Takeaways

* Admission Control runs **after authn/authz, before etcd**
* Mutating controllers run **first**
* Validating controllers decide **allow or deny**
* Webhooks enable **custom policy enforcement**
* etcd only stores **approved and compliant objects**

---

If you want, next we can:

* Compare Admission Controller vs Controller Manager
* Deep dive into PodSecurity Admission
* Build a simple validating webhook step by step
