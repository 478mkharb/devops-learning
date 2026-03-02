# Kubernetes DaemonSet – Deep Dive with Examples

## What is a DaemonSet?

A **DaemonSet** is a Kubernetes workload resource that ensures **exactly one Pod runs on every eligible Node** in the cluster (or on a selected subset of nodes).

> 📌 As nodes are **added**, a DaemonSet **automatically schedules Pods** on them. As nodes are **removed**, the Pods are **automatically cleaned up**.

Think of a DaemonSet as:

> **“One pod per node, always.”**

Unlike Deployments or ReplicaSets, you **do not specify replicas**. Kubernetes decides the number of Pods based on the number of nodes.

---

## Why DaemonSet Exists (Problem It Solves)

Some workloads must:

* Run on **every node**
* Have **direct access to node-level resources**
* Work independently per node

Deployments fail here because:

* They schedule pods **randomly**
* They scale by **replica count**, not node count

DaemonSet solves this by **binding pod lifecycle to node lifecycle**.

---

## How DaemonSet Works Internally

1. DaemonSet controller watches:

   * Nodes
   * DaemonSet definitions
2. When a node appears:

   * Controller checks node selectors / tolerations
   * Creates a pod **directly bound** to that node
3. When a node disappears:

   * Pod is garbage-collected
4. Scheduler behavior:

   * Pods created by DaemonSet are **pre-assigned to nodes**
   * Scheduler is mostly bypassed

---

## Core Characteristics of DaemonSet

| Feature          | Behavior                  |
| ---------------- | ------------------------- |
| Replicas         | ❌ Not defined             |
| Scaling          | Automatic (node-based)    |
| Pod placement    | One pod per node          |
| Scheduler        | Node pre-selected         |
| Restart behavior | Same as Pod restartPolicy |

---

## Common Real-World Use Cases of DaemonSet

### 1️⃣ Log Collection Agents

📌 Example: Fluentd, Filebeat, Logstash

**Why DaemonSet?**

* Logs live on each node
* Each node needs its own agent

Each pod:

* Mounts `/var/log`
* Ships logs to ELK / Loki

---

### 2️⃣ Monitoring Agents

📌 Example: Prometheus Node Exporter, Datadog Agent

**Why DaemonSet?**

* Collect CPU, memory, disk, network metrics per node
* Needs host-level access

---

### 3️⃣ Networking Components (CNI)

📌 Example: Calico, Flannel, Weave

**Why DaemonSet?**

* Every node needs networking
* Pods require host networking and privileged access

---

### 4️⃣ Storage & CSI Node Plugins

📌 Example: EBS CSI Node Driver

**Why DaemonSet?**

* Storage drivers interact with node kernel
* Must run on nodes using that storage

---

### 5️⃣ Security & Compliance Agents

📌 Example: Falco, Sysdig

**Why DaemonSet?**

* Observe system calls
* Monitor kernel-level activity

---

## Basic DaemonSet Example

### Scenario

Run an Nginx pod on **every node**.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-daemonset
  namespace: default
spec:
  selector:
    matchLabels:
      app: nginx-ds
  template:
    metadata:
      labels:
        app: nginx-ds
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

### Result

If cluster has:

* 3 nodes → 3 pods
* 5 nodes → 5 pods

---

## DaemonSet with Node Selector

### Scenario

Run pods **only on worker nodes**.

```yaml
spec:
  template:
    spec:
      nodeSelector:
        node-role.kubernetes.io/worker: ""
```

Only nodes with that label will run the pod.

---

## DaemonSet with Tolerations (Control Plane Nodes)

### Scenario

Run monitoring agent on **master/control-plane nodes**.

```yaml
spec:
  template:
    spec:
      tolerations:
      - key: "node-role.kubernetes.io/control-plane"
        operator: "Exists"
        effect: "NoSchedule"
```

Without this, DaemonSet pods **won’t run on tainted nodes**.

---

## DaemonSet with Host Access (Real Use Case)

### Fluentd Log Collector Example

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd:v1.16
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

This gives:

* One Fluentd per node
* Direct access to node logs

---

## Rolling Updates in DaemonSet

DaemonSet supports **rolling updates** (like Deployments).

```yaml
updateStrategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
```

This ensures:

* Only one node is impacted at a time
* Safe upgrades of node-level agents

---

