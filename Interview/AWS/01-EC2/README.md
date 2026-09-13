# EC2

## Interview Questions & Answers

### Q1. What is Amazon EC2?

**Answer:** Amazon EC2 (Elastic Compute Cloud) is an AWS service that provides **resizable virtual servers**, called instances. An EC2 instance provides compute capacity such as CPU and memory, along with networking and storage.

When launching an instance, you typically choose an **AMI, instance type, VPC/subnet, security groups, IAM instance profile, and storage configuration**. AWS manages the underlying physical infrastructure, while you are responsible for the guest operating system, applications, and configuration.

---

### Q2. What is an EC2 instance?

**Answer:** An EC2 instance is a **running virtual server** provisioned on AWS infrastructure. It is launched from an AMI and an instance type determines the available compute characteristics.

For example:

| Component | Determines |
|---|---|
| AMI | Operating system and initial software |
| Instance type | vCPU, memory, network capability, accelerators |
| EBS / Instance Store | Storage |
| ENI | Network connectivity |
| Security Group | Network access control |
| IAM Role | AWS API permissions |

---

### Q3. What is an AMI?

**Answer:** An AMI (Amazon Machine Image) is a **template used to launch EC2 instances**. It contains the operating system and software configuration needed to create an instance and includes block-device mappings.

An AMI can be created from an existing EC2 environment, copied to another Region, or shared with other AWS accounts depending on the AMI and ownership model.

---

### Q4. What is an instance type?

**Answer:** An EC2 instance type defines the **hardware characteristics presented to the virtual machine**, including vCPUs, memory, network performance, and, for some families, local storage or accelerators.

Instance types are organized into families based on workload requirements:

| Family | Typical workload |
|---|---|
| General Purpose | Web servers, application servers |
| Compute Optimized | CPU-intensive workloads |
| Memory Optimized | In-memory databases, large caches |
| Storage Optimized | High local-storage I/O |
| Accelerated Computing | GPU/accelerator workloads |

---

### Q5. What is user data?

**Answer:** EC2 user data is information, commonly a shell script or cloud-init configuration, supplied when an instance is launched. It is primarily used for **bootstrapping and initialization**.

Typical uses include installing packages, creating configuration files, starting services, registering the instance with a load balancer or monitoring system, and performing first-boot configuration.

---

### Q6. What is instance metadata?

**Answer:** EC2 instance metadata is information about the running instance exposed through the **Instance Metadata Service (IMDS)**. Applications can use it to obtain information such as the instance ID, local IP address, availability zone, and other instance attributes.

If an IAM role is attached to the instance, temporary credentials can also be obtained through IMDS. **IMDSv2** is recommended because it uses a session-oriented token mechanism that provides stronger protection against certain credential-access attacks.

---

### Q7. Explain On-Demand, Reserved Instances, Savings Plans, Spot Instances, and Dedicated Hosts/Instances.

**Answer:** These are different EC2 purchasing and tenancy options:

| Option | Main idea | Best suited for |
|---|---|---|
| On-Demand | Pay for capacity without a long-term commitment | Unpredictable or short-term workloads |
| Reserved Instances | Discount for a commitment to an eligible configuration | Stable, predictable usage |
| Savings Plans | Discount for committing to a specified compute spend per hour | Steady usage where flexibility is important |
| Spot Instances | Uses spare EC2 capacity at a large discount; can be interrupted | Fault-tolerant and interruptible workloads |
| Dedicated Instances | Instances run on dedicated hardware for the account | Certain isolation/compliance requirements |
| Dedicated Host | Entire physical server dedicated to the account | Licensing, compliance, and host-level visibility requirements |

The correct choice depends on **workload predictability, interruption tolerance, commitment, licensing, and isolation requirements**.

---

### Q8. When should you use Spot Instances?

**Answer:** Spot Instances should be used when the workload can tolerate **interruption and capacity replacement**.

Good examples include:

