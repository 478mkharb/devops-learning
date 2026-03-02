# 🧭 API Deep Dive for DevOps — Interactive Guide (Part 2)

## Types of APIs (REST, GraphQL, gRPC, WebSocket) — *When & Why to Use Which*

> This part is **decision-focused**. By the end, you should be able to say:
> **“For THIS problem, I’ll use THIS API type — and here’s why.”**

---

## 🧠 Warm‑up: One Problem, Many API Choices

**Problem:** A frontend needs user data.

Possible solutions:

* REST
* GraphQL
* gRPC
* WebSocket

❓ Are all correct?

✅ **Yes — but context decides the winner.**

---

## 1️⃣ REST API — The Default Workhorse

### How REST Thinks

* Everything is a **resource**
* Identified by **URLs**
* Manipulated using **HTTP verbs**

### Example

```http
GET /users/10
PUT /users/10
DELETE /users/10
```

```json
{
  "id": 10,
  "name": "Mukesh",
  "role": "DevOps"
}
```

### Why REST is everywhere

* Simple
* Human-readable
* HTTP-native
* Cache-friendly

### ❓ Interactive Check

**Q:** Can REST work without browsers?

✅ Yes — curl, scripts, CI tools, microservices

### When to use REST ✅

* Public APIs
* CRUD operations
* Cloud & DevOps tooling
* Simple microservices

### When REST struggles ❌

* Over/under-fetching data
* Very high-performance internal calls
* Real-time streaming

---

## 2️⃣ GraphQL — Client Is the Boss

### How GraphQL Thinks

* Client defines **exact data shape**
* Single endpoint
* Strong schema

### Example Query

```graphql
{
  user(id: 10) {
    name
    email
  }
}
```

### What the server returns

```json
{
  "name": "Mukesh",
  "email": "mukesh@gmail.com"
}
```

👉 No extra fields. No missing fields.

### ❓ Interactive Check

**Q:** Who controls the response?

✅ **Client**

### When to use GraphQL ✅

* Frontend-heavy apps
* Mobile apps (bandwidth sensitive)
* Complex UI screens

### When GraphQL is risky ❌

* Caching complexity
* Query abuse (needs limits)
* Simple CRUD APIs

### DevOps Note

* Needs query depth limits
* Harder to cache with CDN

---

## 3️⃣ gRPC — Speed Above All

### How gRPC Thinks

* **Function calls**, not URLs
* Binary data (Protocol Buffers)
* HTTP/2

### Example (Conceptual)

```text
getUser(userId=10)
```

### Why gRPC is FAST

* Binary payload
* Persistent connections
* Multiplexing

### ❓ Interactive Check

**Q:** Can browsers directly call gRPC?

❌ No (without gRPC-Web)

### When to use gRPC ✅

* Internal microservices
* High throughput systems
* Low latency requirements

### When NOT to use gRPC ❌

* Public APIs
* Browser-first apps
* Debug-heavy environments

### DevOps Reality

* Used inside Kubernetes clusters
* Often hidden behind REST gateways

---

## 4️⃣ WebSocket — Always Connected

### How WebSocket Thinks

* **Persistent connection**
* Bi-directional
* Event-driven

### Flow

```text
Client ⇄ Server (open connection)
```

### Example Use Cases

* Chat apps
* Live dashboards
* Trading systems
* Real-time alerts

### ❓ Interactive Check

**Q:** Is WebSocket stateless?

❌ No — connection maintains state

### When to use WebSocket ✅

* Real-time updates
* Push notifications
* Live monitoring UI

### When NOT to use ❌

* Simple request-response
* Highly scalable stateless APIs

---

## 5️⃣ REST vs GraphQL vs gRPC vs WebSocket (Decision Table)

| Feature          | REST     | GraphQL | gRPC    | WebSocket |
| ---------------- | -------- | ------- | ------- | --------- |
| Style            | Resource | Query   | RPC     | Event     |
| Data format      | JSON     | JSON    | Binary  | Any       |
| Browser friendly | ✅        | ✅       | ❌       | ✅         |
| Performance      | Medium   | Medium  | 🔥 High | High      |
| Caching          | Easy     | Hard    | N/A     | N/A       |
| Real-time        | ❌        | ❌       | ❌       | ✅         |
| DevOps usage     | 🔥🔥🔥   | 🔥      | 🔥🔥    | 🔥        |

---

## 6️⃣ Decision Scenarios (Interview Gold)

### Scenario 1

**Public cloud API (AWS, GitHub)**

✅ REST

---

### Scenario 2

**Mobile app with many UI variations**

✅ GraphQL

---

### Scenario 3

**Internal microservices in Kubernetes**

✅ gRPC

---

### Scenario 4

**Live metrics dashboard**

✅ WebSocket

---

## 🧠 Memory Trick (You’ll Thank Me Later)

* **REST** → *Resources*
* **GraphQL** → *Exact data*
* **gRPC** → *Speed*
* **WebSocket** → *Real-time*

---

## 🎯 DevOps Takeaway

As a DevOps engineer:

* You mostly **consume REST**
* You sometimes **support GraphQL**
* You often **run gRPC internally**
* You **monitor WebSockets**, not design them

---

## ⏭ What’s Next (Part 3)

Choose one:

1️⃣ Real DevOps APIs (Jenkins, Kubernetes, AWS, Terraform) with curl examples
2️⃣ API Security + Monitoring (JWT, OAuth, rate limits, Prometheus)
3️⃣ Architecture patterns (API Gateway, BFF, Service Mesh)

👉 Reply with **NEXT: <number>**
