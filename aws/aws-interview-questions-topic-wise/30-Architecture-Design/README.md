# AWS Architecture & Design

[⬅️ Back to AWS Topics](../README.md)

## 🔑 Keywords

🏛️ Well-Architected | HA | Scalability | RTO | RPO | Backup | Pilot Light | Warm Standby | Active-Active

## 🧠 Core Memory

🧠 **Remember:** **HA + Scale + Resilience + Security + Cost + Performance**; RTO = recovery time, RPO = acceptable data loss.

---

## ❓ Interview Questions

### 📌 Well-Architected

#### Q1. What are the AWS Well-Architected Framework pillars?

**💡 Answer:** The AWS Well-Architected Framework organizes architectural guidance around operational excellence, security, reliability, performance efficiency, cost optimization, and sustainability.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q2. What is reliability?

**💡 Answer:** Reliability focuses on a workload's ability to recover from failures, meet demand, and operate correctly through resilient architecture, monitoring, recovery, and change management.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q3. What is operational excellence?

**💡 Answer:** Operational excellence focuses on running and monitoring systems effectively, automating operations, learning from events, and continually improving processes.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q4. What is security?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q5. What is performance efficiency?

**💡 Answer:** Performance efficiency focuses on using computing resources efficiently and selecting appropriate architectures, technologies, and scaling strategies as requirements change.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q6. What is cost optimization?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q7. What is sustainability?

**💡 Answer:** Sustainability focuses on reducing the environmental impact of workloads by improving resource utilization, selecting efficient architectures, and minimizing unnecessary resource consumption.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 High Availability

#### Q8. How do Multi-AZ architectures improve availability?

**💡 Answer:** RDS Multi-AZ is primarily a high-availability and failover capability. RDS maintains a standby in another Availability Zone and can fail over when the primary becomes unavailable; the standby is not the normal read-scaling mechanism.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q9. Why distribute workloads across AZs?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q10. How does an ALB improve application availability?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q11. How does ASG improve resilience?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Scalability

#### Q12. What is horizontal scaling?

**💡 Answer:** Horizontal scaling adds or removes instances or workers to change capacity. It is commonly implemented with Auto Scaling Groups, load balancers, queues, and distributed services.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q13. What is vertical scaling?

**💡 Answer:** Vertical scaling increases or decreases the size of an individual resource, such as moving to a larger EC2 instance. It is simple but has limits and may require downtime depending on the resource.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q14. How do queues decouple workloads?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q15. How can caching reduce database load?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q16. How do read replicas support read scaling?

**💡 Answer:** An RDS read replica is a separate database instance that receives replicated changes from a source database and can serve read traffic. It is primarily used for read scaling and some migration/DR patterns, not the same purpose as a Multi-AZ standby.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Disaster Recovery

#### Q17. What is backup and restore?

**💡 Answer:** Backup and restore periodically copies data and recreates infrastructure or restores data after a disaster. It is usually the simplest DR strategy but can have the highest recovery time.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q18. What is pilot light?

**💡 Answer:** Pilot light keeps only the core components required to recreate the workload running, with other capacity started during recovery. It reduces cost compared with warm standby but increases recovery work.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q19. What is warm standby?

**💡 Answer:** Warm standby maintains a scaled-down but functional copy of the workload in the recovery environment. It can recover faster than backup-and-restore while costing more.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q20. What is active-active?

**💡 Answer:** Active-active runs production workloads in multiple environments or Regions simultaneously. It can provide very low recovery time but is more complex to operate and synchronize.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q21. How do RTO and RPO influence architecture?

**💡 Answer:** RTO (Recovery Time Objective) is the maximum acceptable time to restore service after a disruption.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Design

#### Q22. How would you design a highly available 3-tier application?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q23. How would you design a private application with outbound Internet access?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q24. How would you design a decoupled order-processing system?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q25. How would you design a multi-Region application?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q26. How do you choose between managed services and self-managed infrastructure?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

## 🚀 Last-Minute Revision

> 🧠 **Remember:** **HA + Scale + Resilience + Security + Cost + Performance**; RTO = recovery time, RPO = acceptable data loss.

[⬆️ Back to top](#aws-architecture-design)

[⬅️ Back to AWS Topics](../README.md)