- Batch processing
- CI/CD build workers
- Distributed data processing
- Stateless web/application capacity
- Rendering workloads
- Large parallel jobs

A resilient architecture normally uses multiple Spot capacity pools, an Auto Scaling Group, or another scheduler rather than depending on one Spot instance.

---

### Q9. What happens when a Spot Instance is interrupted?

**Answer:** AWS can reclaim a Spot Instance when the underlying capacity is required. The instance receives a **Spot interruption notice**, giving the application an opportunity to react.

A resilient application should:

1. Stop accepting new work.
2. Save or checkpoint important work.
3. Drain active work where possible.
4. Terminate or allow the instance to be terminated.
5. Replace the capacity through an ASG or scheduler.

Spot should therefore be treated as **interruptible capacity**, not guaranteed capacity.

---

### Q10. When are Reserved Instances useful?

**Answer:** Reserved Instances are useful when you have **predictable and steady EC2 usage** and expect to use an eligible configuration for the commitment period.

For example, if a production application continuously requires a known EC2 configuration, an RI can reduce the effective cost compared with running the same usage entirely On-Demand.

RIs are primarily a **billing discount mechanism**, not a special type of EC2 instance.

---

### Q11. How do Savings Plans differ from Reserved Instances?

**Answer:** Both provide discounts in exchange for commitment, but their commitment models differ.

| Feature | Reserved Instances | Savings Plans |
|---|---|---|
| Commitment | Eligible instance configuration/usage | Committed hourly compute spend |
| Flexibility | More configuration-specific | Generally more flexible |
| Main benefit | Discount for predictable eligible usage | Discount while retaining more compute flexibility |
| Best consideration | Stable configuration | Stable spend with changing eligible compute usage |

The exact flexibility depends on the type of RI or Savings Plan selected.

---

### Q12. Explain general purpose, compute optimized, memory optimized, storage optimized, and accelerated computing families.

**Answer:** EC2 instance families are designed around different resource requirements:

| Family | Optimization | Examples of workloads |
|---|---|---|
| General Purpose | Balanced CPU and memory | Web/application servers |
| Compute Optimized | High CPU performance | Batch processing, compute-heavy services |
| Memory Optimized | Large memory capacity | In-memory databases, large caches |
| Storage Optimized | High local storage performance | Data processing, high-I/O workloads |
| Accelerated Computing | GPUs or specialized accelerators | ML, graphics, scientific workloads |

The family should be selected based on the **actual bottleneck** rather than simply choosing the largest instance.

---

### Q13. Which family is suitable for CPU-heavy workloads?

**Answer:** A **compute-optimized family**, such as the C family, is generally appropriate for CPU-bound workloads.

Examples include:

- High-performance web servers
- Batch processing
- Video encoding
- Scientific calculations
- CPU-intensive application services

The key requirement is a workload where **CPU capacity is the primary bottleneck**, rather than memory or local storage.

---

### Q14. Which family is suitable for in-memory databases?

**Answer:** A **memory-optimized family**, such as the R family, is suitable when the workload requires a large amount of RAM relative to CPU.

Typical examples include:

- In-memory databases
- Large caching layers
- Real-time analytics
- Memory-intensive applications

The objective is to provide enough memory to keep the working dataset in RAM and avoid excessive disk access.

---

### Q15. Which family is suitable for high local NVMe storage workloads?

**Answer:** A **storage-optimized family**, such as supported I-family instances, is appropriate when the workload requires high-performance local storage and very high I/O.

Typical workloads include:

- Distributed data processing
- Local caching
- Temporary high-performance datasets
- Applications requiring high local NVMe throughput

The important point is that **Instance Store is local and ephemeral**, so important persistent data should not rely on it.

---

### Q16. When would you use GPU-based instances?

**Answer:** GPU or accelerated-computing instances are appropriate when the workload can use parallel processing provided by GPUs or other accelerators.

Typical examples include:

