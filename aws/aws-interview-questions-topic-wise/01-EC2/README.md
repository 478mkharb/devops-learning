# EC2

[⬅️ Back to AWS Topics](../README.md)

## 🔑 Keywords

🖥️ EC2 | Instance | AMI | Instance Type | EBS | ENI | User Data | IMDS

## 🧠 Core Memory

🧠 **Remember:** EC2 = **virtual server**. AWS manages the physical infrastructure; you manage the OS and workload.

---

## ❓ Interview Questions

### 📌 Core Concepts

#### Q1. What is Amazon EC2?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q2. What is an EC2 instance?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q3. What is an AMI?

**💡 Answer:** An AMI is a reusable template used to launch EC2 instances. It defines the operating-system image and block-device mappings and can be copied or shared according to AWS rules.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q4. What is an instance type?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q5. What is user data?

**💡 Answer:** EC2 user data is bootstrap data supplied at launch. It is commonly used to install packages, configure services, create files, or run initialization scripts automatically.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q6. What is instance metadata?

**💡 Answer:** The EC2 Instance Metadata Service exposes instance information and, when an IAM role is attached, temporary credentials. IMDSv2 uses a session token and is recommended for stronger protection against credential abuse.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Pricing Models

#### Q7. Explain On-Demand, Reserved Instances, Savings Plans, Spot Instances, and Dedicated Hosts/Instances.

**💡 Answer:** Spot Instances use spare EC2 capacity at a discount. They can be interrupted, so they are best for fault-tolerant, restartable, stateless, or distributed workloads such as batch processing and CI workers.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q8. When should you use Spot Instances?

**💡 Answer:** Spot Instances use spare EC2 capacity at a discount. They can be interrupted, so they are best for fault-tolerant, restartable, stateless, or distributed workloads such as batch processing and CI workers.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q9. What happens when a Spot Instance is interrupted?

**💡 Answer:** Spot Instances use spare EC2 capacity at a discount. They can be interrupted, so they are best for fault-tolerant, restartable, stateless, or distributed workloads such as batch processing and CI workers.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q10. When are Reserved Instances useful?

**💡 Answer:** Reserved Instances provide a billing discount for eligible EC2 usage in exchange for a commitment. They are useful for steady, predictable workloads where the relevant configuration is expected to remain in use.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q11. How do Savings Plans differ from Reserved Instances?

**💡 Answer:** Reserved Instances provide a billing discount for eligible EC2 usage in exchange for a commitment. They are useful for steady, predictable workloads where the relevant configuration is expected to remain in use.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Instance Families

#### Q12. Explain general purpose, compute optimized, memory optimized, storage optimized, and accelerated computing families.

**💡 Answer:** General-purpose instances provide a balanced ratio of compute, memory, and networking. They are a good default for web servers, application servers, and many development workloads.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q13. Which family is suitable for CPU-heavy workloads?

**💡 Answer:** An AMI is a reusable template used to launch EC2 instances. It defines the operating-system image and block-device mappings and can be copied or shared according to AWS rules.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q14. Which family is suitable for in-memory databases?

**💡 Answer:** An AMI is a reusable template used to launch EC2 instances. It defines the operating-system image and block-device mappings and can be copied or shared according to AWS rules.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q15. Which family is suitable for high local NVMe storage workloads?

**💡 Answer:** An AMI is a reusable template used to launch EC2 instances. It defines the operating-system image and block-device mappings and can be copied or shared according to AWS rules.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q16. When would you use GPU-based instances?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Networking & Storage

#### Q17. What are private and public IPv4 addresses?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q18. What is an Elastic IP?

**💡 Answer:** An Elastic IP is a static public IPv4 address that can be associated with supported AWS resources. It is useful when a stable public address is required, although DNS names are usually preferable for application endpoints.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q19. What is an ENI?

**💡 Answer:** An Elastic Network Interface is a virtual network interface containing attributes such as private IP addresses, security groups, and a MAC address. It provides network connectivity for supported VPC resources.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q20. What is EBS?

**💡 Answer:** Amazon EBS provides persistent block storage for EC2. EBS volumes can be used for boot disks, filesystems, and databases, and snapshots can provide backup and recovery.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q21. What is instance store?

**💡 Answer:** Instance store is local ephemeral storage physically attached to the host. It can provide very high I/O performance, but data is not durable like EBS and can be lost when the instance or underlying host is stopped or terminated.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q22. What is the difference between EBS and instance store?

**💡 Answer:** Instance store is local ephemeral storage physically attached to the host. It can provide very high I/O performance, but data is not durable like EBS and can be lost when the instance or underlying host is stopped or terminated.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Operations

#### Q23. What are stop, start, reboot, and terminate?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q24. What happens to EBS volumes when an instance is terminated?

**💡 Answer:** Amazon EBS provides persistent block storage for EC2. EBS volumes can be used for boot disks, filesystems, and databases, and snapshots can provide backup and recovery.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q25. How does an IAM role attach to EC2?

**💡 Answer:** AWS IAM controls authentication and authorization to AWS resources. It includes identities such as users, groups, and roles and policies that determine allowed actions.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q26. What is an EC2 placement group?

**💡 Answer:** A placement group controls how EC2 instances are placed on underlying hardware. Cluster is optimized for low-latency/high-throughput networking, spread emphasizes hardware isolation, and partition isolates logical groups of instances.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q27. Explain cluster, spread, and partition placement groups.

**💡 Answer:** A placement group controls how EC2 instances are placed on underlying hardware. Cluster is optimized for low-latency/high-throughput networking, spread emphasizes hardware isolation, and partition isolates logical groups of instances.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q28. How do security groups protect EC2?

**💡 Answer:** A security group is a stateful virtual firewall attached to an ENI. It defines allowed inbound and outbound traffic. Return traffic for an allowed connection is automatically permitted.

**🔑 Keywords:** `EC2` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

## 🚀 Last-Minute Revision

> 🧠 **Remember:** EC2 = **virtual server**. AWS manages the physical infrastructure; you manage the OS and workload.

[⬆️ Back to top](#ec2)

[⬅️ Back to AWS Topics](../README.md)