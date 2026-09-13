# API Interview Notes

## Table of Contents

1. [Introduction](#1-introduction)
2. [What is an API?](#2-what-is-an-api)
3. [Why Are APIs Used?](#3-why-are-apis-used)
4. [API vs UI vs CLI](#4-api-vs-ui-vs-cli)
5. [API vs Web Service](#5-api-vs-web-service)
6. [How Does an API Work?](#6-how-does-an-api-work)
7. [Core API Components](#7-core-api-components)
8. [Resource Concept](#8-resource-concept)
9. [Types of APIs](#9-types-of-apis)
10. [API Architectural Styles](#10-api-architectural-styles)
11. [What is REST API?](#11-what-is-rest-api)
12. [REST Constraints](#12-rest-constraints)
13. [REST vs RESTful](#13-rest-vs-restful)
14. [HTTP Methods](#14-http-methods)
15. [HTTP Status Codes](#15-http-status-codes)
16. [Request and Response](#16-request-and-response)
17. [HTTP Headers](#17-http-headers)
18. [Path Parameters, Query Parameters, and Request Body](#18-path-parameters-query-parameters-and-request-body)
19. [Authentication and Authorization](#19-authentication-and-authorization)
20. [API Data Formats](#20-api-data-formats)
21. [Statelessness](#21-statelessness)
22. [API Versioning](#22-api-versioning)
23. [API Security](#23-api-security)
24. [API Documentation — OpenAPI and Swagger](#24-api-documentation--openapi-and-swagger)
25. [API Testing](#25-api-testing)
26. [REST vs GraphQL vs gRPC vs WebSocket](#26-rest-vs-graphql-vs-grpc-vs-websocket)
27. [DevOps APIs](#27-devops-apis)
28. [API Monitoring and Observability](#28-api-monitoring-and-observability)
29. [Common API Commands](#29-common-api-commands)
30. [Frequently Asked Interview Questions](#30-frequently-asked-interview-questions)
31. [APIs in OT-Microservices](#31-apis-in-ot-microservices)

---

# 1. Introduction

An **API (Application Programming Interface)** is a defined interface through which one software component communicates with another.

APIs allow applications to:

* Exchange data
* Request services
* Execute operations
* Integrate different systems
* Communicate between microservices

Example:

```text
Frontend
    │
    │ HTTP Request
    ▼
Employee API
    │
    ▼
ScyllaDB
```

The frontend does not need to know how the Employee API stores data internally.

It only needs to follow the API contract.

---

# 2. What is an API?

**API** stands for **Application Programming Interface**.

An API acts as a contract between a client and a service.

For example:

```http
GET /employees/101
```

The client is requesting employee `101`.

The server may return:

```json
{
  "employee_id": 101,
  "name": "Mukesh",
  "department": "DevOps"
}
```

The client does not need to know:

* Which programming language the API uses
* Which database is behind it
* How the business logic is implemented
* Where the application is physically running

It only needs to know the API contract.

---

# 3. Why Are APIs Used?

## Integration

Different applications can communicate through APIs.

```text
Application A
      │
      ▼
     API
      │
      ▼
Application B
```

## Abstraction

The internal implementation remains hidden from the client.

```text
Frontend
   │
   ▼
Employee API
   │
   ▼
ScyllaDB
```

The frontend does not directly access ScyllaDB.

## Reusability

The same API can serve different clients.

```text
Web App ──────┐
              │
Mobile App ───┼──► API
              │
CLI ──────────┘
```

## Microservices Communication

APIs allow independent services to communicate.

```text
Employee API
      │
      ▼
Salary API
      │
      ▼
Notification Service
```

---

# 4. API vs UI vs CLI

These are different ways of interacting with software.

## UI

A **User Interface** is primarily designed for human interaction.

```text
Human
  │
  ▼
Web UI
  │
  ▼
Application
```

Example:

```text
Click "Create Employee"
```

## API

An API is designed for programmatic communication.

```text
Program
  │
  ▼
API
  │
  ▼
Application
```

Example:

```http
POST /employees
```

## CLI

A **Command-Line Interface** allows users or scripts to operate a system through commands.

Example:

```bash
kubectl get pods
```

```bash
aws ec2 describe-instances
```

Many modern CLIs act as clients of APIs.

Conceptually:

```text
CLI
 │
 ▼
API
 │
 ▼
Service
```

This is why a DevOps engineer often interacts with APIs without directly writing HTTP requests.

---

# 5. API vs Web Service

An API is a broader concept than a web service.

An API can exist without HTTP.

Examples:

```text
Programming Library API
Operating System API
Database API
Web API
```

A **Web API** is an API exposed over web technologies such as HTTP.

A simplified relationship is:

```text
API
├── Local / Library APIs
└── Web APIs
     ├── REST
     ├── SOAP
     ├── GraphQL
     └── Other HTTP-based APIs
```

Therefore:

> Every web service exposes an API, but not every API is a web service.

---

# 6. How Does an API Work?

Consider:

```http
GET /employees/101
```

The basic flow is:

```text
Client
   │
   │ HTTP Request
   ▼
API Server
   │
   │ Authentication
   │ Validation
   │ Business Logic
   ▼
Database / Cache / External Service
   │
   ▼
API Server
   │
   │ HTTP Response
   ▼
Client
```

## Step 1: Client Sends Request

```http
GET /employees/101 HTTP/1.1
Host: example.com
```

## Step 2: API Receives Request

The server processes:

* HTTP method
* URL
* Headers
* Authentication information
* Parameters
* Request body

## Step 3: Business Logic

Example:

```text
Validate employee ID
        ↓
Authenticate caller
        ↓
Authorize operation
        ↓
Query database
        ↓
Build response
```

## Step 4: API Sends Response

```json
{
  "employee_id": 101,
  "name": "Mukesh",
  "department": "DevOps"
}
```

---

# 7. Core API Components

A typical HTTP API consists of:

```text
API
├── Endpoint / URL
├── HTTP Method
├── Headers
├── Parameters
├── Request Body
├── Authentication
├── Business Logic
├── Response
└── Status Code
```

Example:

```http
POST /employees?notify=true HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer <token>

{
  "name": "Mukesh",
  "department": "DevOps"
}
```

Here:

```text
POST                    → HTTP Method
/employees              → Endpoint / Path
notify=true             → Query Parameter
Content-Type            → Header
Authorization           → Header
JSON                     → Request Body
```

---

# 8. Resource Concept

REST APIs are commonly designed around **resources**.

A resource represents an entity such as:

* Employee
* Attendance record
* Salary
* Order
* Customer
* Product

Examples:

```text
/employees
/attendance
/salaries
/orders
/customers
```

A specific resource can be identified by an ID:

```text
/employees/101
/salaries/5001
/orders/2001
```

The HTTP method defines the operation.

```text
GET    /employees/101
PUT    /employees/101
PATCH  /employees/101
DELETE /employees/101
```

---

# 9. Types of APIs

APIs can also be classified by who is allowed to use them.

## Public API

A public API is exposed for external consumers.

```text
External Developers
        │
        ▼
     Public API
```

Examples include APIs offered by payment, mapping, or communication providers.

## Private API

A private API is intended for use inside an organization.

```text
Internal Service A
        │
        ▼
    Private API
        │
        ▼
Internal Service B
```

Microservice-to-microservice APIs are often private APIs.

## Partner API

A partner API is exposed to approved external organizations.

```text
Company A
    │
    ▼
Partner API
    │
    ▼
Company B
```

Access is generally controlled through authentication and authorization.

## Composite API

A composite API combines multiple backend operations into a single client-facing request.

```text
Client
  │
  ▼
Composite API
  │
  ├── Employee Service
  ├── Attendance Service
  └── Salary Service
```

This can reduce the number of network calls required by the client.

---

# 10. API Architectural Styles

Common API approaches include:

* REST
* SOAP
* GraphQL
* gRPC
* WebSocket

They solve different problems.

```text
REST      → Resource-oriented HTTP APIs
SOAP      → Formal XML-based messaging
GraphQL   → Client-defined data queries
gRPC      → High-performance RPC
WebSocket → Long-lived real-time communication
```

---

# 11. What is REST API?

**REST** stands for **Representational State Transfer**.

REST is an architectural style for designing networked applications.

REST APIs commonly use:

* HTTP
* Resources
* HTTP methods
* HTTP status codes
* JSON
* Stateless communication

Example:

```http
GET /employees/101
```

```http
POST /employees
```

```http
PUT /employees/101
```

```http
DELETE /employees/101
```

---

# 12. REST Constraints

REST is based on architectural constraints.

## Client-Server

Client and server responsibilities are separated.

```text
Client
  │
  ▼
Server
```

## Stateless

Each request contains the information needed to process it.

## Cacheable

Responses can be cached when appropriate.

## Uniform Interface

Clients interact with resources through a consistent interface.

## Layered System

Intermediate components may exist between the client and server.

```text
Client
   │
   ▼
Load Balancer
   │
   ▼
API Gateway
   │
   ▼
Application
```

## Code-on-Demand

REST permits executable code to be sent from server to client as an optional constraint. It is not commonly required by typical REST APIs.

---

# 13. REST vs RESTful

These terms are related but not exactly identical.

## REST

REST is the **architectural style**.

It defines constraints and principles for designing networked systems.

## RESTful

An API is commonly called **RESTful** when its design follows REST principles.

Example:

```text
GET    /employees
GET    /employees/101
POST   /employees
PUT    /employees/101
DELETE /employees/101
```

A useful interview statement is:

> REST is the architectural style; RESTful describes an API designed according to REST principles.

---

# 14. HTTP Methods

HTTP methods communicate the intended operation.

| Method | Purpose | Example |
|---|---|---|
| GET | Read data | `GET /employees/101` |
| POST | Create a resource or submit an operation | `POST /employees` |
| PUT | Replace a resource | `PUT /employees/101` |
| PATCH | Partially update a resource | `PATCH /employees/101` |
| DELETE | Delete a resource | `DELETE /employees/101` |
| HEAD | Get headers without the response body | `HEAD /employees/101` |
| OPTIONS | Discover supported communication options | `OPTIONS /employees` |

## GET

```http
GET /employees/101
```

Used to retrieve a representation.

## POST

```http
POST /employees
```

Example:

```json
{
  "name": "Mukesh",
  "department": "DevOps"
}
```

## PUT

```http
PUT /employees/101
```

Generally used to replace the selected resource representation.

## PATCH

```http
PATCH /employees/101
```

Generally used for a partial modification.

## DELETE

```http
DELETE /employees/101
```

Used to request deletion of a resource.

---

# 15. HTTP Status Codes

## 2xx — Success

### 200 OK

Request succeeded.

### 201 Created

A new resource was created.

### 202 Accepted

The request was accepted for processing but may not have completed.

### 204 No Content

Request succeeded and there is no response body.

---

## 3xx — Redirection

### 301 Moved Permanently

The resource has a permanent new location.

### 302 Found

Temporary redirection.

### 304 Not Modified

The cached representation can be reused.

---

## 4xx — Client Errors

### 400 Bad Request

Request is invalid or malformed.

### 401 Unauthorized

Authentication is missing or invalid.

### 403 Forbidden

The caller is authenticated but lacks permission.

### 404 Not Found

Requested resource was not found.

### 405 Method Not Allowed

HTTP method is not supported for the resource.

### 409 Conflict

Request conflicts with the current resource state.

### 429 Too Many Requests

The client has exceeded a rate limit.

---

## 5xx — Server Errors

### 500 Internal Server Error

Generic server-side failure.

### 502 Bad Gateway

A gateway or proxy received an invalid response from an upstream service.

### 503 Service Unavailable

The service is temporarily unable to handle the request.

### 504 Gateway Timeout

A gateway or proxy did not receive a timely response from an upstream service.

---

# 16. Request and Response

An HTTP request commonly contains:

```text
Request
├── Method
├── URL
├── Headers
├── Parameters
└── Body
```

Example:

```http
POST /employees HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer <token>

{
  "name": "Mukesh",
  "department": "DevOps"
}
```

A response commonly contains:

```text
Response
├── Status Code
├── Headers
└── Body
```

Example:

```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "employee_id": 101,
  "name": "Mukesh",
  "department": "DevOps"
}
```

---

# 17. HTTP Headers

Headers carry metadata about the request or response.

## Content-Type

Describes the request or response body format.

```http
Content-Type: application/json
```

## Accept

Specifies acceptable response formats.

```http
Accept: application/json
```

## Authorization

Carries authentication credentials such as a bearer token.

```http
Authorization: Bearer <token>
```

## User-Agent

Identifies the client software.

```http
User-Agent: curl/8.x
```

## Cache-Control

Controls caching behavior.

```http
Cache-Control: no-cache
```

## Location

Often identifies a newly created or redirected resource.

```http
Location: /employees/101
```

---

# 18. Path Parameters, Query Parameters, and Request Body

## Path Parameter

Identifies a particular resource.

```http
GET /employees/101
```

Here:

```text
101 = Path Parameter
```

## Query Parameter

Used for filtering, sorting, pagination, or optional request controls.

```http
GET /employees?department=DevOps
```

Multiple parameters:

```http
GET /employees?department=DevOps&page=2&size=20
```

## Request Body

Carries data submitted to the server.

```http
POST /employees
```

```json
{
  "name": "Mukesh",
  "department": "DevOps"
}
```

### Comparison

| Type | Example | Common Purpose |
|---|---|---|
| Path Parameter | `/employees/101` | Identify a resource |
| Query Parameter | `/employees?page=2` | Filter, sort, paginate, or modify request behavior |
| Request Body | JSON object | Submit resource data |

---

# 19. Authentication and Authorization

These are different concepts.

## Authentication

Authentication answers:

> Who are you?

Examples:

* Username/password
* API key
* JWT
* OAuth 2.0
* Mutual TLS

## Authorization

Authorization answers:

> What are you allowed to do?

Example:

```text
User
 └── Read employee

Admin
 ├── Read employee
 ├── Create employee
 ├── Update employee
 └── Delete employee
```

## JWT

JWT stands for **JSON Web Token**.

A simplified flow:

```text
Client
  │
  │ Login
  ▼
Authentication Service
  │
  │ JWT
  ▼
Client
  │
  │ Authorization: Bearer <JWT>
  ▼
API
```

The API validates the token before processing a protected request.

---

# 20. API Data Formats

APIs can exchange data using different formats.

Common formats include:

* JSON
* XML
* Form data
* Plain text
* Protocol Buffers

## JSON

```json
{
  "employee_id": 101,
  "name": "Mukesh",
  "department": "DevOps"
}
```

JSON is very common for REST APIs because it is:

* Human-readable
* Lightweight
* Easy to parse
* Supported by many languages

## XML

```xml
<employee>
    <employee_id>101</employee_id>
    <name>Mukesh</name>
    <department>DevOps</department>
</employee>
```

---

# 21. Statelessness

Statelessness is an important REST constraint.

A stateless API does not depend on stored conversational session state on the server to understand each individual request.

For example:

```http
GET /employees/101
Authorization: Bearer <token>
```

The request contains the information needed to authenticate and process it.

This makes horizontal scaling easier:

```text
             Load Balancer
                  │
         ┌────────┼────────┐
         ▼        ▼        ▼
      API 1     API 2     API 3
```

A request can be routed to any suitable instance.

---

# 22. API Versioning

API versioning allows a service to evolve without unexpectedly breaking existing clients.

Example:

```text
/api/v1/employees
/api/v2/employees
```

Suppose v1 returns:

```json
{
  "name": "Mukesh"
}
```

While v2 returns:

```json
{
  "first_name": "Mukesh",
  "department": "DevOps"
}
```

An existing client may still depend on the v1 contract.

Common versioning approaches include:

```text
URL:
 /api/v1/employees

Header:
 Accept: application/vnd.example.v1+json

Query Parameter:
 /employees?version=1
```

---

# 23. API Security

Important API security practices include:

## HTTPS

Use TLS encryption.

```text
HTTP  ❌
HTTPS ✅
```

## Authentication

Verify the identity of the caller.

## Authorization

Verify that the caller has permission for the operation.

## Input Validation

Validate request parameters and bodies.

## Rate Limiting

Control excessive traffic.

```text
Client
  │
  ▼
Rate Limiter
  │
  ├── Allowed → API
  └── Too Many → 429
```

## Avoid Secrets in URLs

Do not put passwords, tokens, or other secrets in query strings.

Bad:

```text
/login?password=secret123
```

## CORS

CORS controls, in browsers, whether requests from one origin are permitted to access resources from another origin.

CORS is primarily a browser-side security mechanism; it is not a replacement for authentication or authorization.

---

# 24. API Documentation — OpenAPI and Swagger

## OpenAPI

**OpenAPI** is a specification for describing HTTP APIs.

It can describe:

* Endpoints
* HTTP methods
* Parameters
* Request bodies
* Response structures
* Authentication schemes
* Status codes

Example concept:

```yaml
paths:
  /employees/{id}:
    get:
      parameters:
        - name: id
          in: path
          required: true
```

## Swagger

Swagger is the name commonly associated with a set of tools around the OpenAPI specification.

Common Swagger-related tools include:

* Swagger UI
* Swagger Editor

## Why API Documentation Matters

A documented API provides a common contract between developers, testers, frontend teams, and consumers.

```text
OpenAPI Specification
          │
          ├── Documentation
          ├── Testing
          ├── Client Generation
          └── Server Stubs
```

## Postman

Postman is primarily an API development and testing tool.

It is not the same thing as OpenAPI.

```text
OpenAPI → API specification
Swagger → Tooling around OpenAPI
Postman → API client/testing tool
```

---

# 25. API Testing

APIs can be tested using:

* curl
* Postman
* Insomnia
* Swagger UI
* Automated unit tests
* Integration tests
* Load-testing tools

## Functional Testing

Verify expected behavior.

```http
GET /employees/101
```

Expected:

```http
200 OK
```

## Negative Testing

Send invalid data.

```http
GET /employees/abc
```

The API should return an appropriate error.

## Authentication Testing

Verify protected APIs reject missing or invalid authentication.

## Authorization Testing

Verify a user cannot perform operations beyond their permissions.

## Load Testing

Measure API performance under traffic.

Common tools:

* JMeter
* k6
* Gatling

---

# 26. REST vs GraphQL vs gRPC vs WebSocket

Different API approaches solve different problems.

| Technology | Main Model | Typical Data | Main Strength |
|---|---|---|---|
| REST | Resources | JSON | General-purpose web APIs |
| GraphQL | Queries | JSON | Client selects required data |
| gRPC | Remote Procedure Calls | Protocol Buffers | High-performance service-to-service communication |
| WebSocket | Persistent connection / events | Text or binary | Real-time, bidirectional communication |

## REST

```text
Client
  │
  ▼
HTTP Resource API
```

Good for:

* Web applications
* CRUD operations
* Public APIs
* General microservice APIs

## GraphQL

The client specifies the data it needs.

```graphql
query {
  employee(id: 101) {
    name
    department
  }
}
```

Good when clients need flexible data selection.

## gRPC

Uses RPC-style service definitions and commonly uses Protocol Buffers.

```text
Service A
    │
    │ gRPC
    ▼
Service B
```

Good for internal service-to-service communication where high performance and strongly defined contracts are important.

## WebSocket

Maintains a long-lived connection.

```text
Client ◄──────────────► Server
       persistent
       connection
```

Useful for:

* Chat
* Live notifications
* Real-time dashboards
* Live status updates

---

# 27. DevOps APIs

APIs are fundamental to DevOps tools.

A common mental model is:

```text
CLI / UI / Automation
          │
          ▼
         API
          │
          ▼
       Service
```

## Jenkins API

Jenkins exposes HTTP endpoints that can be used for automation and integration.

Example concept:

```text
Automation
    │
    ▼
Jenkins API
    │
    ▼
Build / Job
```

A Jenkins job can be triggered through its HTTP API.

Example:

```bash
curl -X POST \
http://jenkins.example.com/job/my-job/build
```

Authentication may be required depending on the Jenkins security configuration.

## Kubernetes API

The Kubernetes API server is the central API for the cluster.

```text
kubectl
   │
   ▼
Kubernetes API Server
   │
   ├── Pods
   ├── Deployments
   ├── Services
   └── ConfigMaps
```

When you run:

```bash
kubectl get pods
```

`kubectl` acts as a client of the Kubernetes API server.

The API server validates and processes the request and returns the requested resources.

## AWS API

AWS services expose APIs.

For example:

```bash
aws ec2 describe-instances
```

Conceptually:

```text
AWS CLI
   │
   ▼
AWS EC2 API
   │
   ▼
EC2 Service
```

The AWS CLI is a client for AWS service APIs.

## Terraform and Cloud APIs

Terraform normally does not directly "create AWS resources by magic."

A simplified flow is:

```text
Terraform
    │
    ▼
AWS Provider
    │
    ▼
AWS API
    │
    ▼
AWS Resource
```

For example:

```text
Terraform
    │
    ▼
hashicorp/aws provider
    │
    ▼
EC2 / VPC / S3 APIs
    │
    ▼
AWS Resources
```

This is why provider configuration and API credentials are important in Terraform.

---

# 28. API Monitoring and Observability

APIs should be monitored for availability, performance, and errors.

Important metrics include:

* Request rate
* Error rate
* Response time
* Latency
* Throughput
* Status-code distribution
* Availability
* Saturation

Example:

```text
                API
                 │
        ┌────────┼────────┐
        ▼        ▼        ▼
     Metrics    Logs     Traces
```

For example:

```text
Request Rate       → 500 req/s
Error Rate         → 0.4%
Average Latency    → 80 ms
P95 Latency        → 150 ms
P99 Latency        → 300 ms
```

## DevOps Monitoring Stack

A common setup can be:

```text
API
 │
 ├── Metrics ──► Prometheus ──► Grafana
 │
 ├── Logs ─────► Elasticsearch ──► Kibana
 │
 └── Traces ───► Tracing System
```

Monitoring helps identify:

* Slow endpoints
* Failed requests
* Dependency failures
* Capacity problems
* Traffic spikes

---

# 29. Common API Commands

## GET

```bash
curl http://localhost:8080/employees
```

## GET with Path Parameter

```bash
curl http://localhost:8080/employees/101
```

## GET with Query Parameter

```bash
curl "http://localhost:8080/employees?department=DevOps"
```

## POST

```bash
curl -X POST http://localhost:8080/employees \
-H "Content-Type: application/json" \
-d '{
  "name": "Mukesh",
  "department": "DevOps"
}'
```

## PUT

```bash
curl -X PUT http://localhost:8080/employees/101 \
-H "Content-Type: application/json" \
-d '{
  "name": "Mukesh",
  "department": "Cloud DevOps"
}'
```

## PATCH

```bash
curl -X PATCH http://localhost:8080/employees/101 \
-H "Content-Type: application/json" \
-d '{
  "department": "Platform Engineering"
}'
```

## DELETE

```bash
curl -X DELETE http://localhost:8080/employees/101
```

## Authentication

```bash
curl http://localhost:8080/employees \
-H "Authorization: Bearer <token>"
```

## Show Response Headers

```bash
curl -i http://localhost:8080/employees
```

## Verbose Mode

```bash
curl -v http://localhost:8080/employees
```

---

# 30. Frequently Asked Interview Questions

| Question | Detailed Answer |
|---|---|
| **What is an API?** | An API is an interface or contract that allows software components to communicate with each other. |
| **What does API stand for?** | API stands for Application Programming Interface. |
| **What is the difference between API and UI?** | UI is primarily designed for human interaction, while an API is designed for programmatic communication. |
| **Is every API a web service?** | No. APIs can exist as library, operating system, database, or other interfaces without being web services. |
| **What is a REST API?** | A REST API is an HTTP-based API designed using REST architectural principles. |
| **What is the difference between REST and RESTful?** | REST is the architectural style; RESTful commonly describes an API that follows REST principles. |
| **What is a resource in REST?** | A resource represents a logical entity such as an employee, order, or salary record. |
| **What are the HTTP methods?** | Common methods are GET, POST, PUT, PATCH, DELETE, HEAD, and OPTIONS. |
| **What is the difference between GET and POST?** | GET is normally used to retrieve a representation, while POST commonly creates a resource or submits an operation. |
| **What is the difference between PUT and PATCH?** | PUT is generally used to replace a resource representation, while PATCH is generally used for partial modifications. |
| **What is the difference between 401 and 403?** | 401 indicates missing or invalid authentication; 403 indicates that the caller is authenticated but lacks sufficient permission. |
| **What is 404?** | 404 Not Found means the requested resource could not be found. |
| **What is 500?** | 500 Internal Server Error indicates a generic server-side failure. |
| **What is a path parameter?** | It identifies a resource, such as `/employees/101`. |
| **What is a query parameter?** | It modifies or filters a request, such as `/employees?page=2`. |
| **What is a request body?** | It carries data submitted to the server, commonly JSON. |
| **What is authentication?** | Authentication verifies the identity of the caller. |
| **What is authorization?** | Authorization determines what an authenticated caller is allowed to do. |
| **What is JWT?** | JWT is a token format commonly used to carry signed claims between parties. |
| **What does stateless mean in REST?** | The server does not rely on stored conversational state to interpret an individual request; the request carries the information needed for processing. |
| **Why is HTTPS used?** | HTTPS uses TLS to protect traffic confidentiality and integrity. |
| **What is API versioning?** | API versioning allows APIs to evolve without unexpectedly breaking existing clients. |
| **What is rate limiting?** | Rate limiting controls how many requests a client can make within a specified period. |
| **What is CORS?** | CORS is a browser mechanism that controls whether cross-origin web requests are permitted. |
| **What is OpenAPI?** | OpenAPI is a specification for describing HTTP APIs. |
| **What is Swagger?** | Swagger is a set of tools associated with designing, documenting, and interacting with OpenAPI-described APIs. |
| **What is Postman?** | Postman is an API development and testing tool used to send requests, build collections, automate tests, and inspect responses. |
| **What is idempotency?** | An HTTP method is idempotent when repeating the same request has the same intended effect as making it once. GET, HEAD, PUT, and DELETE are defined as idempotent by HTTP semantics. |
| **Is POST idempotent?** | POST is not defined as idempotent by HTTP semantics, but applications can implement idempotency using mechanisms such as idempotency keys. |
| **What is an API Gateway?** | An API gateway is an intermediary that can handle routing, authentication, rate limiting, TLS termination, and other cross-cutting concerns. |
| **What is the difference between REST and SOAP?** | REST is an architectural style commonly implemented over HTTP, while SOAP is a protocol based on structured XML messaging. |
| **What is GraphQL?** | GraphQL allows a client to specify the data it wants through a query against a defined schema. |
| **What is gRPC?** | gRPC is an RPC framework commonly using Protocol Buffers and HTTP/2 for service-to-service communication. |
| **What is WebSocket?** | WebSocket provides a persistent, bidirectional connection suitable for real-time communication. |
| **How does `kubectl` communicate with Kubernetes?** | `kubectl` acts as a client of the Kubernetes API server. |
| **How does the AWS CLI communicate with AWS?** | The AWS CLI calls AWS service APIs. |
| **How does Terraform communicate with AWS?** | Terraform uses the AWS provider, which interacts with AWS APIs to manage resources. |
| **Why are APIs important in DevOps?** | APIs enable automation and integration between CI/CD systems, cloud services, Kubernetes, infrastructure tools, monitoring systems, and applications. |

---

# 31. APIs in OT-Microservices

APIs are the primary communication mechanism between the frontend and backend services in the OT-Microservices project.

The architecture can be represented as:

```text
                         Client
                           │
                           ▼
                    NGINX / Frontend
                           │
           ┌───────────────┼────────────────┐
           ▼               ▼                ▼
      Employee API    Attendance API    Salary API
           │               │                │
           ▼               ▼                ▼
       ScyllaDB        PostgreSQL        ScyllaDB
           │                                │
           │                                ▼
           │                          Elasticsearch
           │                                │
           │                                ▼
           │                        Notification Worker
           │
           └──────────────► Redis Cache
```

## Employee API

The Employee API is a Go-based service.

Typical resource-oriented endpoints include:

```text
POST /employees
GET  /employees/{id}
```

Flow:

```text
Frontend
   │
   ▼
Employee API
   │
   ▼
ScyllaDB
   │
   ▼
Employee Data
```

## Attendance API

The Attendance API is a Python Flask service.

Flow:

```text
Frontend
   │
   ▼
Attendance API
   │
   ▼
Redis
   │
   ├── Cache Hit → Response
   │
   └── Cache Miss
          │
          ▼
      PostgreSQL
          │
          ▼
      Redis Cache
          │
          ▼
       Response
```

PostgreSQL remains the durable source of truth for attendance data.

## Salary API

The Salary API is a Java Spring Boot service.

Flow:

```text
Frontend
   │
   ▼
Salary API
   │
   ▼
ScyllaDB
   │
   ▼
Salary Data
   │
   └────────► Elasticsearch
                 │
                 ▼
            Search / Processing
```

ScyllaDB remains the source of truth for salary data, while Elasticsearch can act as a search/indexing layer.

## Notification Service

A notification worker can consume salary-related information from Elasticsearch.

```text
Salary API
    │
    ▼
ScyllaDB
    │
    │ Index / Mirror
    ▼
Elasticsearch
    │
    ▼
Notification Worker
    │
    ├── Generate PDF
    │
    └── Send Email
```

## Overall API Flow

```text
Browser
   │
   ▼
NGINX
   │
   ▼
Frontend
   │
   ├──────────────► Employee API ─────► ScyllaDB
   │
   ├──────────────► Attendance API ───► PostgreSQL
   │
   └──────────────► Salary API ───────► ScyllaDB
                                          │
                                          ▼
                                    Elasticsearch
                                          │
                                          ▼
                                Notification Worker
```

The API layer provides clear contracts between clients and services while allowing each microservice to own its business logic and persistence layer.
