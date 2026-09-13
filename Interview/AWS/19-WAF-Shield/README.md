# WAF & Shield

### Q1. What is AWS WAF?

**Answer:** AWS WAF is a web application firewall that evaluates HTTP(S) requests using web ACLs and rules. It can allow, block, count, challenge, or rate-limit requests based on request characteristics.

---

### Q2. Where can WAF be associated?

**Answer:** AWS WAF can be associated with supported resources such as CloudFront distributions, Application Load Balancers, API Gateway APIs, and other supported regional application resources. The exact association options depend on the resource type and Region.

---

### Q3. What is a web ACL?

**Answer:** A WAF web ACL is a collection of rules and rule groups evaluated against web requests. It is associated with supported resources such as CloudFront distributions and regional application resources.

---

### Q4. What is a rule?

**Answer:** A WAF rule defines a condition for matching web requests and an action to take when the request matches, such as Allow, Block, Count, CAPTCHA, Challenge, or rate-based mitigation. Rules can inspect IPs, headers, query strings, URI paths, request bodies, and other supported fields.

---

### Q5. What is a rule group?

**Answer:** A WAF rule group is a reusable collection of WAF rules managed together and referenced by web ACLs. It is useful for standardizing a set of protections across multiple applications.

---

### Q6. What is a managed rule group?

**Answer:** A managed rule group contains preconfigured WAF rules maintained by AWS or an AWS Marketplace provider. It can provide protection against common web threats without creating every rule manually.

---

### Q7. How does WAF protect against SQL injection?

**Answer:** WAF can inspect request components for patterns associated with SQL injection and block or count matching requests using managed or custom rules.

---

### Q8. How does WAF protect against XSS?

**Answer:** WAF can inspect request components for patterns associated with cross-site scripting and block or count matching requests using managed or custom rules.

---

### Q9. What is rate-based protection?

**Answer:** A WAF rate-based rule tracks request rates and can mitigate clients that exceed a configured threshold. It is useful for reducing abusive or unusually high request volumes.

---

### Q10. What is CAPTCHA/challenge?

**Answer:** WAF CAPTCHA and Challenge actions require clients to demonstrate that they are legitimate before the request is allowed to proceed. They can reduce automated abusive traffic.

---

### Q11. What is IP set matching?

**Answer:** A WAF IP set is a reusable collection of IP addresses or CIDR ranges that WAF rules can match for allow, block, or other actions.

---

### Q12. What is AWS Shield Standard?

**Answer:** AWS Shield Standard provides automatic DDoS protection for common network and transport-layer attacks against AWS services at no additional charge.

---

### Q13. What is AWS Shield Advanced?

**Answer:** AWS Shield Advanced provides enhanced DDoS protection, additional visibility and response capabilities, and eligibility for features such as DDoS cost protection for supported resources.

---

### Q14. How does Shield differ from WAF?

**Answer:** WAF primarily filters HTTP(S) requests using application-layer rules such as IP, path, header, SQL injection, XSS, and rate-based controls. Shield provides DDoS protection, with Shield Advanced adding enhanced visibility, response capabilities, and additional protections for eligible resources.

---

### Q15. When is Shield Advanced appropriate?

**Answer:** Shield Advanced is appropriate for business-critical public applications where enhanced DDoS protection, attack visibility and response capabilities, and DDoS-related cost protection justify the additional service cost.

---
