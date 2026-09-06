# RDS & Aurora

### Q1. What is Amazon RDS?

**Answer:** Amazon RDS is a managed relational database service. AWS manages much of the underlying infrastructure, provisioning, backups, patching, and high-availability plumbing while the customer manages database data, schema, and configuration.

---

### Q2. Which responsibilities remain with AWS in RDS?

**Answer:** AWS manages the underlying RDS infrastructure and the managed service operations defined by the selected configuration, including provisioning, hardware, storage infrastructure, and many backup, patching, and availability tasks. The exact boundary depends on the RDS engine and features used.

---

### Q3. Which responsibilities remain with the customer?

**Answer:** The customer remains responsible for the database schema and data, database users and permissions, application queries, data modeling, and workload-specific configuration. The customer also chooses availability, backup retention, sizing, and security settings appropriate to the application.

---

### Q4. What engines does RDS support?

**Answer:** Amazon RDS supports managed relational engines including Amazon Aurora, PostgreSQL, MySQL, MariaDB, Oracle Database, and Microsoft SQL Server. Exact engine versions and features vary by Region.

---

### Q5. What is a DB subnet group?

**Answer:** An RDS DB subnet group is a collection of subnets in a VPC that RDS can use for database instances. For high availability, it should span at least two Availability Zones.

---

### Q6. What is RDS Multi-AZ?

**Answer:** RDS Multi-AZ is primarily a high-availability and failover feature. RDS maintains a standby in another Availability Zone and can fail over when the primary becomes unavailable; the standby is not normally used for read scaling.

---

### Q7. What is a standby instance?

**Answer:** An RDS Multi-AZ standby is a synchronized standby database instance maintained in another Availability Zone for high availability and automatic failover. It is not the normal target for application read traffic.

---

### Q8. Does Multi-AZ primarily provide read scaling?

**Answer:** No. RDS Multi-AZ primarily provides high availability and automatic failover. The standby is maintained for failover and is not normally used by applications for read traffic; read replicas are used for read scaling.

---

### Q9. What is an RDS read replica?

**Answer:** An RDS read replica is a separate database instance that receives replicated changes from a source and can serve read traffic. It is primarily used for read scaling and some migration or DR patterns.

---

### Q10. How does a read replica differ from Multi-AZ?

**Answer:** A read replica is a separate readable database instance used primarily to scale read traffic and can also support migration or DR scenarios. Multi-AZ primarily maintains a standby for high availability and automatic failover.

---

### Q11. What is Amazon Aurora?

**Answer:** Amazon Aurora is a MySQL- and PostgreSQL-compatible relational database engine built for AWS. It uses a distributed storage architecture and supports replicas, high availability, and global database features.

---

### Q12. How does Aurora storage differ from standard RDS storage?

**Answer:** Aurora uses a distributed storage subsystem that automatically replicates data across multiple Availability Zones and separates storage from database compute. Traditional RDS engines generally use storage attached to individual DB instances, with high availability and read scaling provided through separate mechanisms.

---

### Q13. What are Aurora Replicas?

**Answer:** Aurora Replicas are read-capable compute instances in an Aurora cluster that share the cluster's distributed storage. They can scale reads and improve availability.

---

### Q14. What is Aurora Serverless?

**Answer:** Aurora Serverless provides an automatically scaling database-capacity model for supported Aurora versions. It is useful when demand is variable or unpredictable and you do not want to manage fixed database capacity.

---

### Q15. What is Aurora Global Database?

**Answer:** Aurora Global Database provides a primary Region and read-only secondary Regions with managed cross-Region replication. It is designed for global read latency and disaster recovery.

---

### Q16. What is automated backup?

**Answer:** RDS automated backups support point-in-time recovery within the configured retention period. AWS manages the backup process for the database instance.

---

### Q17. What is a DB snapshot?

**Answer:** An RDS DB snapshot is a user-initiated backup of a database instance. It is retained until deleted and can be used to restore a new DB instance.

---

### Q18. What is point-in-time recovery?

**Answer:** Point-in-time recovery restores an RDS database to a selected time within the available automated-backup retention window.

---

### Q19. What is a maintenance window?

**Answer:** An RDS maintenance window is the preferred time period during which AWS can apply certain maintenance operations, such as patches or infrastructure changes, when required.

---

### Q20. What is RDS encryption?

**Answer:** RDS encryption protects database storage, automated backups, read replicas in supported configurations, and snapshots using AWS KMS. Encryption is normally selected when the DB instance is created, and a KMS key controls the encryption operations.

---

### Q21. How do parameter groups and option groups differ?

**Answer:** A parameter group controls database engine configuration parameters, such as settings that affect database behavior. An option group enables supported engine-specific features or options. They serve different configuration purposes and availability depends on the RDS engine.

---
