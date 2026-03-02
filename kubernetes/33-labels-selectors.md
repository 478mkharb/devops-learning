## Labels and Selectors in Kubernetes

### What are Labels?

**Labels** are key–value pairs attached to Kubernetes objects (Pods, Services, Deployments, etc.) to **identify and group resources**.

They are used for:

* Organizing resources
* Selecting subsets of objects
* Applying operations (via Services, Deployments, NetworkPolicies, etc.)

**Example labels:**

```text
app=nginx
env=prod
tier=frontend
```

---

### What are Selectors?

**Selectors** are used to **filter and select Kubernetes objects based on labels**.

They answer the question:

> “Which objects match these labels?”

Selectors are heavily used by:

* Services
* Deployments / ReplicaSets
* NetworkPolicies
* kubectl commands

---

### Example 1: Pod with Labels

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
    env: prod
spec:
  containers:
  - name: nginx
    image: nginx
```

---

### Example 2: Service Using Label Selector

This Service will send traffic **only to Pods with `app=nginx`**.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

---

### How Labels & Selectors Work Together

```text
Pod Labels        →  app=nginx, env=prod
Service Selector  →  app=nginx
Result            →  Pod is selected
```

---

### Example 3: Deployment Selector (MatchLabels)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```

> ⚠️ Deployment selector **must match Pod template labels**.

---

### Types of Selectors

#### 1. Equality-based

```text
app=nginx
env!=dev
```

#### 2. Set-based

```text
env in (prod, staging)
tier notin (backend)
```

---

### kubectl Examples

```bash
kubectl get pods -l app=nginx
kubectl get pods -l env=prod
```

---

### In One Line

* **Labels** → attach metadata to objects
* **Selectors** → choose objects using labels

They are the **core mechanism for grouping and service discovery in Kubernetes**.
