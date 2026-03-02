# QoS Eviction Order in Kubernetes

## What is QoS (Quality of Service)?

In Kubernetes, **QoS (Quality of Service)** is a classification mechanism used by the kubelet to decide **which Pods should be evicted first** when a node experiences resource pressure (most commonly **memory pressure**, but also disk or PID pressure).

Every Pod is automatically assigned a QoS class based on how **CPU and memory requests and limits** are defined in its Pod specification.

---

## The Three QoS Classes

### 1. Guaranteed

A Pod is classified as **Guaranteed** when:

* Every container has **CPU and memory requests set**
* Requests are **equal to limits** for CPU and memory

**Use case:** Mission‑critical workloads (APIs, databases, core services)

---

### 2. Burstable

A Pod is classified as **Burstable** when:

* Requests are set, but
* Limits are higher than requests **or not set at all**

**Use case:** Applications that usually stay within limits but may need bursts

---

### 3. BestEffort

A Pod is classified as **BestEffort** when:

* No CPU or memory requests
* No CPU or memory limits

**Use case:** Non‑critical or experimental workloads

---

## QoS Eviction Order

When a node runs out of resources, Kubernetes evicts Pods in the following order:

1. **BestEffort Pods** (evicted first)
2. **Burstable Pods**
3. **Guaranteed Pods** (evicted last)

> Within the same QoS class, Pods that consume **more resources than they requested** are evicted first.

---

## Why QoS Eviction Order Is Important

### 1. Protects Critical Applications

Guaranteed Pods are usually business‑critical services. Evicting them last ensures application availability during resource pressure.

---

### 2. Prevents Node Instability

Instead of allowing the node to crash due to Out‑Of‑Memory (OOM), Kubernetes proactively evicts lower‑priority Pods to stabilize the node.

---

### 3. Encourages Proper Resource Planning

Teams are incentivized to define accurate requests and limits. Poorly defined Pods are penalized during eviction.

---

### 4. Predictable Behavior During Failures

Operators can clearly predict **which workloads will survive** during high load or outages.

---

## Detailed Example Scenario

### Node Capacity

* Total Memory: **8 GB**

### Running Pods

| Pod   | QoS Class  | Memory Request | Memory Usage |
| ----- | ---------- | -------------- | ------------ |
| Pod‑A | BestEffort | None           | 1.5 GB       |
| Pod‑B | Burstable  | 1 GB           | 3 GB         |
| Pod‑C | Guaranteed | 2 GB           | 2 GB         |

---

### What Happens During Memory Pressure?

The node crosses the memory pressure threshold.

**Eviction process:**

1. **Pod‑A (BestEffort)** is evicted first

   * No guarantees, easiest to remove

2. **Pod‑B (Burstable)** is evicted next

   * Using significantly more than its request

3. **Pod‑C (Guaranteed)** remains running

   * Fully protected unless the node is completely exhausted

---

## YAML Examples and Their QoS Classes

### BestEffort Pod

```yaml
resources: {}
```

---

### Burstable Pod

```yaml
resources:
  requests:
    memory: "500Mi"
    cpu: "250m"
  limits:
    memory: "1Gi"
```

---

### Guaranteed Pod

```yaml
resources:
  requests:
    memory: "1Gi"
    cpu: "500m"
  limits:
    memory: "1Gi"
    cpu: "500m"
```

---

## Best Practices

* Always define **CPU and memory requests**
* Use **Guaranteed QoS** for critical services
* Avoid **BestEffort Pods in production**
* Combine QoS with **PriorityClass** for stronger control
* Monitor resource usage to tune requests accurately

---

## One‑Line Summary

**QoS eviction order ensures Kubernetes evicts the least important workloads first, keeping critical applications running when node resources are under pressure.**
