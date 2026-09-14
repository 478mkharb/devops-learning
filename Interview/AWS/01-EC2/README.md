# Amazon EC2 Interview & Practical Notes

> **Scope:** Amazon EC2 and its directly related capabilities only.  
> **Audience:** DevOps Engineers, Cloud Engineers, System Administrators, and AWS interview candidates.

[![AWS](https://img.shields.io/badge/AWS-EC2-orange?logo=amazonaws)](https://aws.amazon.com/ec2/)
[![Focus](https://img.shields.io/badge/Focus-Interview%20%2B%20Hands--On-blue)](#learning-roadmap)

---

## Table of Contents

| Stage | Topic |
|---:|---|
| 1 | [EC2 Fundamentals](#1-ec2-fundamentals) |
| 2 | [AMI and Instance Types](#2-ami-and-instance-types) |
| 3 | [User Data, Metadata and IAM](#3-user-data-metadata-and-iam) |
| 4 | [EC2 Networking](#4-ec2-networking) |
| 5 | [EC2 Storage](#5-ec2-storage) |
| 6 | [EC2 Lifecycle](#6-ec2-lifecycle) |
| 7 | [EC2 Pricing Options](#7-ec2-pricing-options) |
| 8 | [Placement Groups and Tenancy](#8-placement-groups-and-tenancy) |
| 9 | [Security Groups and Key Pairs](#9-security-groups-and-key-pairs) |
| 10 | [Monitoring and Troubleshooting](#10-monitoring-and-troubleshooting) |
| 11 | [Launch Templates and Auto Scaling](#11-launch-templates-and-auto-scaling) |
| 12 | [High Availability Architecture](#12-high-availability-architecture) |
| 13 | [AWS CLI and Linux Cheat Sheet](#13-aws-cli-and-linux-cheat-sheet) |
| 14 | [Final Interview Revision](#14-final-interview-revision) |
| 15 | [References](#15-references) |

---

## Learning Roadmap

| Level | Study order | Expected outcome |
|---|---|---|
| Beginner | Fundamentals → AMI → Instance Type → User Data | Launch and explain an EC2 instance |
| Intermediate | Networking → Storage → Lifecycle → IAM | Operate and secure an EC2 server |
| Advanced | Spot → Placement → Monitoring → ASG | Design resilient and cost-efficient EC2 workloads |
| Interview-ready | Troubleshooting → Architecture → Scenario questions | Explain decisions, not only definitions |

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
| What is the shared responsibility model for EC2? | AWS manages the underlying cloud infrastructure; the customer manages the guest OS, applications, data and configuration. |
| What is the difference between vertical and horizontal scaling? | Vertical scaling increases the size of one instance; horizontal scaling adds more instances. |

---

# 2. AMI and Instance Types

## Amazon Machine Image (AMI)

An **AMI** is a template used to launch EC2 instances.

| AMI contains or defines | Explanation |
|---|---|
| Root volume template | OS and installed software |
| Block-device mapping | Volumes attached during launch |
| Launch permissions | Private, shared or public access |
| Boot configuration | Information needed to boot the instance |

### Golden AMI pattern

```text
Base AMI
  ↓
Install dependencies
  ↓
Configure application
  ↓
Apply security hardening
  ↓
Create Golden AMI
  ↓
Launch consistent EC2 instances
```

### Example: create an AMI

```bash
aws ec2 create-image \
  --instance-id i-xxxxxxxxxxxxxxxxx \
  --name "dev-web-golden-v1" \
  --description "NGINX and monitoring preinstalled" \
  --no-reboot
```

> `--no-reboot` can preserve uptime but may produce an inconsistent image if applications are actively writing data. Use it carefully.

## Instance Type

An instance type defines the compute characteristics exposed to an EC2 instance.

| Characteristic | Examples |
|---|---|
| CPU | vCPUs |
| Memory | RAM |
| Network | Network bandwidth |
| EBS | EBS bandwidth and limits |
| Accelerators | GPU or other accelerators |
| Local storage | Instance Store, where supported |

### Instance families

| Family | Optimization | Typical workload |
|---|---|---|
| T | Burstable general purpose | Dev/test, low or variable traffic |
| M | Balanced general purpose | Web and application servers |
| C | Compute optimized | CPU-heavy jobs, batch processing |
| R | Memory optimized | Caches, in-memory databases |
| I/D/H | Storage optimized | High local I/O and data processing |
| P/G/Inf/Trn | Accelerated computing | ML, graphics and parallel workloads |

### Selection rule

| Bottleneck | Choose |
|---|---|
| CPU | Compute optimized |
| Memory | Memory optimized |
| Storage I/O | Storage optimized |
| GPU/accelerator | Accelerated computing |
| Balanced workload | General purpose |
| Low or variable CPU usage | Burstable general purpose |

### Example

| Instance | Typical decision |
|---|---|
| `t3.small` | Small development server |
| `m7i.large` | Balanced application server |
| `c7i.large` | CPU-intensive workload |
| `r7i.large` | Memory-intensive workload |

### Interview Checkpoint — AMI and Instance Types

| Question | Answer |
|---|---|
| What is an AMI? | A launch template containing the OS/software image and block-device mappings. |
| Can one AMI launch multiple instances? | Yes, an AMI can be used to launch many instances. |
| What is a Golden AMI? | A tested, preconfigured image used to launch consistent servers. |
| What happens if the AMI used for launch is deregistered? | New launches depending on it fail because the image is unavailable. |
| What is an instance type? | A predefined combination of compute, memory, network and storage capabilities. |
| When should you use a T-family instance? | For workloads with low or variable CPU usage that can use CPU credits. |
| How do you select an instance type? | Measure CPU, memory, network, storage and accelerator requirements, then validate cost and performance. |

---

# 3. User Data, Metadata and IAM

## EC2 User Data

User data is launch-time bootstrap information, commonly a shell script on Linux.

### Common uses

| Use | Example |
|---|---|
| Install packages | Install NGINX |
| Configure files | Create application config |
| Start services | Enable and start systemd service |
| Install agents | SSM, monitoring or security agent |
| Register instance | Register with a target or configuration system |

### Example: Linux user data

```bash
#!/bin/bash
set -eux

apt-get update -y
apt-get install -y nginx

systemctl enable nginx
systemctl start nginx

echo "Hello from EC2" > /var/www/html/index.html
```

### Important points

| Point | Explanation |
|---|---|
| Execution | Usually runs during first boot |
| Secret storage | Do not store passwords or access keys in user data |
| Debugging | Check cloud-init logs |
| Repeatability | Make scripts idempotent where possible |

Useful logs:

```bash
sudo cloud-init status --long
sudo tail -f /var/log/cloud-init-output.log
```

## Instance Metadata and IMDSv2

The Instance Metadata Service provides information about the running instance.

| Metadata example | Use |
|---|---|
| Instance ID | Identify the current server |
| Local IPv4 | Internal networking |
| Availability Zone | Placement awareness |
| IAM role credentials | Temporary AWS credentials |
| Instance identity document | Instance verification |

### IMDSv2 example

```bash
TOKEN=$(curl -sS -X PUT \
  "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

curl -sS \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id
```

## IAM Role and Instance Profile

Use an IAM role instead of long-lived AWS access keys on the server.

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
AWS API
```

| Term | Meaning |
|---|---|
| IAM role | Identity with permissions and trust policy |
| Instance profile | Container used to pass a role to EC2 |
| Temporary credentials | Short-lived credentials delivered to the instance |
| Least privilege | Grant only the permissions required |

### Interview Checkpoint — Bootstrap, Metadata and IAM

| Question | Answer |
|---|---|
| What is user data? | Launch-time bootstrap configuration. |
| Is user data secure for passwords? | No. Use Secrets Manager, Parameter Store or IAM roles. |
| What is IMDS? | A service that provides instance information from inside EC2. |
| What is the difference between IMDSv1 and IMDSv2? | IMDSv2 uses a session token and provides stronger protection against certain credential-access paths. |
| Why use IAM roles on EC2? | They provide temporary credentials without storing permanent access keys. |
| What is an instance profile? | The mechanism that associates an IAM role with an EC2 instance. |
| How can you troubleshoot user data? | Check cloud-init status and `/var/log/cloud-init-output.log`. |

---

# 4. EC2 Networking

## Network hierarchy

```text
Region
  ↓
Availability Zone
  ↓
VPC
  ↓
Subnet
  ↓
ENI
  ↓
EC2 Instance
```

| Component | Responsibility |
|---|---|
| VPC | Logical network boundary |
| Subnet | IP range within one AZ |
| Route table | Determines next hop |
| Internet Gateway | Internet connectivity for public routing |
| NAT Gateway | Outbound internet access from private subnets |
| ENI | Network interface attached to EC2 |
| Security group | Stateful instance-level filtering |
| NACL | Stateless subnet-level filtering |

## Private IPv4, Public IPv4 and Elastic IP

| Property | Private IPv4 | Auto-assigned public IPv4 | Elastic IP |
|---|---|---|---|
| VPC communication | Yes | Yes | Yes |
| Internet-routable | No | Yes | Yes |
| Stable across stop/start | Usually yes | No | Yes |
| Typical use | Internal traffic | Temporary public access | Stable public endpoint |

> Prefer DNS names over hard-coding public IP addresses for application endpoints.

## Public subnet vs private subnet

| Public subnet | Private subnet |
|---|---|
| Route to an Internet Gateway | No direct route to an Internet Gateway |
| Instance may have public IPv4 | Instance normally has no public IPv4 |
| Suitable for public-facing components | Suitable for backend systems |
| Inbound access still requires security rules | Outbound internet may use NAT Gateway |

### Private subnet internet access

```text
Private EC2
  ↓
Private route table
  ↓
NAT Gateway
  ↓
Internet Gateway
  ↓
Internet
```

A public IP alone is not enough. Routing, security groups, NACLs and the gateway path must also be correct.

## Elastic Network Interface (ENI)

An ENI can contain:

| ENI attribute | Example |
|---|---|
| Primary private IPv4 | `10.0.1.10` |
| Secondary private IPv4s | Additional application addresses |
| IPv6 addresses | IPv6 connectivity |
| Security groups | Traffic control |
| MAC address | Interface identity |
| Subnet association | Network placement |

### Interview Checkpoint — Networking

| Question | Answer |
|---|---|
| Can a subnet span multiple AZs? | No. A subnet belongs to one Availability Zone. |
| Is a public IP enough for internet access? | No. A valid route through an Internet Gateway and appropriate security rules are also required. |
| How does a private EC2 instance download patches? | Route outbound traffic through a NAT Gateway, or use suitable VPC endpoints where supported. |
| What is an ENI? | A virtual network interface with IP addresses, security groups and subnet association. |
| What happens to an auto-assigned public IP after stop/start? | It is released on stop and a new one may be assigned on start. |
| How do you retain a stable public IPv4? | Associate an Elastic IP when a static address is genuinely required. |
| What is the difference between SG and NACL? | SG is stateful and attached to ENIs; NACL is stateless and applies at subnet level. |
| Why is a private subnet preferred for backend EC2? | It reduces direct internet exposure and allows controlled access through load balancers, bastions, SSM or private connectivity. |

---

# 5. EC2 Storage

## Storage models

| Storage | Description | Use |
|---|---|---|
| EBS | Persistent network-attached block storage | OS, databases, application data |
| Instance Store | Local ephemeral block storage | Cache, scratch and temporary data |

## EBS

Amazon Elastic Block Store provides persistent block storage for EC2.

| Capability | EBS |
|---|---|
| Root volume | Supported |
| Data volume | Supported |
| Encryption | Supported |
| Snapshots | Supported |
| Resize | Supported for supported configurations |
| Persistence | Independent of normal instance stop/start |

### Example: format and mount a Linux EBS volume

```bash
lsblk

sudo mkfs.ext4 /dev/xvdf
sudo mkdir -p /data
sudo mount /dev/xvdf /data

df -h
```

> Confirm the device name with `lsblk`. Formatting the wrong device destroys data.

## Instance Store

Instance Store is local storage available only on supported instance types.

| Advantage | Limitation |
|---|---|
| Very low latency | Ephemeral |
| High local I/O | Not durable application storage |
| High throughput | Data can be lost when the instance or host lifecycle requires it |

Good uses:

- Cache
- Scratch space
- Temporary processing files
- Re-creatable intermediate data

Bad use:

```text
Only copy of important database data
```

## EBS vs Instance Store

| Feature | EBS | Instance Store |
|---|---|---|
| Attachment | Network-attached | Host-local |
| Persistence | Persistent | Ephemeral |
| Survives normal stop | Yes | No |
| Snapshot support | Yes | No EBS-style snapshot |
| Best for | OS and durable data | Cache and scratch |
| Performance | Good to very high | Very high local I/O |

## gp3

`gp3` is a general-purpose SSD EBS volume type.

| Property | Typical gp3 baseline/limit |
|---|---:|
| Size | 1 GiB–64 TiB |
| Baseline IOPS | 3,000 |
| Baseline throughput | 125 MiB/s |
| Maximum provisioned IOPS | Up to 80,000 |
| Maximum provisioned throughput | Up to 2,000 MiB/s |

Actual limits depend on the volume, instance and Region.

### Interview Checkpoint — Storage

| Question | Answer |
|---|---|
| What is EBS? | Persistent block storage designed for EC2. |
| What is Instance Store? | Local ephemeral storage available on supported instance types. |
| When should EBS be used? | When data must survive instance stop/start and host lifecycle changes. |
| When should Instance Store be used? | For temporary, cache or recreatable high-performance data. |
| What is gp3? | General-purpose SSD EBS with independently provisioned storage, IOPS and throughput. |
| What is the difference between EBS and Instance Store? | EBS is persistent network storage; Instance Store is local and ephemeral. |
| What is a snapshot? | A point-in-time backup of an EBS volume stored in AWS. |
| What happens to EBS on termination? | Deletion depends on the volume’s `DeleteOnTermination` setting. |

---

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
| What is hibernation? | RAM state is saved to the root EBS volume and restored later, where supported. |
| What is `DeleteOnTermination`? | An EBS setting that controls whether a volume is deleted with the instance. |
| Do EBS charges stop when an instance is stopped? | No. EBS storage charges continue while the volume exists. |

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
| How much normal Spot interruption notice is provided? | Two minutes. |
| What is a rebalance recommendation? | A signal that the instance has elevated interruption risk. |
| How do you design resilient Spot capacity? | Use multiple instance types, AZs and capacity pools, plus checkpointing and replacement. |

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
| Dedicated Instance vs Dedicated Host? | Dedicated Instance provides dedicated hardware for instances; Dedicated Host allocates the entire physical server. |

---

# 9. Security Groups and Key Pairs

## Security Groups

A Security Group is a **stateful virtual firewall** associated with an ENI.

| Characteristic | Security Group |
|---|---|
| Scope | ENI/resource level |
| State | Stateful |
| Rules | Allow only |
| Explicit deny | Not supported |
| Return traffic | Automatically allowed for permitted flows |
| Evaluation | All applicable rules are evaluated |

### Recommended application pattern

```text
Internet
  ↓ HTTPS 443
Application Load Balancer
  ↓ Application port
EC2 Target Security Group
```

The EC2 security group should normally allow application traffic from the ALB security group rather than from `0.0.0.0/0`.

## Key pairs

Key pairs are commonly used for SSH access to Linux EC2 instances.

```bash
chmod 400 devops.pem
ssh -i devops.pem ubuntu@<public-ip>
```

Best practices:

| Practice | Reason |
|---|---|
| Never commit private keys | Prevent credential exposure |
| Restrict SSH source IPs | Reduce attack surface |
| Prefer SSM Session Manager | Avoid direct SSH and key distribution |
| Use least privilege | Limit impact of compromise |

### Interview Checkpoint — Security

| Question | Answer |
|---|---|
| What is a Security Group? | A stateful virtual firewall attached to an ENI. |
| Can a Security Group contain deny rules? | No, it supports allow rules only. |
| What does stateful mean? | Return traffic for an allowed connection is automatically permitted. |
| How should EC2 be protected behind an ALB? | Allow the backend SG to receive application traffic from the ALB SG. |
| What is a key pair? | A public/private key pair used for supported EC2 access methods such as SSH. |
| Why prefer SSM over SSH? | It reduces exposed ports and avoids managing private SSH keys on administrators’ machines. |

---

# 10. Monitoring and Troubleshooting

## EC2 status checks

| Check | Detects |
|---|---|
| System status check | Underlying AWS infrastructure or host problems |
| Instance status check | Guest OS, boot or networking problems |
| EBS status information | Storage I/O-related issues where reported |

## Useful CloudWatch metrics

| Metric | Meaning |
|---|---|
| `CPUUtilization` | CPU usage |
| `NetworkIn` | Incoming network bytes |
| `NetworkOut` | Outgoing network bytes |
| `DiskReadOps` | Read operations |
| `DiskWriteOps` | Write operations |
| `DiskReadBytes` | Read bytes |
| `DiskWriteBytes` | Write bytes |
| `StatusCheckFailed` | Failed status checks |

> Standard EC2 metrics do not automatically provide guest OS memory utilization. Install an agent for memory metrics.

## Troubleshooting: application unreachable

| Check order | Command or area |
|---:|---|
| 1 | Instance state |
| 2 | Security group |
| 3 | NACL |
| 4 | Route table and gateway |
| 5 | Application listener |
| 6 | OS firewall |
| 7 | Load balancer target health |

```bash
ss -lntp
curl http://localhost:<port>
sudo systemctl status <service>
sudo journalctl -u <service> -n 100 --no-pager
```

## Troubleshooting: SSH timeout

| Check | Why |
|---|---|
| Instance running | Server may be stopped |
| Public IP or bastion path | Correct destination |
| SG TCP 22 | Inbound SSH access |
| Route table | Return and forward path |
| NACL | Stateless traffic rules |
| OS firewall | Local filtering |
| Key and username | Authentication after connectivity works |
| SSM | Alternative access method |

> A timeout usually indicates a connectivity/path problem, not necessarily an authentication problem.

## Troubleshooting: EC2 has no internet access

| Requirement | Check |
|---|---|
| Public subnet | Route to Internet Gateway |
| Private subnet | Route to NAT Gateway or VPC endpoint |
| Security group | Egress rules |
| NACL | Both directions because it is stateless |
| DNS | Resolver and hostname resolution |
| Instance route | `ip route` |

## Troubleshooting: AWS API access fails

```text
IAM Role
  ↓
Instance Profile
  ↓
IMDS / Credentials
  ↓
IAM Permissions
  ↓
Network path to AWS API
```

Do not solve this by placing permanent access keys on the instance.

### Interview Checkpoint — Monitoring and Troubleshooting

| Scenario question | Expected answer structure |
|---|---|
| EC2 is running but app is unreachable | Check SG → NACL → route → listener → OS firewall → target health. |
| SSH times out | Check state, route, SG TCP 22, NACL, public/bastion path and OS firewall. |
| Private EC2 cannot download packages | Check NAT route, NAT subnet, IGW, SG/NACL and DNS. |
| AWS CLI works locally but not on EC2 | Check instance profile, IAM permissions, IMDS and network access. |
| EC2 stopped unexpectedly | Check CloudTrail, state-change events, Spot events and ASG activity. |
| CPU is normal but application is slow | Check memory, disk I/O, network, application logs and dependency latency. |

---

# 11. Launch Templates and Auto Scaling

## Launch Template

A Launch Template defines how EC2 instances should be launched.

| Setting | Example |
|---|---|
| AMI | Golden application AMI |
| Instance type | `t3.small` |
| Security groups | Application SG |
| IAM profile | EC2 SSM role |
| User data | Bootstrap script |
| EBS mappings | Root and data volumes |
| Network settings | Subnet/ENI configuration |
| Monitoring | Detailed monitoring |
| Metadata options | IMDSv2 requirement |

## Auto Scaling Group (ASG)

An ASG maintains a desired number of EC2 instances and can replace unhealthy instances or scale capacity.

Example:

| Setting | Value |
|---|---:|
| Minimum capacity | 2 |
| Desired capacity | 3 |
| Maximum capacity | 6 |

```text
Application Load Balancer
          ↓
Auto Scaling Group
     ┌────┼────┐
     ↓    ↓    ↓
   EC2-1 EC2-2 EC2-3
```

### Replacement flow

```text
3 instances
  ↓
One instance fails
  ↓
2 healthy instances
  ↓
ASG launches replacement
  ↓
3 healthy instances
```

### Interview Checkpoint — Launch Templates and ASG

| Question | Answer |
|---|---|
| What is a Launch Template? | A reusable definition for launching EC2 instances. |
| Why use Launch Templates? | They standardize configuration and reduce manual errors. |
| What is an ASG? | A construct that maintains and scales a fleet of EC2 instances. |
| What is desired capacity? | The number of instances the ASG attempts to maintain. |
| What happens when an ASG instance becomes unhealthy? | The ASG can terminate and replace it according to health-check configuration. |
| Why combine ASG with ALB? | ALB distributes traffic while ASG scales and replaces instances. |
| Why use multiple AZs in an ASG? | To reduce dependence on a single AZ and improve availability. |

---

# 12. High Availability Architecture

## Recommended production pattern

```text
                         Internet
                            ↓
                           ALB
                       /         \
                    AZ-A         AZ-B
                      ↓            ↓
                    EC2-1        EC2-2
                      \            /
                       Auto Scaling
                            ↓
                     Application tier
```

| Component | Role |
|---|---|
| ALB | Distributes traffic and performs health checks |
| Target Group | Registers and checks EC2 targets |
| Launch Template | Defines consistent instance configuration |
| ASG | Maintains and scales capacity |
| Multiple AZs | Reduces single-AZ dependency |
| Security Groups | Restricts traffic paths |
| CloudWatch | Metrics and alarms |
| IAM Roles | Least-privilege AWS API access |

### Interview Checkpoint — Architecture

| Scenario question | Strong answer |
|---|---|
| Design a highly available EC2 web tier | ALB across AZs, ASG across multiple AZs, Launch Template, target health checks and restricted SGs. |
| How do you avoid configuration drift? | Build and test a Golden AMI and use a Launch Template. |
| How do you handle instance failure? | ASG health checks replace failed instances; ALB routes traffic only to healthy targets. |
| How do you secure backend EC2? | Private subnets where appropriate, no direct public access, ALB-to-EC2 SG references and SSM access. |
| How do you reduce cost for batch jobs? | Use Spot capacity with checkpointing and multiple capacity pools. |
| How do you improve operational access? | Use Systems Manager Session Manager instead of exposing SSH broadly. |

---

# 13. AWS CLI and Linux Cheat Sheet

## EC2 lifecycle commands

```bash
aws ec2 describe-instances

aws ec2 start-instances \
  --instance-ids i-xxxxxxxxxxxxxxxxx

aws ec2 stop-instances \
  --instance-ids i-xxxxxxxxxxxxxxxxx

aws ec2 reboot-instances \
  --instance-ids i-xxxxxxxxxxxxxxxxx

aws ec2 terminate-instances \
  --instance-ids i-xxxxxxxxxxxxxxxxx
```

## Discovery commands

```bash
aws ec2 describe-instance-types
aws ec2 describe-images --owners self
aws ec2 describe-security-groups
aws ec2 describe-network-interfaces
aws ec2 describe-volumes
aws ec2 describe-snapshots --owner-ids self
aws ec2 describe-instance-status
aws ec2 describe-tags
```

## Linux diagnostics

| Purpose | Command |
|---|---|
| Host/kernel | `uname -a` |
| Hostname | `hostname` |
| IP addresses | `ip addr` |
| Routes | `ip route` |
| Listening ports | `ss -lntp` |
| Disk devices | `lsblk` |
| Disk usage | `df -h` |
| Memory | `free -h` |
| Load | `uptime` |
| Processes | `top` |
| Service status | `systemctl status <service>` |
| Service logs | `journalctl -u <service>` |
| Local application test | `curl http://localhost:<port>` |

---

# 14. Final Interview Revision

## Frequently Asked Questions

| Question | Short answer |
|---|---|
| EC2 vs Lambda? | EC2 provides server control; Lambda runs event-driven code without server management. |
| AMI vs snapshot? | AMI launches EC2; an EBS snapshot backs up a volume. |
| Private IP vs Elastic IP? | Private IP is internal; Elastic IP is a static public IPv4. |
| SG vs NACL? | SG is stateful/ENI-level; NACL is stateless/subnet-level. |
| Stop vs terminate? | Stop allows restart; terminate permanently removes the instance. |
| EBS vs Instance Store? | EBS is persistent; Instance Store is ephemeral. |
| On-Demand vs Spot? | On-Demand is predictable capacity; Spot is discounted and interruptible. |
| Launch Template vs AMI? | AMI is the image; Launch Template defines the complete launch configuration. |
| ASG vs Launch Template? | Launch Template defines instances; ASG maintains and scales the fleet. |
| Why multiple AZs? | To improve availability and reduce failure-domain dependency. |
| Why use IAM role? | To avoid long-lived credentials on the instance. |
| Why use IMDSv2? | It adds token-based protection for metadata access. |
| What is gp3? | General-purpose SSD EBS with separately provisioned performance. |
| What is user data? | Launch-time bootstrap configuration. |
| What is a Golden AMI? | A tested, preconfigured image for consistent deployments. |

## Scenario-Based Questions

| Scenario | What the interviewer expects |
|---|---|
| EC2 is running but port 8080 is unreachable | Network path, SG, NACL, route, listener and OS firewall analysis |
| Private EC2 cannot run `apt update` | NAT Gateway route, gateway path, DNS and egress checks |
| EC2 cannot access S3 | IAM role, instance profile, IMDS, endpoint/NAT and bucket permissions |
| Public IP changed after restart | Explain auto-assigned public IPv4 behavior and Elastic IP |
| ASG keeps replacing instances | Check health checks, bootstrap failure, AMI, user data and target health |
| Spot instance is interrupted | Checkpoint, drain, replace and diversify capacity |
| Disk is full | `df -h`, `du`, log cleanup, volume expansion and filesystem resize |
| Application is slow but CPU is low | Memory, disk I/O, network, dependencies and application logs |
| Need secure admin access without SSH | SSM Session Manager with IAM permissions and agent connectivity |
| Need identical servers across environments | Golden AMI plus Launch Template and automated provisioning |

## Interview Answer Framework

Use this structure for scenario questions:

| Step | Explain |
|---:|---|
| 1 | State the likely failure domain |
| 2 | Explain the checks in order |
| 3 | Give one or two commands |
| 4 | Explain the fix |
| 5 | Mention the preventive design |

Example:

> **Question:** EC2 is running but the application is unreachable.  
> **Answer:** I first verify instance state and target health. Then I check the security group, NACL, route table, subnet gateway path, application listener and OS firewall. I use `ss -lntp` and `curl localhost:<port>` to separate an application issue from a network issue. Finally, I review logs and add monitoring or health checks to prevent recurrence.

---

# 15. References

## Official AWS documentation

| Topic | Reference |
|---|---|
| EC2 User Guide | [AWS EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/) |
| Instance Types | [EC2 Instance Types](https://docs.aws.amazon.com/ec2/latest/instancetypes/ec2-instance-types.html) |
| AMIs | [Amazon Machine Images](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) |
| User Data | [EC2 User Data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) |
| Metadata | [EC2 Instance Metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html) |
| IMDS retrieval | [Access Instance Metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html) |
| IAM roles | [IAM Roles for EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) |
| Lifecycle | [EC2 Instance Lifecycle](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html) |
| Stop/Start | [Stop and Start EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/how-ec2-instance-stop-start-works.html) |
| EBS | [Amazon EBS User Guide](https://docs.aws.amazon.com/ebs/latest/userguide/what-is-ebs.html) |
| gp3 | [EBS General Purpose SSD](https://docs.aws.amazon.com/ebs/latest/userguide/general-purpose.html) |
| Spot | [EC2 Spot Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances.html) |
| Placement Groups | [EC2 Placement Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html) |
| Security Groups | [EC2 Security Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html) |
| Launch Templates | [EC2 Launch Templates](https://docs.aws.amazon.com/autoscaling/ec2/userguide/launch-templates.html) |
| Auto Scaling | [EC2 Auto Scaling](https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html) |
| Pricing | [Amazon EC2 Pricing](https://aws.amazon.com/ec2/pricing/) |

## Interview research

The interview checkpoints were expanded around recurring themes found in:

- [GeeksforGeeks — AWS Interview Questions](https://www.geeksforgeeks.org/cloud-computing/aws-interview-questions/)
- [GeeksforGeeks — AWS Solutions Architect Interview Questions](https://www.geeksforgeeks.org/blogs/aws-solution-architect-associate-job-interview-questions-and-answers/)
- [EC2 Interview Questions — InterviewQuestions.guru](https://interviewquestions.guru/aws-ec2-interview-questions/)
- [AWS EC2 Interview Questions — MyInternships](https://myinternships.in/aws-interview-questions/ec2)

> **Note:** AWS limits, supported instance families, pricing, defaults and service behavior can change. Verify production decisions against current AWS documentation for the exact Region, instance type and configuration.