- Machine learning training and inference
- Deep learning
- 3D rendering
- Graphics processing
- Scientific and engineering simulations

A GPU instance should be selected only when the application is designed to take advantage of the accelerator; otherwise, the additional cost may not provide a benefit.

---

### Q17. What are private and public IPv4 addresses?

**Answer:** A **private IPv4 address** is used for communication inside a VPC and connected private networks. A **public IPv4 address** provides Internet-routable addressing when the subnet routing and security configuration permit it.

| Property | Private IPv4 | Public IPv4 |
|---|---|---|
| VPC communication | Yes | Yes |
| Direct Internet routing | No | Yes, through an Internet Gateway |
| Typical use | Internal communication | Internet-facing connectivity |
| Stability | Associated with the ENI | Can change after stop/start unless using an Elastic IP |

A public IP alone does not make an application accessible; **routing and security controls must also allow the traffic**.

---

### Q18. What is an Elastic IP?

**Answer:** An Elastic IP (EIP) is a **static public IPv4 address allocated to an AWS account** that can be associated with supported AWS resources.

It is useful when an application requires a stable public IPv4 address. However, for most application endpoints, using a **DNS name** is preferable because DNS provides better abstraction and makes infrastructure changes easier.

---

### Q19. What is an ENI?

**Answer:** An Elastic Network Interface (ENI) is a **virtual network interface in a VPC**. It provides network connectivity to supported resources and contains attributes such as private IPv4 addresses, security groups, MAC address, and subnet association.

An EC2 instance can have one or more ENIs depending on its instance type and configuration.

---

### Q20. What is EBS?

**Answer:** Amazon Elastic Block Store (EBS) provides **persistent block storage for EC2 instances**. An EBS volume behaves like a block device that can be formatted with a filesystem or used directly by applications such as databases.

EBS supports different volume types, encryption, resizing for supported configurations, and point-in-time snapshots.

Typical uses include:

- Operating-system/root volumes
- Application filesystems
- Database storage
- Persistent application data

---

### Q21. What is instance store?

**Answer:** Instance Store is **local ephemeral block storage physically attached to the host running an EC2 instance**. It can provide very high I/O performance and low latency.

However, Instance Store is not persistent storage like EBS. Data can be lost when the instance is stopped, terminated, or when the underlying host fails, depending on the event.

It is therefore appropriate for **temporary data**, such as caches, scratch space, buffers, and intermediate processing data that can be recreated.

---

### Q22. What is the difference between EBS and instance store?

**Answer:** The key difference is **persistence versus local ephemeral performance**.

| Feature | EBS | Instance Store |
|---|---|---|
| Storage type | Network-attached block storage | Local block storage |
| Persistence | Persistent | Ephemeral |
| Survives normal EC2 stop | Yes | No |
| Snapshot support | Yes | No EBS-style snapshots |
| Performance | High | Very high local I/O |
| Typical use | OS, databases, persistent data | Cache, scratch, temporary data |

Also, Instance Store is available only on **supported EC2 instance types**. Most modern EC2 instances use EBS for the root volume.

---

### Q23. What are stop, start, reboot, and terminate?

**Answer:** These operations have different effects:

| Operation | Effect |
|---|---|
| Stop | Shuts down an EBS-backed instance while retaining its EBS volumes according to their configuration |
| Start | Starts a previously stopped EBS-backed instance |
| Reboot | Restarts the operating system on the same EC2 instance |
| Terminate | Permanently removes the EC2 instance |

When an instance is terminated, an attached EBS volume is deleted only if its **DeleteOnTermination** attribute is set to `true`.

Instance Store data should not be treated as persistent through stop/terminate operations.

---

### Q24. What happens to EBS volumes when an instance is terminated?

**Answer:** It depends on the volume's **DeleteOnTermination** setting.

| DeleteOnTermination | Result after EC2 termination |
|---|---|
| `true` | EBS volume is deleted |
| `false` | EBS volume remains and can potentially be attached elsewhere |

