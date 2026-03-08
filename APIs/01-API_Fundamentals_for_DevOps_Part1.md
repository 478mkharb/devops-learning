# 🚀 API – Complete Guide (DevOps & System Design)

![API](https://img.shields.io/badge/API-Concept-blue) ![REST](https://img.shields.io/badge/REST-Architecture-green) ![DevOps](https://img.shields.io/badge/DevOps-Learning-orange)

A **beginner → advanced guide** to understand APIs, REST, RESTful services, Web Services, and modern API‑driven architecture used in **cloud, DevOps, and microservices systems**.

---

# 📑 Table of Contents

* [1. What is an API](#-1-what-is-an-api)
* [2. Why APIs Are Required](#-2-why-apis-are-required)
* [3. UI vs API](#️-3-ui-vs-api)
* [4. Commands vs APIs](#-4-commands-vs-apis)
* [5. API Communication Flow](#-5-api-communication-flow)
* [6. Core API Components](#-6-core-api-components)
* [7. Resource Concept](#-7-resource-concept)
* [8. HTTP Methods](#-8-http-methods)
* [9. API Response Formats](#-9-api-response-formats)
* [10. Types of APIs](#-10-types-of-apis)
* [11. API Architectural Styles](#-11-api-architectural-styles)
* [12. What is REST](#-12-what-is-rest)
* [13. REST Constraints](#-13-rest-constraints)
* [14. REST vs RESTful](#️-14-rest-vs-restful)
* [15. Web Services](#-15-web-services)
* [16. API vs Web Service](#-16-api-vs-web-service)
* [17. API Documentation](#-17-api-documentation)
* [18. DevOps Examples](#️-18-devops-examples)
* [19. API‑Driven Architecture](#-19-api-driven-architecture)

---

# 📌 1. What is an API

**API (Application Programming Interface)** is a set of rules that allows **two software systems to communicate with each other.**

| Step          | Description                    |
| ------------- | ------------------------------ |
| 📥 Request    | Client asks for data or action |
| ⚙️ Processing | Server processes request       |
| 📤 Response   | Server returns result          |

### 🧠 Simple Idea

```
Application A → API → Application B
```

### Example

```http
GET /weather/delhi
```

```json
{
 "city": "Delhi",
 "temperature": 33
}
```

💡 **Tip:** APIs allow **automation, integrations, and microservices communication**.

---

# ❓ 2. Why APIs Are Required

| Problem Without APIs ❌ | Solution With APIs ✅       |
| ---------------------- | -------------------------- |
| Direct DB access       | Controlled interface       |
| Security risks         | Authentication             |
| Tight coupling         | Decoupled systems          |
| Difficult automation   | Programmatic communication |

### Architecture Example

```
Payment App
     ↓
   Bank API
     ↓
 Bank Server
     ↓
  Database
```

---

# 🖥️ 3. UI vs API

| Feature      | UI           | API          |
| ------------ | ------------ | ------------ |
| Designed for | Humans       | Programs     |
| Interaction  | Manual       | Automated    |
| Interface    | Graphical    | Programmatic |
| Example      | Click button | POST /users  |

### Example

UI

```
Browser → Click "Create User"
```

API

```http
POST /users
```

---

# 💻 4. Commands vs APIs

| Feature   | Command       | API             |
| --------- | ------------- | --------------- |
| Execution | Shell command | Network request |
| Access    | Local / SSH   | Remote service  |
| Output    | Text          | JSON/XML        |

### Command Example

```bash
systemctl restart nginx
```

### API Equivalent

```http
POST /services/nginx/restart
```

⚠️ **Important**

Most CLI tools internally **call APIs**.

Example

```bash
kubectl get pods
```

Internally calls

```http
GET /api/v1/pods
```

---

# 🔄 5. API Communication Flow

```
Client
   ↓
HTTP Request
   ↓
API Server
   ↓
Application Logic
   ↓
Database
   ↓
JSON Response
```

---

# 🧩 6. Core API Components

| Component         | Description          | Example       |
| ----------------- | -------------------- | ------------- |
| 🔗 Endpoint       | URL where API exists | /users        |
| ⚡ Method          | Action to perform    | GET, POST     |
| 📥 Request        | Data sent by client  | JSON body     |
| 📤 Response       | Data returned        | JSON/XML      |
| 🔐 Authentication | Access control       | API Key / JWT |

---

# 📦 7. Resource Concept

In REST APIs everything is treated as a **resource**.

| Resource        | Example Endpoint |
| --------------- | ---------------- |
| Users           | /users           |
| Orders          | /orders          |
| Products        | /products        |
| Kubernetes Pods | /pods            |

Specific resource

```
/users/101
```

---

# 🔧 8. HTTP Methods

| Method | Purpose         | Example           |
| ------ | --------------- | ----------------- |
| GET    | Retrieve data   | GET /users        |
| POST   | Create resource | POST /users       |
| PUT    | Update resource | PUT /users/101    |
| PATCH  | Partial update  | PATCH /users/101  |
| DELETE | Remove resource | DELETE /users/101 |

---

# 📄 9. API Response Formats

| Format | Usage                        |
| ------ | ---------------------------- |
| JSON   | Most modern APIs             |
| XML    | Older enterprise systems     |
| Binary | High‑performance APIs (gRPC) |

Example

```json
{
 "id": 101,
 "name": "Mukesh"
}
```

---

# 🌍 10. Types of APIs

| Type         | Description                 | Example           |
| ------------ | --------------------------- | ----------------- |
| 🌐 Public    | Open to external developers | GitHub API        |
| 🏢 Private   | Internal company APIs       | Microservices     |
| 🤝 Partner   | Shared with partners        | Amazon Seller API |
| 🧩 Composite | Combines multiple APIs      | /user-dashboard   |

---

# 🏗️ 11. API Architectural Styles

| Style     | Description             | Used In             |
| --------- | ----------------------- | ------------------- |
| REST      | HTTP based APIs         | Web services        |
| SOAP      | XML protocol            | Enterprise apps     |
| GraphQL   | Flexible data queries   | GitHub              |
| RPC       | Remote procedure calls  | Distributed systems |
| WebSocket | Real‑time communication | Chat apps           |

---

# 🌐 12. What is REST

**REST (Representational State Transfer)** is an **architectural style for designing APIs using HTTP.**

Resources example

```
/users
/users/101
/orders
```

Request

```http
GET /users/101
```

Response

```json
{
 "id": 101,
 "name": "Mukesh"
}
```

---

# 📐 13. REST Constraints

| Constraint        | Description                     |
| ----------------- | ------------------------------- |
| Client‑Server     | Separation of client and server |
| Stateless         | Each request independent        |
| Cacheable         | Responses may be cached         |
| Uniform Interface | Consistent API structure        |
| Layered System    | Multiple architecture layers    |
| Code on Demand    | Optional executable code        |

Architecture example

```
Client → API Gateway → Services → Database
```

---

# ⚖️ 14. REST vs RESTful

| REST                | RESTful                   |
| ------------------- | ------------------------- |
| Architectural style | Implementation of REST    |
| Design guidelines   | API following those rules |

Example RESTful endpoints

```
GET /users
GET /users/101
POST /users
DELETE /users/101
```

---

# 🌍 15. Web Services

A **web service** is a service accessible **over the internet using web protocols.**

Example

```http
GET https://api.weather.com/current
```

---

# 🔗 16. API vs Web Service

| Feature          | API                        | Web Service             |
| ---------------- | -------------------------- | ----------------------- |
| Definition       | Interface between software | API exposed via network |
| Network required | Not always                 | Always                  |
| Examples         | OS API, Library API        | REST, SOAP              |

Relationship

```
API
 ├ Local APIs
 └ Web APIs
      ├ REST
      └ SOAP
```

✅ **All web services are APIs, but not all APIs are web services.**

---

# 📚 17. API Documentation

| Tool    | Purpose           |
| ------- | ----------------- |
| Swagger | API documentation |
| OpenAPI | API specification |
| Postman | API testing       |

Documentation includes

* Endpoints
* Request format
* Response format
* Authentication

---

# ⚙️ 18. DevOps Examples

| Tool       | Command               | API Used          |
| ---------- | --------------------- | ----------------- |
| AWS CLI    | aws ec2 run-instances | AWS APIs          |
| Kubernetes | kubectl get pods      | Kubernetes API    |
| Docker     | docker run nginx      | Docker Engine API |

Example Kubernetes call

```http
GET /api/v1/pods
```

---

# 🏗️ 19. API‑Driven Architecture

Modern platforms follow **API‑first architecture**.

```
UI
CLI
Mobile Apps
Automation Tools
        ↓
       APIs
        ↓
Backend Services
        ↓
Database
```

---

# 🎯 Key Takeaways

| Concept      | Summary                               |
| ------------ | ------------------------------------- |
| API          | Enables communication between systems |
| REST         | Architectural style                   |
| RESTful      | Implementation of REST                |
| Web Service  | API accessible over network           |
| Modern Cloud | Fully API driven                      |

---

⭐ **If this guide helped you, consider starring the repository!**
