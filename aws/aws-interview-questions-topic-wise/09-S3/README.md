# S3

### Q1. What is Amazon S3?

**Answer:** Amazon S3 (Simple Storage Service) is an object storage service. It stores data as objects inside buckets and provides features such as versioning, lifecycle management, replication, encryption, access control, and event notifications.

---

### Q2. What is an S3 bucket?

**Answer:** An S3 bucket is a container for storing objects. A bucket is created in a specific AWS Region and its name must be globally unique within the S3 bucket namespace.

---

### Q3. What is an S3 object?

**Answer:** An S3 object is the fundamental unit of data stored in S3. It consists of the object data, a key, metadata, and other object attributes.

---

### Q4. What is an S3 object key?

**Answer:** An object key is the unique name used to identify an object within a bucket. S3 does not have traditional folders; what appears as a folder structure is created using prefixes in object keys.

For example:

```text
my-bucket/
└── logs/2026/application.log
```

Here, `logs/2026/application.log` is the object key, and `logs/2026/` is the prefix.

---

### Q5. What is an S3 storage class?

**Answer:** An S3 storage class determines how objects are stored and priced based on factors such as access frequency, availability requirements, retrieval requirements, and storage cost.

| Storage class | Typical use | Access pattern | Key characteristic |
|---|---|---|---|
| **S3 Standard** | Frequently accessed data | Frequent | General-purpose, low-latency access |
| **S3 Intelligent-Tiering** | Unknown or changing access patterns | Variable | Automatically moves objects between access tiers |
| **S3 Standard-IA** | Infrequently accessed data | Infrequent | Lower storage cost with retrieval charges |
| **S3 One Zone-IA** | Re-creatable infrequent data | Infrequent | Stored in a single Availability Zone |
| **S3 Glacier Instant Retrieval** | Rarely accessed archive data | Rare | Millisecond retrieval |
| **S3 Glacier Flexible Retrieval** | Archive data | Rare | Retrieval typically takes minutes to hours |
| **S3 Glacier Deep Archive** | Long-term archival | Very rare | Very low storage cost for long-term archive use |


---

### Q6. Is S3 regional or global?

**Answer:** S3 is globally accessible as a service, but S3 buckets are Regional resources. When a bucket is created, it is associated with a specific AWS Region, and its objects are stored in that Region by default.

Replication can be configured to copy objects to another Region.

---

### Q7. What is S3 Standard?

**Answer:** S3 Standard is designed for frequently accessed data that requires low-latency access. It provides high durability and availability and is suitable for general-purpose workloads.

---

### Q8. What is S3 Intelligent-Tiering?

**Answer:** S3 Intelligent-Tiering automatically moves objects between access tiers based on changing access patterns. It is useful when access patterns are unpredictable because S3 automatically optimizes storage cost without requiring you to manually predict future access.

---

### Q9. What is S3 Standard-IA?

**Answer:** S3 Standard-Infrequent Access is designed for data that is accessed less frequently but still requires rapid access when needed. It has lower storage costs than S3 Standard but generally has retrieval and minimum-storage-duration considerations.

---

### Q10. What is S3 One Zone-IA?

**Answer:** S3 One Zone-Infrequent Access stores objects in a single Availability Zone. It provides lower storage cost than multi-AZ S3 storage classes and is suitable for infrequently accessed, re-creatable data where single-AZ storage is acceptable.

---

### Q11. What is S3 Glacier Instant Retrieval?

**Answer:** S3 Glacier Instant Retrieval is designed for rarely accessed archive data that still requires immediate, millisecond-level access when retrieved.

---

### Q12. What is S3 Glacier Flexible Retrieval?

**Answer:** S3 Glacier Flexible Retrieval is designed for archive data where retrieval can take from minutes to hours. It provides lower storage cost than frequently accessed storage classes and is suitable when immediate access is not required.

---

### Q13. What is S3 Glacier Deep Archive?

**Answer:** S3 Glacier Deep Archive is designed for long-term retention of data that is rarely accessed and can tolerate longer retrieval times. It provides very low storage costs and is commonly used for compliance and long-term archival.

---

### Q14. What is an S3 Lifecycle configuration?

