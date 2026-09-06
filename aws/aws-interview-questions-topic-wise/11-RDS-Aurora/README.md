# RDS & Aurora

[⬅️ Back to AWS Topics](../README.md)

## 🔑 Keywords

🗄️ RDS | Multi-AZ | Read Replica | Backup | PITR | Aurora | Replica | Global Database

## 🧠 Core Memory

🧠 **Remember:** **Multi-AZ = HA/failover**, **Read Replica = read scaling**.

---

## ❓ Interview Questions

### 📌 RDS

#### Q1. What is Amazon RDS?

**💡 Answer:** Amazon RDS is a managed relational database service. AWS handles much of the underlying infrastructure, backups, patching, and high-availability plumbing while you manage database data, schema, and configuration.

**🔑 Keywords:** `RDS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q2. Which responsibilities remain with AWS in RDS?

**💡 Answer:** Amazon RDS is a managed relational database service. AWS handles much of the underlying infrastructure, backups, patching, and high-availability plumbing while you manage database data, schema, and configuration.

**🔑 Keywords:** `RDS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q3. Which responsibilities remain with the customer?

**💡 Answer:** Explain the managed relational-database behavior, then distinguish availability, read scaling, backup/recovery, and operational responsibilities.

**🔑 Keywords:** `RDS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q4. What engines does RDS support?

**💡 Answer:** Amazon RDS is a managed relational database service. AWS handles much of the underlying infrastructure, backups, patching, and high-availability plumbing while you manage database data, schema, and configuration.

**🔑 Keywords:** `RDS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q5. What is a DB subnet group?

**💡 Answer:** A subnet is an IP address range inside a VPC and is associated with one Availability Zone. A subnet is considered public when its route table provides a path to an Internet Gateway; otherwise it is commonly private.

**🔑 Keywords:** `RDS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 High Availability

#### Q6. What is RDS Multi-AZ?

**💡 Answer:** Amazon RDS is a managed relational database service. AWS handles much of the underlying infrastructure, backups, patching, and high-availability plumbing while you manage database data, schema, and configuration.

**🔑 Keywords:** `RDS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q7. What is a standby instance?

**💡 Answer:** Explain the managed relational-database behavior, then distinguish availability, read scaling, backup/recovery, and operational responsibilities.

**🔑 Keywords:** `RDS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q8. Does Multi-AZ primarily provide read scaling?

**💡 Answer:** RDS Multi-AZ is primarily a high-availability and failover capability. RDS maintains a standby in another Availability Zone and can fail over when the primary becomes unavailable; the standby is not the normal read-scaling mechanism.

**🔑 Keywords:** `RDS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q9. What is an RDS read replica?

**💡 Answer:** Amazon RDS is a managed relational database service. AWS handles much of the underlying infrastructure, backups, patching, and high-availability plumbing while you manage database data, schema, and configuration.

**🔑 Keywords:** `RDS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q10. How does a read replica differ from Multi-AZ?

**💡 Answer:** RDS Multi-AZ is primarily a high-availability and failover capability. RDS maintains a standby in another Availability Zone and can fail over when the primary becomes unavailable; the standby is not the normal read-scaling mechanism.

**🔑 Keywords:** `RDS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Aurora

#### Q11. What is Amazon Aurora?

**💡 Answer:** Amazon Aurora is a MySQL- and PostgreSQL-compatible relational database engine designed for AWS. It separates compute from a distributed storage layer and supports replicas and managed high availability.

**🔑 Keywords:** `RDS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q12. How does Aurora storage differ from standard RDS storage?

**💡 Answer:** Amazon RDS is a managed relational database service. AWS handles much of the underlying infrastructure, backups, patching, and high-availability plumbing while you manage database data, schema, and configuration.

**🔑 Keywords:** `RDS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q13. What are Aurora Replicas?

**💡 Answer:** Amazon Aurora is a MySQL- and PostgreSQL-compatible relational database engine designed for AWS. It separates compute from a distributed storage layer and supports replicas and managed high availability.

**🔑 Keywords:** `RDS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q14. What is Aurora Serverless?

**💡 Answer:** Amazon Aurora is a MySQL- and PostgreSQL-compatible relational database engine designed for AWS. It separates compute from a distributed storage layer and supports replicas and managed high availability.

**🔑 Keywords:** `RDS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q15. What is Aurora Global Database?

**💡 Answer:** Amazon Aurora is a MySQL- and PostgreSQL-compatible relational database engine designed for AWS. It separates compute from a distributed storage layer and supports replicas and managed high availability.

**🔑 Keywords:** `RDS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Operations

#### Q16. What is automated backup?

**💡 Answer:** RDS automated backups provide point-in-time recovery within the configured retention period. AWS creates and manages the underlying backup data.

**🔑 Keywords:** `RDS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q17. What is a DB snapshot?

**💡 Answer:** An EBS snapshot is a point-in-time backup of an EBS volume. Snapshots are incremental after the first snapshot and can be copied across Regions for disaster-recovery designs.

**🔑 Keywords:** `RDS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q18. What is point-in-time recovery?

**💡 Answer:** Point-in-time recovery restores an RDS database to a selected time within the available automated-backup retention window.

**🔑 Keywords:** `RDS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q19. What is a maintenance window?

**💡 Answer:** Explain the managed relational-database behavior, then distinguish availability, read scaling, backup/recovery, and operational responsibilities.

**🔑 Keywords:** `RDS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q20. What is RDS encryption?

**💡 Answer:** Amazon RDS is a managed relational database service. AWS handles much of the underlying infrastructure, backups, patching, and high-availability plumbing while you manage database data, schema, and configuration.

**🔑 Keywords:** `RDS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q21. How do parameter groups and option groups differ?

**💡 Answer:** A DB parameter group controls database engine configuration parameters. An option group is used by supported RDS engines to enable engine-specific options and features.

**🔑 Keywords:** `RDS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

## 🚀 Last-Minute Revision

> 🧠 **Remember:** **Multi-AZ = HA/failover**, **Read Replica = read scaling**.

[⬆️ Back to top](#rds-aurora)

[⬅️ Back to AWS Topics](../README.md)