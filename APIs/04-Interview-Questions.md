# 🔥 DevOps Web API Interview Questions

## Beginner → Advanced → Brutal 

> This is the **final part**. If you can answer these well, APIs will **never** be your weak area in DevOps interviews.

---

## 🟢 Beginner Level (Foundation Check)

### Q1. What is an API?

**Expected answer:**

> An API is a contract that allows two software systems to communicate in a structured way.

---

### Q2. Why are APIs important for DevOps?

**Answer:**

* CI/CD tools are API-driven
* Cloud resources are managed via APIs
* Kubernetes works entirely through APIs

---

### Q3. What is REST?

**Answer:**

> REST is an architectural style where systems communicate by transferring representations of resource state using HTTP.

---

### Q4. REST vs SOAP (one line)

**Answer:**

> REST is resource-based and lightweight, while SOAP is a strict XML-based protocol.

---

### Q5. What HTTP methods are commonly used in REST?

**Answer:**
GET, POST, PUT, PATCH, DELETE

---

## 🟡 Intermediate Level (Real Understanding)

### Q6. Explain Representational State Transfer in simple terms

**Answer:**

> Clients and servers exchange representations (JSON/XML) of resource state instead of sharing objects or sessions.

---

### Q7. What are the six REST constraints?

**Answer:**

* Client–Server
* Stateless
* Cacheable
* Uniform Interface
* Layered System
* Code on Demand (optional)

---

### Q8. If an API uses JSON, is it REST?

**Answer:**
❌ No. JSON alone does not make an API REST. REST must follow its constraints.

---

### Q9. Is REST always stateless?

**Answer:**
✅ Yes. If state is stored on the server, it is not REST-compliant.

---

### Q10. If I use GET and PUT, is it SOAP?

**Answer:**
❌ No. Using HTTP verbs semantically indicates REST, not SOAP.

---

## 🟠 Advanced Level (DevOps-Focused)

### Q11. How does kubectl work internally?

**Answer:**

> kubectl is a client that talks to the Kubernetes API Server using REST APIs.

---

### Q12. Is Kubernetes an API or a tool?

**Answer:**

> Kubernetes is an API platform; kubectl and controllers are API clients.

---

### Q13. How does Terraform create infrastructure?

**Answer:**

> Terraform calls cloud provider APIs via providers; it does not create resources itself.

---

### Q14. REST vs gRPC — when would you choose gRPC?

**Answer:**

> For internal microservices requiring low latency, high throughput, and strong contracts.

---

### Q15. What happens if an API is not stateless?

**Answer:**

* Scaling becomes difficult
* Load balancers need sticky sessions
* Failures impact users

---

## 🔴 Brutal / Senior-Level Questions

### Q16. Is REST dependent on HTTP?

**Answer:**

> No. REST can work over other protocols, but HTTP fits it best.

---

### Q17. Can REST APIs use XML?

**Answer:**

> Yes. REST is format-agnostic; JSON is just more common.

---

### Q18. Why is caching important in REST?

**Answer:**

> Caching reduces server load and latency, enabling internet-scale performance.

---

### Q19. How would you secure APIs in production?

**Answer:**

* TLS
* OAuth/JWT
* API Gateway
* Rate limiting
* Secrets management

---

### Q20. How do you monitor APIs as a DevOps engineer?

**Answer:**

* Request rate
* Error rate
* Latency
* Prometheus + Grafana

---

### Q21. REST vs WebSocket — can WebSocket replace REST?

**Answer:**
❌ No. WebSocket is for real-time communication; REST is for stateless request-response.

---

## 🧠 Scenario-Based (Interview Favorite)

### Q22. Jenkins job is not triggering from GitHub webhook. How do you debug?

**Answer approach:**

* Check webhook delivery
* Validate Jenkins API endpoint
* Check auth token
* Inspect HTTP status codes

---

### Q23. Terraform apply fails but console works. Why?

**Answer:**

* API permissions missing
* Wrong region
* Provider auth issue

---

### Q24. API works via curl but not via browser. Why?

**Answer:**

* CORS
* Missing headers
* Auth issues

---

## 🎯 Ultimate One-Liners (Use These)

* "Every DevOps tool is an API with a CLI wrapper."
* "Stateless APIs scale better."
* "Terraform orchestrates APIs; it doesn’t provision directly."

---
