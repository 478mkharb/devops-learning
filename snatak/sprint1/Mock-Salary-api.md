# Mock Questions & Answers – Salary API (OT-Microservices)

---

## 1. Explain the complete architecture flow of the Salary API.

### Answer

The Salary API is a Java Spring Boot-based microservice responsible for salary processing, salary calculations, and payroll-related operations.

The request flow is:

```text
Frontend / Client
        ↓
NGINX / Reverse Proxy
        ↓
Salary API (Spring Boot)
        ↓
Controllers & Services
        ↓
Redis Cache Lookup
   ↙                 ↘
Cache Hit         Cache Miss
   ↓                  ↓
Fast Response      ScyllaDB Query
                        ↓
                   Salary Records
                        ↓
                   JSON Response
```

ScyllaDB is used as the primary distributed database, while Redis is used for caching frequently accessed salary records.

---

## 2. In which programming language is the Salary API written and what are its main dependencies?

### Answer

The Salary API is developed using Java and the Spring Boot framework.

Java is preferred because it provides:

* Enterprise-grade reliability
* Strong multithreading support
* Mature ecosystem
* Excellent scalability
* Large enterprise adoption
* Strong dependency injection support

Main dependencies used in the Salary API:

| Dependency      | Purpose                              |
| --------------- | ------------------------------------ |
| Spring Boot     | Backend framework                    |
| Maven           | Dependency management and build tool |
| ScyllaDB Driver | Database connectivity                |
| Redis Client    | Cache integration                    |
| Jackson         | JSON serialization/deserialization   |
| Lombok          | Boilerplate code reduction           |
| Spring Web      | REST API development                 |
| Spring Data     | Data access abstraction              |

Dependencies are managed using:

```text
pom.xml
```

---

## 3. Explain how the Salary API executes internally.

### Answer

The Salary API executes as a Java Spring Boot application.

Execution lifecycle:

1. JVM starts the application
2. Spring Boot initializes the application context
3. Configuration files are loaded
4. Dependency injection initializes components
5. Database and Redis connections are established
6. REST controllers are initialized
7. Application starts listening on configured port
8. Requests are processed through services and repositories
9. JSON responses are returned to clients

The application is built using:

```bash
mvn clean install -DskipTests
```

The generated JAR artifact is then executed.

Spring Boot internally manages:

* Dependency injection
* Bean lifecycle
* Embedded server startup
* Request routing
* Exception handling

---

## 4. Explain the complete request lifecycle inside the Salary API.

### Answer

The Salary API follows a layered request-processing architecture.

Flow:

```text
Client Request
      ↓
HTTP Route
      ↓
Controller Layer
      ↓
Service Layer
      ↓
Repository Layer
      ↓
ScyllaDB / Redis Access
      ↓
Business Logic Processing
      ↓
JSON Response
```

Detailed lifecycle:

1. Client sends HTTP request
2. Spring controller maps endpoint
3. Request validation occurs
4. Service layer executes salary logic
5. Repository layer interacts with database
6. Redis cache may serve repeated requests
7. Salary response is generated
8. JSON response is returned

This layered architecture improves:

* Maintainability
* Scalability
* Code separation
* Testing capability
* Microservice modularity

---

## 5. Explain why ScyllaDB is used in the Salary API instead of PostgreSQL or MongoDB.

### Answer

ScyllaDB is used because the Salary API requires distributed scalability, high-throughput operations, and efficient handling of large payroll datasets.

Salary systems may process:

* Large employee salary records
* Concurrent payroll operations
* Bulk salary generation
* Distributed API access
* High-volume financial transactions

ScyllaDB provides:

| Feature                  | Benefit                             |
| ------------------------ | ----------------------------------- |
| Distributed Architecture | Horizontal scalability              |
| High Throughput          | Handles large request volumes       |
| Low Latency              | Faster read/write performance       |
| Fault Tolerance          | High availability                   |
| Cassandra Compatibility  | Distributed NoSQL ecosystem support |

