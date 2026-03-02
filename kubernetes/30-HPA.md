# Horizontal Pod Autoscaler (HPA)

## What is HPA?

**Horizontal Pod Autoscaler (HPA)** is a Kubernetes feature that automatically **scales the number of pod replicas** in a Deployment, ReplicaSet, or StatefulSet based on **observed metrics** such as:

* CPU utilization
* Memory utilization
* Custom metrics (via Prometheus / Metrics Server)
* External metrics (queue length, requests per second, etc.)

In short:

> **More load → more pods | Less load → fewer pods**

---

## Why HPA is Needed

Without HPA:

* You manually change replica count
* Scaling is slow and error-prone

With HPA:

* Automatic scaling based on real traffic
* Better resource utilization
* Improved application availability

---

## How HPA Works (Step-by-Step)

1. **Metrics Server** collects resource metrics (CPU/Memory)
2. **HPA Controller** queries metrics periodically (default: every 15s)
3. HPA compares **current metrics vs target metrics**
4. Kubernetes **scales replicas up or down**

### Formula (CPU-based scaling)

```
Desired Replicas = Current Replicas × (Current CPU Utilization / Target CPU Utilization)
```

---

## Prerequisites for HPA

* Metrics Server must be installed
* Pods must define **resource requests**

```yaml
resources:
  requests:
    cpu: "200m"
    memory: "256Mi"
```

---

## HPA Example (CPU-based Scaling)

### Step 1: Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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
        resources:
          requests:
            cpu: "200m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
```

---

### Step 2: Create HPA

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

---

## What Happens During Scaling?

### Scenario

* Current replicas: **2**
* Target CPU: **50%**
* Actual CPU usage: **100%**

### Calculation

```
Desired Replicas = 2 × (100 / 50) = 4
```

➡ Kubernetes scales pods from **2 → 4**

---

## Scale Down Example

* CPU drops to **20%**

```
Desired Replicas = 4 × (20 / 50) = 1.6 ≈ 2
```

➡ Kubernetes scales pods down to **2** (respects minReplicas)

---

## Commands to Work with HPA

```bash
kubectl get hpa
kubectl describe hpa nginx-hpa
kubectl top pods
kubectl top nodes
```

---

## HPA with Memory (Example)

```yaml
metrics:
- type: Resource
  resource:
    name: memory
    target:
      type: Utilization
      averageUtilization: 70
```

---

## HPA vs Manual Scaling

| Feature             | Manual Scaling | HPA             |
| ------------------- | -------------- | --------------- |
| Automation          | ❌ No           | ✅ Yes           |
| Reaction to traffic | ❌ Slow         | ✅ Fast          |
| Resource efficiency | ❌ Poor         | ✅ Optimized     |
| Production ready    | ❌ Risky        | ✅ Best Practice |

---

## Key Points to Remember

* HPA scales **pods**, not nodes
* Requires Metrics Server
* Uses **requests**, not limits, for calculations
* Works best with stateless applications

---

## One-Line Summary

> **HPA automatically adjusts pod replicas based on real-time metrics to handle varying load efficiently.**
