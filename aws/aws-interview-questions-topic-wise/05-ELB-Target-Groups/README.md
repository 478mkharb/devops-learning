# ELB & Target Groups

[⬅️ Back to AWS Topics](../README.md)

## 🔑 Keywords

⚖️ ALB | NLB | GWLB | Target Group | Health Check | Deregistration | Layer 7 | Layer 4

## 🧠 Core Memory

🧠 **Remember:** **ALB = Layer 7**, **NLB = Layer 4**, **GWLB = network appliances**.

---

## ❓ Interview Questions

### 📌 ELB Types

#### Q1. What is Elastic Load Balancing?

**💡 Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

**🔑 Keywords:** `ELB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q2. Explain Application Load Balancer.

**💡 Answer:** An Application Load Balancer operates at the application layer and supports HTTP/HTTPS-aware routing such as host and path routing. It is a strong choice for web applications and microservices.

**🔑 Keywords:** `ELB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q3. Explain Network Load Balancer.

**💡 Answer:** A Network Load Balancer operates at the transport layer and is designed for very high-performance TCP/UDP/TLS traffic. It provides low latency and supports static IP addresses and source-IP preservation scenarios.

**🔑 Keywords:** `ELB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q4. Explain Gateway Load Balancer.

**💡 Answer:** A Gateway Load Balancer is designed to deploy and scale virtual network appliances such as firewalls and intrusion-prevention systems. It uses GENEVE encapsulation and works with a Gateway Load Balancer endpoint.

**🔑 Keywords:** `ELB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q5. When would you choose ALB over NLB?

**💡 Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

**🔑 Keywords:** `ELB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q6. When would you choose NLB over ALB?

**💡 Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

**🔑 Keywords:** `ELB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q7. What is a Classic Load Balancer and why is it generally legacy?

**💡 Answer:** Classic Load Balancer is the older Elastic Load Balancing generation. ALB and NLB provide newer capabilities and are normally selected for new architectures.

**🔑 Keywords:** `ELB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 ALB

#### Q8. Which OSI layer does ALB primarily operate at?

**💡 Answer:** A Lambda layer packages reusable libraries or other dependencies separately from function code. Multiple functions can share the same layer.

**🔑 Keywords:** `ELB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q9. What are ALB target types?

**💡 Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

**🔑 Keywords:** `ELB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q10. What is ALB cross-zone load balancing?

**💡 Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

**🔑 Keywords:** `ELB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q11. What is connection draining/ deregistration delay?

**💡 Answer:** Deregistration delay allows existing connections to finish before a target is fully removed from service. It helps deployments and scale-in operations avoid abruptly dropping in-flight requests.

**🔑 Keywords:** `ELB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q12. What is ALB access logging?

**💡 Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

**🔑 Keywords:** `ELB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 NLB

#### Q13. Which OSI layer does NLB primarily operate at?

**💡 Answer:** A Lambda layer packages reusable libraries or other dependencies separately from function code. Multiple functions can share the same layer.

**🔑 Keywords:** `ELB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q14. What is a static IP capability of NLB?

**💡 Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

**🔑 Keywords:** `ELB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q15. What is TLS termination on NLB?

**💡 Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

**🔑 Keywords:** `ELB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q16. What is source IP preservation?

**💡 Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

**🔑 Keywords:** `ELB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q17. When is NLB useful for very high-performance TCP/UDP workloads?

**💡 Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

**🔑 Keywords:** `ELB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Target Groups

#### Q18. What is a target group?

**💡 Answer:** A target group is a logical set of backend targets used by a load balancer. It defines target type, protocol/port, and health-check settings. A load balancer forwards traffic to healthy registered targets.

**🔑 Keywords:** `ELB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q19. What target types can target groups support?

**💡 Answer:** A target group is a logical set of backend targets used by a load balancer. It defines target type, protocol/port, and health-check settings. A load balancer forwards traffic to healthy registered targets.

**🔑 Keywords:** `ELB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q20. What is a health check?

**💡 Answer:** A load-balancer target health check periodically tests a configured protocol, port, and path or connection behavior. Only healthy targets receive traffic.

**🔑 Keywords:** `ELB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q21. Which health-check settings can be configured?

**💡 Answer:** An ASG can use EC2 health checks and, when configured, ELB health checks. When an instance is considered unhealthy, the ASG terminates it and launches a replacement to restore desired capacity.

**🔑 Keywords:** `ELB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q22. What happens when a target fails health checks?

**💡 Answer:** A load-balancer target health check periodically tests a configured protocol, port, and path or connection behavior. Only healthy targets receive traffic.

**🔑 Keywords:** `ELB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q23. Can one target belong to multiple target groups?

**💡 Answer:** A target group is a logical set of backend targets used by a load balancer. It defines target type, protocol/port, and health-check settings. A load balancer forwards traffic to healthy registered targets.

**🔑 Keywords:** `ELB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q24. How does deregistration delay work?

**💡 Answer:** Deregistration delay allows existing connections to finish before a target is fully removed from service. It helps deployments and scale-in operations avoid abruptly dropping in-flight requests.

**🔑 Keywords:** `ELB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

## 🚀 Last-Minute Revision

> 🧠 **Remember:** **ALB = Layer 7**, **NLB = Layer 4**, **GWLB = network appliances**.

[⬆️ Back to top](#elb-target-groups)

[⬅️ Back to AWS Topics](../README.md)