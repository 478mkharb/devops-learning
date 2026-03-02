# 🌐 Fanout vs Host-based Routing in Kubernetes Ingress

Kubernetes **Ingress** lets you expose multiple services through a single external entry point (usually an Ingress Controller like NGINX, Traefik, or ALB). Two of the most common routing strategies are **Fanout routing** and **Host-based routing**.

This document explains both concepts in detail, with clear examples and YAML manifests.

---

## 1️⃣ What is Fanout Routing?

**Fanout routing** (also called *path-based routing*) routes traffic to different backend services **based on the URL path**.

### 🔹 How it works

* Same **host / domain**
* Different **paths**
* Each path maps to a different service

### 🔹 Real-world analogy

Think of a shopping website:

* `/products` → Product service
* `/orders` → Order service
* `/users` → User service

All are served from:

```
https://shop.example.com
```

---

## 🧭 Fanout Routing Architecture

```
Client
  |
  v
Ingress Controller
  |
  |-- /products  --> product-service
  |-- /orders    --> order-service
  |-- /users     --> user-service
```

---

## 📄 Fanout Routing – Ingress YAML Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: fanout-ingress
spec:
  rules:
  - host: shop.example.com
    http:
      paths:
      - path: /products
        pathType: Prefix
        backend:
          service:
            name: product-service
            port:
              number: 80
      - path: /orders
        pathType: Prefix
        backend:
          service:
            name: order-service
            port:
              number: 80
      - path: /users
        pathType: Prefix
        backend:
          service:
            name: user-service
            port:
              number: 80
```

### 🔹 Request Flow Example

| Request URL                 | Routed To       |
| --------------------------- | --------------- |
| `shop.example.com/products` | product-service |
| `shop.example.com/orders`   | order-service   |
| `shop.example.com/users`    | user-service    |

---

## 2️⃣ What is Host-based Routing?

**Host-based routing** routes traffic **based on the domain name (Host header)** of the incoming request.

### 🔹 How it works

* Different **subdomains or domains**
* Each host points to a different service
* Path is usually `/`

### 🔹 Real-world analogy

Think of multiple apps under the same organization:

* `api.example.com` → Backend API
* `admin.example.com` → Admin UI
* `blog.example.com` → Blog service

---

## 🧭 Host-based Routing Architecture

```
Client
  |
  v
Ingress Controller
  |
  |-- api.example.com    --> api-service
  |-- admin.example.com  --> admin-service
  |-- blog.example.com   --> blog-service
```

---

## 📄 Host-based Routing – Ingress YAML Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hostbased-ingress
spec:
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80

  - host: admin.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: admin-service
            port:
              number: 80
```

### 🔹 Request Flow Example

| Request Host        | Routed To     |
| ------------------- | ------------- |
| `api.example.com`   | api-service   |
| `admin.example.com` | admin-service |

---

## ⚖️ Fanout vs Host-based Routing – Comparison

| Feature          | Fanout Routing              | Host-based Routing          |
| ---------------- | --------------------------- | --------------------------- |
| Routing based on | URL path                    | Host / domain               |
| Domain usage     | Single domain               | Multiple domains/subdomains |
| Best for         | Microservices under one app | Multiple apps or tenants    |
| URL structure    | `/app1`, `/app2`            | `app1.example.com`          |
| Common use case  | API gateway style           | SaaS, multi-app platforms   |

---

## 🧠 When to Use What?

### ✅ Use Fanout Routing when:

* You want **one domain**
* Services are part of the **same application**
* Clean API-style paths

### ✅ Use Host-based Routing when:

* You manage **multiple applications**
* Need **clear separation**
* TLS certs per subdomain

---

## 🚀 Bonus: You Can Combine Both

Ingress supports **host + path-based routing together**:

```yaml
host: api.example.com
path: /v1
```

This is common in real-world production setups.

---

## ✅ Summary

* **Fanout routing** = Path-based (`/service1`, `/service2`)
* **Host-based routing** = Domain-based (`service.example.com`)
* Both are powerful, flexible, and widely used in Kubernetes Ingress

This knowledge is essential for designing scalable, clean Kubernetes networking architectures.
