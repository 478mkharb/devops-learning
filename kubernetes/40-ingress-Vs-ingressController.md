# Kubernetes Ingress – Complete Explanation

## 1. What is Ingress?

**Ingress** is a Kubernetes API object that manages **external HTTP/HTTPS access** to services inside a cluster.

Instead of exposing every service with its own LoadBalancer or NodePort, Ingress provides **centralized routing rules** based on:

* Hostname (domain-based routing)
* URL path (path-based routing)

Ingress works at **Layer 7 (HTTP/HTTPS)**.

### Why Ingress exists

* Avoid multiple cloud LoadBalancers
* Centralized routing
* URL‑based and host‑based routing
* TLS (HTTPS) termination

### Simple flow

```
User → Ingress Controller → Kubernetes Service → Pod
```

---

## 2. Difference Between Ingress and Service Type LoadBalancer

| Feature                   | Ingress           | LoadBalancer        |
| ------------------------- | ----------------- | ------------------- |
| Kubernetes object         | Yes               | Yes (Service type)  |
| Layer                     | L7 (HTTP/HTTPS)   | L4 (TCP/UDP)        |
| Routing                   | Host & Path based | No routing          |
| Cloud LoadBalancer needed | One shared        | One per service     |
| TLS termination           | Yes               | Limited             |
| Cost                      | Low               | High (multiple LBs) |
| Protocols                 | HTTP / HTTPS      | Any TCP/UDP         |

### Key difference in plain words

* **LoadBalancer** exposes *one service = one external IP*
* **Ingress** exposes *multiple services using one external IP*

### Example comparison

**Without Ingress**

```
frontend-service  → LoadBalancer → EXTERNAL-IP-1
backend-service   → LoadBalancer → EXTERNAL-IP-2
```

**With Ingress**

```
frontend.example.com → Ingress → frontend-service
api.example.com      → Ingress → backend-service
```

---

## 3. What is an Ingress Controller?

An **Ingress Controller** is the actual component that:

* Watches Ingress resources
* Reads routing rules
* Configures a real proxy/load balancer

⚠️ **Ingress alone does nothing** — it requires an Ingress Controller.

### Common Ingress Controllers

* NGINX Ingress Controller
* HAProxy Ingress
* Traefik
* AWS ALB Ingress Controller
* GKE Ingress

### Responsibility of Ingress Controller

* Accept external traffic
* Apply routing rules
* Forward traffic to correct service
* Handle TLS certificates
* Load balance traffic across pods

---

## 4. How Ingress, Controller, Service, and Pod Work Together

```
Client
  ↓
Cloud LoadBalancer (created by Ingress Controller)
  ↓
Ingress Controller (NGINX / ALB)
  ↓
Ingress Rules
  ↓
Service (ClusterIP)
  ↓
Pods
```

---

## 5. Example: Ingress Controller (NGINX) – Basic YAML

Below is a **minimal but realistic example** of deploying the **NGINX Ingress Controller** using Kubernetes manifests.

### Step 1: Ingress Controller Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ingress-nginx
```

---

### Step 2: Ingress Controller Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ingress-nginx
  template:
    metadata:
      labels:
        app: ingress-nginx
    spec:
      containers:
      - name: controller
        image: registry.k8s.io/ingress-nginx/controller:v1.9.0
        args:
          - /nginx-ingress-controller
          - --publish-service=$(POD_NAMESPACE)/ingress-nginx-controller
        env:
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        ports:
        - name: http
          containerPort: 80
        - name: https
          containerPort: 443
```

---

### Step 3: Expose Ingress Controller (Service)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  type: LoadBalancer
  selector:
    app: ingress-nginx
  ports:
  - name: http
    port: 80
    targetPort: 80
  - name: https
    port: 443
    targetPort: 443
```

---

### What this setup does

* Deploys NGINX as the Ingress Controller
* Creates **one external LoadBalancer IP**
* All Ingress rules use this controller

---

## 6. Example: Ingress with Two Services

### Services

* `frontend-service` (port 80)
* `backend-service` (port 8080)

### Ingress YAML Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  rules:
  - host: frontend.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80

  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 8080
```

### Traffic behavior

* Request to `frontend.example.com` → frontend pods
* Request to `api.example.com` → backend pods

---

## 6. Ingress vs Ingress Controller (Very Important)

| Component          | Purpose               |
| ------------------ | --------------------- |
| Ingress            | Defines routing rules |
| Ingress Controller | Implements routing    |

### Easy analogy

* **Ingress** → Traffic rules (configuration)
* **Ingress Controller** → Traffic police (execution)

---

## 7. When to Use What?

### Use LoadBalancer when

* Non‑HTTP traffic (TCP/UDP)
* Simple setup
* Very few services

### Use Ingress when

* HTTP/HTTPS workloads
* Many services
* Domain/path routing required
* TLS termination needed
* Cost optimization matters

---

## 8. One‑Line Summary

> **Ingress exposes multiple HTTP services using a single external entry point, while the Ingress Controller makes those rules work in real traffic.**