The root EBS volume is commonly configured with `DeleteOnTermination = true`, while important data volumes may be configured to remain.

Before terminating an instance containing important data, always verify the volume's DeleteOnTermination setting.

---

### Q25. How does an IAM role attach to EC2?

**Answer:** An IAM role is not attached directly to an EC2 instance as a standalone object. It is associated with the instance through an **IAM instance profile**.

The flow is:

```text
IAM Role
   ↓
Instance Profile
   ↓
EC2 Instance
   ↓
Instance Metadata Service (IMDS)
   ↓
Temporary AWS Credentials
   ↓
Application calls AWS APIs
```

The role's permissions determine what AWS APIs the application can call. This avoids storing long-term access keys on the EC2 instance and is the recommended approach for workload authentication.

---

### Q26. What is an EC2 placement group?

**Answer:** An EC2 placement group is a logical placement strategy that controls how EC2 instances are placed on the underlying AWS infrastructure.

The main placement strategies are:

| Strategy | Purpose |
|---|---|
| Cluster | Low-latency, high-throughput networking between instances |
| Spread | Keep instances on separate underlying hardware for failure isolation |
| Partition | Separate groups of instances into isolated partitions for distributed workloads |

Placement groups are useful when **network performance or failure-domain placement** is an important architectural requirement.

---

### Q27. Explain cluster, spread, and partition placement groups.

**Answer:** The three strategies solve different problems:

**Cluster placement:** Places instances close together within a single Availability Zone to achieve **low-latency and high-throughput networking**.

**Spread placement:** Places instances on distinct underlying hardware to reduce the chance that a single hardware failure affects multiple instances. It is useful for workloads where **individual-instance isolation** is important.

**Partition placement:** Divides instances into logical partitions. Instances in different partitions are placed on separate hardware groups, making it useful for distributed systems such as **Hadoop, Kafka, and Cassandra**, where you want to reduce correlated hardware failures.

---

### Q28. How do security groups protect EC2?

**Answer:** A security group is a **stateful virtual firewall associated with an EC2 instance's network interface**. It controls which inbound and outbound traffic is allowed.

Important characteristics:

| Characteristic | Security Group |
|---|---|
| Scope | ENI / resource |
| State | Stateful |
| Inbound rules | Allow rules |
| Outbound rules | Allow rules |
| Explicit deny rules | Not supported |
| Return traffic | Automatically allowed for an established permitted connection |

For example, an application server might allow inbound TCP `443` from an ALB security group while denying direct application access from the Internet.

Security groups therefore provide **network-level access control**, while IAM controls AWS API permissions.

---

---

### Q29. What is the normal Spot Instance interruption notice period?

**Answer:** When Amazon EC2 interrupts a Spot Instance because it needs to reclaim the capacity, the normal interruption notice is **two minutes**.

This gives the application a short window to:
- Stop accepting new work
- Save or checkpoint state
- Drain work where possible
- Start replacement capacity

The two-minute warning does **not** apply in the same way to hibernation because the hibernation process begins immediately. AWS also provides a **rebalance recommendation** that can arrive before the interruption notice when a Spot Instance is at elevated risk.

---

### Q30. Does a Spot Instance have a fixed lifetime, such as 1 hour or 24 hours?

**Answer:** No. A Spot Instance does **not** have a fixed lifetime.

It can continue running while capacity is available and until one of these occurs:
- You stop or terminate it
- Amazon EC2 interrupts it because AWS needs the capacity back
- Another configured lifecycle condition causes it to stop or terminate

Therefore, Spot should be treated as **interruptible capacity**, not capacity with a guaranteed runtime.

---

### Q31. How many private IPv4 addresses can be assigned to an EC2 instance?

**Answer:** There is **no single maximum number that applies to every EC2 instance**.

The limit depends on the **instance type**, specifically:
- Maximum number of network interfaces (ENIs)
- Maximum number of private IPv4 addresses supported per ENI

