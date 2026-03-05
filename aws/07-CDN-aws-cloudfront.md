# AWS CloudFront / CDN

This guide explains common **DevOps interview questions about Amazon CloudFront and Content Delivery Networks (CDN)**.

---

# 1. How CloudFront Caches Content at Edge Locations

CloudFront caches content in **edge locations** that are geographically closer to users.

Flow:

```
User Request
     │
     ▼
Nearest Edge Location
     │
     ├─ If cached → return content immediately
     │
     └─ If not cached
            │
            ▼
         Origin Server
         (S3 / ALB / EC2)
            │
            ▼
Edge location caches response
            │
            ▼
User receives content
```

This reduces repeated requests to the origin and improves performance.

---

# 2. Difference Between CloudFront Origin vs Edge Location

| Component     | Description                                                |
| ------------- | ---------------------------------------------------------- |
| Origin        | The original source of content (S3, ALB, EC2, API Gateway) |
| Edge Location | CDN servers that cache content closer to users             |

Example:

```
User → Edge Location → Origin
```

Origin only receives traffic when content is **not cached**.

---

# 3. How to Secure an S3 Bucket Behind CloudFront

Best practice is to **block public access to S3** and allow access only through CloudFront.

Steps:

1. Create CloudFront distribution
2. Configure S3 as origin
3. Enable Origin Access Control (OAC)
4. Block public S3 access
5. Update S3 bucket policy to allow only CloudFront

Architecture:

```
User
 │
 ▼
CloudFront
 │
 ▼
Private S3 Bucket
```

Users cannot access the S3 bucket directly.

---

# 4. What is Origin Access Control (OAC)

Origin Access Control allows CloudFront to **securely access private S3 buckets**.

Key features:

* Uses **signed requests** between CloudFront and S3
* Prevents direct public access to S3
* Replaces the older **Origin Access Identity (OAI)** method

Flow:

```
User → CloudFront → Signed request → S3
```

---

# 5. How CloudFront Invalidation Works

Invalidation removes cached objects from edge locations.

Example command:

```
Invalidate /index.html
```

Process:

```
Admin triggers invalidation
       │
       ▼
CloudFront removes object from cache
       │
       ▼
Next request fetches latest version from origin
```

Used when **new application versions are deployed**.

---

# 6. What Happens When Cache Expires

Each object has a **TTL (Time To Live)**.

When TTL expires:

```
User request
     │
     ▼
Edge cache expired
     │
     ▼
CloudFront requests object from origin
     │
     ▼
Updated content cached again
```

Cache settings:

* Minimum TTL
* Default TTL
* Maximum TTL

---

# 7. Delivering Dynamic Content via CloudFront

CloudFront can deliver dynamic content by forwarding requests to origin.

Methods:

* Disable caching for certain paths
* Forward headers, cookies, query strings
* Use API Gateway or ALB as origin

Example:

```
User → CloudFront → ALB → Application
```

CloudFront still provides:

* TLS termination
* global routing
* DDoS protection

---

# 8. CloudFront vs Global Accelerator

| Feature  | CloudFront                 | Global Accelerator           |
| -------- | -------------------------- | ---------------------------- |
| Purpose  | CDN content caching        | Network traffic acceleration |
| Layer    | Layer 7 (HTTP/HTTPS)       | Layer 4 (TCP/UDP)            |
| Caching  | Yes                        | No                           |
| Use Case | Static/dynamic web content | Gaming, APIs, real-time apps |

CloudFront improves performance using caching, while Global Accelerator improves **network routing efficiency**.

---

# 9. How Signed URLs Protect Private Content

Signed URLs restrict access to content.

Process:

```
User requests protected content
        │
        ▼
Application generates signed URL
        │
        ▼
User accesses CloudFront using signed URL
        │
        ▼
CloudFront validates signature
        │
        ▼
Content delivered
```

Benefits:

* Only authorized users can access content
* Expiration time can be set

Used for:

* video streaming
* paid content
* secure downloads

---

# 10. How CloudFront Reduces Latency

CloudFront improves performance using **global edge locations**.

Instead of:

```
User (India) → US server
```

CloudFront provides:

```
User (India)
     │
     ▼
Nearby Edge Location
     │
     ▼
Cached content
```

Benefits:

* Lower latency
* Reduced origin load
* Faster content delivery worldwide

---

# Quick DevOps Summary

```
CloudFront = Global CDN

Key features:
- Edge caching
- S3 + ALB origins
- Signed URLs
- Cache invalidation
- Global latency reduction
```
