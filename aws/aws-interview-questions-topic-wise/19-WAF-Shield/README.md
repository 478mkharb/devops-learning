# WAF & Shield

## Interview Questions & Answers

### Q1. What is AWS WAF?

**Answer:** AWS WAF is a web application firewall for inspecting HTTP(S) requests. Web ACLs contain rules that can allow, block, count, challenge, or rate-limit requests based on request characteristics.

---

### Q2. Where can WAF be associated?

**Answer:** AWS WAF is a web application firewall for inspecting HTTP(S) requests. Web ACLs contain rules that can allow, block, count, challenge, or rate-limit requests based on request characteristics.

---

### Q3. What is a web ACL?

**Answer:** A WAF web ACL is a collection of rules and rule groups evaluated against web requests. It is associated with supported AWS resources such as CloudFront distributions and regional application endpoints.

---

### Q4. What is a rule?

**Answer:** A WAF rule defines a statement that matches web requests and an action to take when the statement matches. Actions can include Allow, Block, Count, CAPTCHA, Challenge, or rate-based mitigation depending on the rule.

---

### Q5. What is a rule group?

**Answer:** A WAF rule group is a reusable collection of WAF rules. Custom rule groups can be shared across web ACL designs, while managed rule groups provide prebuilt protections maintained by AWS or an AWS Marketplace provider.

---

### Q6. What is a managed rule group?

**Answer:** A managed rule group is a prebuilt set of WAF rules maintained by AWS or an AWS Marketplace security provider. It can provide protection against common web exploits without requiring every detection rule to be written manually.

---

### Q7. How does WAF protect against SQL injection?

**Answer:** AWS WAF is a web application firewall for inspecting HTTP(S) requests. Web ACLs contain rules that can allow, block, count, challenge, or rate-limit requests based on request characteristics.

---

### Q8. How does WAF protect against XSS?

**Answer:** AWS WAF is a web application firewall for inspecting HTTP(S) requests. Web ACLs contain rules that can allow, block, count, challenge, or rate-limit requests based on request characteristics.

---

### Q9. What is rate-based protection?

**Answer:** A WAF rate-based rule tracks request rates from clients and can automatically mitigate sources that exceed a configured threshold. It is useful for reducing abusive request floods.

---

### Q10. What is CAPTCHA/challenge?

**Answer:** CAPTCHA and Challenge are WAF actions that require the client to demonstrate that it is likely a legitimate browser/user before the request is allowed to continue. They are useful for mitigating automated bots and abusive traffic.

---

### Q11. What is IP set matching?

**Answer:** A WAF IP set is a reusable collection of IP addresses or CIDR ranges that rules can match. It can be used for allowlists, blocklists, or trusted-source controls.

---

### Q12. What is AWS Shield Standard?

**Answer:** AWS Shield Standard provides automatic DDoS protection for common network and transport-layer attacks against AWS services at no additional charge.

---

### Q13. What is AWS Shield Advanced?

**Answer:** AWS Shield Advanced provides enhanced DDoS protection, additional visibility and response capabilities, and access to DDoS cost-protection and specialist support features for eligible resources.

---

### Q14. How does Shield differ from WAF?

**Answer:** AWS WAF is a web application firewall for inspecting HTTP(S) requests. Web ACLs contain rules that can allow, block, count, challenge, or rate-limit requests based on request characteristics.

---

### Q15. When is Shield Advanced appropriate?

**Answer:** AWS Shield Advanced provides enhanced DDoS protection, additional visibility and response capabilities, and access to DDoS cost-protection and specialist support features for eligible resources.

---
