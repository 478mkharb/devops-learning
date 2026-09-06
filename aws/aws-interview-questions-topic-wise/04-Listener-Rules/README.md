# Listener & Listener Rules

### Q1. What is an ALB listener?

**Answer:** An ALB listener checks incoming connections on a configured protocol and port, such as HTTP:80 or HTTPS:443, and evaluates listener rules to determine how matching requests are handled.

---

### Q2. What protocols can an ALB listener use?

**Answer:** ALB listeners support HTTP and HTTPS for application-layer traffic. An HTTPS listener uses TLS and an ACM or suitable certificate to secure the client connection; an HTTP listener does not encrypt the connection.

---

### Q3. What is the difference between HTTP and HTTPS listeners?

**Answer:** An HTTP listener accepts unencrypted HTTP traffic, typically on port 80. An HTTPS listener uses TLS, typically on port 443, and requires a certificate. HTTPS allows the ALB to terminate TLS before forwarding traffic to targets using the configured backend protocol.

---

### Q4. What is a default listener action?

**Answer:** The default listener action is the fallback action for a request that does not match any higher-priority custom rule. It commonly forwards the request to a target group, but it can also return a fixed response or perform another supported action.

---

### Q5. Why is HTTPS commonly terminated at the ALB?

**Answer:** TLS termination at the ALB centralizes certificate management and removes TLS processing from backend servers. The ALB decrypts the client connection and forwards traffic to the targets using the configured target-group protocol. HTTPS can also be used from the ALB to targets when end-to-end encryption is required.

---

### Q6. What is a listener rule?

**Answer:** An ALB listener rule contains conditions and an action. Conditions can match request attributes such as host headers or paths, and actions can forward, redirect, or return a fixed response.

---

### Q7. What is rule priority?

**Answer:** Listener rules have priorities that determine evaluation order. Custom rules are evaluated from the lowest numerical priority to the highest; the first matching rule is used, and the default rule is evaluated if no custom rule matches.

---

### Q8. What is the default rule?

**Answer:** The default listener rule is the final rule evaluated when no custom listener rule matches. It provides the listener's fallback action, such as forwarding to a target group or returning a fixed response.

---

### Q9. What conditions can listener rules evaluate?

**Answer:** ALB listener rules can evaluate supported request attributes such as host headers, path patterns, HTTP headers, HTTP request methods, query strings, source IP addresses, and other supported conditions. A matching rule then executes its configured action.

---

### Q10. Explain host-header routing.

**Answer:** Host-header routing examines the HTTP Host header and can send different domain names to different target groups. For example, api.example.com and app.example.com can use separate backend services behind one ALB.

---

### Q11. Explain path-pattern routing.

**Answer:** Path-pattern routing examines the request path and can send different URL paths to different target groups. For example, /api/* can go to an API service while /web/* goes to a web service.

---

### Q12. Can a rule have multiple conditions?

**Answer:** Yes. A listener rule can contain multiple conditions, such as a host-header condition combined with a path-pattern condition. The request must satisfy the rule's conditions for the rule to match, subject to the supported condition combinations.

---

### Q13. Can a rule forward to different target groups?

**Answer:** Yes. Different listener rules can forward requests to different target groups. For example, `/api/*` can forward to an API target group while `/web/*` forwards to a web target group.

---

### Q14. What is a fixed-response action?

**Answer:** A fixed-response action makes the load balancer return a configured HTTP response directly, without forwarding the request to a target. It is useful for simple errors, maintenance responses, or controlled endpoints.

---

### Q15. What is a redirect action?

**Answer:** A redirect action instructs the client to make another request to a specified URL. A common example is redirecting HTTP traffic to HTTPS.

---

### Q16. How would you route /api to one target group and /web to another?

**Answer:** A target group is a logical collection of backend targets used by a load balancer. It defines the target type, protocol and port, health checks, and the targets that can receive forwarded traffic.

---

### Q17. How does host-based routing support multiple applications?

**Answer:** Host-based routing examines the HTTP Host header. One ALB can therefore route api.example.com to an API target group, app.example.com to a web target group, and admin.example.com to another application.

---

### Q18. What happens if no custom rule matches?

**Answer:** The ALB evaluates the default listener rule. Its configured default action is then executed, such as forwarding the request to a target group or returning a fixed response.

---

### Q19. How do listener rules differ from Route 53 routing?

**Answer:** Amazon Route 53 is AWS's managed DNS service. It provides authoritative DNS hosting, domain registration, health checks, and several DNS routing policies.

---

### Q20. How do listener rules differ from security groups?

**Answer:** The security pillar focuses on protecting information and systems through strong identity controls, detection, infrastructure protection, data protection, and incident response.

---
