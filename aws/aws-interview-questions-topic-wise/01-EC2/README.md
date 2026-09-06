# EC2

## Interview Questions & Answers

### Q1. What is Amazon EC2?

**Answer:** Amazon EC2 (Elastic Compute Cloud) provides resizable virtual servers in AWS. You select an AMI, instance type, networking, storage, and security controls, then run the operating system and application. AWS manages the underlying physical infrastructure; you manage the guest OS and workload.

---

### Q2. What is an EC2 instance?

**Answer:** An EC2 instance is a virtual compute server launched from an AMI using a selected instance type. It provides compute, networking, and storage resources according to the launch configuration.

---

### Q3. What is an AMI?

**Answer:** An AMI is a reusable template used to launch EC2 instances. It defines the operating-system image and block-device mappings and can be copied or shared according to AWS rules.

---

### Q4. What is an instance type?

**Answer:** An EC2 instance type defines the compute characteristics exposed to the instance, including vCPUs, memory, network performance, and, for some families, local storage or accelerators. The instance family and size should match the workload.

---

### Q5. What is user data?

**Answer:** EC2 user data is bootstrap data supplied at launch. It is commonly used to install packages, configure services, create files, or run initialization scripts automatically.

---

### Q6. What is instance metadata?

**Answer:** The EC2 Instance Metadata Service exposes instance information and, when an IAM role is attached, temporary credentials. IMDSv2 uses a session token and is recommended for stronger protection against credential abuse.

---

### Q7. Explain On-Demand, Reserved Instances, Savings Plans, Spot Instances, and Dedicated Hosts/Instances.

**Answer:** Spot Instances use spare EC2 capacity at a discount. They can be interrupted, so they are best for fault-tolerant, restartable, stateless, or distributed workloads such as batch processing and CI workers.

---

### Q8. When should you use Spot Instances?

**Answer:** Spot Instances use spare EC2 capacity at a discount. They can be interrupted, so they are best for fault-tolerant, restartable, stateless, or distributed workloads such as batch processing and CI workers.

---

### Q9. What happens when a Spot Instance is interrupted?

**Answer:** Spot Instances use spare EC2 capacity at a discount. They can be interrupted, so they are best for fault-tolerant, restartable, stateless, or distributed workloads such as batch processing and CI workers.

---

### Q10. When are Reserved Instances useful?

**Answer:** Reserved Instances provide a billing discount for eligible EC2 usage in exchange for a commitment. They are useful for steady, predictable workloads where the relevant configuration is expected to remain in use.

---

### Q11. How do Savings Plans differ from Reserved Instances?

**Answer:** Reserved Instances provide a billing discount for eligible EC2 usage in exchange for a commitment. They are useful for steady, predictable workloads where the relevant configuration is expected to remain in use.

---

### Q12. Explain general purpose, compute optimized, memory optimized, storage optimized, and accelerated computing families.

**Answer:** General-purpose instances provide a balanced ratio of compute, memory, and networking. They are a good default for web servers, application servers, and many development workloads.

---

### Q13. Which family is suitable for CPU-heavy workloads?

**Answer:** An AMI is a reusable template used to launch EC2 instances. It defines the operating-system image and block-device mappings and can be copied or shared according to AWS rules.

---

### Q14. Which family is suitable for in-memory databases?

**Answer:** An AMI is a reusable template used to launch EC2 instances. It defines the operating-system image and block-device mappings and can be copied or shared according to AWS rules.

---

### Q15. Which family is suitable for high local NVMe storage workloads?

**Answer:** An AMI is a reusable template used to launch EC2 instances. It defines the operating-system image and block-device mappings and can be copied or shared according to AWS rules.

---

### Q16. When would you use GPU-based instances?

**Answer:** Use GPU or other accelerated-computing instances when the workload can exploit parallel acceleration, such as machine-learning training/inference, graphics rendering, video processing, or scientific workloads.

---

### Q17. What are private and public IPv4 addresses?

**Answer:** A private IPv4 address is used for communication inside the VPC and connected private networks. A public IPv4 address provides Internet-reachable addressing when the subnet has an Internet Gateway route and the resource is configured for public access.

---

### Q18. What is an Elastic IP?

**Answer:** An Elastic IP is a static public IPv4 address that can be associated with supported AWS resources. It is useful when a stable public address is required, although DNS names are usually preferable for application endpoints.

---

### Q19. What is an ENI?

**Answer:** An Elastic Network Interface is a virtual network interface containing attributes such as private IP addresses, security groups, and a MAC address. It provides network connectivity for supported VPC resources.

---

### Q20. What is EBS?

**Answer:** Amazon EBS provides persistent block storage for EC2. EBS volumes can be used for boot disks, filesystems, and databases, and snapshots can provide backup and recovery.

---

### Q21. What is instance store?

**Answer:** Instance store is local ephemeral storage physically attached to the host. It can provide very high I/O performance, but data is not durable like EBS and can be lost when the instance or underlying host is stopped or terminated.

---

### Q22. What is the difference between EBS and instance store?

**Answer:** Instance store is local ephemeral storage physically attached to the host. It can provide very high I/O performance, but data is not durable like EBS and can be lost when the instance or underlying host is stopped or terminated.

---

### Q23. What are stop, start, reboot, and terminate?

**Answer:** Stop shuts down an EBS-backed instance while preserving its EBS volumes; start boots it again. Reboot restarts the operating system. Terminate permanently removes the EC2 instance; EBS volumes are deleted or retained according to DeleteOnTermination settings.

---

### Q24. What happens to EBS volumes when an instance is terminated?

**Answer:** Amazon EBS provides persistent block storage for EC2. EBS volumes can be used for boot disks, filesystems, and databases, and snapshots can provide backup and recovery.

---

### Q25. How does an IAM role attach to EC2?

**Answer:** AWS IAM controls authentication and authorization to AWS resources. It includes identities such as users, groups, and roles and policies that determine allowed actions.

---

### Q26. What is an EC2 placement group?

**Answer:** A placement group controls how EC2 instances are placed on underlying hardware. Cluster is optimized for low-latency/high-throughput networking, spread emphasizes hardware isolation, and partition isolates logical groups of instances.

---

### Q27. Explain cluster, spread, and partition placement groups.

**Answer:** A placement group controls how EC2 instances are placed on underlying hardware. Cluster is optimized for low-latency/high-throughput networking, spread emphasizes hardware isolation, and partition isolates logical groups of instances.

---

### Q28. How do security groups protect EC2?

**Answer:** A security group is a stateful virtual firewall attached to an ENI. It defines allowed inbound and outbound traffic. Return traffic for an allowed connection is automatically permitted.

---