---

### Compared to PostgreSQL

PostgreSQL is excellent for relational transactional systems.

However the Salary API requires:

* Massive scalability
* Distributed workload handling
* High-performance concurrent processing

ScyllaDB handles these workloads more efficiently in distributed environments.

---

### Compared to MongoDB

MongoDB is document-oriented and schema-flexible.

However ScyllaDB provides:

* Better distributed write optimization
* Predictable low-latency performance
* Better large-scale distributed architecture handling

---

## 6. Explain how configuration management works in the Salary API.

### Answer

The Salary API uses configuration files for environment-specific settings.

Main configuration file:

```text
salary-api/src/main/resources/application.yml
```

This file stores:

* Database configuration
* Redis configuration
* Ports
* Application settings
* Logging settings

Example:

```yaml
spring:
  data:
    cassandra:
      contact-points: 127.0.0.1
```

Benefits of configuration management:

* Environment separation
* Easier deployment
* Avoids hardcoded values
* Supports DevOps automation
* Improves maintainability

In production, configurations may also be managed using:

* Environment variables
* Kubernetes ConfigMaps
* Jenkins secrets
* Vault systems

---

## 7. Explain why artifact generation is important in the Salary API.

### Answer

The Salary API generates a deployable JAR artifact.

Build command:

```bash
mvn clean install -DskipTests
```

Artifact generation is important because:

* Standardizes deployments
* Creates reusable build artifacts
* Supports CI/CD pipelines
* Simplifies production deployment
* Ensures build consistency

Generated artifacts contain:

* Compiled Java classes
* Dependencies
* Spring Boot runtime configuration

The generated JAR file becomes the deployable unit for the Salary API.

---

## 8. Explain how migrations are executed in the Salary API.

### Answer

The Salary API uses migration scripts to initialize and manage ScyllaDB schema changes.

Migration execution ensures:

* Tables are created automatically
* Schema changes remain consistent
* CI/CD deployments remain standardized
* Manual database setup is minimized

Migration configurations are generally managed using:

```text
migration.json
```

Migrations are important because they:

* Standardize database initialization
* Improve deployment automation
* Reduce operational errors
* Maintain schema consistency across environments

---

## 9. Explain the role of `main class`, `pom.xml`, and `migration.json` files in the Salary API.

### Answer

These files are critical components of the Salary API project.

---

### 1. Main Class

The main class is the application entry point.

Responsibilities:

* Starts Spring Boot
* Initializes application context
* Loads configurations
* Starts embedded server
* Initializes components and services

Execution begins from:

```java
SpringApplication.run()
```

---

### 2. pom.xml

`pom.xml` is the primary Maven configuration file.

It stores:

* Project metadata
* Dependencies
* Plugin configuration
* Build settings
* Java version requirements

Purpose:

* Dependency management
* Build automation
* CI/CD consistency
* Plugin execution

---

### 3. migration.json

`migration.json` stores migration-related database configuration.

Responsibilities:

* Database connectivity settings
* Keyspace configuration
* Migration execution configuration
* Schema initialization settings

It helps automate schema management inside the Salary API project.

---

## 10. Explain why Java Spring Boot is preferred for the Salary API.

### Answer

Spring Boot is preferred because it simplifies enterprise backend development and provides production-ready features.

Advantages:

| Feature              | Benefit                               |
| -------------------- | ------------------------------------- |
| Dependency Injection | Loose coupling                        |
| Embedded Server      | Simplified deployment                 |
| Spring MVC           | REST API support                      |
| Auto Configuration   | Reduced manual setup                  |
| Enterprise Ecosystem | Large enterprise adoption             |
| Security Support     | Enterprise-grade security integration |

Spring Boot is highly suitable for salary and payroll systems because such systems require:

* Enterprise scalability
* Reliable transaction handling
* Strong backend architecture
* Maintainable code structure
* Production-grade operational stability

Its mature ecosystem and enterprise tooling make it an excellent choice for large-scale payroll microservices.
