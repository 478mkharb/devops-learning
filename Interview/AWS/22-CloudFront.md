# CloudFront

### Q1. What is Amazon CloudFront?

**Answer:** Amazon CloudFront is AWS's content delivery network (CDN). It delivers content from geographically distributed edge locations close to viewers and can retrieve content from origins such as Amazon S3, Application Load Balancers, API Gateway, or custom HTTP servers.

---

### Q2. What is a CloudFront edge location?

**Answer:** A CloudFront edge location is an AWS point of presence where CloudFront can cache content and process supported requests closer to viewers. Serving cached content from an edge location can reduce latency and origin load.

---

### Q3. What is a CloudFront distribution?

**Answer:** A CloudFront distribution is the configuration that defines how CloudFront delivers content. It specifies settings such as origins, cache behaviors, viewer protocol policies, TLS certificates, and security controls.

---

### Q4. What is a CloudFront origin?

**Answer:** A CloudFront origin is the backend from which CloudFront retrieves content when it is not available in the edge cache.

Common origins include:

| Origin | Example use |
|---|---|
| **S3 bucket** | Static websites, images, videos, files |
| **Application Load Balancer** | Dynamic web applications |
| **API Gateway** | API endpoints |
| **Custom HTTP server** | Applications running on EC2 or other HTTP servers |

---

### Q5. What is a CloudFront cache behavior?

**Answer:** A cache behavior defines how CloudFront handles requests matching a specified path pattern. It can determine the origin, allowed HTTP methods, cache policy, origin request policy, viewer protocol policy, and other request-processing settings.

For example:

```text
/*          → S3 origin
/api/*      → ALB origin
/images/*   → S3 origin
```

---

### Q6. What is TTL in CloudFront?

**Answer:** TTL (Time to Live) determines how long CloudFront can keep an object in its cache before it needs to check whether the cached response is still valid according to the configured cache policy and origin response headers.

| TTL | Effect |
|---|---|
| **Long TTL** | Better cache hit ratio and fewer origin requests, but content may remain stale longer |
| **Short TTL** | Fresher content, but more origin requests and potentially lower cache efficiency |

---

### Q7. What is a CloudFront cache miss?

**Answer:** A cache miss occurs when the requested object is not available in the selected CloudFront cache for the relevant cache key. CloudFront forwards the request to the origin, retrieves the object, returns it to the viewer, and can cache the response according to the configured policies.

---

### Q8. What is CloudFront invalidation?

**Answer:** A CloudFront invalidation explicitly removes selected cached objects from CloudFront edge caches so subsequent requests retrieve the current content from the origin.

It is useful when content needs to be refreshed before its normal TTL expires.

Example:

```text
CloudFront cache
      |
      | Invalidation
      ↓
Cached object removed
      |
      | Next request
      ↓
Origin → Fresh content
```

---

### Q9. How does a CloudFront cache policy work?

**Answer:** A CloudFront cache policy determines which request values are included in the CloudFront cache key. These can include selected query strings, headers, and cookies.

Requests that produce the same cache key can share the same cached response.

---

### Q10. How do query strings affect CloudFront caching?

**Answer:** CloudFront can include selected query-string parameters in the cache key.

| Configuration | Result |
|---|---|
| Parameter included in cache key | Different parameter values can create different cached responses |
| Parameter excluded from cache key | Different values can share the same cached response, if other cache-key components are identical |

For example, if `productId` is included:

```text
/products?id=100 → Cache object A
/products?id=200 → Cache object B
```

If `id` is excluded from the cache key, those requests may use the same cached response, which is only correct when the response does not actually depend on that parameter.

---

### Q11. How can CloudFront use HTTPS?

**Answer:** CloudFront can use HTTPS for viewer connections by configuring a valid TLS certificate, typically from AWS Certificate Manager (ACM). CloudFront can also use HTTPS when communicating with an HTTPS origin.

This provides two separate TLS connections when HTTPS is used on both sides:

```text
Viewer
  |
  | HTTPS
  ↓
CloudFront
  |
  | HTTPS
  ↓
Origin
```

---

### Q12. What is Origin Access Control (OAC) for S3?

**Answer:** Origin Access Control (OAC) allows CloudFront to authenticate requests to an S3 origin so that the S3 bucket can remain private while CloudFront retrieves and serves the content.

Typical architecture:

```text
User
  |
  ↓
CloudFront
  |
  | Authenticated request
  ↓
Private S3 Bucket
```

---

### Q13. How does AWS WAF integrate with CloudFront?

**Answer:** An AWS WAF web ACL can be associated with a CloudFront distribution. WAF evaluates applicable viewer requests at the CloudFront edge before allowed requests are forwarded to the origin.

This can provide controls such as:

- IP-based rules
- Managed rule groups
- Request filtering
- Rate-based rules
- Protection against common web exploits

---

### Q14. What is a CloudFront signed URL?

**Answer:** A CloudFront signed URL provides time-limited, controlled access to a specific protected resource. It is useful when access needs to be granted to individual files or resources.

---

### Q15. What is a CloudFront signed cookie?

**Answer:** A CloudFront signed cookie provides access to multiple protected resources without requiring a separate signed URL for every resource.

---

### Q16. What is the difference between a CloudFront signed URL and signed cookie?

**Answer:** Both provide controlled access to private CloudFront content, but they are suited to different access patterns.

| Feature | Signed URL | Signed Cookie |
|---|---|---|
| Access model | Individual resource/request | Multiple protected resources |
| URL modification | Access information is embedded in the URL | URL remains unchanged |
| Best suited for | Single files or specific resources | Multiple files or a collection of protected content |
| Example | Download one private PDF | Access an entire private video/content library |

---

### Q17. What is a CloudFront cache key?

**Answer:** A CloudFront cache key is the set of request attributes CloudFront uses to determine whether two requests can use the same cached response. Depending on the cache policy, it can include values such as the URL path, selected query strings, headers, and cookies.

---

### Q18. What is the difference between a cache policy and an origin request policy?

**Answer:** A cache policy controls what contributes to the cache key and therefore determines how responses are cached. An origin request policy controls which request values CloudFront forwards to the origin without necessarily including those values in the cache key.

| Feature | Cache Policy | Origin Request Policy |
|---|---|---|
| Main purpose | Control caching | Control what is forwarded to origin |
| Affects cache key | Yes | No |
| Controls forwarded headers/cookies/query strings | Can affect caching | Yes |
| Main concern | Cache efficiency and correctness | Origin request requirements |

---

### Q19. What is a CloudFront Origin Shield?

**Answer:** Origin Shield provides an additional centralized caching layer between CloudFront edge locations and the origin. It can improve cache efficiency and reduce the number of requests reaching the origin, especially when viewers are distributed across many edge locations.

---

### Q20. What is the difference between CloudFront and a traditional CDN?

**Answer:** CloudFront is AWS's CDN and integrates directly with AWS services such as S3, ALB, WAF, ACM, and Shield.

| Capability | CloudFront |
|---|---|
| Content caching | Yes |
| Global edge network | Yes |
| S3 integration | Native |
| ALB integration | Native |
| AWS WAF integration | Yes |
| ACM certificate integration | Yes |
| Signed URLs/cookies | Yes |
| Origin Shield | Yes |