## DaemonSet vs Deployment vs StatefulSet

| Feature         | DaemonSet   | Deployment    | StatefulSet   |
| --------------- | ----------- | ------------- | ------------- |
| Pods per node   | 1           | Any           | Any           |
| Scaling         | Node-based  | Replica-based | Replica-based |
| Stable identity | ❌           | ❌             | ✅             |
| Typical use     | Node agents | Apps          | Databases     |

---

## When NOT to Use DaemonSet ❌

Avoid DaemonSet if:

* App doesn’t need node-level access
* Horizontal scaling is required
* Load balancing across pods is needed

---

## System DaemonSets vs User DaemonSets (Full Detail)

### What is a System DaemonSet?

**System DaemonSets** run core Kubernetes or infrastructure components that are **mandatory for cluster operation**.

They are usually:

* Installed during **cluster bootstrap** (kubeadm, EKS, GKE, AKS)
* Managed by **Kubernetes or cloud providers**
* Deployed in the `kube-system` namespace

> 🚨 Deleting or misconfiguring a system DaemonSet can **break the cluster**.

---

### Common Examples of System DaemonSets

| DaemonSet             | Purpose                                  |
| --------------------- | ---------------------------------------- |
| kube-proxy            | Service networking & iptables/ipvs rules |
| calico-node / flannel | Pod-to-Pod networking (CNI)              |
| aws-node (EKS)        | VPC CNI integration                      |
| cilium                | eBPF-based networking                    |
| node-local-dns        | Faster DNS resolution                    |

Namespace:

```bash
kube-system
```

---

### Characteristics of System DaemonSets

* Critical for **cluster networking & communication**
* Usually **privileged** containers
* Often use `hostNetwork: true`
* Tolerate **all taints**, including control-plane taints
* Run on **every node**, including masters

Example toleration:

```yaml
tolerations:
- operator: Exists
```

---

### What is a User DaemonSet?

**User DaemonSets** are created by **cluster admins or DevOps teams** to run **node-level support workloads**.

They enhance:

* Observability
* Security
* Logging
* Node maintenance

> ✅ Safe to modify or delete (with operational awareness).

---

### Common Examples of User DaemonSets

| Use Case       | Tool                         |
| -------------- | ---------------------------- |
| Log collection | Fluentd, Filebeat            |
| Monitoring     | Node Exporter, Datadog Agent |
| Security       | Falco, Sysdig                |
| Custom agents  | Backup, cleanup, scanners    |

Namespaces:

```bash
default
monitoring
logging
security
```

---

### Characteristics of User DaemonSets

* Optional (non-critical)
* Fully managed by users
* Often restricted to **worker nodes only**
* Use nodeSelectors / affinities
* Rarely need control-plane access

Example nodeSelector:

```yaml
nodeSelector:
  node-role.kubernetes.io/worker: ""
```

---

### Side-by-Side Comparison

| Feature         | System DaemonSet      | User DaemonSet  |
| --------------- | --------------------- | --------------- |
| Purpose         | Cluster functionality | App/ops support |
| Namespace       | kube-system           | Any namespace   |
| Created by      | Kubernetes / Cloud    | Admin / DevOps  |
| Privileged      | Almost always         | Sometimes       |
| Runs on masters | Yes                   | Usually no      |
| Safe to delete  | ❌ No                  | ✅ Yes           |
| Upgrade control | Provider-managed      | User-managed    |

---

### What Happens If You Delete Them?

❌ **Deleting System DaemonSet**

```bash
kubectl delete ds kube-proxy -n kube-system
```

Impact:

* Services stop routing
* Networking breaks
* Production outage

✅ **Deleting User DaemonSet**

```bash
kubectl delete ds fluentd -n logging
```

Impact:

* Logs stop flowing
* Applications continue running

---

## Updated Key Interview Takeaways

* DaemonSet ensures **one pod per node**
* **System DaemonSets** are cluster-critical and provider-managed
* **User DaemonSets** support logging, monitoring, security
* System DaemonSets usually run in `kube-system`
* User DaemonSets are safe to customize
* DaemonSets scale with **nodes, not replicas**

---

✅ This document now includes:

* DaemonSet fundamentals
* All real-world use cases
* YAML examples
* Rolling updates
* System vs User DaemonSets (deep dive)
* Interview-ready explanations
