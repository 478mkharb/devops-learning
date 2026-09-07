# EBS

### Q1. What is Amazon EBS?

**Answer:** Amazon Elastic Block Store (EBS) provides persistent block storage for Amazon EC2. EBS volumes are attached to EC2 instances in an Availability Zone and are commonly used for operating-system disks, application filesystems, and databases. EBS also supports snapshots and encryption.

---

### Q2. What is an EBS volume?

**Answer:** An EBS volume is a persistent block-storage device that can be attached to supported EC2 instances in the same Availability Zone. The selected volume type determines its performance and cost characteristics.

---

### Q3. What is an EBS snapshot?

**Answer:** An EBS snapshot is a point-in-time backup of an EBS volume. Snapshots can be used to create new EBS volumes, copy backups across Regions, and support backup and disaster-recovery workflows.

---

### Q4. What is the difference between EBS and Instance Store?

**Answer:** EBS is persistent, network-attached block storage, while Instance Store is local ephemeral storage physically associated with the EC2 host.

| Feature | EBS | Instance Store |
|---|---|---|
| Storage type | Network-attached block storage | Local block storage |
| Persistence | Persistent | Ephemeral |
| Survives EC2 stop | Yes, normally | Not designed as persistent storage |
| Survives instance termination | Depends on `DeleteOnTermination` | No |
| Snapshots | Yes | No EBS-style snapshots |
| Typical use | OS, applications, databases | Temporary, cache, scratch data |
| Performance | Depends on EBS volume type | Very high local I/O performance |

---

### Q5. Can an EBS volume persist after an EC2 instance is stopped?

**Answer:** Yes. An EBS volume normally remains available when an EC2 instance is stopped. When the instance starts again, the volume can be reattached and used according to the instance's block-device configuration.

---

### Q6. What happens to an EBS volume when an EC2 instance is terminated?

**Answer:** The behavior depends on the volume's **`DeleteOnTermination`** setting.

| `DeleteOnTermination` | Result when EC2 is terminated |
|---|---|
| `true` | EBS volume is deleted |
| `false` | EBS volume is retained |

The root EBS volume is commonly configured to be deleted when the instance is terminated, while additional data volumes may be configured for retention.

---

### Q7. Compare gp2 and gp3.

**Answer:** Both are general-purpose SSD EBS volume types, but gp3 allows storage capacity, IOPS, and throughput to be provisioned more independently than gp2.

| Feature | gp2 | gp3 |
|---|---|---|
| Volume type | General-purpose SSD | General-purpose SSD |
| Performance model | IOPS scales with volume size | IOPS and throughput can be provisioned independently of size |
| Baseline IOPS | 3 IOPS/GiB, subject to volume limits | 3,000 IOPS |
| Baseline throughput | Scales with volume size | 125 MiB/s |
| Main advantage | Simple size-based performance model | More flexible performance provisioning |

---

### Q8. When should you use io2?

**Answer:** Use io2 for I/O-intensive workloads that require consistently high IOPS, low latency, and high durability, such as demanding relational databases and other critical workloads.

---

### Q9. What are st1 and sc1 designed for?

**Answer:** Both are HDD-based EBS volume types intended for throughput-oriented workloads rather than low-latency transactional workloads.

| Volume type | Designed for | Example workloads |
|---|---|---|
| **st1** | Frequently accessed, throughput-intensive sequential workloads | Big-data processing, data warehouses, log processing |
| **sc1** | Infrequently accessed, throughput-oriented workloads | Cold data and large sequential datasets |

---

### Q10. What is provisioned IOPS?

**Answer:** Provisioned IOPS means explicitly provisioning a target number of I/O operations per second for supported EBS volume types. It is useful for workloads with demanding and predictable I/O performance requirements.

---

### Q11. What is the difference between IOPS and throughput?

**Answer:** **IOPS** measures the number of I/O operations performed per second, while **throughput** measures the amount of data transferred per second.

