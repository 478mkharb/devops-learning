# AWS Architecture & Design

## Interview Questions & Answers

### Q1. What are the AWS Well-Architected Framework pillars?

**Answer:** The AWS Well-Architected Framework organizes architectural guidance around operational excellence, security, reliability, performance efficiency, cost optimization, and sustainability.

---

### Q2. What is reliability?

**Answer:** Reliability focuses on a workload's ability to recover from failures, meet demand, and operate correctly through resilient architecture, monitoring, recovery, and change management.

---

### Q3. What is operational excellence?

**Answer:** Operational excellence focuses on running and monitoring systems effectively, automating operations, learning from events, and continually improving processes.

---

### Q4. What is security?

**Answer:** Security in the AWS Well-Architected Framework focuses on protecting information and systems through strong identity and access control, detection, infrastructure protection, data protection, and incident response.

---

### Q5. What is performance efficiency?

**Answer:** Performance efficiency focuses on using computing resources efficiently and selecting appropriate architectures, technologies, and scaling strategies as requirements change.

---

### Q6. What is cost optimization?

**Answer:** Cost Optimization focuses on delivering business value at the lowest appropriate cost. Practices include cost visibility, rightsizing, appropriate purchasing models, lifecycle policies, eliminating idle resources, and choosing efficient architectures.

---

### Q7. What is sustainability?

**Answer:** Sustainability focuses on reducing the environmental impact of workloads by improving resource utilization, selecting efficient architectures, and minimizing unnecessary resource consumption.

---

### Q8. How do Multi-AZ architectures improve availability?

**Answer:** RDS Multi-AZ is primarily a high-availability and failover capability. RDS maintains a standby in another Availability Zone and can fail over when the primary becomes unavailable; the standby is not the normal read-scaling mechanism.

---

### Q9. Why distribute workloads across AZs?

**Answer:** Availability Zones are separate failure domains within a Region. Distributing application capacity across multiple AZs reduces the chance that a single AZ failure removes all capacity and supports highly available designs.

---

### Q10. How does an ALB improve application availability?

**Answer:** An ALB distributes requests across healthy targets in multiple Availability Zones and stops routing to unhealthy targets. Combined with an ASG, it supports automatic replacement and horizontal scaling of application instances.

---

### Q11. How does ASG improve resilience?

**Answer:** An ASG maintains desired capacity, replaces unhealthy instances, and scales capacity according to demand. When instances are distributed across Availability Zones, the ASG can maintain service capacity during individual instance or AZ failures.

---

### Q12. What is horizontal scaling?

**Answer:** Horizontal scaling adds or removes instances or workers to change capacity. It is commonly implemented with Auto Scaling Groups, load balancers, queues, and distributed services.

---

### Q13. What is vertical scaling?

**Answer:** Vertical scaling increases or decreases the size of an individual resource, such as moving to a larger EC2 instance. It is simple but has limits and may require downtime depending on the resource.

---

### Q14. How do queues decouple workloads?

**Answer:** A queue separates producers from consumers and buffers work. Producers can continue during temporary consumer slowdowns, while consumers process messages asynchronously at their own rate. SQS is the common AWS implementation.

---

### Q15. How can caching reduce database load?

**Answer:** A cache stores frequently requested data closer to the application. Cache hits avoid repeated database reads, reducing database CPU/I/O and improving response latency. The application must define appropriate expiration and invalidation behavior.

---

### Q16. How do read replicas support read scaling?

**Answer:** An RDS read replica is a separate database instance that receives replicated changes from a source database and can serve read traffic. It is primarily used for read scaling and some migration/DR patterns, not the same purpose as a Multi-AZ standby.

---

### Q17. What is backup and restore?

**Answer:** Backup and restore periodically copies data and recreates infrastructure or restores data after a disaster. It is usually the simplest DR strategy but can have the highest recovery time.

---

### Q18. What is pilot light?

**Answer:** Pilot light keeps only the core components required to recreate the workload running, with other capacity started during recovery. It reduces cost compared with warm standby but increases recovery work.

---

### Q19. What is warm standby?

**Answer:** Warm standby maintains a scaled-down but functional copy of the workload in the recovery environment. It can recover faster than backup-and-restore while costing more.

---

### Q20. What is active-active?

**Answer:** Active-active runs production workloads in multiple environments or Regions simultaneously. It can provide very low recovery time but is more complex to operate and synchronize.

---

### Q21. How do RTO and RPO influence architecture?

**Answer:** RTO (Recovery Time Objective) is the maximum acceptable time to restore service after a disruption.

---

### Q22. How would you design a highly available 3-tier application?

**Answer:** Use Route 53 for DNS, an internet-facing ALB across multiple Availability Zones, stateless application servers in private subnets managed by an ASG, and a Multi-AZ database in private subnets. Use NAT Gateways or VPC endpoints for required outbound/service access and CloudWatch for monitoring.

---

### Q23. How would you design a private application with outbound Internet access?

**Answer:** Place application instances in private subnets with no direct route to an Internet Gateway. Route Internet-bound traffic to a NAT Gateway in a public subnet. Use VPC endpoints for supported AWS services when private service access is preferable.

---

### Q24. How would you design a decoupled order-processing system?

**Answer:** Expose an API through API Gateway or an ALB, persist the order, publish asynchronous work to SQS, and process it with Lambda or EC2 workers. Use a DLQ for repeated failures, idempotent consumers, and CloudWatch monitoring.

---

### Q25. How would you design a multi-Region application?

**Answer:** Deploy application capacity in multiple Regions, replicate data with a service-appropriate mechanism, and use Route 53 or another global routing mechanism for traffic management. Define consistency, failover, RTO, and RPO requirements before choosing active-passive or active-active architecture.

---

### Q26. How do you choose between managed services and self-managed infrastructure?

**Answer:** Prefer managed services when they meet requirements because they reduce operational work for patching, scaling, backups, and availability. Choose self-managed infrastructure when a required feature, compatibility constraint, performance requirement, or control requirement justifies the additional operational burden.

---
