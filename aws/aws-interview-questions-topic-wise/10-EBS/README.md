# EBS

## Interview Questions & Answers

### Q1. What is Amazon EBS?

**Answer:** Amazon EBS provides persistent block storage for EC2. EBS volumes can be used for boot disks, filesystems, and databases, and snapshots can provide backup and recovery.

---

### Q2. What is an EBS volume?

**Answer:** Amazon EBS provides persistent block storage for EC2. EBS volumes can be used for boot disks, filesystems, and databases, and snapshots can provide backup and recovery.

---

### Q3. What is an EBS snapshot?

**Answer:** Amazon EBS provides persistent block storage for EC2. EBS volumes can be used for boot disks, filesystems, and databases, and snapshots can provide backup and recovery.

---

### Q4. What is the difference between EBS and instance store?

**Answer:** Instance store is local ephemeral storage physically attached to the host. It can provide very high I/O performance, but data is not durable like EBS and can be lost when the instance or underlying host is stopped or terminated.

---

### Q5. Can EBS volumes persist after an EC2 stop?

**Answer:** Amazon EBS provides persistent block storage for EC2. EBS volumes can be used for boot disks, filesystems, and databases, and snapshots can provide backup and recovery.

---

### Q6. Compare gp3 and gp2.

**Answer:** gp3 is a general-purpose SSD EBS volume type that lets you provision IOPS and throughput independently from storage capacity within supported limits. It is often a cost-effective default for general workloads.

---

### Q7. When should you use io2?

**Answer:** io2 is a high-performance SSD EBS volume type designed for workloads requiring high IOPS, low latency, and strong durability, such as demanding databases.

---

### Q8. What are st1 and sc1 designed for?

**Answer:** st1 is a throughput-optimized HDD volume type intended for large sequential workloads such as big-data processing and log processing.

---

### Q9. What is provisioned IOPS?

**Answer:** Provisioned IOPS means you explicitly provision a target I/O rate for supported EBS volume types. It is useful for latency-sensitive databases and other workloads with predictable I/O requirements.

---

### Q10. How do throughput and IOPS differ?

**Answer:** Throughput measures the amount of data transferred per unit time, typically MB/s. IOPS measures the number of I/O operations per second. A workload can be throughput-heavy or IOPS-heavy depending on its I/O pattern.

---

### Q11. How do EBS snapshots work?

**Answer:** Amazon EBS provides persistent block storage for EC2. EBS volumes can be used for boot disks, filesystems, and databases, and snapshots can provide backup and recovery.

---

### Q12. Are EBS snapshots incremental?

**Answer:** Amazon EBS provides persistent block storage for EC2. EBS volumes can be used for boot disks, filesystems, and databases, and snapshots can provide backup and recovery.

---

### Q13. Can snapshots be copied across Regions?

**Answer:** An EBS snapshot is a point-in-time backup of an EBS volume. Snapshots are incremental after the first snapshot and can be copied across Regions for disaster-recovery designs.

---

### Q14. What is Fast Snapshot Restore?

**Answer:** An EBS snapshot is a point-in-time backup of an EBS volume. Snapshots are incremental after the first snapshot and can be copied across Regions for disaster-recovery designs.

---

### Q15. How can snapshots support backup and disaster recovery?

**Answer:** An EBS snapshot is a point-in-time backup of an EBS volume. Snapshots are incremental after the first snapshot and can be copied across Regions for disaster-recovery designs.

---

### Q16. What is EBS encryption?

**Answer:** Amazon EBS provides persistent block storage for EC2. EBS volumes can be used for boot disks, filesystems, and databases, and snapshots can provide backup and recovery.

---

### Q17. Can encrypted snapshots be copied?

**Answer:** An EBS snapshot is a point-in-time backup of an EBS volume. Snapshots are incremental after the first snapshot and can be copied across Regions for disaster-recovery designs.

---

### Q18. Can you resize an EBS volume?

**Answer:** Amazon EBS provides persistent block storage for EC2. EBS volumes can be used for boot disks, filesystems, and databases, and snapshots can provide backup and recovery.

---

### Q19. What is Elastic Volumes?

**Answer:** EBS Elastic Volumes lets you modify supported volume properties such as size, type, and performance without detaching the volume in many cases.

---

### Q20. What is Multi-Attach and when is it supported?

**Answer:** EBS Multi-Attach allows a supported io2 volume to be attached to multiple Nitro-based EC2 instances in the same Availability Zone. Applications must coordinate concurrent writes correctly.

---
