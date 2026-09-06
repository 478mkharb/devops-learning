# AWS Architecture & Design

### Q1. What are the AWS Well-Architected Framework pillars?

**Answer:** The AWS Well-Architected Framework provides architectural guidance around six pillars: operational excellence, security, reliability, performance efficiency, cost optimization, and sustainability.

---

### Q2. What is reliability?

**Answer:** Reliability focuses on a workload's ability to recover from failures, meet demand, and operate correctly through resilient architecture, monitoring, recovery, and controlled change.

---

### Q3. What is operational excellence?

**Answer:** Operational excellence focuses on running and monitoring workloads effectively, automating operations, learning from events, and continually improving processes.

---

### Q4. What is security?

**Answer:** The security pillar focuses on protecting information and systems through strong identity controls, detection, infrastructure protection, data protection, and incident response.

---

### Q5. What is performance efficiency?

**Answer:** Performance efficiency focuses on using computing resources efficiently and selecting appropriate technologies and architectures as requirements change.

---

### Q6. What is cost optimization?

**Answer:** Cost optimization focuses on delivering business value at the lowest appropriate cost through rightsizing, elasticity, pricing models, storage optimization, and ongoing cost visibility.

---

### Q7. What is sustainability?

**Answer:** Sustainability focuses on reducing the environmental impact of workloads through efficient resource use, appropriate architectures, and minimizing unnecessary consumption.

---

### Q8. How do Multi-AZ architectures improve availability?

**Answer:** RDS Multi-AZ is primarily a high-availability and failover feature. RDS maintains a standby in another Availability Zone and can fail over when the primary becomes unavailable; the standby is not normally used for read scaling.

---

### Q9. Why distribute workloads across AZs?

**Answer:** Distributing workloads across Availability Zones reduces dependence on a single failure domain. If one AZ experiences an infrastructure or power/network failure, resources in other AZs can continue serving traffic, improving availability and resilience.

---

### Q10. How does an ALB improve application availability?

**Answer:** An ALB distributes requests across healthy targets and can span multiple Availability Zones. It continuously performs target health checks and stops routing to unhealthy targets, allowing healthy instances to continue serving traffic when individual instances fail.

---

### Q11. How does ASG improve resilience?

**Answer:** An ASG improves resilience by maintaining desired capacity, replacing unhealthy instances, and scaling the fleet when demand changes. Deploying the ASG across multiple Availability Zones prevents a single instance or AZ failure from removing the entire application capacity.

---

### Q12. What is horizontal scaling?

**Answer:** Horizontal scaling adds or removes instances or workers to change capacity. It is commonly implemented with Auto Scaling Groups, load balancers, queues, and distributed services.

---

### Q13. What is vertical scaling?

**Answer:** Vertical scaling changes the size of an individual resource, such as moving to a larger EC2 instance. It is simple but constrained by instance limits and can require downtime depending on the resource.

---

### Q14. How do queues decouple workloads?

**Answer:** Queues decouple workloads by allowing producers to submit work without waiting for consumers to process it. Consumers can scale independently, traffic spikes can be absorbed by the queue, and temporary consumer failures do not necessarily stop producers.

---

### Q15. How can caching reduce database load?

**Answer:** Caching reduces database load by serving frequently requested data from a faster cache instead of querying the database for every request. This reduces database CPU, connections, and I/O while improving response latency.

---

### Q16. How do read replicas support read scaling?

**Answer:** Read replicas support read scaling by maintaining additional readable copies of a database. Applications can route eligible read traffic to replicas, reducing read load on the primary database. Replication lag must be considered where strong read-after-write consistency is required.

---

### Q17. What is backup and restore?

**Answer:** Backup and restore periodically copies data and recreates or restores the workload after a failure. It is usually the simplest DR strategy but often has the longest recovery time.

---

### Q18. What is pilot light?

**Answer:** A pilot-light DR strategy keeps only the core components required to recreate the workload continuously available. During recovery, additional infrastructure is started and configured.

---

### Q19. What is warm standby?

**Answer:** Warm standby maintains a scaled-down but functional copy of the workload in the recovery environment. It can recover faster than backup-and-restore but costs more.

---

### Q20. What is active-active?

**Answer:** An active-active architecture runs production capacity in multiple environments or Regions simultaneously. It can provide very low recovery time but requires more complex data synchronization and operations.

---

### Q21. How do RTO and RPO influence architecture?

**Answer:** RTO (Recovery Time Objective) is the maximum acceptable time to restore a workload after a disruption.

---

### Q22. How would you design a highly available 3-tier application?

**Answer:** For a highly available 3-tier application, place an internet-facing ALB across multiple Availability Zones, application servers in private subnets across those AZs using an ASG, and a Multi-AZ relational database in private database subnets. Use security groups between tiers, NAT Gateways or VPC endpoints for required outbound/service access, IAM roles for workloads, backups, monitoring, and health checks.

---

### Q23. How would you design a private application with outbound Internet access?

**Answer:** Place the application in private subnets with no direct route to an Internet Gateway. Put a NAT Gateway in a public subnet and route the private subnet's Internet-bound traffic to it. The public subnet routes to the Internet Gateway. Use security groups, NACLs as required, and VPC endpoints for supported AWS services when private access is preferable.

---

### Q24. How would you design a decoupled order-processing system?

**Answer:** Use an API or application tier to validate the order and publish an order message to SQS. Worker instances or Lambda consume the queue asynchronously and update the database or downstream services. Add a DLQ, idempotent processing, monitoring, and scaling based on queue depth so temporary downstream problems do not block order submission.

---

### Q25. How would you design a multi-Region application?

**Answer:** A multi-Region application deploys application capacity in multiple AWS Regions and uses an appropriate global routing mechanism such as Route 53 to direct users. Data must be replicated using a service and consistency model appropriate to the workload. The design must define RTO, RPO, failover, DNS TTL, deployment, and rollback procedures.

---

### Q26. How do you choose between managed services and self-managed infrastructure?

**Answer:** Prefer managed services when they satisfy the requirement because AWS handles more infrastructure operations such as patching, scaling, backups, and availability. Choose self-managed infrastructure when the workload requires capabilities or control that the managed service cannot provide, while accepting the additional operational responsibility.

---
