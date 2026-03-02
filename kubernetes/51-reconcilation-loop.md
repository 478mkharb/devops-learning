# Reconciliation Loop in Kubernetes

## What is a Reconciliation Loop?

The **reconciliation loop** is the core control mechanism of Kubernetes. It is the continuous process where Kubernetes **compares the desired state** (defined by the user in YAML manifests) with the **current state** of the cluster, and then **takes corrective actions** to make the current state match the desired state.

In simple terms:

> **You declare *what you want***, Kubernetes continuously works to ensure ***that is what actually exists***.

This loop runs **forever**, not once.

---

## Desired State vs Current State

* **Desired State**

  * Stored in **etcd**
  * Defined via Kubernetes objects (Deployment, Pod, Service, etc.)
  * Example: `replicas: 3`

* **Current State**

  * Observed from the cluster
  * Reported by **kubelet**, container runtime, and nodes
  * Example: Only 2 Pods are running

If these two differ → **reconciliation happens**.

---

## Who Performs the Reconciliation?

Reconciliation is performed by **Controllers**, such as:

* Deployment Controller
* ReplicaSet Controller
* StatefulSet Controller
* Job Controller
* Node Controller
* Custom Controllers (Operators)

All controllers run inside **kube-controller-manager**.

Each controller follows the same pattern:

1. Watch Kubernetes API
2. Compare desired vs current state
3. Act to fix the difference

---

## High-Level Reconciliation Flow

```
User
 │
 │ kubectl apply -f deployment.yaml
 ▼
API Server
 │
 │ stores desired state
 ▼
ETCD  <──────────────┐
 │                   │
 │ watch events      │
 ▼                   │
Controller           │
 │                   │
 │ reconcile         │
 ▼                   │
API Server           │
 │                   │
 ▼                   │
Scheduler            │
 │                   │
 ▼                   │
Node (kubelet)       │
 │                   │
 ▼                   │
Container Runtime    │
 │                   │
 ▼                   │
Running Pods ────────┘
```

This loop never stops.

---

## Detailed Step-by-Step Example (Deployment)

### Example Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 3
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
        image: nginx:latest
```

---

### Step 1: Desired State is Created

* User applies the Deployment
* API Server validates the request
* Deployment object is stored in **etcd**

Desired State:

```
Deployment nginx-deploy
replicas = 3
```

---

### Step 2: Deployment Controller Watches API Server

* Deployment Controller sees a **new Deployment**
* It checks current state: **0 Pods exist**

Mismatch detected:

```
Desired: 3 Pods
Current: 0 Pods
```

---

### Step 3: Controller Takes Action

* Deployment Controller creates a **ReplicaSet**
* ReplicaSet Controller creates **3 Pod objects**

These Pod objects are written back to the API Server.

---

### Step 4: Scheduler Assigns Pods to Nodes

* Scheduler watches for Pods with no node assigned
* Chooses suitable nodes
* Updates Pod spec with `nodeName`

---

### Step 5: Kubelet Creates Containers

* Kubelet on each node watches assigned Pods
* Calls **CRI** (container runtime)
* Pulls image and starts containers

Current State becomes:

```
3 Pods Running
```

---

### Step 6: Continuous Monitoring

Controller keeps checking:

```
Desired: 3 Pods
Current: 3 Pods
```

No action needed → loop continues silently.

---

## Failure Scenario: Pod Crash

### Pod Dies Unexpectedly

* One Pod crashes
* Kubelet reports status to API Server
* Current State becomes:

```
Desired: 3 Pods
Current: 2 Pods
```

---

### Reconciliation Happens Again

* ReplicaSet Controller detects mismatch
* Creates **1 new Pod**
* Scheduler schedules it
* Kubelet starts container

Cluster returns to desired state.

---

## Full Reconciliation Loop (Expanded Diagram)

```
┌────────────┐
│   User     │
└─────┬──────┘
      │ kubectl apply
      ▼
┌────────────┐
│ API Server │◄─────────────┐
└─────┬──────┘              │
      │ stores state        │ watch
      ▼                     │
┌────────────┐              │
│    ETCD    │              │
└────────────┘              │
                            │
        ┌───────────────────┘
        ▼
┌────────────────────────┐
│ Controller (Reconcile) │
└───────────┬────────────┘
            │ create/update
            ▼
     ┌────────────┐
     │   Scheduler│
     └─────┬──────┘
           │ assign node
           ▼
     ┌────────────┐
     │  Kubelet   │
     └─────┬──────┘
           │ CRI calls
           ▼
┌──────────────────────┐
│ Container Runtime    │
└──────────────────────┘
           │
           ▼
     Running Containers

(Loop repeats continuously)
```

---

## Key Characteristics of Reconciliation Loop

* **Declarative** – user defines *what*, not *how*
* **Continuous** – always running
* **Eventually consistent** – state converges over time
* **Self-healing** – fixes failures automatically
* **Controller-driven** – logic lives in controllers

---

## One-Line Summary

> The reconciliation loop is Kubernetes' control mechanism that continuously ensures the cluster’s actual state matches the desired state stored in etcd, by observing, comparing, and correcting through controllers.

---

If you want, I can next:

* Explain reconciliation in **Custom Controllers / Operators**
* Map reconciliation to **real production incidents**
* Compare reconciliation with **imperative systems**
