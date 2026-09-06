# AWS Migration

### Q1. What is AWS Application Migration Service?

**Answer:** AWS Application Migration Service (MGN) provides automated lift-and-shift migration for servers by continuously replicating source machines and launching them as EC2 instances for testing and cutover.

---

### Q2. What is Database Migration Service?

**Answer:** AWS Database Migration Service (DMS) migrates and replicates databases. It supports homogeneous and heterogeneous migrations and can use ongoing replication to reduce downtime.

---

### Q3. What is Migration Hub?

**Answer:** AWS Migration Hub provides a central view for tracking migration progress across supported AWS migration tools and workloads.

---

### Q4. What is DataSync?

**Answer:** AWS DataSync is a managed high-speed data transfer service for moving data between on-premises storage and AWS storage services or between supported AWS storage locations.

---

### Q5. What is Snowball?

**Answer:** AWS Snowball is a physical data-transfer service/device used to move large datasets to or from AWS when network transfer is impractical or too slow.

---

### Q6. What is rehost?

**Answer:** Rehost, or lift-and-shift, moves a workload to AWS with minimal changes. It is usually fast and low-risk but may not fully use cloud-native capabilities.

---

### Q7. What is replatform?

**Answer:** Replatform moves a workload to AWS with limited modifications that provide operational, performance, or cost benefits without a full redesign.

---

### Q8. What is refactor?

**Answer:** Refactor, or re-architect, substantially changes the application to use cloud-native architecture. It can provide greater long-term benefits but requires more engineering effort.

---

### Q9. What is repurchase?

**Answer:** Repurchase replaces an existing product or licensing model with a different solution, often moving to a cloud-based or SaaS product.

---

### Q10. What is retain?

**Answer:** Retain means keeping a workload in its existing environment for now because migration is not currently justified, feasible, or strategically required.

---

### Q11. What is retire?

**Answer:** Retire means decommissioning a workload that is no longer needed rather than migrating it.

---

### Q12. When is Snowball preferable to network transfer?

**Answer:** Snowball is preferable when the dataset is large enough that transferring it over the available network would take too long, cost too much, or consume unacceptable network capacity. Physical transfer can be more practical for bulk migrations.

---

### Q13. When should DMS be used?

**Answer:** Use AWS DMS when you need to migrate or continuously replicate data between supported source and target databases. It is especially useful when minimizing downtime because ongoing replication can keep the target synchronized before the final cutover.

---

### Q14. How can migration minimize downtime?

**Answer:** Minimize migration downtime by pre-provisioning the target, continuously replicating data where supported, testing the destination, keeping the source available during synchronization, and performing a controlled final cutover. For databases, DMS ongoing replication is a common approach.

---

### Q15. What is a cutover?

**Answer:** A cutover is the controlled transition from the source environment to the target environment. A good cutover includes final synchronization, validation, traffic or connection switching, monitoring, and a tested rollback plan.

---
