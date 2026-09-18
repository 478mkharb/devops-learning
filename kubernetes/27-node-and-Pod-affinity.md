# Kubernetes Affinity and Anti-Affinity

## Overview

## Important Terminology Clarification

Do not confuse these two different concepts:

```yaml
key: topology.kubernetes.io/zone
```

and:

```yaml
topologyKey: kubernetes.io/hostname
```

They are not the same field.

- `key` is used in **Node Affinity** to inspect a node label.
- `topologyKey` is used in **Pod Affinity** and **Pod Anti-Affinity** to define the topology domain.
- `topologyKey` is also used by **Topology Spread Constraints**, which is a separate scheduling feature.
- Normal `nodeAffinity` does **not** contain a `topologyKey` field.

For example, this is valid Node Affinity:

```yaml
nodeAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
      - matchExpressions:
          - key: topology.kubernetes.io/zone
            operator: In
            values:
              - us-east-1a
```

Here, `topology.kubernetes.io/zone` is simply the name of a node label.

By contrast, this is Pod Anti-Affinity:

```yaml
podAntiAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchLabels:
          app: myapp
      topologyKey: kubernetes.io/hostname
```

Here, `topologyKey` defines the placement boundary as the node.

---

Kubernetes affinity rules control where Pods are scheduled in relation to:

- **Node properties** — using **Node Affinity**
- **Other Pods** — using **Pod Affinity**
- **Other Pods that should be kept away** — using **Pod Anti-Affinity**

These rules are evaluated by **kube-scheduler**.

---

# 1. Node Affinity

## What is Node Affinity?

**Node Affinity** constrains which nodes a Pod can run on based on node labels.

Common use cases:

- GPU-enabled nodes
- SSD-backed nodes
- Specific availability zones
- Dedicated production nodes
- CPU architecture, such as ARM or AMD64
- Compliance or licensing requirements

Node affinity is more expressive than `nodeSelector`.

## Label a node

```bash
kubectl label nodes node-1 disktype=ssd
kubectl get nodes --show-labels
```

## Types of Node Affinity

### Required node affinity

```yaml
requiredDuringSchedulingIgnoredDuringExecution
```

A **hard requirement**. If no node satisfies the rule, the Pod remains `Pending`.

### Preferred node affinity

```yaml
preferredDuringSchedulingIgnoredDuringExecution
```

A **soft preference**. The scheduler tries to satisfy it but can use another suitable node.

## Required Node Affinity example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ssd-app
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
    - name: nginx
      image: nginx:1.27
```

Meaning:

> Schedule the Pod only on a node with `disktype=ssd`.

## Preferred Node Affinity example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: zone-preferred-app
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 80
          preference:
            matchExpressions:
              - key: topology.kubernetes.io/zone
                operator: In
                values:
                  - us-east-1a
  containers:
    - name: nginx
      image: nginx:1.27
```

- `weight` is between `1` and `100`.
- Higher weight gives the preference more importance.
- The Pod may still run in another zone.

## Node Affinity operators

| Operator | Meaning |
|---|---|
| `In` | Label value must match one listed value |
| `NotIn` | Label value must not match listed values |
| `Exists` | Label key must exist |
| `DoesNotExist` | Label key must not exist |
| `Gt` | Numeric label value must be greater than the given value |
| `Lt` | Numeric label value must be less than the given value |

## Negative Node Affinity

Kubernetes has no separate feature called **Node Anti-Affinity**. To avoid nodes, use negative node affinity with `NotIn` or `DoesNotExist`.

### Example: avoid GPU nodes

Assume GPU nodes have:

```text
accelerator=gpu
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: normal-workload
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: accelerator
                operator: NotIn
                values:
                  - gpu
  containers:
    - name: nginx
      image: nginx:1.27
```

Meaning:

> Schedule this Pod only on nodes where `accelerator` is not `gpu`.

## When to use Node Affinity

