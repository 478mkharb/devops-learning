# EC2 Interview Notes

## Table of Contents

1. [Introduction](#1-introduction)
2. [What is Amazon EC2?](#2-what-is-amazon-ec2)
3. [What is an EC2 Instance?](#3-what-is-an-ec2-instance)
4. [EC2 Launch Components](#4-ec2-launch-components)
5. [What is an AMI?](#5-what-is-an-ami)
6. [What is an Instance Type?](#6-what-is-an-instance-type)
7. [EC2 Instance Families](#7-ec2-instance-families)
8. [User Data](#8-user-data)
9. [Instance Metadata and IMDSv2](#9-instance-metadata-and-imdsv2)
10. [IAM Role and Instance Profile](#10-iam-role-and-instance-profile)
11. [EC2 Networking](#11-ec2-networking)
12. [Private IPv4, Public IPv4, and Elastic IP](#12-private-ipv4-public-ipv4-and-elastic-ip)
13. [Elastic Network Interface (ENI)](#13-elastic-network-interface-eni)
14. [EC2 Storage](#14-ec2-storage)
15. [EBS](#15-ebs)
16. [Instance Store](#16-instance-store)
17. [EBS vs Instance Store](#17-ebs-vs-instance-store)
18. [gp3 EBS Volume](#18-gp3-ebs-volume)
19. [EC2 Lifecycle](#19-ec2-lifecycle)
20. [Stop, Start, Reboot, Hibernate, and Terminate](#20-stop-start-reboot-hibernate-and-terminate)
21. [EBS DeleteOnTermination](#21-ebs-deleteontermination)
22. [Spot Instances](#22-spot-instances)
23. [Spot Interruption and Rebalance Recommendation](#23-spot-interruption-and-rebalance-recommendation)
24. [On-Demand, Reserved Instances, and Savings Plans](#24-on-demand-reserved-instances-and-savings-plans)
25. [Dedicated Instances and Dedicated Hosts](#25-dedicated-instances-and-dedicated-hosts)
26. [Placement Groups](#26-placement-groups)
27. [Security Groups](#27-security-groups)
28. [EC2 Key Pairs](#28-ec2-key-pairs)
29. [Monitoring and Status Checks](#29-monitoring-and-status-checks)
30. [Auto Scaling with EC2](#30-auto-scaling-with-ec2)
31. [Launch Template](#31-launch-template)
32. [EC2 High Availability Architecture](#32-ec2-high-availability-architecture)
33. [Common EC2 Failure Scenarios](#33-common-ec2-failure-scenarios)
34. [Useful AWS CLI Commands](#34-useful-aws-cli-commands)
35. [Frequently Asked Interview Questions](#35-frequently-asked-interview-questions)
36. [One-Line Interview Answers](#36-one-line-interview-answers)
37. [Official References](#37-official-references)

---

# 1. Introduction

**Amazon EC2 (Elastic Compute Cloud)** is an AWS service that provides resizable compute capacity in the AWS Cloud.

An EC2 instance is a virtual server running on AWS infrastructure.

When launching an instance, you typically select:

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
```

AWS manages the underlying physical infrastructure, while the customer is responsible for the guest operating system, applications, configuration, and workload.

---

# 2. What is Amazon EC2?

Amazon EC2 provides on-demand, resizable compute capacity.

It allows organizations to provision virtual machines without purchasing physical servers.

Typical EC2 workloads include:

* Web servers
* Application servers
* CI/CD agents
* Jenkins workers
* Databases
* Containers
* Batch processing
* Data processing
* Monitoring systems
* Development environments

Simplified:

```text
AWS Infrastructure
        │
        ▼
      EC2
        │
   ┌────┴────┐
   ▼         ▼
Instance  Instance
   │         │
   ▼         ▼
Application Application
```

---

# 3. What is an EC2 Instance?

An **EC2 instance** is a running virtual server provisioned from an AMI using a selected instance type and configuration.

Example:

```text
EC2 Instance
├── AMI
├── vCPU
├── Memory
├── Network Interface
├── Security Groups
├── EBS Volumes
├── IAM Role / Instance Profile
└── Optional User Data
```

Important components:

| Component | Determines |
|---|---|
| AMI | OS and initial software |
| Instance Type | vCPU, memory, network, accelerator characteristics |
| EBS / Instance Store | Storage |
| ENI | Network connectivity |
| Security Group | Network access control |
| IAM Role | AWS API permissions |

---

# 4. EC2 Launch Components

A typical launch decision looks like:

```text
1. Select AMI
       ↓
2. Select Instance Type
       ↓
3. Select VPC / Subnet
       ↓
4. Configure Network / ENI
       ↓
5. Configure Security Groups
       ↓
6. Attach IAM Instance Profile
       ↓
7. Configure Storage
       ↓
8. Configure User Data
       ↓
9. Launch
```

The final behavior of the instance depends on the complete configuration, not just the instance type.

---

# 5. What is an AMI?

**AMI (Amazon Machine Image)** is a template used to launch EC2 instances.

An AMI defines the software image and block-device mapping used during launch.

It can contain:

* Operating system
* Installed packages
* Application software
* Configuration
* Boot configuration

Example:

```text
Golden AMI
    │
    ├── Ubuntu
    ├── Java
    ├── Nginx
    ├── Monitoring Agent
    └── Security Configuration
           │
           ▼
      EC2 Instances
```

AMI sources can include:

* AWS-provided images
* Custom images
* Marketplace images
* Shared images
* Images copied between Regions, where supported

## Golden AMI Pattern

A common DevOps pattern is:

```text
Base AMI
   ↓
Install Dependencies
   ↓
Configure Application
   ↓
Security Hardening
   ↓
Create Golden AMI
   ↓
Launch Multiple EC2 Instances
```

This reduces configuration drift between instances.

---

# 6. What is an Instance Type?

An EC2 instance type defines the compute characteristics presented to the instance.

It influences:

* vCPUs
* Memory
* Network performance
* EBS performance capability
* Accelerator availability
* Local instance-store availability on supported types

Example notation:

```text
t3.small
c7i.large
r7i.large
```

The family indicates the general optimization, while the size indicates the relative capacity within that family.

---

# 7. EC2 Instance Families

## General Purpose

Examples:

```text
T
M
```

Used for:

* Web servers
* Application servers
* Development environments
* General-purpose workloads

## Compute Optimized

Example:

```text
C
```

Used for:

* CPU-heavy workloads
* Batch processing
* High-performance web services
* Video processing
* Scientific workloads

## Memory Optimized

Example:

```text
R
```

Used for:

* In-memory databases
* Large caches
* Real-time analytics
* Memory-intensive applications

## Storage Optimized

Examples include supported storage-optimized families.

Used for:

* High local storage I/O
* Data processing
* Local caching
* Temporary high-throughput datasets

## Accelerated Computing

Examples include GPU/accelerator families.

Used for:

* Machine learning
* Deep learning
* Graphics
* Scientific simulations
* Parallel compute workloads

### Selection Rule

Choose the instance family based on the **actual bottleneck**:

```text
CPU bottleneck      → Compute optimized
Memory bottleneck   → Memory optimized
Storage I/O         → Storage optimized
GPU workload        → Accelerated computing
Balanced workload   → General purpose
```

---

# 8. User Data

**EC2 user data** is information provided during instance launch, commonly a shell script on Linux or PowerShell/cloud-init-related configuration on Windows.

Typical uses:

* Install packages
* Create configuration files
* Start services
* Register systems
* Bootstrap agents
* Perform first-boot configuration

Example:

```bash
#!/bin/bash

apt update -y
apt install -y nginx
systemctl enable nginx
systemctl start nginx
```

Flow:

```text
EC2 Launch
    │
    ▼
User Data
    │
    ▼
Bootstrap
    │
    ├── Install packages
    ├── Configure application
    └── Start services
```

### Important Considerations

User data is not a secret-management mechanism. AWS documentation warns that instance metadata and user data are accessible from inside the instance, so sensitive credentials should not be placed in user data.

For sensitive configuration, use services such as:

```text
IAM Roles
AWS Secrets Manager
SSM Parameter Store
```

---

# 9. Instance Metadata and IMDSv2

**Instance Metadata Service (IMDS)** provides information about the running EC2 instance.

Examples include:

* Instance ID
* Local IP
* Availability Zone
* Network information
* Instance identity information

Applications can access metadata from inside the instance.

A common IMDSv2 flow is:

```text
Application
    │
    ▼
Request IMDSv2 token
    │
    ▼
IMDS
    │
    ▼
Instance metadata
```

## IMDSv2

IMDSv2 uses a session-oriented token mechanism.

Example on a Linux instance:

```bash
TOKEN=$(curl -X PUT \
  "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
```

Then:

```bash
curl \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id
```

IMDSv2 is recommended because it provides stronger protection against some credential-access attack paths.

---

# 10. IAM Role and Instance Profile

An EC2 application should use an IAM role instead of long-lived AWS access keys stored on the instance.

The relationship is:

```text
IAM Role
   ↓
Instance Profile
   ↓
EC2 Instance
   ↓
IMDS
   ↓
Temporary Credentials
   ↓
Application
   ↓
AWS APIs
```

Example:

```text
EC2
 │
 └── IAM Role: S3ReadRole
          │
          ▼
       S3 API
```

This allows an application to access AWS APIs without embedding permanent credentials in configuration files.

---

# 11. EC2 Networking

An EC2 instance is connected to a VPC through one or more **Elastic Network Interfaces (ENIs)**.

Basic structure:

```text
Region
  │
  ▼
Availability Zone
  │
  ▼
VPC
  │
  ▼
Subnet
  │
  ▼
ENI
  │
  ▼
EC2 Instance
```

Important networking concepts include:

* VPC
* Subnet
* ENI
* Private IPv4
* Public IPv4
* Elastic IP
* IPv6
* Route Tables
* Internet Gateway
* NAT Gateway
* Security Groups
* Network ACLs

---

# 12. Private IPv4, Public IPv4, and Elastic IP

## Private IPv4

Used for communication inside the VPC and connected networks.

```text
10.0.1.10
```

Private addresses are normally retained across stop/start operations because the ENI remains attached.

## Public IPv4

Provides Internet-routable addressing when the routing and security configuration permit it.

An automatically assigned public IPv4 address is normally released on stop and a new one can be assigned on a subsequent start.

## Elastic IP

An **Elastic IP (EIP)** is a static public IPv4 address allocated to the account.

Use it when a stable public IPv4 address is required.

However:

> For most application endpoints, DNS is preferable to hard-coding public IP addresses.

### Comparison

| Property | Private IPv4 | Public IPv4 | Elastic IP |
|---|---|---|---|
| VPC communication | ✅ | ✅ | ✅ |
| Internet-routable | ❌ | ✅ | ✅ |
| Stable across stop/start | ✅ | Usually no | ✅ |
| Typical use | Internal traffic | Internet access | Stable public IPv4 |

---

# 13. Elastic Network Interface (ENI)

An **ENI** is a virtual network interface in a VPC.

It can contain:

* Primary private IPv4 address
* Secondary private IPv4 addresses
* IPv6 addresses
* Security groups
* MAC address
* Subnet association
* Elastic IP association through supported configurations

An EC2 instance can have one or more ENIs depending on the instance type and configuration.

Simplified:

```text
EC2
 │
 ├── eth0 / Primary ENI
 │
 └── eth1 / Secondary ENI
```

ENIs are useful for:

* Multi-homed instances
* Network separation
* Application/network appliances
* Failover designs
* Secondary interfaces

---

# 14. EC2 Storage

EC2 can use two major storage models:

```text
EC2 Storage
├── EBS
└── Instance Store
```

### EBS

Persistent network-attached block storage.

### Instance Store

Local ephemeral block storage physically attached to the host.

---

# 15. EBS

**Amazon Elastic Block Store (EBS)** provides persistent block storage for EC2.

Typical uses:

* Root volume
* Application filesystem
* Database storage
* Persistent application data

EBS supports:

* Snapshots
* Encryption
* Different volume types
* Elastic volume modifications for supported configurations

Example:

```text
EC2
 │
 ├── Root EBS
 │
 └── Data EBS
```

An EBS volume can be formatted with a filesystem:

```bash
sudo mkfs.ext4 /dev/xvdf
sudo mkdir /data
sudo mount /dev/xvdf /data
```

---

# 16. Instance Store

Instance Store is **local ephemeral storage** attached to supported EC2 instance types.

Advantages:

* Very high local I/O
* Low latency
* High throughput

Disadvantages:

* Ephemeral
* Data is not persistent like EBS
* Cannot be treated as durable application storage

Good uses:

* Cache
* Scratch space
* Temporary processing data
* Intermediate files

Bad use:

```text
Only copy of database data
```

---

# 17. EBS vs Instance Store

| Feature | EBS | Instance Store |
|---|---|---|
| Storage | Network-attached block | Local host-attached block |
| Persistence | ✅ | ❌ |
| Survives normal stop | ✅ | ❌ |
| Snapshot | ✅ | ❌ EBS-style snapshot |
| High local I/O | Good to very high depending on volume/instance | Very high |
| Typical use | OS, database, persistent data | Cache, scratch, temporary data |

### Interview Answer

> Use EBS when data must persist independently of the instance lifecycle. Use Instance Store for temporary or recreatable data where local performance matters.

---

# 18. gp3 EBS Volume

`gp3` is a general-purpose SSD EBS volume type.

Current AWS documentation states:

```text
Size:
1 GiB → 64 TiB

Baseline IOPS:
3,000

Baseline Throughput:
125 MiB/s
```

Additional performance can be provisioned up to:

```text
80,000 IOPS
2,000 MiB/s
```

subject to the volume and EC2 instance limits. citeturn912561search0

Example:

```text
gp3
 ├── Storage = 100 GiB
 ├── IOPS    = 6000
 └── Throughput = 250 MiB/s
```

One important advantage of gp3 is that storage size, IOPS, and throughput can be provisioned more independently than gp2.

---

# 19. EC2 Lifecycle

EC2 instance states include:

```text
pending
   ↓
running
   ↓
stopping
   ↓
stopped
   ↓
pending
   ↓
running
```

Or:

```text
running
   ↓
shutting-down
   ↓
terminated
```

Simplified:

```text
                 ┌──────────────┐
                 │   pending    │
                 └──────┬───────┘
                        ▼
                 ┌──────────────┐
                 │   running    │
                 └──┬────────┬──┘
                    │        │
                 stop       terminate
                    │        │
                    ▼        ▼
                stopped   terminated
```

---

# 20. Stop, Start, Reboot, Hibernate, and Terminate

## Reboot

Restarts the operating system.

```text
Running
   ↓
Reboot
   ↓
Running
```

The instance generally remains on the same host computer.

## Stop

For EBS-backed instances:

```text
Running
   ↓
Stopped
```

EBS volumes and ENIs persist, while RAM, instance-store data, and the automatically assigned public IPv4 are lost. citeturn912561search3turn912561search5

## Start

Starts a stopped EBS-backed instance.

The private IPv4 normally remains the same, while an automatically assigned public IPv4 can change. citeturn912561search3turn912561search5

## Hibernate

Hibernation saves RAM state to the root EBS volume and later restores it when the instance starts, subject to the supported instance/AMI configuration.

## Terminate

Permanently removes the instance.

Whether attached EBS volumes are deleted depends on `DeleteOnTermination`.

---

# 21. EBS DeleteOnTermination

When an EBS-backed EC2 instance is terminated:

```text
DeleteOnTermination = true
        ↓
Volume deleted

DeleteOnTermination = false
        ↓
Volume preserved
```

Check the attribute before terminating important systems.

Example:

```bash
aws ec2 describe-instances \
  --instance-ids <instance-id> \
  --query "Reservations[].Instances[].BlockDeviceMappings[].{Device:DeviceName,DeleteOnTermination:Ebs.DeleteOnTermination}"
```

---

# 22. Spot Instances

Spot Instances use spare EC2 capacity at a potentially large discount compared with On-Demand.

They are appropriate when applications can tolerate interruption.

Good workloads:

* Batch processing
* CI/CD workers
* Rendering
* Distributed data processing
* Stateless application capacity
* Parallel jobs

Architecture:

```text
Auto Scaling Group
        │
   ┌────┼────┐
   ▼    ▼    ▼
 Spot Spot Spot
```

A resilient design should use multiple capacity pools rather than depending on a single Spot instance.

---

# 23. Spot Interruption and Rebalance Recommendation

When AWS needs to reclaim Spot capacity, a normal Spot interruption provides a **two-minute interruption notice**.

Recommended response:

```text
Interruption Notice
       ↓
Stop accepting work
       ↓
Checkpoint state
       ↓
Drain active work
       ↓
Terminate / Replace
```

AWS can also issue a **rebalance recommendation** before an interruption when a Spot Instance is at elevated risk.

The two signals differ:

| Signal | Meaning |
|---|---|
| Rebalance Recommendation | Increased interruption risk |
| Interruption Notice | Interruption has been scheduled |

Spot therefore should always be treated as:

> **Interruptible capacity, not guaranteed capacity.**

---

# 24. On-Demand, Reserved Instances, and Savings Plans

## On-Demand

Pay for compute without a long-term commitment.

Best for:

* Unpredictable workloads
* Short-term environments
* Development/testing
* New workloads

## Reserved Instances

Commit to an eligible configuration in exchange for a pricing discount.

Best for:

* Predictable, steady workloads
* Stable EC2 usage

Important:

> Reserved Instances are primarily a billing discount mechanism, not a unique instance type.

## Savings Plans

Commit to a consistent compute spend per hour in exchange for discounted pricing.

Compared with standard Reserved Instances, Savings Plans can provide more flexibility around eligible compute usage, depending on the plan type.

AWS currently recommends considering Savings Plans for their flexibility. citeturn912561search2

### Comparison

| Option | Commitment | Best Use |
|---|---|---|
| On-Demand | None | Variable usage |
| Reserved Instance | Eligible configuration | Stable configuration |
| Savings Plan | Hourly compute spend | Stable spend with greater compute flexibility |
| Spot | No long-term commitment | Interruptible workloads |

---

# 25. Dedicated Instances and Dedicated Hosts

## Dedicated Instance

Runs instances on hardware dedicated to a single AWS account.

Use cases can include:

* Isolation requirements
* Certain compliance requirements

## Dedicated Host

Provides an entire physical server dedicated to the account.

Useful for:

* Certain software licensing requirements
* Compliance
* Host-level placement/visibility needs

### Difference

```text
Dedicated Instance
→ Dedicated hardware, but you manage instances

Dedicated Host
→ Entire physical server allocated to your account
```

---

# 26. Placement Groups

Placement groups influence how EC2 instances are physically placed.

There are three common strategies:

| Strategy | Main Purpose |
|---|---|
| Cluster | Low-latency, high-throughput networking |
| Spread | Individual instance failure isolation |
| Partition | Failure-domain separation for distributed systems |

## Cluster

```text
Node 1
Node 2
Node 3
   │
   ▼
Close placement
```

Useful for tightly coupled workloads.

## Spread

```text
Node 1 → Hardware A
Node 2 → Hardware B
Node 3 → Hardware C
```

Useful when each individual instance should be isolated from others.

## Partition

```text
Partition 1 → Group of hardware
Partition 2 → Group of hardware
Partition 3 → Group of hardware
```

Useful for distributed systems such as Kafka and Cassandra where correlated hardware failures should be reduced.

---

# 27. Security Groups

A **Security Group** is a stateful virtual firewall associated with an EC2 network interface.

Characteristics:

| Characteristic | Security Group |
|---|---|
| Scope | ENI / resource |
| State | Stateful |
| Inbound rules | Allow |
| Outbound rules | Allow |
| Explicit deny | Not supported |
| Return traffic | Automatically allowed for permitted stateful flows |

Example:

```text
Internet
   │
   │ HTTPS 443
   ▼
ALB
   │
   │ Application port
   ▼
EC2 Security Group
```

A common architecture is:

```text
Internet
   ↓
ALB SG
   ↓
EC2 Target SG
```

The EC2 security group should normally permit application traffic from the appropriate ALB security group rather than exposing the backend directly.

---

# 28. EC2 Key Pairs

A key pair is used for SSH access to Linux EC2 instances or corresponding administrative access patterns for supported instance configurations.

Typical flow:

```text
EC2 Launch
    │
    ▼
Key Pair
    │
    ▼
Private Key
    │
    ▼
SSH
```

Example:

```bash
chmod 400 devops.pem

ssh -i devops.pem ubuntu@<public-ip>
```

### Important

Do not store private keys in Git repositories.

Also, when using AWS Systems Manager Session Manager, direct SSH key-based access may not be required.

---

# 29. Monitoring and Status Checks

EC2 provides status checks and CloudWatch monitoring.

## System Status Check

Checks the underlying AWS infrastructure.

Example failures can indicate:

* Host hardware problems
* Power issues
* AWS networking problems

## Instance Status Check

Checks whether the instance's guest OS and networking stack are functioning correctly.

## EBS Status Check

EBS can also report I/O-related status information for supported scenarios.

## CloudWatch

Useful EC2 metrics include:

```text
CPUUtilization
NetworkIn
NetworkOut
DiskReadOps
DiskWriteOps
DiskReadBytes
DiskWriteBytes
StatusCheckFailed
```

For memory utilization, install an agent because standard EC2 CloudWatch metrics do not provide guest OS memory utilization automatically.

---

# 30. Auto Scaling with EC2

An **Auto Scaling Group (ASG)** can automatically maintain a desired number of EC2 instances.

Simplified architecture:

```text
            Load Balancer
                 │
                 ▼
          Auto Scaling Group
           /       |       \
          ▼        ▼        ▼
        EC2-1    EC2-2    EC2-3
```

ASG can maintain:

```text
Desired Capacity = 3
Minimum Capacity = 2
Maximum Capacity = 6
```

If one instance fails:

```text
3 instances
    │
Instance fails
    ↓
2 instances
    │
    ▼
ASG launches replacement
    │
    ▼
3 instances
```

---

# 31. Launch Template

A **Launch Template** defines how new EC2 instances should be launched.

It can contain:

* AMI
* Instance type
* Security groups
* IAM instance profile
* User data
* Block device mappings
* Network configuration
* Monitoring settings
* Metadata options

Example:

```text
Launch Template
      │
      ├── AMI
      ├── t3.small
      ├── Security Group
      ├── IAM Role
      ├── User Data
      └── EBS
            │
            ▼
       Auto Scaling Group
            │
            ▼
         EC2 Fleet
```

A Launch Template is commonly preferred for modern EC2/ASG designs.

---

# 32. EC2 High Availability Architecture

A production application should avoid depending on a single instance.

Example:

```text
                 Internet
                    │
                    ▼
                   ALB
               /         \
             AZ-A       AZ-B
              │           │
           EC2-1        EC2-2
              │           │
              └─────┬─────┘
                    ▼
               Application
```

Combine:

```text
ALB
 +
Target Groups
 +
ASG
 +
Multiple AZs
 +
Health Checks
```

This provides:

* Automatic replacement
* Horizontal scaling
* Failure isolation
* Better availability

---

# 33. Common EC2 Failure Scenarios

## Scenario 1: EC2 is Running but Application Is Not Reachable

Check:

```text
Security Group
   ↓
NACL
   ↓
Route Table
   ↓
Subnet
   ↓
Application listening port
   ↓
OS firewall
```

Use:

```bash
ss -lntp
```

and:

```bash
curl http://localhost:<port>
```

---

## Scenario 2: SSH Times Out

Check:

```text
Is instance running?
        ↓
Does it have network connectivity?
        ↓
Security Group TCP 22
        ↓
Subnet route
        ↓
NACL
        ↓
Public IP / Bastion / SSM
```

A timeout is not automatically an authentication failure.

---

## Scenario 3: EC2 Has No Public Internet Access

A public IP alone is not enough.

For Internet access, check:

```text
Public IP
   +
Route Table
   +
Internet Gateway
   +
Security Group
   +
NACL
```

For private subnets:

```text
EC2
 ↓
Private Subnet
 ↓
NAT Gateway
 ↓
Internet Gateway
 ↓
Internet
```

---

## Scenario 4: Application Cannot Call AWS APIs

Check:

```text
IAM Role
   ↓
Instance Profile
   ↓
IMDS / Credentials
   ↓
IAM Permissions
```

Do not solve this by placing permanent AWS access keys on the server.

---

## Scenario 5: EC2 Stops Unexpectedly

Check:

```text
CloudTrail
CloudWatch
EC2 state-change events
Spot interruption events
Auto Scaling activity
```

Determine whether the instance was:

```text
Stopped
Rebooted
Terminated
Interrupted as Spot
```

---

# 34. Useful AWS CLI Commands

## Describe Instances

```bash
aws ec2 describe-instances
```

Specific instance:

```bash
aws ec2 describe-instances \
  --instance-ids i-0123456789abcdef0
```

## Start Instance

```bash
aws ec2 start-instances \
  --instance-ids i-0123456789abcdef0
```

## Stop Instance

```bash
aws ec2 stop-instances \
  --instance-ids i-0123456789abcdef0
```

## Reboot Instance

```bash
aws ec2 reboot-instances \
  --instance-ids i-0123456789abcdef0
```

## Terminate Instance

```bash
aws ec2 terminate-instances \
  --instance-ids i-0123456789abcdef0
```

## Describe Instance Types

```bash
aws ec2 describe-instance-types
```

## Describe AMIs

```bash
aws ec2 describe-images \
  --owners self
```

## Describe Security Groups

```bash
aws ec2 describe-security-groups
```

## Describe ENIs

```bash
aws ec2 describe-network-interfaces
```

## Describe EBS Volumes

```bash
aws ec2 describe-volumes
```

## Describe Snapshots

```bash
aws ec2 describe-snapshots \
  --owner-ids self
```

## Describe Instance Status

```bash
aws ec2 describe-instance-status \
  --instance-ids i-0123456789abcdef0
```

## Describe Spot Requests

```bash
aws ec2 describe-spot-instance-requests
```

## Describe Tags

```bash
aws ec2 describe-tags
```

---

# 35. Frequently Asked Interview Questions

| Question | Answer |
|---|---|
| **What is EC2?** | AWS service providing resizable virtual compute capacity. |
| **What is an EC2 instance?** | A running virtual server provisioned from an AMI using a selected instance type and configuration. |
| **What is an AMI?** | A template used to launch EC2 instances containing the OS/software configuration and block-device mappings needed for launch. |
| **What is an instance type?** | A definition of the compute characteristics exposed to the instance, such as vCPU, memory, and network capability. |
| **What is user data?** | Launch-time configuration or bootstrap data used to initialize an instance. |
| **What is instance metadata?** | Information about the running EC2 instance available through the Instance Metadata Service. |
| **What is IMDSv2?** | A session-oriented version of the EC2 Instance Metadata Service that provides stronger protection against certain credential-access attacks. |
| **Why use IAM roles with EC2?** | To provide temporary AWS credentials to applications without storing long-lived access keys on the instance. |
| **What is an instance profile?** | The container used to associate an IAM role with an EC2 instance. |
| **What is an ENI?** | A virtual network interface providing network connectivity and addresses/security-group associations. |
| **What is EBS?** | Persistent block storage designed for EC2. |
| **What is Instance Store?** | Local ephemeral block storage available on supported instance types. |
| **When should EBS be used?** | When data must persist independently of the instance's host lifecycle. |
| **When should Instance Store be used?** | For temporary, cache, scratch, or recreatable high-performance data. |
| **What is gp3?** | General-purpose SSD EBS volume type with independent provisioning of storage, IOPS, and throughput within its supported limits. |
| **What is the baseline gp3 performance?** | 3,000 IOPS and 125 MiB/s throughput, with higher provisioned limits available within supported constraints. |
| **What is the difference between stop and reboot?** | Reboot restarts the OS; stop shuts down an EBS-backed instance and releases compute capacity until it is started again. |
| **What is the difference between stop and terminate?** | Stop preserves the EBS-backed instance resources for restart; terminate permanently removes the instance and may delete EBS volumes depending on DeleteOnTermination. |
| **What happens to a private IPv4 on stop/start?** | The private IPv4 on the persistent ENI is retained. |
| **What happens to an automatically assigned public IPv4 on stop/start?** | It is released on stop and a new public IPv4 may be assigned on start. |
| **How do you keep a stable public IPv4?** | Associate an Elastic IP, when a static public IPv4 is actually required. |
| **What is a Security Group?** | A stateful virtual firewall associated with a resource's ENI. |
| **Can Security Groups have deny rules?** | No. Security Groups support allow rules; explicit deny rules are not supported. |
| **What is a placement group?** | An EC2 placement strategy that controls instance placement for latency, isolation, or distributed-workload failure domains. |
| **What are the placement group strategies?** | Cluster, Spread, and Partition. |
| **What is a Spot Instance?** | EC2 capacity offered at a discount that can be interrupted by AWS. |
| **How much interruption notice does Spot normally provide?** | Two minutes for normal Spot interruption handling. |
| **What is a Spot rebalance recommendation?** | A signal that a Spot instance is at elevated interruption risk, allowing proactive replacement or draining. |
| **Do Spot Instances have a fixed lifetime?** | No. They can continue while capacity is available and until an interruption or other lifecycle event occurs. |
| **What is an On-Demand instance?** | EC2 capacity without a long-term commitment. |
| **What is a Reserved Instance?** | A billing discount commitment for eligible EC2 configurations. |
| **What is a Savings Plan?** | A compute-spend commitment that provides discounted pricing with greater flexibility than some RI models. |
| **What is a Dedicated Instance?** | An EC2 instance running on hardware dedicated to the AWS account. |
| **What is a Dedicated Host?** | An entire physical server dedicated to the AWS account, useful for certain licensing, compliance, and host-level requirements. |
| **What is a Launch Template?** | A reusable EC2 launch configuration used by EC2 launches and commonly by Auto Scaling Groups. |
| **What is an Auto Scaling Group?** | A service construct that maintains a desired number of EC2 instances and can scale/replace them according to policies and health. |
| **Why use multiple Availability Zones?** | To reduce dependency on a single AZ and improve application availability. |
| **What is the difference between EBS root and Instance Store root?** | EBS root storage can support stop/start; Instance Store root is ephemeral and cannot be stopped/started in the same way. |
| **What happens when an EC2 instance is terminated?** | The instance is permanently removed; EBS volumes are deleted or retained according to DeleteOnTermination. |
| **What is DeleteOnTermination?** | EBS volume behavior that determines whether a volume is deleted with the EC2 instance. |
| **What is EC2 user data used for?** | Bootstrapping and first-start configuration. |
| **Should passwords be stored in user data?** | No. Use IAM roles, Secrets Manager, or Parameter Store for sensitive information. |
| **What is the difference between private IP and Elastic IP?** | Private IP is used for internal VPC communication; Elastic IP is a static public IPv4 address. |
| **What is an ENI used for?** | Network connectivity, secondary IPs, security groups, and multi-interface networking. |
| **What is a status check?** | AWS health checks that identify underlying system or instance-level problems. |
| **How do you troubleshoot an unreachable EC2 server?** | Check instance state, routes, security groups, NACLs, application listeners, OS firewall, and connectivity path. |
| **What is the purpose of an AMI in an ASG?** | It provides a consistent machine image for launching replacement and scaling instances. |
| **Why use Launch Templates instead of manually configuring every instance?** | They standardize and automate the instance configuration used for repeated launches and Auto Scaling. |

---

# 36. One-Line Interview Answers

| Concept | One-Line Answer |
|---|---|
| **EC2** | AWS service providing resizable virtual compute capacity. |
| **Instance** | Running EC2 virtual server. |
| **AMI** | Template used to launch EC2 instances. |
| **Instance Type** | Defines vCPU, memory, networking, and other instance capabilities. |
| **User Data** | Launch-time bootstrap configuration. |
| **IMDS** | Service providing metadata about the running EC2 instance. |
| **IMDSv2** | Token/session-oriented version of EC2 metadata access. |
| **IAM Role** | Identity whose permissions can be used by workloads. |
| **Instance Profile** | Mechanism associating an IAM role with an EC2 instance. |
| **ENI** | Virtual network interface attached to a VPC resource. |
| **Private IPv4** | Internal VPC address. |
| **Public IPv4** | Internet-routable IPv4 address. |
| **Elastic IP** | Static public IPv4 allocated to an account. |
| **EBS** | Persistent network-attached block storage. |
| **Instance Store** | Local ephemeral block storage. |
| **gp3** | General-purpose SSD EBS volume with independently provisioned performance. |
| **Security Group** | Stateful virtual firewall. |
| **Placement Group** | Controls instance placement for performance or failure isolation. |
| **Spot** | Discounted, interruptible EC2 capacity. |
| **On-Demand** | EC2 capacity without long-term commitment. |
| **Reserved Instance** | Commitment-based pricing discount for eligible configurations. |
| **Savings Plan** | Commitment-based compute-spend discount. |
| **Launch Template** | Reusable EC2 launch configuration. |
| **Auto Scaling Group** | Maintains and scales a fleet of EC2 instances. |
| **Stop** | Stops an EBS-backed instance while preserving eligible persistent resources. |
| **Start** | Starts a stopped EBS-backed instance. |
| **Reboot** | Restarts the operating system. |
| **Hibernate** | Saves memory state and later restores it, where supported. |
| **Terminate** | Permanently removes the EC2 instance. |
| **DeleteOnTermination** | Controls whether an EBS volume is deleted when the instance terminates. |
| **Status Check** | Indicates whether AWS infrastructure and/or the instance are healthy. |

---

# Practical EC2 Cheat Sheet

## Identify the Host

```bash
uname -a
hostname
```

## Check Network

```bash
ip addr
ip route
```

## Check Listening Ports

```bash
ss -lntp
```

## Check Disk

```bash
lsblk
df -h
```

## Check Memory

```bash
free -h
```

## Check CPU / Load

```bash
uptime
top
```

## Query Instance Metadata with IMDSv2

```bash
TOKEN=$(curl -X PUT \
  "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

curl \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id
```

## Check Instance Identity

```bash
curl \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/dynamic/instance-identity/document
```

## AWS CLI

```bash
aws ec2 describe-instances
aws ec2 describe-volumes
aws ec2 describe-network-interfaces
aws ec2 describe-security-groups
aws ec2 describe-instance-status
```

---

# EC2 Architecture to Remember

```text
                           AWS Region
                               │
                    ┌──────────┴──────────┐
                    │                     │
                   AZ-A                  AZ-B
                    │                     │
                 Subnet                Subnet
                    │                     │
                  EC2-1                 EC2-2
                    │                     │
              ┌─────┼─────┐         ┌─────┼─────┐
              │     │     │         │     │     │
             ENI   EBS  SG          ENI   EBS  SG
                    │                     │
                    └──────────┬──────────┘
                               │
                              ALB
                               │
                            Clients
```

A production architecture commonly combines:

```text
AMI
 +
Launch Template
 +
Auto Scaling Group
 +
Multiple AZs
 +
Load Balancer
 +
Target Groups
 +
Security Groups
 +
CloudWatch
 +
IAM Roles
```

This provides:

* Consistent instance configuration
* Horizontal scaling
* Failure replacement
* High availability
* Centralized monitoring
* Least-privilege AWS API access

---

# Official References

The following AWS documentation should be used to verify current EC2 behavior, limits, defaults, and pricing features:

1. [Amazon EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/)
2. [EC2 Instance Types](https://docs.aws.amazon.com/ec2/latest/instancetypes/ec2-instance-types.html)
3. [Amazon Machine Images (AMIs)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html)
4. [EC2 User Data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html)
5. [EC2 Instance Metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)
6. [Access EC2 Instance Metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html)
7. [EC2 IAM Roles and Instance Profiles](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)
8. [EC2 Instance Lifecycle](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html)
9. [Stop and Start EC2 Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/how-ec2-instance-stop-start-works.html)
10. [Amazon EBS User Guide](https://docs.aws.amazon.com/ebs/latest/userguide/what-is-ebs.html)
11. [Amazon EBS General Purpose SSD (gp3)](https://docs.aws.amazon.com/ebs/latest/userguide/general-purpose.html)
12. [EC2 Spot Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances.html)
13. [EC2 Placement Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html)
14. [EC2 Security Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html)
15. [EC2 Launch Templates](https://docs.aws.amazon.com/autoscaling/ec2/userguide/launch-templates.html)
16. [EC2 Auto Scaling](https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html)
17. [Amazon EC2 Pricing](https://aws.amazon.com/ec2/pricing/)
18. [AWS Savings Plans](https://aws.amazon.com/savingsplans/)
19. [Amazon EC2 Reserved Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-reserved-instances.html)

> **Note:** AWS instance families, limits, supported features, EBS performance, pricing models, and service behavior can change. Always verify production decisions against the current AWS documentation for the specific Region, instance type, and configuration you are using.
