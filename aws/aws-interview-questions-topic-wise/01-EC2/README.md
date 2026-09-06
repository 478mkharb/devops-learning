# EC2

### Q1. What is Amazon EC2?

**Answer:** Amazon EC2 (Elastic Compute Cloud) is a service that provides resizable virtual servers called instances. You choose the operating system, instance type, storage, networking, and security settings, while AWS manages the underlying physical infrastructure.

---

### Q2. What is an EC2 instance?

**Answer:** An EC2 instance is a virtual server running on AWS infrastructure. It is launched from an AMI and uses a selected instance type that determines resources such as vCPU, memory, network performance, and supported storage.

---

### Q3. What is an AMI?

**Answer:** An AMI (Amazon Machine Image) is a template used to launch EC2 instances. It specifies the operating-system image and block-device mappings needed to create the instance.

---

### Q4. What is an instance type?

**Answer:** An EC2 instance type defines the compute resources and capabilities of an instance, including vCPUs, memory, networking, and sometimes local storage or accelerators. Instance families are optimized for different workloads.

---

### Q5. What is user data?

**Answer:** EC2 user data is configuration or a startup script supplied when an instance is launched. It is commonly used to install software, configure services, and bootstrap an instance automatically.

---

### Q6. What is instance metadata?

**Answer:** EC2 instance metadata is information available from the Instance Metadata Service about the running instance, such as instance ID, networking information, and, when an IAM role is attached, temporary credentials. IMDSv2 is the recommended access method.

---

### Q7. Explain On-Demand, Reserved Instances, Savings Plans, Spot Instances, and Dedicated Hosts/Instances.

**Answer:** EC2 purchasing options serve different cost and commitment models. On-Demand has no long-term commitment and is billed for usage. Reserved Instances provide a discount for eligible usage with a commitment. Savings Plans provide a discount in exchange for a committed hourly spend. Spot uses spare EC2 capacity at lower cost but can be interrupted. Dedicated Instances run on dedicated hardware, while Dedicated Hosts provide a dedicated physical server with visibility into socket/core placement and are useful for licensing or compliance requirements.

---

### Q8. When should you use Spot Instances?

**Answer:** Use Spot Instances for fault-tolerant and interruptible workloads such as batch jobs, CI workers, distributed processing, and stateless applications. Do not depend on a single Spot instance for critical stateful capacity.

---

### Q9. What happens when a Spot Instance is interrupted?

**Answer:** AWS can interrupt a Spot Instance when the capacity is needed back. The application should handle interruption gracefully by saving work, draining the instance, and replacing capacity through an ASG or another scheduler.

---

### Q10. When are Reserved Instances useful?

**Answer:** Reserved Instances are useful when you have predictable, steady EC2 usage and expect to keep an eligible configuration for the commitment period. They reduce the effective hourly cost compared with On-Demand pricing.

---

### Q11. How do Savings Plans differ from Reserved Instances?

**Answer:** Savings Plans provide a discount in exchange for a committed hourly compute spend and generally offer more flexibility across eligible compute usage. Reserved Instances are tied more closely to a specific instance configuration and Region, depending on the RI type. Choose based on the flexibility and commitment model your workload requires.

---

### Q12. Explain general purpose, compute optimized, memory optimized, storage optimized, and accelerated computing families.

**Answer:** General-purpose families provide a balanced CPU-to-memory ratio. Compute-optimized families favor CPU-intensive workloads. Memory-optimized families provide high memory capacity. Storage-optimized families are designed for high local storage I/O. Accelerated-computing families use GPUs or other accelerators for specialized workloads.

---

### Q13. Which family is suitable for CPU-heavy workloads?

**Answer:** Use a compute-optimized family, such as the C family, for CPU-bound workloads where high compute capacity is more important than unusually large memory capacity.

---

### Q14. Which family is suitable for in-memory databases?

**Answer:** Use a memory-optimized family, such as the R family, when the workload needs a large amount of RAM relative to CPU, for example in-memory databases, large caches, or memory-intensive analytics.

---

### Q15. Which family is suitable for high local NVMe storage workloads?

**Answer:** Use a storage-optimized family, such as an I family, when the workload needs high-performance local NVMe storage and very high local I/O, such as some analytics and distributed data-processing workloads.

---

### Q16. When would you use GPU-based instances?

**Answer:** Use GPU or other accelerated EC2 instances for workloads that benefit from parallel acceleration, such as machine learning, graphics rendering, scientific computing, and some high-performance workloads.

---

### Q17. What are private and public IPv4 addresses?

**Answer:** A private IPv4 address is used for communication inside the VPC and connected private networks. A public IPv4 address provides Internet-reachable addressing when routing and security controls permit it. Private addresses remain stable for the network interface, while public IPv4 can change when an instance is stopped and started unless a stable public address mechanism is used.

---

### Q18. What is an Elastic IP?

**Answer:** An Elastic IP is a static public IPv4 address allocated to an AWS account and associated with a supported resource. It is useful when a stable public IP is required, although DNS is generally preferred for application endpoints.

---

### Q19. What is an ENI?

**Answer:** An Elastic Network Interface is a virtual network interface in a VPC. It contains properties such as private IP addresses, security groups, and a MAC address and can be attached to supported resources.

---

### Q20. What is EBS?

**Answer:** Amazon Elastic Block Store provides persistent block storage for EC2 instances. EBS volumes can be used for operating-system disks, application filesystems, and databases, and they support snapshots and encryption.

---

### Q21. What is instance store?

**Answer:** Instance store is local ephemeral storage physically attached to the EC2 host. It can provide very high I/O performance, but its data is not persistent like EBS and can be lost when the instance or underlying host is stopped or terminated.

---

### Q22. What is the difference between EBS and instance store?

**Answer:** EBS is persistent network-attached block storage that supports snapshots and is designed to survive an EC2 stop. Instance store is local ephemeral storage with very high local performance but data-loss risk tied to the instance/host lifecycle.

---

### Q23. What are stop, start, reboot, and terminate?

**Answer:** Stop shuts down an EBS-backed instance while preserving its EBS volumes according to their settings; start boots it again. Reboot restarts the operating system on the same instance. Terminate permanently removes the EC2 instance and usually deletes EBS volumes configured with DeleteOnTermination.

---

### Q24. What happens to EBS volumes when an instance is terminated?

**Answer:** EBS volumes have a DeleteOnTermination attribute. If it is true, the volume is deleted when the instance terminates; if false, the volume remains and can be attached elsewhere. The root volume is commonly configured to delete on termination, while important data volumes are often retained.

---

### Q25. How does an IAM role attach to EC2?

**Answer:** An IAM role is an identity that trusted principals can assume to obtain temporary credentials. Roles are commonly used by EC2, Lambda, federated users, and cross-account access.

---

### Q26. What is an EC2 placement group?

**Answer:** A placement group controls the physical placement strategy of EC2 instances. Cluster placement is optimized for low-latency, high-throughput networking; spread placement emphasizes hardware isolation; partition placement isolates groups of instances for distributed workloads.

---

### Q27. Explain cluster, spread, and partition placement groups.

**Answer:** Cluster placement groups place instances close together for low-latency, high-throughput networking. Spread placement groups place instances on distinct underlying hardware to reduce correlated failure risk. Partition placement groups divide instances into partitions so hardware failures can be isolated between groups.

---

### Q28. How do security groups protect EC2?

**Answer:** The security pillar focuses on protecting information and systems through strong identity controls, detection, infrastructure protection, data protection, and incident response.

---