**Answer:** An S3 Lifecycle configuration consists of rules that automatically manage objects throughout their lifecycle. The two primary lifecycle actions are **transition** and **expiration**.

| Action | Purpose | Example |
|---|---|---|
| **Transition** | Move objects to another storage class | Standard → Glacier Flexible Retrieval |
| **Expiration** | Remove objects or versions after retention conditions are met | Delete objects after 365 days |


---

### Q15. What is an S3 Lifecycle transition rule?

**Answer:** A transition rule automatically moves objects to another S3 storage class when specified conditions are met, such as when an object reaches a certain age.

For example:

```text
S3 Standard
     |
   30 days
     ↓
Glacier Flexible Retrieval
     |
   180 days
     ↓
Glacier Deep Archive
```

---

### Q16. What is an S3 Lifecycle expiration rule?

**Answer:** An expiration rule automatically removes objects or noncurrent object versions when the configured lifecycle conditions are met.

---

### Q17. How can S3 Lifecycle rules reduce storage costs?

**Answer:** Lifecycle rules automatically move less frequently accessed data to lower-cost storage classes and can expire objects or old versions when they are no longer required. This reduces manual management and aligns storage costs with the data lifecycle.

---

### Q18. Can S3 Lifecycle rules target object prefixes?

**Answer:** Yes. Lifecycle rules can use prefixes as filters.

For example:

```text
logs/
archive/
backups/
```

A rule targeting `logs/` can apply only to objects whose keys begin with that prefix.

---

### Q19. Can S3 Lifecycle rules use object tags?

**Answer:** Yes. Lifecycle rules can use object tags as filters. This allows different lifecycle policies to be applied based on application-defined classifications.

For example:

```text
Environment=dev
Environment=prod
DataType=archive
```

---

### Q20. How would you automatically move old S3 objects to Glacier?

**Answer:** Create an S3 Lifecycle rule that matches the required objects and configure a transition to the appropriate Glacier storage class after a specified number of days.

For example:

```text
Day 0       → S3 Standard
Day 30      → Glacier Flexible Retrieval
Day 180     → Glacier Deep Archive
```

The actual transition periods should be selected according to the workload's access and retention requirements.

---

### Q21. What is an S3 bucket policy?

**Answer:** An S3 bucket policy is a resource-based JSON policy attached to an S3 bucket. It defines which principals can perform which actions on which S3 resources under specified conditions.

---

### Q22. What is S3 Block Public Access?

**Answer:** S3 Block Public Access provides controls at the account and bucket levels to prevent public access through common bucket and access-policy configurations. It acts as a defense-in-depth mechanism against accidental public exposure.

---

### Q23. What is S3 Object Ownership?

**Answer:** S3 Object Ownership controls object ownership and the use of ACLs. With **Bucket owner enforced**, ACLs are disabled and the bucket owner automatically owns objects uploaded to the bucket.

---

### Q24. What is SSE-S3?

**Answer:** SSE-S3 is server-side encryption where Amazon S3 manages the encryption keys and performs encryption of objects at rest.

---

### Q25. What is SSE-KMS?

**Answer:** SSE-KMS is server-side encryption for S3 objects using AWS Key Management Service (KMS) keys. It provides additional control over key permissions, auditing, and key management compared with SSE-S3.

---

### Q26. What is client-side encryption?

**Answer:** Client-side encryption encrypts data before it is uploaded to S3. The client or application performs the encryption, and S3 stores the resulting ciphertext.

### SSE-S3 vs SSE-KMS vs Client-Side Encryption

| Feature | SSE-S3 | SSE-KMS | Client-side encryption |
|---|---|---|---|
| Encryption location | S3 | S3 | Client/application |
| Data sent to S3 as plaintext | Yes, assuming HTTPS is used for transit | Yes, assuming HTTPS is used for transit | No |
| Key management | S3 | AWS KMS | Customer/application |
| Key-level access control | Managed by S3 | KMS IAM/key-policy controls | Customer-controlled |
| Audit/key-usage controls | Basic S3 controls | KMS provides key-usage auditing | Depends on implementation |
| Application complexity | Low | Low to moderate | Higher |
| Typical use | General encryption at rest | Additional key control and auditing | When plaintext must not be uploaded to S3 |


