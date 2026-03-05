# AWS S3 – DevOps Interview Guide

This document explains important **Amazon S3 concepts frequently asked in DevOps interviews**, including bucket types, storage classes, lifecycle management, replication, CORS, and secure access from EC2.

---

# 1. Types of S3 Buckets

Amazon S3 provides different bucket types depending on workload requirements.

| Bucket Type             | Description                                   | Use Case                        |
| ----------------------- | --------------------------------------------- | ------------------------------- |
| General Purpose Buckets | Standard S3 buckets used for most workloads   | Static websites, backups, logs  |
| Directory Buckets       | Optimized for low latency and high throughput | Real-time analytics workloads   |
| Table Buckets           | Used for storing structured tabular data      | Data analytics and AI workloads |
| Vector Buckets          | Designed for vector-based similarity search   | AI/ML, recommendation systems   |

General purpose buckets are the most commonly used in production systems.

---

# 2. S3 Storage Classes (8 Types)

S3 provides multiple storage tiers optimized for different access patterns.

| Storage Class              | Availability | Access Pattern             | Typical Use Case        |
| -------------------------- | ------------ | -------------------------- | ----------------------- |
| S3 Standard                | 99.99%       | Frequent access            | Active application data |
| S3 Standard-IA             | 99.9%        | Infrequent access          | Backups                 |
| S3 Intelligent-Tiering     | 99.9%        | Unknown access patterns    | Cost optimization       |
| S3 Express One Zone        | 99.9%        | Very frequent, low latency | High performance apps   |
| S3 One Zone-IA             | 99.5%        | Infrequent access          | Secondary backups       |
| Glacier Instant Retrieval  | 99.9%        | Rare but fast access       | Archives                |
| Glacier Flexible Retrieval | 99.9%        | Rare access                | Long-term archives      |
| Glacier Deep Archive       | 99.9%        | Very rare access           | Compliance storage      |

---

# 3. How S3 Lifecycle Management Works

S3 Lifecycle policies automatically **transition or delete objects based on age**.

Example lifecycle policy:

```
Day 0  → Store in S3 Standard
Day 30 → Move to Standard-IA
Day 90 → Move to Glacier
Day 365 → Delete object
```

Lifecycle rules help reduce storage costs by automatically moving data to cheaper storage classes.

Flow:

```
Upload object
     │
     ▼
Lifecycle rule evaluation
     │
     ▼
Transition to cheaper storage
     │
     ▼
Optional deletion
```

---

# 4. Cross-Region Replication (CRR)

Cross-Region Replication automatically copies objects from **one S3 bucket to another bucket in a different AWS region**.

Architecture:

```
Primary Bucket (Region A)
           │
           ▼
Replication Rule
           │
           ▼
Replica Bucket (Region B)
```

Benefits:

* Disaster recovery
* Global availability
* Compliance requirements

Requirements:

* Versioning enabled
* IAM replication role

---

# 5. What is CORS in S3

CORS (Cross-Origin Resource Sharing) allows **web applications hosted on different domains to access S3 resources**.

Example:

```
Frontend (example.com)
       │
       ▼
Access file in S3 bucket
```

CORS configuration example:

```
Allow origin: https://example.com
Allow methods: GET, PUT
```

This is commonly used when **static websites or frontend apps fetch assets from S3**.

---

# 6. Designing Secure S3 Access from EC2 Without Internet

Best practice is to use a **VPC Gateway Endpoint for S3**.

Architecture:

```
EC2 Instance (Private Subnet)
        │
        ▼
VPC Gateway Endpoint
        │
        ▼
Amazon S3
```

Benefits:

* Traffic stays inside AWS network
* No NAT gateway cost
* More secure

Implementation steps:

1. Create VPC Gateway Endpoint for S3
2. Update route table
3. Restrict bucket policy to allow only endpoint access

Example bucket policy condition:

```
aws:sourceVpce
```

---

# Quick DevOps Summary

```
S3 Bucket Types:
- General Purpose
- Directory
- Table
- Vector

Storage Classes:
Standard → IA → Glacier → Deep Archive

Key Features:
- Lifecycle management
- Cross-region replication
- CORS for web apps
- VPC endpoint for secure access
```