| Metric | Measures | Example |
|---|---|---|
| **IOPS** | Number of I/O operations per second | 20,000 I/O operations/sec |
| **Throughput** | Amount of data transferred per second | 500 MiB/s |

Small random I/O workloads are often more sensitive to IOPS, while large sequential transfers are often more sensitive to throughput.

---

### Q12. How do EBS snapshots work?

**Answer:** An EBS snapshot captures the state of an EBS volume at a point in time. EBS snapshots use incremental storage, so after the initial snapshot, subsequent snapshots store blocks that have changed since the previous snapshot. A snapshot can later be used to create a new EBS volume.

---

### Q13. Are EBS snapshots incremental?

**Answer:** Yes. EBS snapshots are incremental at the storage level. After the first snapshot, subsequent snapshots store only changed blocks relative to the previous snapshot. Each snapshot still represents the complete volume state logically.

---

### Q14. Can EBS snapshots be copied across Regions?

**Answer:** Yes. An EBS snapshot can be copied to another AWS Region. The destination receives its own snapshot, which can be used to create EBS volumes in that Region.

This is commonly used for cross-Region backup and disaster recovery.

---

### Q15. What is Fast Snapshot Restore?

**Answer:** Fast Snapshot Restore (FSR) enables EBS volumes created from a configured snapshot to achieve full performance immediately in the Availability Zones where FSR is enabled. It avoids the normal need for lazy initialization of snapshot blocks.

---

### Q16. How can EBS snapshots support backup and disaster recovery?

**Answer:** Snapshots provide point-in-time copies of EBS volume data that can be retained independently of the source volume. They can be copied to other Regions and used to create replacement volumes.

A typical disaster-recovery workflow is:

```text
Production EBS Volume
        |
        | Create Snapshot
        ↓
   EBS Snapshot
        |
        | Copy to another Region
        ↓
Destination Snapshot
        |
        | Create EBS Volume
        ↓
Replacement EBS Volume
```

---

### Q17. What is EBS encryption?

**Answer:** EBS encryption protects EBS data at rest. It also encrypts snapshots and data transferred between the EC2 instance and the EBS volume. AWS KMS keys are used for encryption key management.

---

### Q18. Can encrypted EBS snapshots be copied?

**Answer:** Yes. Encrypted EBS snapshots can be copied, including across Regions. For a cross-Region copy, the destination snapshot is encrypted using a KMS key available in the destination Region.

---

### Q19. Can you resize an EBS volume?

**Answer:** Yes. Supported EBS volumes can be modified using Elastic Volumes. You can increase the volume size and, where supported, modify the volume type, IOPS, or throughput without first detaching the volume.

After increasing the EBS volume size, the operating-system partition and filesystem may also need to be extended.

---

### Q20. What is EBS Elastic Volumes?

**Answer:** EBS Elastic Volumes allows supported volume properties such as size, volume type, IOPS, and throughput to be modified while the volume remains attached in many cases.

This allows EBS capacity and performance to be adjusted without the traditional detach-and-recreate workflow.

---

### Q21. What is EBS Multi-Attach and when is it supported?

**Answer:** EBS Multi-Attach allows a supported EBS volume to be attached simultaneously to multiple supported EC2 instances in the same Availability Zone. It is primarily supported with **io2** volumes on supported **Nitro-based EC2 instances**.

| Requirement | Multi-Attach |
|---|---|
| Volume type | Supported `io2` volumes |
| EC2 platform | Supported Nitro-based instances |
| Availability Zone | Instances must be in the same AZ |
| Concurrent access | Application/filesystem must coordinate access correctly |
| Automatic shared-filesystem safety | No |

Example:

```text
             ┌── EC2 Instance A
             │
io2 Volume ──┼── EC2 Instance B
             │
             └── EC2 Instance C
```

Multi-Attach does not automatically make a filesystem safe for concurrent writes. The application and filesystem architecture must support shared block-device access.
