# AWS Storage Services

### Q1. What is EBS?

**Answer:** Amazon EBS is persistent block storage designed primarily for EC2 instances. It provides disk-like volumes for operating systems, filesystems, and databases and supports features such as snapshots, resizing, and encryption.

---

### Q2. When is block storage appropriate?

**Answer:** Block storage is appropriate when a workload needs a disk-like device with filesystem or database access, low-latency random I/O, and control over the filesystem. EBS is a common AWS choice for persistent block storage attached to EC2.

---

### Q3. What is instance store?

**Answer:** Instance store is local ephemeral storage physically attached to the EC2 host. It can provide very high I/O performance, but data is temporary and should not be treated as durable storage.

---

### Q4. What is EFS?

**Answer:** Amazon EFS is managed elastic file storage that can be mounted concurrently by multiple compute resources. It is designed for shared POSIX-compatible file-system access.

---

### Q5. What is FSx?

**Answer:** Amazon FSx provides managed file systems optimized for specific workloads, including Windows File Server and high-performance file-system use cases.

---

### Q6. How does EFS differ from EBS?

**Answer:** EFS is managed shared file storage that multiple compute resources can mount concurrently and is designed for elastic filesystem access. EBS is block storage normally attached to an EC2 instance and is suited to operating systems, databases, and applications that need a disk-like device.

---

### Q7. When is shared file storage required?

**Answer:** Shared file storage is required when multiple compute resources need concurrent access to the same filesystem and directory structure. EFS is a common choice for Linux workloads, while appropriate FSx services support specific filesystem and application requirements.

---

### Q8. What is S3?

**Answer:** Amazon S3 is an object-storage service. Data is stored as objects in buckets and can be protected, versioned, replicated, transitioned between storage classes, and accessed through APIs.

---

### Q9. Why is S3 object storage rather than block storage?

**Answer:** An S3 object is a unit of stored data consisting of the object data, key, metadata, and related attributes. Objects are stored inside buckets.

---

### Q10. What is S3 Glacier?

**Answer:** S3 Glacier refers to S3 archival storage classes designed for long-term, infrequently accessed data. Glacier Instant Retrieval provides rapid access, Glacier Flexible Retrieval supports retrieval from minutes to hours, and Glacier Deep Archive targets very long-term archival at very low storage cost.

---

### Q11. What is Storage Gateway?

**Answer:** AWS Storage Gateway provides hybrid cloud storage integrations between on-premises environments and AWS storage services. It includes File Gateway, Volume Gateway, and Tape Gateway.

---

### Q12. What are File Gateway, Volume Gateway, and Tape Gateway?

**Answer:** Volume Gateway provides iSCSI block-storage volumes backed by AWS cloud storage and supports cached and stored volume modes.

---

### Q13. When would you use DataSync?

**Answer:** AWS DataSync is a managed high-speed data transfer service for moving data between on-premises storage and AWS storage services or between supported AWS storage locations.

---