| Use case | Example |
|---|---|
| GPU workloads | Run ML Pods on GPU nodes |
| Storage | Run database Pods on SSD nodes |
| Availability zones | Prefer or require a zone |
| CPU architecture | Run ARM workloads on ARM nodes |
| Compliance | Use approved nodes only |
| Dedicated pools | Run production workloads on production nodes |

---

# 2. Pod Affinity

## What is Pod Affinity?

**Pod Affinity** tells Kubernetes to place a Pod near other matching Pods.

It is based on **Pod labels**, not node labels.

Common use cases:

- Frontend near backend
- Application near cache
- Related services in the same zone
- Frequently communicating workloads

## Required Pod Affinity example

Assume a backend Pod has:

```yaml
labels:
  app: backend
```

A frontend Pod can request placement on a node where a backend Pod exists:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
              - key: app
                operator: In
                values:
                  - backend
          topologyKey: kubernetes.io/hostname
  containers:
    - name: nginx
      image: nginx:1.27
```

Meaning:

> Schedule the frontend Pod on a node where a Pod labeled `app=backend` is running.

## Preferred Pod Affinity example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend-preferred
spec:
  affinity:
    podAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 70
          podAffinityTerm:
            labelSelector:
              matchLabels:
                app: backend
            topologyKey: kubernetes.io/hostname
  containers:
    - name: nginx
      image: nginx:1.27
```

This is a preference, not a strict requirement.

---

# 3. Pod Anti-Affinity

## What is Pod Anti-Affinity?

**Pod Anti-Affinity** tells Kubernetes to avoid placing a Pod near matching Pods.

It is commonly used to distribute replicas across nodes or availability zones for high availability.

## Without Pod Anti-Affinity

```text
Node 1
├── App Pod 1
├── App Pod 2
└── App Pod 3
```

If Node 1 fails, all replicas are lost.

## With Pod Anti-Affinity

```text
Node 1       Node 2       Node 3
├── App 1    ├── App 2    └── App 3
```

If Node 1 fails, only one replica is affected.

## Required Pod Anti-Affinity example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
  labels:
    app: myapp
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
              - key: app
                operator: In
                values:
                  - myapp
          topologyKey: kubernetes.io/hostname
  containers:
    - name: nginx
      image: nginx:1.27
```

Meaning:

> Do not schedule this Pod on the same node as another Pod labeled `app=myapp`.

## Preferred Pod Anti-Affinity example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-preferred
  labels:
    app: myapp
spec:
  affinity:
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 100
          podAffinityTerm:
            labelSelector:
              matchLabels:
                app: myapp
            topologyKey: kubernetes.io/hostname
  containers:
    - name: nginx
      image: nginx:1.27
```

The scheduler tries to spread matching Pods across nodes, but can place them together if necessary.

## Spread across availability zones

Use:

```yaml
topologyKey: topology.kubernetes.io/zone
```

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: zone-spread-app
  labels:
    app: myapp
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchLabels:
              app: myapp
          topologyKey: topology.kubernetes.io/zone
  containers:
    - name: nginx
      image: nginx:1.27
