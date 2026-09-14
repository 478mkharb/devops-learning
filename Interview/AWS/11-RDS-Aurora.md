# RDS & Aurora

### Q1. What is Amazon RDS?

**Answer:** Amazon RDS is a managed relational database service. AWS manages much of the underlying infrastructure and managed-service operations, while the customer manages database data, schema, users, queries, and workload-specific configuration.

---

### Q2. Which responsibilities remain with AWS in RDS?

**Answer:** AWS manages the underlying RDS infrastructure and managed-service operations defined by the selected engine and configuration, including provisioning, hardware, storage infrastructure, and many backup, patching, and availability tasks. The exact responsibility boundary depends on the engine and features used.

---

### Q3. Which responsibilities remain with the customer?

**Answer:** The customer remains responsible for database schema and data, database users and permissions, application queries, data modeling, and workload-specific configuration. The customer also chooses appropriate sizing, backup retention, availability, networking, and security settings.

---

### Q4. What database engines does Amazon RDS support?

**Answer:** Amazon RDS supports managed relational database engines including **Amazon Aurora, PostgreSQL, MySQL, MariaDB, Oracle Database, and Microsoft SQL Server**. Available engine versions and features vary by Region.

| Engine | Type |
|---|---|
| **Amazon Aurora** | MySQL- and PostgreSQL-compatible |
| **PostgreSQL** | Open-source relational database |
| **MySQL** | Open-source relational database |
| **MariaDB** | Open-source relational database |
| **Oracle** | Commercial relational database |
| **Microsoft SQL Server** | Commercial relational database |

---

### Q5. What is an RDS DB subnet group?

**Answer:** An RDS DB subnet group is a collection of subnets in a VPC that RDS can use for database instances. For a Multi-AZ deployment, the subnet group should include subnets in at least two Availability Zones.

---

### Q6. What is RDS Multi-AZ?

**Answer:** RDS Multi-AZ is primarily a **high-availability and failover** feature. RDS maintains a standby database in another Availability Zone and can automatically fail over when the primary becomes unavailable.

The standby is not normally used to serve application read traffic.

---

### Q7. What is an RDS standby instance?

**Answer:** An RDS Multi-AZ standby is a synchronized standby database instance maintained in another Availability Zone for high availability and automatic failover. It is not normally used as a read-scaling target.

---

### Q8. Does RDS Multi-AZ provide read scaling?

**Answer:** No. Traditional RDS Multi-AZ is primarily designed for high availability and automatic failover. The standby is not normally used for application read traffic.

For read scaling, use **RDS Read Replicas** or, where appropriate, Aurora Replicas.

---

### Q9. What is an RDS Read Replica?

**Answer:** An RDS Read Replica is a separate database instance that receives replicated changes from a source database and can serve read traffic. It is primarily used to scale read workloads and can also support certain migration and disaster-recovery architectures.

---

### Q10. What is the difference between RDS Multi-AZ and Read Replicas?

**Answer:** Multi-AZ and Read Replicas solve different problems.

| Feature | Multi-AZ | Read Replica |
|---|---|---|
| Primary purpose | High availability and failover | Read scaling |
| Read traffic | Standby is not normally used | Replica can serve reads |
| Failover | Automatic failover capability | Not primarily a HA standby mechanism |
| Location | Commonly another AZ | Can be same or different AZ/Region depending on engine/configuration |
| Replication purpose | Maintain standby | Replicate data for readable copy |
| Typical use | Production HA | Scale read-heavy workloads |

---

### Q11. What is Amazon Aurora?

**Answer:** Amazon Aurora is an AWS-managed relational database engine compatible with MySQL and PostgreSQL. Aurora uses a distributed storage architecture and supports features such as Aurora Replicas, high availability, and Aurora Global Database.

---

### Q12. How does Aurora storage differ from standard RDS storage?

**Answer:** Aurora separates database compute from its distributed storage subsystem. Aurora storage automatically maintains multiple copies across Availability Zones.

| Aspect | Traditional RDS engines | Aurora |
|---|---|---|
| Storage architecture | Storage associated with DB instances | Distributed cluster storage |
| Storage replication | Depends on engine/HA configuration | Built into Aurora storage architecture |
| Compute and storage | More closely associated | More decoupled |
| Read scaling | Read Replicas | Aurora Replicas |
| Typical advantage | Broad engine choice | AWS-native distributed architecture and scaling features |

---

### Q13. What are Aurora Replicas?

**Answer:** Aurora Replicas are read-capable Aurora DB instances in an Aurora cluster that share the cluster's distributed storage. They can serve read traffic and improve availability by providing additional database instances.

---

### Q14. What is Aurora Serverless?

**Answer:** Aurora Serverless provides an automatically scaling compute-capacity model for supported Aurora configurations. It is useful for workloads with variable or unpredictable demand where continuously running fixed database capacity may not be appropriate.

---

### Q15. What is Aurora Global Database?

**Answer:** Aurora Global Database provides a primary Aurora Region and one or more read-only secondary Regions with managed cross-Region replication. It is designed for globally distributed applications that need lower read latency in multiple Regions and faster regional disaster recovery.

Example:

```text
                 Aurora Global Database
                         |
             ┌───────────┴───────────┐
             ↓                       ↓
       Primary Region          Secondary Region
          us-east-1               eu-west-1
             |                       |
          Read/Write               Read
```

---

### Q16. What are RDS automated backups?

**Answer:** RDS automated backups provide point-in-time recovery within the configured backup retention period. AWS manages the backup process for the DB instance.

---

### Q17. What is an RDS DB snapshot?

**Answer:** An RDS DB snapshot is a user-initiated backup of a database instance. Unlike automated backups, a user-created snapshot is retained until it is explicitly deleted and can be used to restore a new DB instance.

---

### Q18. What is point-in-time recovery in RDS?

**Answer:** Point-in-time recovery allows an RDS database to be restored to a specific point in time within the available automated-backup retention window.

```text
Automated backups + transaction logs
                 |
                 ↓
        Selected point in time
                 |
                 ↓
        Restored DB instance
```

---

### Q19. What is the difference between an automated backup and a DB snapshot?

**Answer:** Both can support database recovery, but they have different retention and recovery characteristics.

| Feature | Automated backup | DB snapshot |
|---|---|---|
| Created by | RDS automatically | Customer manually |
| Point-in-time recovery | Yes, within retention window | No direct PITR from a single snapshot |
| Retention | Configured backup retention | Until manually deleted |
| Main use | Operational backup and PITR | Long-term backup, migration, cloning, restore |
| Management | AWS-managed backup process | Customer controls snapshot lifecycle |

---

### Q20. What is an RDS maintenance window?

**Answer:** An RDS maintenance window is the preferred time period during which AWS can apply certain maintenance operations, such as patches or infrastructure changes, when required.

It helps customers choose a period that minimizes the potential impact on application traffic.

---

### Q21. What is RDS encryption?

**Answer:** RDS encryption protects database storage and supported backups and snapshots using AWS KMS. Encryption is normally enabled when the DB instance or cluster is created, and the selected KMS key is used for encryption key management.

---

### Q22. How do parameter groups and option groups differ?

**Answer:** A **DB parameter group** controls database engine parameters that affect database behavior, while an **option group** enables supported engine-specific options or features.

| Feature | Parameter Group | Option Group |
|---|---|---|
| Purpose | Configure database parameters | Enable supported engine-specific options |
| Example | Memory or connection-related parameters | Engine-specific features/options |
| Applies to | Supported RDS engines | Only engines that support option groups |
| Configuration type | Parameter values | Options/features |
