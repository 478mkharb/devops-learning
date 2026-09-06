# Listener & Listener Rules

[⬅️ Back to AWS Topics](../README.md)

## 🔑 Keywords

🎧 Listener | Port | Protocol | Priority | Host Header | Path | Forward | Redirect

## 🧠 Core Memory

🧠 **Remember:** **Listener receives → Rule decides → Target Group forwards → Target serves**.

---

## ❓ Interview Questions

### 📌 Listener

#### Q1. What is an ALB listener?

**💡 Answer:** An ALB listener checks incoming connections on a configured protocol and port, such as HTTP:80 or HTTPS:443, and applies listener rules to determine the action. HTTPS listeners can terminate TLS at the load balancer.

**🔑 Keywords:** `Listener` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q2. What protocols can an ALB listener use?

**💡 Answer:** An ALB listener checks incoming connections on a configured protocol and port, such as HTTP:80 or HTTPS:443, and applies listener rules to determine the action. HTTPS listeners can terminate TLS at the load balancer.

**🔑 Keywords:** `Listener` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q3. What is the difference between HTTP and HTTPS listeners?

**💡 Answer:** An ALB listener checks incoming connections on a configured protocol and port, such as HTTP:80 or HTTPS:443, and applies listener rules to determine the action. HTTPS listeners can terminate TLS at the load balancer.

**🔑 Keywords:** `Listener` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q4. What is a default listener action?

**💡 Answer:** An ALB listener checks incoming connections on a configured protocol and port, such as HTTP:80 or HTTPS:443, and applies listener rules to determine the action. HTTPS listeners can terminate TLS at the load balancer.

**🔑 Keywords:** `Listener` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q5. Why is HTTPS commonly terminated at the ALB?

**💡 Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

**🔑 Keywords:** `Listener` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Rules

#### Q6. What is a listener rule?

**💡 Answer:** An ALB listener rule determines how matching requests are handled. Rules can match conditions such as host headers and paths and can forward, redirect, or return a fixed response. Rules are evaluated by priority, followed by the default rule.

**🔑 Keywords:** `Listener` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q7. What is rule priority?

**💡 Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

**🔑 Keywords:** `Listener` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q8. What is the default rule?

**💡 Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

**🔑 Keywords:** `Listener` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q9. What conditions can listener rules evaluate?

**💡 Answer:** An ALB listener rule determines how matching requests are handled. Rules can match conditions such as host headers and paths and can forward, redirect, or return a fixed response. Rules are evaluated by priority, followed by the default rule.

**🔑 Keywords:** `Listener` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q10. Explain host-header routing.

**💡 Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

**🔑 Keywords:** `Listener` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q11. Explain path-pattern routing.

**💡 Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

**🔑 Keywords:** `Listener` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q12. Can a rule have multiple conditions?

**💡 Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

**🔑 Keywords:** `Listener` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q13. Can a rule forward to different target groups?

**💡 Answer:** A target group is a logical set of backend targets used by a load balancer. It defines target type, protocol/port, and health-check settings. A load balancer forwards traffic to healthy registered targets.

**🔑 Keywords:** `Listener` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q14. What is a fixed-response action?

**💡 Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

**🔑 Keywords:** `Listener` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q15. What is a redirect action?

**💡 Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

**🔑 Keywords:** `Listener` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Design

#### Q16. How would you route /api to one target group and /web to another?

**💡 Answer:** A target group is a logical set of backend targets used by a load balancer. It defines target type, protocol/port, and health-check settings. A load balancer forwards traffic to healthy registered targets.

**🔑 Keywords:** `Listener` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q17. How does host-based routing support multiple applications?

**💡 Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

**🔑 Keywords:** `Listener` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q18. What happens if no custom rule matches?

**💡 Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

**🔑 Keywords:** `Listener` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q19. How do listener rules differ from Route 53 routing?

**💡 Answer:** An ALB listener rule determines how matching requests are handled. Rules can match conditions such as host headers and paths and can forward, redirect, or return a fixed response. Rules are evaluated by priority, followed by the default rule.

**🔑 Keywords:** `Listener` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q20. How do listener rules differ from security groups?

**💡 Answer:** An ALB listener rule determines how matching requests are handled. Rules can match conditions such as host headers and paths and can forward, redirect, or return a fixed response. Rules are evaluated by priority, followed by the default rule.

**🔑 Keywords:** `Listener` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

## 🚀 Last-Minute Revision

> 🧠 **Remember:** **Listener receives → Rule decides → Target Group forwards → Target serves**.

[⬆️ Back to top](#listener-listener-rules)

[⬅️ Back to AWS Topics](../README.md)