```

Meaning:

> Do not place this Pod in the same availability zone as another matching Pod.

---

# 4. Understanding `topologyKey`

`topologyKey` defines the topology domain in which Pod affinity or anti-affinity is applied.

| `topologyKey` | Meaning |
|---|---|
| `kubernetes.io/hostname` | Node-level topology |
| `topology.kubernetes.io/zone` | Availability-zone topology |
| `topology.kubernetes.io/region` | Region-level topology |

Examples:

```yaml
topologyKey: kubernetes.io/hostname
```

Means the rule applies at node level.

```yaml
topologyKey: topology.kubernetes.io/zone
```

Means the rule applies at availability-zone level.

**Important:** Normal `nodeAffinity` does not use `topologyKey`; it uses node labels. `topologyKey` is used inside Pod affinity and Pod anti-affinity terms.

---

# 5. Hard Rules vs Soft Rules

| Rule | Type | Behavior |
|---|---|---|
| `requiredDuringSchedulingIgnoredDuringExecution` | Hard | Must be satisfied |
| `preferredDuringSchedulingIgnoredDuringExecution` | Soft | Scheduler tries to satisfy it |

## What does `IgnoredDuringExecution` mean?

The rule is checked during scheduling. If the environment changes later, Kubernetes does not automatically evict the Pod merely because the affinity condition is no longer true.

---

# 6. Comparison Table

| Feature | Based on | Purpose | Example |
|---|---|---|---|
| Node Affinity | Node labels | Select suitable nodes | Run on SSD nodes |
| Negative Node Affinity | Node labels | Avoid unsuitable nodes | Avoid GPU nodes |
| Pod Affinity | Other Pod labels | Place Pods together | Frontend near backend |
| Pod Anti-Affinity | Other Pod labels | Separate Pods | Replicas on different nodes |

---

# 7. Node Affinity vs Taints and Tolerations

## Node Affinity

A Pod says:

> I want to run on nodes with these properties.

Example:

```text
disktype=ssd
```

## Taints and Tolerations

A node says:

> Do not schedule ordinary Pods here.

Example:

```bash
kubectl taint nodes node-1 workload=gpu:NoSchedule
```

A Pod needs a matching toleration to be allowed onto that node.

For dedicated GPU nodes, it is common to use both:

- A node label and node affinity
- A node taint and Pod toleration

---

# 8. Troubleshooting

Inspect a Pending Pod:

```bash
kubectl describe pod <pod-name>
```

Check node labels:

```bash
kubectl get nodes --show-labels
```

Check Pod labels:

```bash
kubectl get pods --show-labels
```

Check placement:

```bash
kubectl get pod <pod-name> -o wide
```

Common causes of Pending Pods:

- No node matches required node affinity
- No node satisfies Pod affinity
- Pod anti-affinity prevents placement
- Required topology labels are missing
- Nodes do not have enough resources

---

# 9. Common Mistakes

1. **Forgetting to label nodes** — required affinity cannot match.
2. **Using required when preferred is enough** — this can leave Pods Pending.
3. **Confusing node and Pod labels** — node affinity checks node labels; Pod affinity checks Pod labels.
4. **Assuming Node Anti-Affinity is a separate feature** — use negative node affinity instead.
5. **Using the wrong topology key** — hostname is node-level; zone is availability-zone-level.
6. **Forgetting matching labels** — selectors must match the intended Pods.

---

# 10. Interview Questions and Answers

## What is Node Affinity?

> Node Affinity is a Kubernetes scheduling rule that places Pods on nodes based on node labels. It supports hard requirements and soft preferences.

## What is Pod Affinity?

> Pod Affinity places a Pod near other matching Pods based on their labels. It is useful when workloads benefit from locality.

## What is Pod Anti-Affinity?

> Pod Anti-Affinity keeps Pods away from matching Pods. It is commonly used to distribute replicas across nodes or zones for high availability.

## Why is there Pod Anti-Affinity but no separate Node Anti-Affinity?

> Pod anti-affinity expresses relationships between workloads, such as keeping replicas apart. For nodes, Kubernetes already provides node affinity with `NotIn` and `DoesNotExist`, plus taints and tolerations. Therefore, a separate Node Anti-Affinity feature is not required.

## What is `topologyKey`?

> `topologyKey` defines the topology domain in which Pod affinity or anti-affinity is applied, such as a node, zone, or region.

## What happens when required affinity cannot be satisfied?

> The Pod remains Pending until a suitable placement becomes available.

## What is the difference between required and preferred affinity?

> Required affinity is a hard requirement. Preferred affinity is a soft preference that the scheduler may ignore if necessary.

---

# Quick Memory Trick

```text
Node Affinity
    = Select or avoid nodes based on node properties

Pod Affinity
    = Keep Pods together

Pod Anti-Affinity
    = Keep Pods apart
```

# Key Takeaway

> **Node Affinity** selects or avoids nodes based on node labels. **Pod Affinity** places Pods near matching Pods. **Pod Anti-Affinity** separates Pods from matching Pods, commonly to improve high availability. `topologyKey` defines whether the Pod relationship applies across nodes, zones, or regions.
