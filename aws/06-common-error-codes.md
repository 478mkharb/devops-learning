# Common HTTP Error Codes

This document lists **commonly seen HTTP error codes** that DevOps engineers encounter while working with **ALB, Nginx, APIs, microservices, and web applications**.

---

# 1. 400 – Bad Request

**Meaning:**
The server cannot process the request because it is malformed.

**Common Causes**

* Invalid request syntax
* Missing parameters
* Incorrect headers

**Example**

```
Client sends invalid JSON payload
```

---

# 2. 401 – Unauthorized

**Meaning:**
Authentication is required but was not provided or failed.

**Common Causes**

* Missing authentication token
* Invalid credentials

---

# 3. 403 – Forbidden

**Meaning:**
The request is understood but the server refuses to authorize it.

**Common Causes**

* Permission denied
* IAM policy restrictions
* Security rule blocking access

---

# 4. 404 – Not Found

**Meaning:**
The requested resource does not exist.

**Common Causes**

* Wrong URL path
* API endpoint missing
* Incorrect routing rule

---

# 5. 408 – Request Timeout

**Meaning:**
The server timed out waiting for the request.

**Common Causes**

* Slow client
* Network latency

---

# 6. 429 – Too Many Requests

**Meaning:**
Client sent too many requests in a short time.

**Common Causes**

* Rate limiting
* API throttling

---

# 7. 500 – Internal Server Error

**Meaning:**
Generic server error.

**Common Causes**

* Application crash
* Unhandled exception
* Misconfigured service

---

# 8. 502 – Bad Gateway

**Meaning:**
A gateway or load balancer received an invalid response from the backend server.

**Common Causes**

* Backend application crashed
* Wrong port configuration
* Container not running

---

# 9. 503 – Service Unavailable

**Meaning:**
Server is temporarily unable to handle the request.

**Common Causes**

* No healthy targets in ALB
* Application overload
* Maintenance mode

---

# 10. 504 – Gateway Timeout

**Meaning:**
The upstream server did not respond in time.

**Common Causes**

* Slow backend service
* Network latency
* Database query delay

---

# Quick DevOps Summary

```
4xx → Client errors
5xx → Server errors

502 → Backend failure
503 → Service unavailable
504 → Backend timeout
```

These errors are commonly seen when troubleshooting:

* Application Load Balancers
* API gateways
* Microservices
* Web servers (Nginx / Apache)
* Kubernetes ingress