Therefore, when an interviewer asks for the maximum number of private IPs, the correct answer is: **it is instance-type dependent**.

For example, the total possible private IPv4 addresses are constrained by the instance's ENI and per-ENI limits rather than by one universal EC2-wide number.

---

### Q32. Does a private IPv4 address change when an EC2 instance is stopped and started?

**Answer:** No. A private IPv4 address remains associated with the network interface when an EC2 instance is stopped and started.

| Event | Private IPv4 address |
|---|---|
| Stop | Retained |
| Start | Retained |
| Hibernate | Retained |
| Terminate | Released |

The automatically assigned public IPv4 address behaves differently: it is normally released when the instance is stopped and a new public IPv4 address can be assigned when the instance starts again. Use an Elastic IP when a persistent public IPv4 address is required.

---

### Q33. What is the maximum size of an Amazon EBS volume?

**Answer:** The maximum supported size of an Amazon EBS volume is **64 TiB**.

The actual usable capacity can also depend on the operating system and partitioning scheme.

For boot volumes:
- **MBR** limits the boot volume to **2 TiB**.
- **GPT** can support boot volumes up to **64 TiB** when the operating system and boot mode support it.

---

### Q34. What is the size range and baseline performance of a gp3 EBS volume?

**Answer:** A gp3 volume supports **1 GiB to 64 TiB** of storage.

Its baseline performance includes:
- **3,000 IOPS**
- **125 MiB/s throughput**

These are independent of volume size within the supported configuration. Additional performance can be provisioned up to:
- **80,000 IOPS**
- **2,000 MiB/s throughput**

The actual maximum performance also depends on the attached EC2 instance's EBS bandwidth capabilities.

---

### Q35. What happens to Instance Store data when an EC2 instance is stopped or terminated?

**Answer:** Instance Store is **ephemeral** storage. Data stored on Instance Store volumes is lost when the instance is stopped or terminated.

It should therefore be used only for data that can be recreated, such as:
- Caches
- Temporary files
- Scratch space
- Intermediate processing data

Persistent data should be stored on services such as EBS, S3, or an appropriate database.

---

### Q36. Can every EC2 instance be stopped and started?

**Answer:** No. The normal stop/start operation applies to instances with an **Amazon EBS volume as the root volume**.

An instance-store-root instance cannot be stopped and started in the same way because its root storage is ephemeral.

For EBS-backed instances, stopping preserves the attached EBS volumes while the instance is stopped, although Instance Store data and the automatically assigned public IPv4 address are lost.

---

### Q37. What happens to an EC2 instance's EBS volumes when the instance is terminated?

**Answer:** The result depends on the volume's **DeleteOnTermination** attribute.

| DeleteOnTermination | Result after termination |
|---|---|
| `true` | EBS volume is deleted |
| `false` | EBS volume is preserved |

The setting should always be checked before terminating an instance that contains important data.

---

### Q38. What happens to an EC2 instance's public and private IP addresses during stop/start?

**Answer:** They behave differently.

| Address | Stop | Start | Terminate |
|---|---|---|---|
| Private IPv4 | Retained | Same private IPv4 | Released |
| Automatically assigned public IPv4 | Released | Normally a new public IPv4 is assigned | Released |
| Elastic IP | Remains associated | Remains associated | Depends on resource association/release |

This is why applications that require a stable public IPv4 address should use an **Elastic IP**, although DNS names are generally preferable for application endpoints.

---

### Q39. What is the difference between the two-minute Spot interruption notice and a Spot rebalance recommendation?

**Answer:** They are different signals with different purposes.

| Signal | Meaning | Purpose |
|---|---|---|
| Rebalance recommendation | Instance is at elevated risk of interruption | Gives an opportunity to proactively replace or drain the instance |
| Spot interruption notice | AWS has scheduled an interruption | Normally provides two minutes to react |

A rebalance recommendation can arrive before the two-minute interruption notice, but AWS does not guarantee that it will always arrive earlier.
