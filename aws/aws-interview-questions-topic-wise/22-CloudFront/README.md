# CloudFront

### Q1. What is Amazon CloudFront?

**Answer:** Amazon CloudFront is AWS's content delivery network. It serves content from edge locations close to viewers and can cache content from origins such as S3, ALB, API Gateway, or custom servers.

---

### Q2. What is an edge location?

**Answer:** A CloudFront edge location is an AWS point of presence where CloudFront can cache content and handle supported request processing close to users.

---

### Q3. What is a distribution?

**Answer:** A CloudFront distribution defines how CloudFront serves content, including origins, cache behaviors, certificates, viewer protocol policies, and security settings.

---

### Q4. What is an origin?

**Answer:** A CloudFront origin is the backend from which CloudFront retrieves content, such as S3, an ALB, API Gateway, or a custom HTTP server.

---

### Q5. What is a cache behavior?

**Answer:** A CloudFront cache behavior defines how requests matching a path pattern are handled, including the origin, allowed methods, cache policy, and viewer protocol policy.

---

### Q6. What is TTL in CloudFront?

**Answer:** CloudFront TTL controls how long an object can remain cached at an edge location before CloudFront considers it stale and needs to revalidate or retrieve it according to the cache policy and origin behavior.

---

### Q7. What causes a cache miss?

**Answer:** A CloudFront cache miss occurs when the requested object is not available in the selected edge cache. CloudFront then forwards the request to the origin, retrieves the object, and may cache it according to the cache behavior.

---

### Q8. What is invalidation?

**Answer:** A CloudFront invalidation explicitly removes selected objects from edge caches. It is used when cached content must be refreshed before its normal TTL expires.

---

### Q9. How do cache policies work?

**Answer:** A CloudFront cache policy determines which request values, such as selected headers, cookies, and query strings, are included in the cache key. Requests with the same cache-key values can share a cached response, improving cache efficiency.

---

### Q10. How can query strings affect caching?

**Answer:** CloudFront can include selected query-string parameters in the cache key. If a parameter is included, different values can produce separate cached objects; if it is excluded, requests with different values may share the same cached response. The cache policy controls this behavior.

---

### Q11. How can CloudFront use HTTPS?

**Answer:** CloudFront can use HTTPS for viewer connections by configuring a valid TLS certificate, typically from AWS Certificate Manager in the required Region for the distribution. CloudFront can also use HTTPS when communicating with an HTTPS origin.

---

### Q12. What is Origin Access Control for S3?

**Answer:** CloudFront Origin Access Control (OAC) allows CloudFront to authenticate requests to an S3 origin so the S3 bucket can remain private while CloudFront serves the content.

---

### Q13. How can WAF integrate with CloudFront?

**Answer:** Associate an AWS WAF web ACL with the CloudFront distribution. WAF evaluates viewer requests at the edge before CloudFront forwards allowed requests to the origin, enabling filtering and rate-based protection close to users.

---

### Q14. What is signed URL?

**Answer:** A CloudFront signed URL grants time-limited access to a specific protected resource. It is useful when individual objects need controlled access.

---

### Q15. What is signed cookie?

**Answer:** A CloudFront signed cookie grants access to multiple protected objects without requiring a separate signed URL for every object.

---
