## What is a Namespace in Kubernetes?

A **namespace** in Kubernetes is a **logical partition inside a cluster** that helps organize, isolate, and manage resources such as Pods, Services, Deployments, and ConfigMaps.

Think of a namespace as a **virtual cluster within a single physical cluster**.

---

## Why Do We Need Namespaces?

Namespaces solve several real-world problems:

### 1. Resource Isolation

* Different teams or applications can run in the same cluster
* Resources in one namespace do not interfere with others

### 2. Environment Separation

* Run **dev**, **test**, and **prod** workloads in the same cluster
* Prevent accidental changes across environments

### 3. Resource Management

* Apply **ResourceQuota** to limit CPU, memory, pods, etc.
* Prevent one team from consuming all cluster resources

### 4. Access Control (RBAC)

* Grant users permissions only for specific namespaces
* Improve security in multi-team clusters

### 5. Name Scoping

* Same resource names can exist in different namespaces
* Example: `nginx-service` in both `dev` and `prod`

---

## Important Characteristics

* Namespaces provide **logical isolation**, not physical isolation
* Nodes are **shared** across namespaces
* Some resources are **cluster-wide** and not namespaced

  * Nodes
  * PersistentVolumes
  * ClusterRoles

---

## Built-in Namespaces

| Namespace       | Purpose                                |
| --------------- | -------------------------------------- |
| default         | Default namespace if none is specified |
| kube-system     | Kubernetes control plane components    |
| kube-public     | Publicly readable objects              |
| kube-node-lease | Node heartbeat and health tracking     |

---

## Example: Using Namespaces

### Step 1: Create a Namespace

```bash
kubectl create namespace dev
```

### Step 2: Deploy an Application in `dev` Namespace

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx
```

Apply it:

```bash
kubectl apply -f pod.yaml
```

### Step 3: Verify Pods in the Namespace

```bash
kubectl get pods -n dev
```

---

## Same Resource Name in Different Namespaces

```bash
kubectl create namespace prod
```

You can now create **another `nginx-pod`** in `prod` without conflict.

---

## Real-World Analogy

A Kubernetes cluster is like an **office building**:

* The building → Kubernetes Cluster
* Floors → Namespaces
* Teams → Applications

Each team works independently, but infrastructure is shared.

---

## In Short

> **Namespaces help organize, isolate, secure, and manage Kubernetes resources within a single cluster.**
