## What is Node Affinity?

**Node Affinity** is a Kubernetes scheduling feature that allows you to control **which nodes a Pod can or prefers to run on**, based on node labels. It provides **advanced and flexible rules** compared to `nodeSelector`.

In simple terms:

> Node affinity tells the scheduler *how strictly* a Pod should be placed on nodes with specific labels.

---

## Why Node Affinity is needed

* Place workloads on specific hardware (SSD, GPU, high-memory nodes)
* Separate environments (prod / dev / staging)
* Optimize performance and cost
* Apply flexible scheduling rules in production

---

## Types of Node Affinity

Node affinity is defined under:

```
spec.affinity.nodeAffinity
```

### 1. RequiredDuringSchedulingIgnoredDuringExecution (Hard Rule)

* Pod **must** be scheduled on matching nodes
* If no node matches → Pod stays **Pending**

### 2. PreferredDuringSchedulingIgnoredDuringExecution (Soft Rule)

* Scheduler **tries** to place Pod on matching nodes
* If not possible → Pod runs on other nodes

---

## Example: Node Labels

```
kubectl label node node1 disktype=ssd env=prod
kubectl label node node2 disktype=hdd env=dev
```

---

## Node Affinity – Required (Hard Rule) Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
  containers:
  - name: app
    image: nginx
```

### Explanation

* Pod runs **only on nodes with `disktype=ssd`**
* If no such node exists → Pod remains **Pending**

---

## Node Affinity – Preferred (Soft Rule) Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod-preferred
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 10
        preference:
          matchExpressions:
          - key: env
            operator: In
            values:
            - prod
  containers:
  - name: app
    image: nginx
```

### Explanation

* Scheduler prefers nodes with `env=prod`
* Pod can still run on other nodes if needed

---

## What is nodeSelector?

`nodeSelector` is the **simplest way** to schedule a Pod onto nodes with specific labels using **exact match**.

### nodeSelector Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-pod
spec:
  nodeSelector:
    disktype: ssd
  containers:
  - name: app
    image: nginx
```

### Explanation

* Pod runs only on nodes labeled `disktype=ssd`
* No flexibility or preferences

---

## Limitations of nodeSelector

* Supports only `key = value`
* No OR conditions
* No NOT conditions
* No soft preference
* No priority or weight

---

## Difference: nodeSelector vs nodeAffinity

| Feature           | nodeSelector | nodeAffinity            |
| ----------------- | ------------ | ----------------------- |
| Matching type     | Exact match  | Expressions & operators |
| OR logic          | ❌ No         | ✅ Yes                   |
| NOT logic         | ❌ No         | ✅ Yes                   |
| Hard & Soft rules | ❌ Only hard  | ✅ Both                  |
| Weight / Priority | ❌ No         | ✅ Yes                   |
| Complexity        | Simple       | Advanced                |
| Production use    | Limited      | Recommended             |

---

## OR Logic Example (Only possible with nodeAffinity)

```yaml
matchExpressions:
- key: disktype
  operator: In
  values:
  - ssd
  - nvme
```

---

## When to use what?

### Use nodeSelector when:

* Requirement is very simple
* Single label match
* Small or test clusters

### Use nodeAffinity when:

* Production workloads
* Multiple conditions
* Preference-based scheduling
* Advanced control is required

---

## Interview-ready Summary

* **nodeSelector** → Simple, rigid, exact match
* **nodeAffinity** → Flexible, expressive, production-grade scheduling
