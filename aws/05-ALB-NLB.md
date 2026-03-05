# AWS Load Balancers (ALB / NLB)

This document explains common **DevOps interview questions related to Application Load Balancer (ALB) and Network Load Balancer (NLB)**.

---

# 1. How ALB Routes Traffic Internally

ALB operates at **Layer 7 (HTTP/HTTPS)** and routes traffic based on request content.

Traffic flow:

```
User Request
     │
     ▼
DNS → ALB
     │
     ▼
Listener (port 80 / 443)
     │
     ▼
Listener Rules
     │
     ▼
Target Group
     │
     ▼
EC2 / ECS / IP / Lambda
```

ALB evaluates rules such as:

* Host header
* Path pattern
* HTTP headers

Then forwards traffic to the correct **target group**.

---

# 2. How ALB Maintains Session Stickiness

Session stickiness ensures that a user's requests go to the **same backend instance**.

ALB implements stickiness using **cookies**.

Process:

```
User request
     │
     ▼
ALB selects instance
     │
     ▼
ALB sends cookie (AWSALB)
     │
     ▼
Future requests include cookie
     │
     ▼
ALB routes to same instance
```

This is useful for **stateful applications**.

---

# 3. Connection-Based vs Request-Based Routing

| Feature          | Connection-Based Routing | Request-Based Routing |
| ---------------- | ------------------------ | --------------------- |
| Layer            | Layer 4                  | Layer 7               |
| Example LB       | NLB                      | ALB                   |
| Routing Decision | Based on connection      | Based on HTTP request |
| Protocol         | TCP/UDP                  | HTTP/HTTPS            |

Connection-based routing forwards the entire connection to a backend.

Request-based routing inspects the HTTP request and makes decisions based on headers, paths, etc.

---

# 4. How NLB Achieves Ultra-Low Latency

NLB works at **Layer 4 (TCP/UDP)** and does minimal processing.

Key reasons for low latency:

* No HTTP inspection
* Static IP addresses
* Flow hashing routing algorithm
* Direct connection forwarding

Traffic flow:

```
Client
   │
   ▼
Network Load Balancer
   │
   ▼
Target Instance
```

---

# 5. When Should You Use NLB Instead of ALB

Use NLB when:

* Extremely **low latency** is required
* Handling **millions of connections per second**
* Using **TCP/UDP protocols**
* Static IP is required

Example workloads:

* Gaming servers
* Real-time applications
* IoT platforms

---

# 6. Path-Based Routing Using ALB

ALB can route traffic based on the URL path.

Example:

```
/api → API target group
/images → Image service
/auth → Authentication service
```

Flow:

```
User request
   │
   ▼
ALB Listener
   │
   ▼
Rule evaluation
   │
   ▼
Correct target group
```

---

# 7. Host-Based Routing in ALB

ALB can route requests based on **domain names**.

Example:

```
api.example.com → API service
shop.example.com → eCommerce service
admin.example.com → Admin service
```

Flow:

```
Request Host Header
      │
      ▼
ALB rule checks host
      │
      ▼
Route to correct target group
```

---

# 8. Routing Microservices Behind One ALB

Multiple microservices can share one ALB.

Example architecture:

```
                ALB
                 │
     ┌───────────┼───────────┐
     ▼           ▼           ▼
  API Service  Auth Service  Payment Service
```

Rules example:

```
/api/* → API target group
/auth/* → Auth target group
/pay/* → Payment target group
```

---

# 9. What Happens When All Targets Fail Health Checks

If all targets fail health checks:

```
ALB cannot route traffic
```

Result:

* ALB returns **503 Service Unavailable**

Possible reasons:

* Application crash
* Health check path incorrect
* Security group blocking

---

# 10. Weighted Traffic Routing Between Versions

Weighted routing allows traffic splitting between versions.

Example:

```
80% → Version v1
20% → Version v2
```

Implementation methods:

* Multiple target groups
* Listener rule weights
* Route53 weighted routing

Used for **canary deployments**.

---

# 11. Troubleshooting 502 / 503 Errors from ALB

## 502 Bad Gateway

Possible causes:

* Application crashed
* Backend closed connection
* Wrong target port

## 503 Service Unavailable

Possible causes:

* No healthy targets
* Target group empty

Troubleshooting steps:

```
Check ALB health checks
Check target group
Check security groups
Check application logs
```

---

# 12. SSL Termination in ALB

SSL termination means **ALB handles HTTPS encryption**.

Flow:

```
User HTTPS request
        │
        ▼
ALB decrypts SSL
        │
        ▼
Forward HTTP request to backend
```

Benefits:

* Reduces backend CPU load
* Centralized certificate management

Certificates are managed using **AWS Certificate Manager (ACM)**.

---

# 13. ALB Listener Rules Priority System

Listener rules are evaluated in **priority order**.

Example:

```
Priority 1 → /api/*
Priority 2 → /images/*
Priority 3 → default rule
```

ALB processes rules from **lowest number to highest number**.

First matching rule is applied.

---

# 14. What Happens If Multiple Listener Rules Match

If multiple rules match a request:

```
ALB selects the rule with the highest priority
(lowest number)
```

Example:

```
Rule 1 → /api/*
Rule 2 → /api/users/*
```

If request:

```
/api/users/123
```

Rule with **higher priority** is selected.

---

# Quick DevOps Summary

```
ALB = Layer 7 load balancer
NLB = Layer 4 load balancer

ALB supports:
- path routing
- host routing
- cookie stickiness

NLB supports:
- ultra low latency
- TCP/UDP
- static IP
```
