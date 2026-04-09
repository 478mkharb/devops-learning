# OT-Microservices — Architecture, Technology Decisions & Interview Reference

---

## 1. Overview

OT-Microservices is a distributed employee management system built using a **microservices architecture**. Each service is independently developed, deployed, and scaled.

### Services

| Service        | Language | Responsibility           |
| -------------- | -------- | ------------------------ |
| Employee API   | Go       | Employee data management |
| Attendance API | Python   | Attendance tracking      |
| Salary API     | Java     | Salary processing        |
| Frontend       | React    | User interface           |
| NGINX          | Native   | Reverse proxy & routing  |

---

## 2. Architecture Principles

### Microservices Architecture

* Independent services
* Independent deployment
* Fault isolation

### Database per Service

* Each service owns its database
* No shared schema
* Avoids tight coupling

### Polyglot Design

* Different technologies used per service
* Chosen based on use case

---

## 3. Technology Decisions

---

### 3.1 Go (Employee API)

**Purpose:** High-performance API for employee data

**Why used:**

* Efficient concurrency model
* Low memory usage
* Fast execution

**Why not alternatives:**

* Python: slower for high-throughput workloads
* Java: higher resource consumption for simple API

---

### 3.2 Python + Flask + Gunicorn (Attendance API)

**Purpose:** Lightweight service for attendance management

**Why used:**

* Rapid development
* Simple CRUD operations
* Easy maintainability

**Why Gunicorn:**

* Production-ready WSGI server
* Supports multiple worker processes

**Why not alternatives:**

* Flask dev server: not production-safe
* Java/Go: unnecessary complexity for simple service

---

### 3.3 Java + Spring Boot (Salary API)

**Purpose:** Structured service for salary processing

**Why used:**

* Layered architecture (Controller, Service, Repository)
* Strong ecosystem
* Enterprise-ready framework

**Why not alternatives:**

* Python: lacks strict structure for complex logic
* Go: limited built-in enterprise patterns

---

### 3.4 React (Frontend)

**Purpose:** User interface and data presentation

**Why used:**

* Component-based architecture
* Efficient rendering
* Widely adopted ecosystem

**Special role:**

* Performs data aggregation across services

---

### 3.5 NGINX (Reverse Proxy)

**Purpose:** Single entry point for all requests

**Responsibilities:**

* Route API requests
* Serve frontend static files
* Hide backend ports

**Why used:**

* High performance
* Lightweight
* Reliable for production

**Why not alternatives:**

* Apache: higher overhead
* Node-based proxy: not ideal for production routing

---

### 3.6 PostgreSQL (Attendance Database)

**Purpose:** Store attendance records

**Why used:**

* Relational data handling
* Strong query capabilities (filtering, aggregation)
* ACID compliance

**Why not alternatives:**

* NoSQL databases: not suitable for relational queries

---

### 3.7 ScyllaDB (Employee & Salary Databases)

**Purpose:** High-performance data storage

**Why used:**

* Optimized for high read throughput
* Low latency
* Cassandra-compatible

**Why not alternatives:**

* PostgreSQL: less efficient at scale for read-heavy workloads

---

### 3.8 Redis (Caching)

**Purpose:** Improve performance

**Why used:**

* In-memory data store
* Reduces database load
* Improves response time

**Why not alternatives:**

* Memcached: limited functionality compared to Redis

---

### 3.9 Swagger (API Documentation)

**Purpose:** API documentation and testing

**Why used:**

* Standardized documentation
* Interactive API testing

**Why not alternatives:**

* Postman: not suitable for shared documentation

---

## 4. Request Flow

```text
Client → NGINX → Service → Cache (Redis) → Database
```

---

## 5. Key Design Decisions

* Services communicate via REST APIs
* No direct database sharing
* Frontend aggregates data across services
* NGINX handles routing instead of a full API Gateway

---

## 6. Interview Reference (Detailed)

### System Context

**Services:**

* Employee API (Go, :8080)
* Attendance API (Python + Flask + Gunicorn, :8000/8081)
* Salary API (Java + Spring Boot, :8082)
* Frontend (React, served via NGINX)
* NGINX (reverse proxy, :80)

**Databases & Cache:**

* ScyllaDB (Cassandra-compatible) → Employee, Salary
* PostgreSQL → Attendance
* Redis → Caching layer for APIs

**API Documentation:**

* Swagger (OpenAPI) → Used across services for API documentation and testing

---

### Q1: Why microservices over monolith?

**Answer:**

* Independent deployability: each service can be released without impacting others
* Independent scalability: read-heavy (employee) vs write-heavy (attendance) can scale differently
* Fault isolation: failure in one service (e.g., attendance) does not bring down salary or employee services
* Technology flexibility: enables polyglot stack selection per service

---

### Q2: Why different languages for each service?

**Answer:**

* Employee API (Go): optimized for high-concurrency, low-latency read operations
* Attendance API (Python): simple CRUD with rapid development and minimal complexity
* Salary API (Java): requires structured, layered architecture for business logic (validation, processing)

This approach follows a **polyglot architecture**, allowing the best-suited language per use case rather than enforcing a single stack.

---

### Q3: Why Redis is used and where?

**Answer:**

* Used as a caching layer in front of databases
* Reduces repeated database reads for frequently accessed endpoints (e.g., employee search)
* Improves latency significantly (memory vs disk access)

**Flow:**
Client → API → Redis (cache hit) → Response
Client → API → Redis (miss) → DB → Redis → Response

---

### Q4: Why PostgreSQL for Attendance API?

**Answer:**

* Attendance data is relational (id, date, status)
* Requires filtering and aggregation (e.g., attendance by date, status)
* PostgreSQL provides strong ACID guarantees and SQL capabilities

**Why not ScyllaDB:**

* No joins or flexible querying
* Not suited for relational queries

---

### Q5: Why ScyllaDB for Employee and Salary APIs?

**Answer:**

* Designed for high-throughput, low-latency workloads
* Optimized for key-based lookups (employee by ID)
* Horizontally scalable

**Trade-off:**

* No joins → data must be aggregated at application (frontend/API) level

---

### Q6: Why Gunicorn is required for Attendance API?

**Answer:**

* Flask development server is single-threaded and not production-ready
* Gunicorn runs multiple worker processes to handle concurrent requests
* Provides process management, better performance, and stability

---

### Q7: Why NGINX is used instead of exposing services directly?

**Answer:**

* Acts as a reverse proxy and single entry point
* Routes requests based on path (/api/v1/employee, /attendance, /salary)
* Hides internal service ports (8080, 8000, 8082)
* Serves static frontend files
* Reduces CORS issues by unifying origin

---

### Q8: How do services communicate?

**Answer:**

* Services are loosely coupled and communicate via REST APIs over HTTP
* No direct database access across services
* Each service owns its data

---

### Q9: Why frontend performs data aggregation?

**Answer:**

* No shared database and no joins across services
* No API Gateway implemented in this setup
* Frontend calls multiple APIs (employee + salary) and merges data

---

### Q10: What are potential bottlenecks in this system?

**Answer:**

* Database latency (ScyllaDB/PostgreSQL)
* Network overhead between services
* Cache misses (Redis)
* NGINX misconfiguration or single-node limitation

---

### Q11: How can this architecture be improved?

**Answer:**

* Introduce API Gateway (Kong / Spring Cloud Gateway)
* Add asynchronous communication (Kafka)
* Implement centralized logging and monitoring
* Containerize services and deploy via Kubernetes

---

## 7. Summary

The system demonstrates:

* Microservices architecture
* Polyglot design
* Reverse proxy pattern
* Database-per-service model

---

End of document
