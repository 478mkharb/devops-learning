# ELB & Target Groups

### Q1. What is Elastic Load Balancing?

**Answer:** Elastic Load Balancing distributes incoming traffic across healthy targets. AWS provides Application Load Balancer for HTTP/HTTPS-aware routing, Network Load Balancer for high-performance transport-layer traffic, and Gateway Load Balancer for network appliances.

---

### Q2. Explain Application Load Balancer.

**Answer:** An Application Load Balancer operates at Layer 7 and understands HTTP/HTTPS requests. It supports host-based and path-based routing, listener rules, redirects, fixed responses, and target groups.

---

### Q3. Explain Network Load Balancer.

**Answer:** A Network Load Balancer operates primarily at Layer 4 and handles TCP, UDP, and TLS traffic with very high throughput and low latency. It supports features such as static IP addresses and source-IP preservation.

---

### Q4. Explain Gateway Load Balancer.

**Answer:** A Gateway Load Balancer is designed to deploy, scale, and manage virtual network appliances such as firewalls and intrusion-prevention systems. It uses GENEVE encapsulation and Gateway Load Balancer endpoints.

---

### Q5. When would you choose ALB over NLB?

**Answer:** Choose an ALB when the workload is HTTP/HTTPS based and needs Layer 7 features such as host-based routing, path-based routing, redirects, or application-aware listener rules.

---

### Q6. When would you choose NLB over ALB?

**Answer:** Choose an NLB when you need high-performance Layer 4 TCP/UDP/TLS traffic handling, very low latency, static IP addresses, or source-IP preservation scenarios that fit NLB capabilities.

---

### Q7. What is a Classic Load Balancer and why is it generally legacy?

**Answer:** Classic Load Balancer is the earlier generation of Elastic Load Balancing. It provides basic Layer 4/7 load balancing but lacks many capabilities available in ALB and NLB, so new architectures generally use ALB, NLB, or GWLB instead.

---

### Q8. Which OSI layer does ALB primarily operate at?

**Answer:** ALB primarily operates at Layer 7, the application layer. It understands HTTP/HTTPS requests and can route traffic using information such as host headers and URL paths.

---

### Q9. What are ALB target types?

**Answer:** ALB target groups can use supported target types such as instance, IP address, and Lambda. The target type determines how the load balancer registers and forwards requests to the backend.

---

### Q10. What is ALB cross-zone load balancing?

**Answer:** Cross-zone load balancing distributes requests across healthy targets in enabled Availability Zones rather than restricting each load balancer node to targets in its own zone. This can improve distribution when target counts differ between zones.

---

### Q11. What is connection draining/ deregistration delay?

**Answer:** Deregistration delay, also called connection draining, gives existing connections time to complete after a target is removed from service. It reduces dropped in-flight requests during deployments and scale-in.

---

### Q12. What is ALB access logging?

**Answer:** ALB access logs record detailed information about requests received by the load balancer, such as client information, request processing details, target information, status codes, and timing. Logs can be delivered to Amazon S3 for analysis and auditing.

---

### Q13. Which OSI layer does NLB primarily operate at?

**Answer:** NLB primarily operates at Layer 4, the transport layer. It handles connections such as TCP, UDP, and TLS without the HTTP-aware routing model of an ALB.

---

### Q14. What is a static IP capability of NLB?

**Answer:** NLB supports static IP addresses for its nodes, including Elastic IP association for internet-facing designs. This is useful when clients or allowlists require stable IP addresses.

---

### Q15. What is TLS termination on NLB?

**Answer:** With a TLS listener, an NLB can terminate TLS at the load balancer using an ACM certificate and then forward traffic to targets using the configured target protocol. This offloads TLS processing from targets.

---

### Q16. What is source IP preservation?

**Answer:** Source IP preservation means the backend can see the original client IP rather than only the load balancer's address. NLB supports source-IP preservation for appropriate configurations and protocols.

---

### Q17. When is NLB useful for very high-performance TCP/UDP workloads?

**Answer:** NLB is useful for workloads requiring high connection rates, high throughput, low latency, TCP/UDP support, static IP addresses, or transport-layer load balancing without HTTP-aware routing.

---

### Q18. What is a target group?

**Answer:** A target group is a logical collection of backend targets used by a load balancer. It defines the target type, protocol and port, health checks, and the targets that can receive forwarded traffic.

---

### Q19. What target types can target groups support?

**Answer:** Depending on the load balancer and target group configuration, supported target types include EC2 instances, IP addresses, and Lambda for ALB, while NLB target groups commonly use instance, IP, or ALB targets. The exact options depend on the load balancer type.

---

### Q20. What is a health check?

**Answer:** A load-balancer health check periodically tests a target using configured protocol, port, path, and success criteria. Traffic is normally sent only to targets considered healthy.

---

### Q21. Which health-check settings can be configured?

**Answer:** Health checks can be configured with settings such as protocol, port, path for HTTP/HTTPS checks, healthy and unhealthy thresholds, timeout, and interval. The supported settings depend on the target group protocol and load balancer type.

---

### Q22. What happens when a target fails health checks?

**Answer:** The load balancer stops routing new traffic to a target that fails its health checks. The target remains registered and can receive traffic again after it passes the configured healthy threshold.

---

### Q23. Can one target belong to multiple target groups?

**Answer:** Yes. A target, such as an EC2 instance or IP address, can be registered with multiple target groups. This is useful when different listeners or rules need to route to the same backend with different configurations.

---

### Q24. How does deregistration delay work?

**Answer:** When a target is deregistered, the load balancer stops sending new connections to it while allowing existing connections to continue for the configured deregistration delay. After the delay, remaining connections are closed according to the load balancer behavior.

---
