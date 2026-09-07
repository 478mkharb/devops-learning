# EBS

### Q1. What is Amazon EBS?

**Answer:** Amazon Elastic Block Store (EBS) provides persistent block storage for EC2. EBS volumes are attached to instances in an Availability Zone and are commonly used for operating-system disks, application filesystems, and databases. EBS also supports snapshots and encryption.

---

### Q2. What is an EBS volume?

**Answer:** An EBS volume is persistent block storage that can be attached to supported EC2 instances in the same Availability Zone. Its volume type determines its performance and cost characteristics.

---

### Q3. What is an EBS snapshot?

**Answer:** An EBS snapshot is a point-in-time backup of an EBS volume. Snapshots can be used to create new volumes, copy data across Regions, and support backup and disaster-recovery workflows.

---

### Q4. What is the difference between EBS and instance store?

**Answer:** EBS is persistent network-attached block storage, while instance store is local ephemeral storage physically associated with the EC2 host.

| Feature | EBS | Instance Store |
|---|---|---|
| Storage type | Network-attached block storage | Local block storage |
| Persistence | Persistent | Ephemeral |
| Survives EC2 stop | Yes, normally | Not applicable as durable storage |
| Survives instance termination | Depends on volume's delete-on-termination setting | No |
| Snapshots | Yes | No EBS-style snapshots |
| Typical use | OS, applications, databases | Temporary/cache/scratch data |
| Performance | Depends on EBS volume type | Very high local I/O performance |

---

### Q5. Can EBS volumes persist after an EC2 stop?

**Answer:** Yes. EBS volumes normally remain available when an EC2 instance is stopped. The volume remains associated with the instance configuration and can be used again when the instance starts.

---

### Q6. Compare gp3 and gp2.

**Answer:** Both are general-purpose SSD volume types, but gp3 separates storage capacity from provisioned performance more effectively than gp2.

| Feature | gp2 | gp3 |
|---|---|---|
| Volume type | General-purpose SSD | General-purpose SSD |
| Performance model | IOPS scales with volume size | IOPS and throughput can be provisioned independently of size |
| Baseline IOPS | 3 IOPS/GiB, subject to volume limits | 3,000 IOPS |
| Baseline throughput | Scales with volume size | 125 MiB/s |
| Typical advantage | Simple size-based performance | More flexible and often more cost-efficient performance provisioning |

---

### Q7. When should you use io2?

**Answer:** Use io2 for I/O-intensive workloads that require consistently high IOPS, low latency, and high durability, such as demanding relational and other critical databases.

---

### Q8. What are st1 and sc1 designed for?

**Answer:** Both are HDD-based EBS volume types intended for throughput-oriented workloads rather than low-latency transactional workloads.

| Volume type | Designed for | Example workloads |
|---|---|---|
| **st1** | Frequently accessed, throughput-intensive sequential workloads | Big-data processing, data warehouses, log processing |
| **sc1** | Infrequently accessed, throughput-oriented workloads | Cold data, large sequential datasets |

---

### Q9. What is provisioned IOPS?

**Answer:** Provisioned IOPS means explicitly provisioning a target number of I/O operations per second for supported EBS volume types. It is useful for workloads with demanding and predictable I/O performance requirements.

---

### Q10. How do throughput and IOPS differ?

**Answer:** **IOPS** measures how many I/O operations can be performed per second, while **throughput** measures how much data can be transferred per second.

| Metric | Measures | Example |
|---|---|---|
| **IOPS** | Number of I/O operations per second | 20,000 I/O operations/sec |
| **Throughput** | Amount of data transferred per second | 500 MiB/s |

Small random I/O workloads are often more sensitive to IOPS, while large sequential transfers are often more sensitive to throughput.

---

### Q11. How do EBS snapshots work?

**Answer:** An EBS snapshot captures the state of an EBS volume at a point in time. EBS snapshots use incremental storage, so after the initial snapshot, subsequent snapshots store changed blocks. A snapshot can later be used to create a new EBS volume.

---

### Q12. Are EBS snapshots incremental?

**Answer:** Yes. EBS snapshots are incremental at the storage level. After the first snapshot, subsequent snapshots store only blocks that have changed since the previous snapshot. Each snapshot can still be used to restore the complete volume state logically.

---

### Q13. Can snapshots be copied across Regions?

**Answer:** Yes. An EBS snapshot can be copied to another AWS Region. The destination receives its own snapshot, which can be used to create EBS volumes in that Region.

This is commonly used for cross-Region backup and disaster recovery.

---

### Q14. What is Fast Snapshot Restore?

**Answer:** Fast Snapshot Restore (FSR) enables EBS volumes created from a configured snapshot to achieve full performance immediately in the Availability Zones where FSR is enabled, avoiding the normal need for lazy initialization of blocks.

---

### Q15. How can snapshots support backup and disaster recovery?

**Answer:** Snapshots provide point-in-time copies of EBS volume data that can be retained independently of the source volume. They can be copied to other Regions and used to create replacement volumes.

A typical disaster-recovery workflow is:

```text
Production EBS Volume
        |
        | Snapshot
        ↓
   EBS Snapshot
        |
        | Copy to another Region
        ↓
Destination Snapshot
        |
        | Create Volume
        ↓
Replacement EBS Volume
```

---

### Q16. What is EBS encryption?

**Answer:** EBS encryption protects EBS data at rest and provides encryption for snapshots and data transferred between the EC2 instance and the EBS volume. AWS KMS keys are used for encryption key management.

---

### Q17. Can encrypted snapshots be copied?

**Answer:** Yes. Encrypted EBS snapshots can be copied, including across Regions. For a cross-Region copy, the destination snapshot is encrypted using a KMS key available in the destination Region.

---

### Q18. Can you resize an EBS volume?

**Answer:** Yes. Supported EBS volumes can be modified using Elastic Volumes. You can increase the volume size and, where supported, modify the volume type, IOPS, or throughput without first detaching the volume. After increasing the volume size, the operating-system partition and filesystem may also need to be extended.

---

### Q19. What is Elastic Volumes?

**Answer:** EBS Elastic Volumes allows supported volume properties such as size, volume type, IOPS, and throughput to be modified while the volume remains attached in many cases. This allows EBS performance and capacity to be adjusted without the traditional detach-and-recreate workflow.

---

### Q20. What is EBS Multi-Attach and when is it supported?

**Answer:** EBS Multi-Attach allows a supported EBS volume to be attached simultaneously to multiple supported EC2 instances in the same Availability Zone. It is primarily supported with **io2** volumes on supported **Nitro-based EC2 instances**.

The application must be designed to coordinate concurrent access correctly because multiple instances can access the same block device.

Example:

```text
             ┌── EC2 Instance A
             │
io2 Volume ──┼── EC2 Instance B
             │
             └── EC2 Instance C
```

Multi-Attach does not automatically make a filesystem safe for concurrent writes; the application and filesystem architecture must support shared access.
