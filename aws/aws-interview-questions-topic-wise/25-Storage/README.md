# AWS Storage Services

## Interview Questions & Answers

### Q1. What is EBS?

**Answer:** Amazon EBS provides persistent block storage for EC2. EBS volumes can be used for boot disks, filesystems, and databases, and snapshots can provide backup and recovery.

---

### Q2. When is block storage appropriate?

**Answer:** Block storage is appropriate when an operating system or application needs a block device, filesystem, or database volume with low-latency random I/O. Amazon EBS is the common persistent block-storage choice for EC2.

---

### Q3. What is instance store?

**Answer:** Instance store is local ephemeral storage physically attached to the host. It can provide very high I/O performance, but data is not durable like EBS and can be lost when the instance or underlying host is stopped or terminated.

---

### Q4. What is EFS?

**Answer:** Amazon EFS is managed elastic file storage that can be mounted by multiple compute resources and is designed for shared file-system access.

---

### Q5. What is FSx?

**Answer:** Amazon FSx provides managed file systems for specific workload requirements, including Windows File Server and high-performance file-system options.

---

### Q6. How does EFS differ from EBS?

**Answer:** Amazon EBS provides persistent block storage for EC2. EBS volumes can be used for boot disks, filesystems, and databases, and snapshots can provide backup and recovery.

---

### Q7. When is shared file storage required?

**Answer:** Shared file storage is required when multiple compute resources need concurrent access to the same filesystem and files. Amazon EFS is a common choice for shared Linux file storage; FSx provides specialized managed file systems.

---

### Q8. What is S3?

**Answer:** Amazon S3 is object storage. Applications store objects in buckets and access them through S3 APIs. It is designed for scalable, highly durable storage rather than presenting a block device to an operating system.

---

### Q9. Why is S3 object storage rather than block storage?

**Answer:** S3 stores complete objects addressed by bucket and key and exposes them through APIs. It does not provide a traditional block device, so an operating system cannot mount an S3 bucket as a normal block disk without an additional filesystem/gateway layer.

---

### Q10. What is S3 Glacier?

**Answer:** S3 Glacier is a family of S3 archival storage classes for infrequently accessed data. Glacier Instant Retrieval provides rapid access, Glacier Flexible Retrieval supports retrieval from minutes to hours, and Glacier Deep Archive targets very long-term, rarely accessed data.

---

### Q11. What is Storage Gateway?

**Answer:** AWS Storage Gateway provides hybrid storage integration between on-premises environments and AWS. It offers file, volume, and tape gateway modes so existing applications can use familiar storage interfaces while data is backed by AWS services.

---

### Q12. What are File Gateway, Volume Gateway, and Tape Gateway?

**Answer:** Storage Gateway File Gateway presents file shares to on-premises applications while storing files as objects in Amazon S3.

---

### Q13. When would you use DataSync?

**Answer:** AWS DataSync transfers data between on-premises storage and AWS storage services or between supported AWS storage locations. It is optimized for high-speed managed data transfer.

---
