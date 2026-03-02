# RBAC in Kubernetes (Role-Based Access Control)

RBAC (Role-Based Access Control) in Kubernetes is a security mechanism that controls **who can do what** within a cluster. It defines **permissions** for users, groups, and service accounts to interact with Kubernetes resources (pods, services, deployments, secrets, etc.).

At a high level, RBAC answers three questions:

1. **Who** is making the request? (User / Group / ServiceAccount)
2. **What** are they allowed to do? (verbs like get, list, create, delete)
3. **On which resources** and **in which scope**? (namespace or cluster-wide)

---

## Core RBAC Components

Kubernetes RBAC is built using four main objects:

* **Role**
* **ClusterRole**
* **RoleBinding**
* **ClusterRoleBinding**

---

## Role

A **Role** defines a set of permissions **within a specific namespace**.

### Key Points

* Namespace-scoped
* Cannot grant access outside its namespace
* Used for fine-grained access control

### Example: Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: dev
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

🔹 This Role allows **read-only access to pods** in the `dev` namespace.

---

## ClusterRole

A **ClusterRole** defines permissions **across the entire cluster** or for **cluster-scoped resources**.

### Key Points

* Cluster-wide scope
* Can access:

  * Cluster-scoped resources (nodes, namespaces, PVs)
  * Namespaced resources across *all* namespaces
* Can also be reused inside a namespace via RoleBinding

### Example: ClusterRole

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-reader-cluster
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

🔹 This ClusterRole allows **read-only access to pods in all namespaces**.

---

## RoleBinding

A **RoleBinding** assigns a **Role or ClusterRole** to a user, group, or service account **within a namespace**.

### Key Points

* Namespace-scoped
* Grants permissions only in the binding namespace
* Can bind either:

  * Role
  * ClusterRole (restricted to namespace)

### Example: RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: dev
subjects:
- kind: User
  name: alice
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

🔹 User `alice` can read pods **only in the `dev` namespace**.

---

## ClusterRoleBinding

A **ClusterRoleBinding** assigns a **ClusterRole** to a user, group, or service account **across the entire cluster**.

### Key Points

* Cluster-wide scope
* Cannot reference a Role
* Grants access to all namespaces

### Example: ClusterRoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-pod-reader-binding
subjects:
- kind: User
  name: bob
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: pod-reader-cluster
  apiGroup: rbac.authorization.k8s.io
```

🔹 User `bob` can read pods **in every namespace**.

---

## Role vs ClusterRole (Comparison)

| Feature                     | Role      | ClusterRole  |
| --------------------------- | --------- | ------------ |
| Scope                       | Namespace | Cluster-wide |
| Access to cluster resources | ❌ No      | ✅ Yes        |
| Reusable across namespaces  | ❌ No      | ✅ Yes        |
| Used for nodes / PVs        | ❌ No      | ✅ Yes        |

---

## RoleBinding vs ClusterRoleBinding (Comparison)

| Feature                     | RoleBinding | ClusterRoleBinding |
| --------------------------- | ----------- | ------------------ |
| Scope                       | Namespace   | Cluster-wide       |
| Binds Role                  | ✅ Yes       | ❌ No               |
| Binds ClusterRole           | ✅ Yes       | ✅ Yes              |
| Access limited to namespace | ✅ Yes       | ❌ No               |

---

## Real-World Scenarios

### 1️⃣ Developer Access (Namespace Only)

* Create a Role for pods, services
* Bind it using RoleBinding

### 2️⃣ Admin Access (Cluster-Wide)

* Use ClusterRole `cluster-admin`
* Bind using ClusterRoleBinding

### 3️⃣ Monitoring Tools (Read-Only Cluster Access)

* Create ClusterRole with `get`, `list`, `watch`
* Bind to ServiceAccount using ClusterRoleBinding

---

## Summary

* **Role** → Permissions inside a namespace
* **ClusterRole** → Permissions across the cluster
* **RoleBinding** → Assigns permissions within a namespace
* **ClusterRoleBinding** → Assigns permissions cluster-wide

RBAC is critical for securing Kubernetes clusters and enforcing the principle of **least privilege**.
