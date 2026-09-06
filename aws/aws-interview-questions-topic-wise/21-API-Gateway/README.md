# API Gateway

### Q1. What is Amazon API Gateway?

**Answer:** Amazon API Gateway is a managed service for creating, publishing, securing, throttling, monitoring, and operating APIs. It supports REST, HTTP, and WebSocket APIs.

---

### Q2. What is a REST API?

**Answer:** An API Gateway REST API provides a feature-rich API model with resources, methods, stages, authorizers, integrations, throttling, and other controls.

---

### Q3. What is an HTTP API?

**Answer:** An API Gateway HTTP API is a simpler API option designed for common HTTP and Lambda/backend integrations. It generally has fewer features and lower cost than REST APIs.

---

### Q4. What is a WebSocket API?

**Answer:** An API Gateway WebSocket API supports persistent two-way communication between clients and backend integrations, making it suitable for real-time applications.

---

### Q5. What is a resource and method?

**Answer:** In an API Gateway REST API, a resource represents a URL path and a method represents an HTTP operation such as GET, POST, PUT, or DELETE on that resource.

---

### Q6. How can API Gateway authenticate clients?

**Answer:** API Gateway can use IAM authorization, Lambda authorizers, JWT authorizers for supported APIs, and integrations with identity providers such as Amazon Cognito. API keys can be used for usage identification and throttling, but they should not be treated as a strong authentication mechanism by themselves.

---

### Q7. What is IAM authorization?

**Answer:** API Gateway IAM authorization uses AWS Signature Version 4-signed requests and IAM policies to determine whether the caller is allowed to invoke the API.

---

### Q8. What is a Lambda authorizer?

**Answer:** A Lambda authorizer is custom authorization code invoked by API Gateway to inspect a request and return an authorization decision or policy.

---

### Q9. What is a JWT authorizer?

**Answer:** A JWT authorizer validates JSON Web Tokens for supported API Gateway APIs, checking configured issuer, audience, and token claims before allowing a request.

---

### Q10. How can WAF protect API Gateway?

**Answer:** Associate a WAF web ACL with a supported API Gateway API. WAF evaluates incoming web requests and can block malicious patterns, restrict IPs, rate-limit abusive clients, and apply managed web-protection rules before requests reach the integration.

---

### Q11. What is throttling?

**Answer:** API Gateway throttling limits request rates and burst traffic to protect APIs and backend services. Limits can be configured at appropriate API or usage levels.

---

### Q12. What is caching?

**Answer:** API Gateway caching stores responses for configured API methods so repeated requests can be served without invoking the backend for every request. It can reduce backend load and latency, but cached data must be appropriate for the required freshness.

---

### Q13. What is a stage?

**Answer:** An API Gateway stage is a named logical deployment environment such as dev, test, or prod. Stage settings can include logging, throttling, variables, and other behavior.

---

### Q14. What is a deployment?

**Answer:** An API Gateway deployment is a point-in-time snapshot of an API configuration that can be associated with a stage. Changes generally need to be deployed before clients using that stage see them.

---

### Q15. What is a usage plan?

**Answer:** An API Gateway usage plan associates API keys with API stages and can apply throttling and quota controls for clients.

---

### Q16. How does API Gateway integrate with Lambda?

**Answer:** API Gateway receives the client HTTP request and invokes the configured Lambda integration. Lambda processes the event and returns a response, which API Gateway transforms as configured and sends back to the client.

---