---

### Q27. What is S3 Versioning?

**Answer:** S3 Versioning maintains multiple versions of an object under the same object key. It helps recover from accidental overwrites and deletions and allows lifecycle management of older versions.

---

### Q28. What is S3 Object Lock?

**Answer:** S3 Object Lock provides WORM-style (Write Once, Read Many) protection. It can prevent objects from being deleted or overwritten during a configured retention period or while a legal hold is active.

| Control | Behavior |
|---|---|
| **Governance mode** | Retention can potentially be bypassed by users with the required special permissions |
| **Compliance mode** | Protected objects cannot be deleted or overwritten until the retention period expires |
| **Legal Hold** | Prevents deletion independently of a configured retention period |

### Versioning vs Object Lock

| Feature | Versioning | Object Lock |
|---|---|---|
| Main purpose | Preserve multiple object versions | Prevent deletion/overwrite during protection |
| Helps recover accidental overwrite | Yes | Not its primary purpose |
| WORM protection | No | Yes |
| Legal hold | No | Yes |
| Works with versions | Provides the versions | Retention can protect object versions |


---

### Q29. What is S3 Replication?

**Answer:** S3 Replication automatically copies eligible objects from a source bucket to one or more destination buckets. Replication can occur across AWS Regions or within the same Region.

| Replication type | Source | Destination | Typical use |
|---|---|---|---|
| **CRR** | One Region | Different Region | Disaster recovery, geographic distribution, compliance |
| **SRR** | One Region | Same Region | Compliance, data separation, replication workflows |

It can also be used for other data-distribution and operational requirements.


---

### Q30. What is S3 Cross-Region Replication (CRR)?

**Answer:** S3 Cross-Region Replication automatically replicates eligible objects from a source bucket to a destination bucket in a different AWS Region.

For example:

```text
us-east-1
Source S3 Bucket
      |
      | CRR
      ↓
ap-south-1
Destination S3 Bucket
```

It can be useful for disaster recovery, compliance, and geographic data distribution.

---

### Q31. What is S3 Same-Region Replication (SRR)?

**Answer:** S3 Same-Region Replication automatically replicates eligible objects to a destination bucket in the same AWS Region as the source bucket. It can be useful for compliance, data separation, and maintaining separate copies of data within a Region.

---

### Q32. What is S3 Transfer Acceleration?

**Answer:** S3 Transfer Acceleration speeds up long-distance transfers to S3 by using AWS edge locations. Instead of sending data directly over the long-distance path to the S3 bucket's Region, the client uploads through an AWS edge location, which forwards the data over AWS's network.

---

### Q33. What is an S3 presigned URL?

**Answer:** An S3 presigned URL provides temporary access to a specific S3 object or operation without requiring the recipient to have AWS credentials.

The permissions available through the URL are based on the permissions of the principal that generated it, and the URL has a configured expiration time.

A common use case is allowing a user to download a private file temporarily.

---

### Q34. What are S3 event notifications?

**Answer:** S3 event notifications allow S3 to generate events when supported bucket or object events occur, such as object creation or deletion. These events can be delivered to supported AWS destinations.


---

### Q35. How can S3 integrate with EventBridge, SQS, SNS, and Lambda?

**Answer:** S3 can generate events when objects are created, deleted, or when other supported events occur. Those events can be sent to different AWS services depending on the required architecture.

| Service | Typical use |
|---|---|
| **SQS** | Buffer events for asynchronous processing |
| **SNS** | Fan out notifications to multiple subscribers |
| **Lambda** | Execute code automatically in response to events |
| **EventBridge** | Route and filter events to multiple AWS targets |

Example:

```text
                 ┌──→ SQS → Worker
                 │
S3 Object ──Event├──→ Lambda → Processing
                 │
                 ├──→ SNS → Subscribers
                 │
                 └──→ EventBridge → Multiple targets
```

A common example is:

```text
User uploads image
       ↓
      S3
       ↓
S3 Object Created event
       ↓
    Lambda
       ↓
Resize / Process image
```
