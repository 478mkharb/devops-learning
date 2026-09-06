# EBS

### Q1. What is Amazon EBS?

**Answer:** Amazon Elastic Block Store (EBS) provides persistent block storage for EC2. EBS volumes are attached to instances in an Availability Zone and are commonly used for operating-system disks, application filesystems, and databases. EBS also supports snapshots and encryption.

---

### Q2. What is an EBS volume?

**Answer:** An EBS volume is persistent block storage that can be attached to supported EC2 instances in the same Availability Zone. Its type determines performance and cost characteristics.

---

### Q3. What is an EBS snapshot?

**Answer:** An EBS snapshot is a point-in-time backup of an EBS volume. Snapshots are stored in AWS and can be used to create new volumes, copy data across Regions, and support backup and disaster-recovery workflows.

---

### Q4. What is the difference between EBS and instance store?

**Answer:** EBS is persistent network-attached block storage that supports snapshots and can normally survive an EC2 stop. Instance store is local ephemeral storage with very high local I/O performance, but its data is not durable across the instance or underlying host lifecycle.

---

### Q5. Can EBS volumes persist after an EC2 stop?

**Answer:** Yes. EBS volumes normally remain available when an EC2 instance is stopped. The volume is detached from the stopped instance and can be reattached when the instance starts, subject to the instance and volume configuration.

---

### Q6. Compare gp3 and gp2.

**Answer:** gp3 is a general-purpose SSD EBS volume type that allows storage size, IOPS, and throughput to be provisioned more independently than gp2. It is commonly a strong default for general workloads.

---

### Q7. When should you use io2?

**Answer:** io2 is a high-performance SSD EBS volume type designed for workloads requiring high IOPS, low latency, and high durability, such as demanding databases.

---

### Q8. What are st1 and sc1 designed for?

**Answer:** st1 is a throughput-optimized HDD EBS volume type intended for large sequential workloads such as big-data processing and log processing.

---

### Q9. What is provisioned IOPS?

**Answer:** Provisioned IOPS means explicitly provisioning a target I/O rate for supported EBS volume types. It is useful for I/O-intensive, latency-sensitive workloads.

---

### Q10. How do throughput and IOPS differ?

**Answer:** IOPS measures the number of I/O operations per second, while throughput measures the amount of data transferred per second. Small random I/O can be IOPS-bound; large sequential I/O can be throughput-bound.

---

### Q11. How do EBS snapshots work?

**Answer:** An EBS snapshot captures the volume's data at a point in time. The first snapshot establishes the baseline and later snapshots store changed blocks, so snapshots can be used efficiently for backup and recovery. A snapshot can be used to create a new EBS volume.

---

### Q12. Are EBS snapshots incremental?

**Answer:** Yes. After the first snapshot, subsequent EBS snapshots store only blocks that have changed since the previous snapshot. Each snapshot therefore represents the complete volume state logically, even though the underlying storage is incremental.

---

### Q13. Can snapshots be copied across Regions?

**Answer:** Yes. EBS snapshots can be copied to another AWS Region. AWS creates a snapshot in the destination Region, where it can be used to create EBS volumes or support disaster recovery. Cross-Region copies are independent of the source snapshot after the copy operation.

---

### Q14. What is Fast Snapshot Restore?

**Answer:** EBS Fast Snapshot Restore creates volumes from a snapshot with full performance available immediately in the configured Availability Zones, avoiding the normal lazy initialization process.

---

### Q15. How can snapshots support backup and disaster recovery?

**Answer:** Snapshots provide point-in-time copies of EBS data that can be retained independently of the source volume. They can be copied across Regions and used to create replacement volumes, making them useful for backup, recovery, and regional disaster-recovery strategies.

---

### Q16. What is EBS encryption?

**Answer:** EBS encryption protects data at rest on the volume, snapshots, and data moving between the instance and the volume using AWS-managed or customer-managed KMS keys. Encrypted volumes and their snapshots retain encryption throughout supported operations.

---

### Q17. Can encrypted snapshots be copied?

**Answer:** Yes. Encrypted EBS snapshots can be copied, including across Regions. For a cross-Region copy, the destination copy uses a KMS key available in the destination Region.

---

### Q18. Can you resize an EBS volume?

**Answer:** Yes. With Elastic Volumes, supported EBS volumes can be modified by increasing size and, where supported, changing volume type, IOPS, or throughput without first detaching the volume. After increasing the volume, the operating system filesystem may also need to be expanded.

---

### Q19. What is Elastic Volumes?

**Answer:** EBS Elastic Volumes allows supported volume properties such as size, type, IOPS, and throughput to be modified without detaching the volume in many cases.

---

### Q20. What is Multi-Attach and when is it supported?

**Answer:** EBS Multi-Attach allows a supported io2 volume to be attached to multiple Nitro-based EC2 instances in the same Availability Zone. Applications must coordinate concurrent access correctly.

---
