# AWS Migration

## Interview Questions & Answers

### Q1. What is AWS Application Migration Service?

**Answer:** AWS Application Migration Service (MGN) automates lift-and-shift migration of servers into AWS by replicating source servers and launching them as AWS instances.

---

### Q2. What is Database Migration Service?

**Answer:** AWS Database Migration Service (DMS) migrates and replicates databases. It supports homogeneous and heterogeneous migrations and can use ongoing replication to reduce downtime.

---

### Q3. What is Migration Hub?

**Answer:** AWS Migration Hub provides a central place to track migration progress across supported migration tools and workloads.

---

### Q4. What is DataSync?

**Answer:** AWS DataSync transfers data between on-premises storage and AWS storage services or between supported AWS storage locations. It is optimized for high-speed managed data transfer.

---

### Q5. What is Snowball?

**Answer:** AWS Snowball is a physical data-transfer device/service used to move large datasets to or from AWS when network transfer is impractical or too slow.

---

### Q6. What is rehost?

**Answer:** Rehost, or lift-and-shift, moves workloads to AWS with minimal application changes. It is usually fast and low-risk but may not fully exploit cloud-native capabilities.

---

### Q7. What is replatform?

**Answer:** Replatform moves a workload to AWS with limited modifications that provide operational or cost benefits without a full redesign.

---

### Q8. What is refactor?

**Answer:** Refactor, or re-architect, substantially changes the application to use cloud-native architecture. It can deliver larger long-term benefits but requires more engineering effort.

---

### Q9. What is repurchase?

**Answer:** Repurchase replaces an existing solution with a different product or service, often moving to a cloud-based licensing or SaaS model.

---

### Q10. What is retain?

**Answer:** Retain means keeping a workload in its current environment for now because migration is not currently justified or feasible.

---

### Q11. What is retire?

**Answer:** Retire means decommissioning a workload that is no longer needed instead of migrating it.

---

### Q12. When is Snowball preferable to network transfer?

**Answer:** AWS Snowball is a physical data-transfer device/service used to move large datasets to or from AWS when network transfer is impractical or too slow.

---

### Q13. When should DMS be used?

**Answer:** Use AWS Database Migration Service when migrating or continuously replicating supported databases. DMS can perform an initial load and ongoing change data capture, which helps reduce application downtime during migration.

---

### Q14. How can migration minimize downtime?

**Answer:** Use replication-based migration where possible, perform testing before cutover, keep the source available during synchronization, complete a final change-data synchronization, and switch application traffic only after the target is validated. DNS TTL reduction can also help for DNS-based cutovers.

---

### Q15. What is a cutover?

**Answer:** Cutover is the controlled point at which production traffic or users are switched from the old environment to the migrated AWS environment after validation and final synchronization.

---
