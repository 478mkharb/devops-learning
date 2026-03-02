# API Deep Dive for DevOps - Part 1

> This is **Part 1** of an interactive, example-heavy API guide.

---

## 🧠 Start With a Thought Experiment

👉 Imagine **there are NO APIs**.

* Jenkins cannot trigger builds
* Terraform cannot create EC2
* Kubernetes cannot create Pods
* Prometheus cannot scrape metrics

❓ Question: *How would DevOps even work?*

➡️ **Answer**: It wouldn’t.

APIs are the **language of DevOps automation**.

---

## 1️⃣ What is an API?

### ❌ Wrong way to think

> API = some HTTP endpoint

### ✅ Correct way to think

> API = **a contract between two systems**

---

## 🎭 Real-Life Analogy

### 🍽 Restaurant Model

| Real Life | Software             |
| --------- | -------------------- |
| Customer  | Client (UI / Script) |
| Waiter    | API                  |
| Kitchen   | Server               |

* You don’t enter the kitchen
* You don’t know how food is made
* You just order & get response

👉 **That abstraction = API**

---

## 2️⃣ API in DevOps

### 🔹 Jenkins API

```bash
curl http://jenkins/job/build
```

➡️ Jenkins exposes an API to trigger pipelines

---

### 🔹 Kubernetes API

```bash
kubectl get pods
```

❓ What really happens?

➡️ `kubectl` calls **Kubernetes API Server**

---

### 🔹 Terraform + Cloud API

```text
Terraform → AWS API → EC2 Created
```

Terraform itself **creates nothing**.
It just talks to APIs.

---

## 🧩 Mini Quiz (Pause & Think)

❓ Is Docker CLI also using APIs?

✅ **Yes** — Docker CLI → Docker Daemon API

---

## 3️⃣ REST — What Problem Did It Solve?

Before REST:

* Tight coupling
* Heavy XML
* Stateful servers

REST solved:

* Scalability
* Simplicity
* Web-scale systems

---

## 4️⃣ Representational State Transfer (Visual Thinking)

### 🔹 Resource

```text
/users/10
```

### 🔹 State (data at this moment)

```json
{
  "id": 10,
  "name": "Mukesh",
  "status": "ACTIVE"
}
```

### 🔹 Representation

* JSON
* XML
* HTML

👉 You **never get the actual object**, only a *representation*.

---

## 🔁 Interactive Flow (Read Slowly)

1️⃣ Client requests resource

```http
GET /users/10
```

2️⃣ Server responds

```json
{
  "name": "Mukesh"
}
```

❓ What was transferred?

✅ **Representation of state**

---

## 5️⃣ HTTP Methods — Action Mapping Game

Match the action 👇

| Action      | HTTP Verb |
| ----------- | --------- |
| Read data   | ❓         |
| Create data | ❓         |
| Update data | ❓         |
| Delete data | ❓         |

### ✅ Answers

* Read → GET
* Create → POST
* Update → PUT / PATCH
* Delete → DELETE

---

## 6️⃣ REST is NOT RPC (Remote Procedure Call) - Critical Interview Concept

### ❌ RPC Style

```text
getUser(10)
```

### ✅ REST Style

```text
GET /users/10
```

👉 REST talks about **things (resources)**
👉 RPC talks about **actions**

---

## **Six REST Constraints (System Rules) - Important**


Let’s understand them like **system rules**.

---

### 1️⃣ Client–Server

🧠 Thought:

* Frontend team changes UI
* Backend team sleeps peacefully 😄

Because both are independent.

---

### 2️⃣ Stateless (VERY IMPORTANT)

❌ Server should NOT remember you

Every request must include:

* Auth token
* Required data

```http
GET /orders
Authorization: Bearer token
```

👉 This enables **load balancing & scaling**

---

### 3️⃣ Cacheable

❓ Why hit server again for same data?

```http
Cache-Control: max-age=3600
```

CDN, browser, proxy can cache it.

---

### 4️⃣ Uniform Interface

Rules:

* Use nouns
* Use HTTP verbs properly

```text
/users
/users/10
```

Predictable = developer friendly

---

### 5️⃣ Layered System

Client sees:

```text
api.company.com
```

Behind the scenes:

* Load balancer
* API Gateway
* Auth service
* Backend

Client never knows.

---

### 6️⃣ Code on Demand (Optional)

Server sends executable code (JS).

Used rarely but allowed.

---

## 🧠 Mini Interview Trap Question

❓ If an API violates statelessness, is it REST?

❌ No — it becomes **REST-like**, not REST.

---
