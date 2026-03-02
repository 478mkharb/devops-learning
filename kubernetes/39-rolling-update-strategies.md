# Kubernetes Deployment Strategies

In Kubernetes, **deployment strategies** define *how* Pods are replaced when a new version of an application is deployed. The goal is to update applications with **minimal or zero downtime**, while maintaining stability.

Kubernetes mainly supports **two deployment strategies** out of the box:

---

## 1️⃣ Recreate Strategy

### 🔹 What it is

* All existing Pods are **terminated first**
* New Pods are created **only after** old Pods are gone

### 🔹 How it works

1. Kubernetes deletes all old Pods
2. Application goes **temporarily down**
3. New Pods are created with the updated version

### 🔹 YAML Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-recreate
spec:
  replicas: 3
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: nginx:latest
```

### 🔹 Pros & Cons

✅ Simple and predictable
❌ Causes downtime

### 🔹 When to use

* Apps that **cannot run multiple versions simultaneously**
* Development or testing environments

---

## 2️⃣ RollingUpdate Strategy (Default)

### 🔹 What it is

Pods are updated **gradually**, ensuring the application remains available during the update.

### 🔹 How it works

* New Pods are created **before** old Pods are deleted
* Traffic is shifted slowly to new Pods

### 🔹 Key Parameters

| Parameter        | Meaning                                            |
| ---------------- | -------------------------------------------------- |
| `maxUnavailable` | Maximum Pods that can be unavailable during update |
| `maxSurge`       | Extra Pods allowed above desired replicas          |

---

## 🔄 What is Rolling Update?

A **Rolling Update** updates Pods in small batches instead of all at once.

Example with `replicas: 4`

* Kubernetes creates new Pods
* Deletes old Pods one by one
* Ensures service availability

---

## 🔹 RollingUpdate YAML Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-rolling
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: nginx:1.25
```

### 🔹 What happens here

* At most **1 Pod down** at a time
* At most **1 extra Pod** created during update

---

## ❓ What if maxSurge or maxUnavailable are NOT specified?

### 🔹 Default Behavior

If you **do not specify** `maxSurge` or `maxUnavailable`:

```yaml
strategy:
  type: RollingUpdate
```

Kubernetes applies **default values**:

| Field          | Default Value |
| -------------- | ------------- |
| maxSurge       | `25%`         |
| maxUnavailable | `25%`         |

---

## 🔍 Default Rolling Update Example

Assume:

```yaml
replicas: 4
```

Defaults applied:

* `maxSurge: 25%` → **1 extra Pod** allowed
* `maxUnavailable: 25%` → **1 Pod can be down**

### Update Flow

1. Create 1 new Pod (total = 5)
2. Delete 1 old Pod (back to 4)
3. Repeat until all Pods are updated

✅ **No downtime**

---

## 🧠 Percentage vs Absolute Values

You can define values as:

```yaml
maxSurge: 2
maxUnavailable: 1
```

OR

```yaml
maxSurge: 50%
maxUnavailable: 0
```

### Rule

* Percentages are **rounded up**
* Kubernetes always ensures at least **1 Pod is available**

---

## 📌 Summary Table

| Strategy      | Downtime | Control | Use Case             |
| ------------- | -------- | ------- | -------------------- |
| Recreate      | ❌ Yes    | Low     | Non-critical apps    |
| RollingUpdate | ✅ No     | High    | Production workloads |

---

## ✅ Key Takeaways

* **RollingUpdate is the default strategy**
* `maxSurge` controls extra Pods
* `maxUnavailable` controls downtime
* Defaults are **25% / 25%** if not specified

---

📌 *Rolling updates are the backbone of zero-downtime deployments in Kubernetes.*
