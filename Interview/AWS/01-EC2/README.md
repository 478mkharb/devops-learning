# Amazon EC2 Interview & Practical Notes

> **Scope:** EC2 service concepts only. This document intentionally excludes EBS, IAM, VPC networking, ENI, NAT Gateway, Security Groups, NACLs, CloudWatch, Load Balancers, Auto Scaling, and other separate AWS services.
> **Audience:** DevOps Engineers, Cloud Engineers, System Administrators, and AWS interview candidates.

## Table of Contents

| Stage | Topic |
|---:|---|
| 1 | [EC2 Fundamentals](#1-ec2-fundamentals) |
| 2 | [Instance Types](#2-instance-types) |
| 3 | [EC2 Lifecycle](#6-ec2-lifecycle) |
| 4 | [EC2 Pricing Options](#7-ec2-pricing-options) |
| 5 | [Placement Groups and Tenancy](#8-placement-groups-and-tenancy) |
| 6 | [Final Interview Revision](#14-final-interview-revision) |
| 7 | [References](#15-references) |

---

# 1. EC2 Fundamentals

## What is Amazon EC2?

**Amazon Elastic Compute Cloud (EC2)** provides resizable virtual compute capacity in AWS.

| EC2 provides | Customer responsibility |
|---|---|
| Physical infrastructure | Guest OS |
| Hypervisor and underlying hardware | OS patches and packages |
| Instance provisioning | Application and configuration |
| Availability-zone infrastructure | Data, permissions and workload |

### Common use cases

| Workload | Example |
|---|---|
| Web server | NGINX or Apache |
| Application server | Java, Python, Go or Node.js |
| CI/CD worker | Jenkins agent |
| Monitoring | Prometheus or exporters |
| Batch processing | Parallel jobs |
| Database | Self-managed database on EC2 |
| Development | Temporary test environment |

## What is an EC2 instance?

An EC2 instance is a virtual server launched from an AMI with a selected instance type and configuration.

| Component | Purpose |
|---|---|
| AMI | OS and initial software |
| Instance type | vCPU, memory and network capability |
| VPC/Subnet | Network placement |
| ENI | Network connectivity |
| Security group | Stateful traffic filtering |
| EBS/Instance Store | Storage |
| IAM instance profile | AWS API permissions |
| User data | Bootstrap configuration |

### EC2 launch flow

```text
AMI
  ↓
Instance Type
  ↓
VPC / Subnet
  ↓
Security Group
  ↓
IAM Instance Profile
  ↓
Storage
  ↓
User Data
  ↓
EC2 Instance
```

### Example: launch an instance

```bash
aws ec2 run-instances \
  --image-id ami-xxxxxxxxxxxxxxxxx \
  --instance-type t3.small \
  --subnet-id subnet-xxxxxxxx \
  --security-group-ids sg-xxxxxxxx \
  --iam-instance-profile Name=ec2-ssm-profile \
  --tag-specifications \
    'ResourceType=instance,Tags=[{Key=Name,Value=dev-web-01}]'
```

> Replace example IDs with values from your AWS account and Region.

### Interview Checkpoint — Fundamentals

| Frequently asked question | Interview-ready answer |
|---|---|
| What is EC2? | A service that provides resizable virtual compute capacity in AWS. |
| Why is EC2 called elastic? | Capacity can be increased, decreased, started, stopped, or replaced according to workload needs. |
| What is an EC2 instance? | A virtual server created from an AMI using an instance type and launch configuration. |
| What are the main EC2 launch components? | AMI, instance type, VPC/subnet, security group, IAM profile, storage and user data. |

---


# 2. Instance Types

## Instance Types

An instance type defines the compute characteristics exposed to an EC2 instance.

| Characteristic | Examples |
|---|---|
| CPU | vCPUs |
| Memory | RAM |
| Network | Network bandwidth |
| EBS | EBS bandwidth and limits |
| Accelerators | GPU, FPGA, Inferentia or Trainium |
| Local storage | Instance Store, where supported |

## Naming Convention

Example:

```text
m7i-flex.large
│││  │    └── Size
│││  └────── Option: Flex
││└───────── Processor/feature option: Intel
│└────────── Generation: 7
└─────────── Family: M
```

| Part | Meaning |
|---|---|
| `m` | General-purpose family |
| `7` | Generation |
| `i` | Intel processor option |
| `flex` | Flex variant |
| `large` | Instance size |

Common suffixes:

| Suffix | Meaning |
|---|---|
| `a` | AMD processor |
| `g` | AWS Graviton processor |
| `i` | Intel processor |
| `d` | Local instance-store volumes |
| `n` | Enhanced network/EBS capability |
| `e` | Extra storage, memory or GPU memory depending on family |
| `z` | High CPU frequency |
| `metal` | Bare-metal instance |

## EC2 Instance Family Classification

EC2 instance families are grouped by the workload they are designed to run.

| Category | Families | Primary use |
|---|---|---|
| **General Purpose** | **A, M, T, Mac** | Balanced CPU, memory and networking; Arm, burstable and macOS workloads |
| **Compute Optimized** | **C** | CPU-intensive applications, batch processing and high-performance web servers |
| **Memory Optimized** | **R, X, Z, U** | In-memory databases, large datasets and high-memory workloads |
| **Storage Optimized** | **I, D, H, Im, Is** | High local I/O, low-latency storage and dense local storage capacity |
| **Accelerated Computing** | **P, G, F, Inf, Trn, VT, Dl** | GPU, graphics, FPGA, ML, video and specialized acceleration |
| **High Performance Computing** | **Hpc** | Scientific simulations and tightly coupled HPC workloads |

### Family-by-family explanation

| Family | Explanation | Example workload |
|---|---|---|
| **A** | Arm-based general-purpose instances | Cost-efficient Arm-compatible applications |
| **M** | Balanced general-purpose instances | Web servers, application servers and backend services |
| **T** | Burstable general-purpose instances using CPU credits | Development servers and variable CPU workloads |
| **Mac** | macOS instances | iOS/macOS build and testing |
| **C** | Compute-optimized instances | CPU-heavy services, batch processing and encoding |
| **R** | Memory-optimized instances | Caches, analytics and in-memory databases |
| **X** | Very memory-intensive instances | Large in-memory datasets |
| **Z** | High-memory instances with high CPU frequency | Low-latency, memory-intensive workloads |
| **U** | High-memory instances with very large RAM capacity | Enterprise databases requiring terabytes of memory |
| **I** | Storage-optimized instances, commonly with high-performance local SSD/NVMe storage | Databases and distributed storage |
| **D** | Dense-storage instances | Large local storage capacity |
| **H** | HDD-storage-optimized instances | High-throughput, data-intensive workloads |
| **Im / Is** | Storage-optimized variants with different memory-to-vCPU ratios | Storage-heavy workloads |
| **P** | GPU-accelerated instances | ML training and GPU computing |
| **G** | Graphics-optimized GPU instances | Graphics, visualization and ML inference |
| **F** | FPGA instances | Custom hardware acceleration and specialized analytics |
| **Inf** | AWS Inferentia-based instances | Machine-learning inference |
| **Trn** | AWS Trainium-based instances | Machine-learning training |
| **VT** | Video-transcoding instances | Video encoding and media processing |
| **Dl** | Deep-learning-focused instances | Deep-learning training and inference |
| **Hpc** | High-performance computing instances | Scientific and engineering simulations |


### Interview Checkpoint

| Question | Interview-ready answer |
|---|---|
| How do you select an instance family? | Match the workload bottleneck—CPU, memory, storage, accelerator or burstability—then validate the choice using performance metrics, cost and scaling requirements. |


# 6. EC2 Lifecycle

## Instance states

| State | Meaning |
|---|---|
| `pending` | Being prepared for launch |
| `running` | Operating and consuming compute capacity |
| `stopping` | Being stopped |
| `stopped` | Not consuming instance compute capacity |
| `shutting-down` | Being terminated |
| `terminated` | Permanently removed |

```text
pending → running → stopped → pending → running
                  └────────→ shutting-down → terminated
```

## Stop vs Start vs Reboot vs Hibernate vs Terminate

| Action | Result | EBS | Public IPv4 |
|---|---|---|---|
| Reboot | Restarts OS | Preserved | Usually unchanged |
| Stop | Releases compute capacity | Preserved by default | Auto public IP released |
| Start | Starts stopped instance | Reattached | New auto public IP may be assigned |
| Hibernate | Saves RAM to root EBS and stops | Preserved, if supported | Auto public IP released |
| Terminate | Permanently removes instance | Depends on `DeleteOnTermination` | Released |

### DeleteOnTermination

```text
DeleteOnTermination = true
  → Volume deleted with instance

DeleteOnTermination = false
  → Volume preserved
```

Check before termination:

```bash
aws ec2 describe-instances \
  --instance-ids i-xxxxxxxxxxxxxxxxx \
  --query "Reservations[].Instances[].BlockDeviceMappings[].{Device:DeviceName,DeleteOnTermination:Ebs.DeleteOnTermination}"
```

### Interview Checkpoint — Lifecycle

| Question | Answer |
|---|---|
| Difference between stop and reboot? | Reboot restarts the OS; stop shuts down the instance and releases compute capacity. |
| Difference between stop and terminate? | A stopped instance can be started again; a terminated instance is permanently removed. |
| What happens to private IP on stop/start? | The private IP on the persistent ENI is normally retained. |
| What happens to auto-assigned public IP? | It is released on stop and may change after start. |

---


# 7. EC2 Pricing Options

| Option | Commitment | Best fit |
|---|---|---|
| On-Demand | None | Variable or short-term workloads |
| Reserved Instance | Eligible configuration commitment | Predictable steady usage |
| Savings Plan | Consistent compute spend commitment | Stable usage with flexibility |
| Spot | Interruptible spare capacity | Fault-tolerant workloads |

## Spot Instances

Spot Instances use spare EC2 capacity at a discount but can be interrupted.

Good workloads:

- Batch jobs
- CI/CD workers
- Rendering
- Distributed processing
- Stateless capacity
- Parallel workloads

### Spot interruption flow

```text
Interruption notice
  ↓
Stop accepting new work
  ↓
Checkpoint state
  ↓
Drain active work
  ↓
Replace capacity
```

A normal Spot interruption provides a two-minute notice. A **rebalance recommendation** indicates increased interruption risk and allows proactive action.

### Interview Checkpoint — Pricing

| Question | Answer |
|---|---|
| When should you use On-Demand? | For unpredictable, short-term or non-interruptible workloads. |
| What is a Reserved Instance? | A pricing commitment for eligible EC2 configurations, not a separate instance type. |
| What is a Savings Plan? | A commitment to eligible compute spend in exchange for discounted pricing. |
| What is a Spot Instance? | Discounted spare EC2 capacity that can be interrupted. |

---


# 8. Placement Groups and Tenancy

## Placement group strategies

| Strategy | Purpose | Typical use |
|---|---|---|
| Cluster | Low latency and high throughput within one AZ | HPC and tightly coupled workloads |
| Spread | Separate instances across hardware | Critical independent instances |
| Partition | Separate groups into failure domains | Kafka, Cassandra and distributed systems |

## Dedicated tenancy

| Option | Meaning | Use case |
|---|---|---|
| Dedicated Instance | Instance runs on hardware dedicated to one AWS account | Certain isolation requirements |
| Dedicated Host | Entire physical server dedicated to the account | Licensing, compliance and host-level visibility |

### Interview Checkpoint — Placement and Tenancy

| Question | Answer |
|---|---|
| What is a placement group? | A strategy controlling how EC2 instances are placed on AWS infrastructure. |
| Cluster vs Spread? | Cluster optimizes low latency; Spread isolates individual instances. |
| What is Partition placement? | It separates instances into partitions to reduce correlated failures. |
| Which placement group suits Kafka? | Partition placement is commonly suitable for distributed systems such as Kafka. |

---
