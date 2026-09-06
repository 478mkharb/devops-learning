# CloudFront

## Interview Questions & Answers

### Q1. What is Amazon CloudFront?

**Answer:** Amazon CloudFront is AWS's content delivery network. It caches content at edge locations and can accelerate dynamic and static applications close to users.

---

### Q2. What is an edge location?

**Answer:** A CloudFront edge location is an AWS point of presence where content can be cached or requests can be handled closer to viewers.

---

### Q3. What is a distribution?

**Answer:** A CloudFront distribution defines how CloudFront serves content, including origins, cache behaviors, certificates, security settings, and viewer protocol policies.

---

### Q4. What is an origin?

**Answer:** A CloudFront origin is the backend from which CloudFront retrieves content, such as S3, an ALB, API Gateway, or a custom HTTP server.

---

### Q5. What is a cache behavior?

**Answer:** A CloudFront cache behavior defines how requests matching a path pattern are handled, including origin selection, allowed methods, caching policies, and viewer protocol behavior.

---

### Q6. What is TTL in CloudFront?

**Answer:** DNS TTL specifies how long a resolver may cache a DNS answer before querying again. A lower TTL can make changes visible sooner but increases DNS query traffic.

---

### Q7. What causes a cache miss?

**Answer:** A cache miss occurs when the requested object is not present as a valid cached object at the selected CloudFront edge location. CloudFront retrieves the object from the configured origin and can cache it according to the cache policy and TTL.

---

### Q8. What is invalidation?

**Answer:** A CloudFront invalidation explicitly removes selected objects from CloudFront caches so subsequent requests retrieve the current version from the origin. It is useful after content changes when versioned object names are not being used.

---

### Q9. How do cache policies work?

**Answer:** A CloudFront cache policy controls the cache key and TTL behavior. It determines which request values, such as query strings, headers, and cookies, are included when CloudFront decides whether two requests can use the same cached object.

---

### Q10. How can query strings affect caching?

**Answer:** If query strings are included in the cache key, different query-string values can create separate cached objects. If they are excluded, requests with different query strings can share the same cached response, which is safe only when the origin response does not depend on those values.

---

### Q11. How can CloudFront use HTTPS?

**Answer:** Amazon CloudFront is AWS's content delivery network. It caches content at edge locations and can accelerate dynamic and static applications close to users.

---

### Q12. What is Origin Access Control for S3?

**Answer:** A CloudFront origin is the backend from which CloudFront retrieves content, such as S3, an ALB, API Gateway, or a custom HTTP server.

---

### Q13. How can WAF integrate with CloudFront?

**Answer:** AWS WAF is a web application firewall for inspecting HTTP(S) requests. Web ACLs contain rules that can allow, block, count, challenge, or rate-limit requests based on request characteristics.

---

### Q14. What is signed URL?

**Answer:** A CloudFront signed URL grants time-limited access to a specific resource for users who are authorized to receive it.

---

### Q15. What is signed cookie?

**Answer:** A CloudFront signed cookie grants access to multiple restricted objects without requiring a separate signed URL for each object.

---
