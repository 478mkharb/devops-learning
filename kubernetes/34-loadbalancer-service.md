## What is a LoadBalancer Service in Kubernetes?

A **LoadBalancer Service** is a Kubernetes Service type that exposes an application **externally** using a **cloud provider’s managed load balancer** (AWS ELB/ALB/NLB, GCP Load Balancer, Azure Load Balancer, etc.).

It is mainly used when you want **direct external access** to a service running inside a Kubernetes cluster.

---

## Why Do We Need LoadBalancer Service?

* Expose applications to the **internet**
* Distribute traffic across **multiple pods**
* Provide a **stable external IP / DNS**
* Offload traffic handling to **cloud-native load balancers**

---

## How LoadBalancer Service Works

1. You create a Service with `type: LoadBalancer`
2. Kubernetes asks the **cloud provider** to create a load balancer
3. The cloud load balancer gets a **public IP / DNS**
4. Traffic flow:

```
User → Cloud Load Balancer → NodePort → Pod
```

Internally, a LoadBalancer Service **automatically creates**:

* A **NodePort** Service
* Uses **kube-proxy** to route traffic to Pods

---

## LoadBalancer Service YAML Example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-lb-service
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
    - port: 80          # External port
      targetPort: 8080  # Pod container port
```

---

## Corresponding Deployment Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: web
          image: nginx
          ports:
            - containerPort: 8080
```

---

## What Happens After Creation?

Run:

```bash
kubectl get svc web-lb-service
```

Output:

```
NAME             TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)
web-lb-service   LoadBalancer   10.96.12.10    35.201.x.x       80:32001/TCP
```

* `EXTERNAL-IP` → Public IP created by cloud provider
* `32001` → Auto-created NodePort

---

## Traffic Flow (Step-by-Step)

```
Internet User
     ↓
Cloud Load Balancer (Public IP)
     ↓
NodePort (on any node)
     ↓
Service (ClusterIP)
     ↓
Pod (via label selector)
```

---

## LoadBalancer vs NodePort

| Feature           | NodePort         | LoadBalancer       |
| ----------------- | ---------------- | ------------------ |
| External Access   | Yes (via NodeIP) | Yes (via Cloud LB) |
| Public IP         | ❌                | ✅                  |
| Cloud Integration | ❌                | ✅                  |
| Production Ready  | ❌                | ✅                  |
| Cost              | Free             | Paid (Cloud LB)    |

---

## When to Use LoadBalancer Service

✅ Public-facing applications

* Web applications
* APIs
* Microservices (external access)

❌ Avoid for:

* Internal services
* Cost-sensitive environments
* Local clusters (minikube, kind)

---

## LoadBalancer in Minikube

Minikube does **not** provide a real cloud load balancer.
Use:

```bash
minikube service web-lb-service
```

---

## Key Takeaways

* LoadBalancer exposes services **outside the cluster**
* Uses **cloud-managed load balancers**
* Automatically creates a **NodePort**
* Best suited for **production workloads**

---

✅ Short Definition:

> A LoadBalancer Service exposes a Kubernetes application externally using a cloud provider’s load balancer and distributes traffic across pods.
