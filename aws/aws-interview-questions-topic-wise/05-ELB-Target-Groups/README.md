# ELB & Target Groups

## Interview Questions & Answers

### Q1. What is Elastic Load Balancing?

**Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

---

### Q2. Explain Application Load Balancer.

**Answer:** An Application Load Balancer operates at the application layer and supports HTTP/HTTPS-aware routing such as host and path routing. It is a strong choice for web applications and microservices.

---

### Q3. Explain Network Load Balancer.

**Answer:** A Network Load Balancer operates at the transport layer and is designed for very high-performance TCP/UDP/TLS traffic. It provides low latency and supports static IP addresses and source-IP preservation scenarios.

---

### Q4. Explain Gateway Load Balancer.

**Answer:** A Gateway Load Balancer is designed to deploy and scale virtual network appliances such as firewalls and intrusion-prevention systems. It uses GENEVE encapsulation and works with a Gateway Load Balancer endpoint.

---

### Q5. When would you choose ALB over NLB?

**Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

---

### Q6. When would you choose NLB over ALB?

**Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

---

### Q7. What is a Classic Load Balancer and why is it generally legacy?

**Answer:** Classic Load Balancer is the older Elastic Load Balancing generation. ALB and NLB provide newer capabilities and are normally selected for new architectures.

---

### Q8. Which OSI layer does ALB primarily operate at?

**Answer:** A Lambda layer packages reusable libraries or other dependencies separately from function code. Multiple functions can share the same layer.

---

### Q9. What are ALB target types?

**Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

---

### Q10. What is ALB cross-zone load balancing?

**Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

---

### Q11. What is connection draining/ deregistration delay?

**Answer:** Deregistration delay allows existing connections to finish before a target is fully removed from service. It helps deployments and scale-in operations avoid abruptly dropping in-flight requests.

---

### Q12. What is ALB access logging?

**Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

---

### Q13. Which OSI layer does NLB primarily operate at?

**Answer:** A Lambda layer packages reusable libraries or other dependencies separately from function code. Multiple functions can share the same layer.

---

### Q14. What is a static IP capability of NLB?

**Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

---

### Q15. What is TLS termination on NLB?

**Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

---

### Q16. What is source IP preservation?

**Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

---

### Q17. When is NLB useful for very high-performance TCP/UDP workloads?

**Answer:** Explain the traffic path from client to load balancer listener, listener rule, target group, and healthy target. Include the protocol/layer involved and the key configuration that controls the behavior.

---

### Q18. What is a target group?

**Answer:** A target group is a logical set of backend targets used by a load balancer. It defines target type, protocol/port, and health-check settings. A load balancer forwards traffic to healthy registered targets.

---

### Q19. What target types can target groups support?

**Answer:** A target group is a logical set of backend targets used by a load balancer. It defines target type, protocol/port, and health-check settings. A load balancer forwards traffic to healthy registered targets.

---

### Q20. What is a health check?

**Answer:** A load-balancer target health check periodically tests a configured protocol, port, and path or connection behavior. Only healthy targets receive traffic.

---

### Q21. Which health-check settings can be configured?

**Answer:** An ASG can use EC2 health checks and, when configured, ELB health checks. When an instance is considered unhealthy, the ASG terminates it and launches a replacement to restore desired capacity.

---

### Q22. What happens when a target fails health checks?

**Answer:** A load-balancer target health check periodically tests a configured protocol, port, and path or connection behavior. Only healthy targets receive traffic.

---

### Q23. Can one target belong to multiple target groups?

**Answer:** A target group is a logical set of backend targets used by a load balancer. It defines target type, protocol/port, and health-check settings. A load balancer forwards traffic to healthy registered targets.

---

### Q24. How does deregistration delay work?

**Answer:** Deregistration delay allows existing connections to finish before a target is fully removed from service. It helps deployments and scale-in operations avoid abruptly dropping in-flight requests.

---
