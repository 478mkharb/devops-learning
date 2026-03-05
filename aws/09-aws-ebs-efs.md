# AWS EBS & EFS

This document explains key **AWS storage concepts related to EBS snapshots, EBS volume types, resizing volumes, and when to use EFS vs EBS**. These are common topics in **DevOps and Cloud Engineer interviews**.

---

# 1. What Happens Internally When an EBS Snapshot is Created?

An **EBS snapshot** is a backup of an EBS volume stored in **Amazon S3**.

When a snapshot is created:

```
EC2 Instance
     │
     ▼
EBS Volume
     │
     ▼
Snapshot process starts
     │
     ▼
Data blocks copied to S3
```

Important behavior:

* Snapshot captures **block-level data** of the volume
* Snapshot process is **asynchronous**
* Only **used blocks** are copied
* AWS stores snapshot data in **S3 internally** (not directly visible)

Flow:

```
Create Snapshot
      │
      ▼
Identify used blocks
      │
      ▼
Copy blocks to S3
      │
      ▼
Snapshot stored
```

Snapshots can later be used to:

* Restore EBS volumes
* Create AMIs
* Create backup copies

---

# 2. How Incremental Snapshot Storage Works

EBS snapshots are **incremental**, meaning only changed blocks are stored after the first snapshot.

Example:

Initial snapshot:

```
Volume blocks
A B C D E

Snapshot1
A B C D E
```

After some data changes:

```
Volume blocks
A B X D Y
```

Next snapshot stores only changed blocks:

```
Snapshot2
X Y
```

AWS internally links snapshots:

```
Snapshot1 → Snapshot2 → Snapshot3
```

Benefits:

* Reduced storage cost
* Faster snapshot creation
* Efficient backups

---

# 3. Difference Between gp3 and io2 Volumes

Both gp3 and io2 are **SSD-based EBS volumes**, but they serve different workloads.

| Feature    | gp3                         | io2                         |
| ---------- | --------------------------- | --------------------------- |
| Type       | General Purpose SSD         | Provisioned IOPS SSD        |
| IOPS       | Up to 16,000                | Up to 256,000               |
| Latency    | Low                         | Very low and consistent     |
| Cost       | Lower                       | Higher                      |
| Best For   | Web apps, general workloads | Databases, high I/O systems |
| Durability | Standard                    | Higher durability           |

Use **gp3** for:

* application servers
* general workloads
* cost optimization

Use **io2** for:

* high-performance databases
* financial systems
* latency-sensitive workloads

---

# 4. How to Expand an EBS Volume Without Downtime

AWS allows **online resizing of EBS volumes**.

Steps:

1. Modify EBS volume size
2. Wait for modification to complete
3. Expand filesystem inside the OS

Architecture:

```
EC2 Instance
     │
     ▼
EBS Volume
     │
     ▼
Modify volume size
```

Example AWS command:

```
Modify Volume
Size: 100GB → 200GB
```

Then expand filesystem inside the instance.

Example for Linux:

```
sudo growpart /dev/xvda 1
sudo resize2fs /dev/xvda1
```

Result:

* No downtime
* Application keeps running

---

# 5. When Should You Use EFS Instead of EBS?

EFS is a **shared network file system**, while EBS is **block storage attached to one instance**.

| Feature      | EBS                 | EFS                    |
| ------------ | ------------------- | ---------------------- |
| Storage Type | Block storage       | File storage           |
| Mount        | Single EC2 instance | Multiple EC2 instances |
| Protocol     | Block device        | NFS                    |
| Scalability  | Manual resizing     | Automatic scaling      |
| Use Case     | Databases, OS disks | Shared storage         |

Use **EFS when:**

* Multiple EC2 instances need shared storage
* Web servers share files
* Container workloads need shared storage

Architecture example:

```
        EFS File System
          │     │     │
          ▼     ▼     ▼
        EC2   EC2   EC2
```

Use **EBS when:**

* High-performance block storage required
* Single instance storage needed
* Database workloads

---

# Quick DevOps Summary

```
EBS Snapshot
Block-level backup stored in S3

Snapshots
Incremental storage

Volume Types

GP3 → general workloads
IO2 → high performance database workloads

Storage Choice

EBS → single instance block storage
EFS → shared file system across instances
```
