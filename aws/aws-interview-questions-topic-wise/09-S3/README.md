# S3

[⬅️ Back to AWS Topics](../README.md)

## 🔑 Keywords

🪣 Bucket | Object | Key | Storage Class | Versioning | Lifecycle | Replication | Encryption | Object Lock | Presigned URL

## 🧠 Core Memory

🧠 **Remember:** **S3 = objects in buckets**. Lifecycle = automatic movement/deletion; Versioning = recovery from overwrite/delete.

---

## ❓ Interview Questions

### 📌 Core

#### Q1. What is Amazon S3?

**💡 Answer:** Explain the S3 bucket/object model, the feature's effect on storage, access, durability, lifecycle, or security, and the workload for which it is appropriate.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q2. What is a bucket?

**💡 Answer:** Explain the S3 bucket/object model, the feature's effect on storage, access, durability, lifecycle, or security, and the workload for which it is appropriate.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q3. What is an object?

**💡 Answer:** Explain the S3 bucket/object model, the feature's effect on storage, access, durability, lifecycle, or security, and the workload for which it is appropriate.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q4. What is an object key?

**💡 Answer:** Explain the S3 bucket/object model, the feature's effect on storage, access, durability, lifecycle, or security, and the workload for which it is appropriate.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q5. What is S3 storage class?

**💡 Answer:** Explain the S3 bucket/object model, the feature's effect on storage, access, durability, lifecycle, or security, and the workload for which it is appropriate.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q6. Is S3 regional or global?

**💡 Answer:** Explain the S3 bucket/object model, the feature's effect on storage, access, durability, lifecycle, or security, and the workload for which it is appropriate.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Storage Classes

#### Q7. Explain S3 Standard.

**💡 Answer:** Explain the S3 bucket/object model, the feature's effect on storage, access, durability, lifecycle, or security, and the workload for which it is appropriate.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q8. Explain S3 Intelligent-Tiering.

**💡 Answer:** S3 Intelligent-Tiering automatically moves objects among access tiers based on access patterns, helping optimize storage cost without requiring you to predict when objects will become infrequently accessed.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q9. Explain S3 Standard-IA.

**💡 Answer:** S3 Standard-Infrequent Access is designed for data accessed less frequently but requiring millisecond access when requested. It has lower storage cost than Standard and higher access charges.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q10. Explain S3 One Zone-IA.

**💡 Answer:** S3 One Zone-Infrequent Access stores data in a single Availability Zone and costs less than multi-AZ storage classes. It is suitable for re-creatable or secondary data where AZ-level resilience is acceptable.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q11. Explain S3 Glacier Instant Retrieval.

**💡 Answer:** S3 Glacier Instant Retrieval is designed for archive data that is rarely accessed but must be retrieved with millisecond access when needed.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q12. Explain S3 Glacier Flexible Retrieval.

**💡 Answer:** S3 Glacier Flexible Retrieval is for archival data that can tolerate retrieval times from minutes to hours, with lower storage cost than frequently accessed classes.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q13. Explain S3 Glacier Deep Archive.

**💡 Answer:** S3 Glacier Deep Archive is intended for very long-lived data that is rarely accessed and can tolerate long retrieval times. It is optimized for the lowest-cost long-term S3 archival storage.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Lifecycle

#### Q14. What is an S3 Lifecycle configuration?

**💡 Answer:** An S3 Lifecycle configuration defines automated transitions and expirations for objects. Rules can target prefixes, object tags, or object-size conditions and can move data to lower-cost storage classes or delete it when no longer needed.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q15. What is a transition rule?

**💡 Answer:** Explain the S3 bucket/object model, the feature's effect on storage, access, durability, lifecycle, or security, and the workload for which it is appropriate.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q16. What is an expiration rule?

**💡 Answer:** Explain the S3 bucket/object model, the feature's effect on storage, access, durability, lifecycle, or security, and the workload for which it is appropriate.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q17. How can lifecycle rules reduce storage cost?

**💡 Answer:** S3 Lifecycle rules automate object transitions between storage classes and object expiration. They are a key cost-optimization mechanism for data with changing access patterns.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q18. Can lifecycle rules target object prefixes?

**💡 Answer:** S3 Lifecycle rules automate object transitions between storage classes and object expiration. They are a key cost-optimization mechanism for data with changing access patterns.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q19. Can lifecycle rules use object tags?

**💡 Answer:** S3 Lifecycle rules automate object transitions between storage classes and object expiration. They are a key cost-optimization mechanism for data with changing access patterns.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q20. How would you move old objects to Glacier classes automatically?

**💡 Answer:** Explain the S3 bucket/object model, the feature's effect on storage, access, durability, lifecycle, or security, and the workload for which it is appropriate.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Security

#### Q21. What is an S3 bucket policy?

**💡 Answer:** Explain the S3 bucket/object model, the feature's effect on storage, access, durability, lifecycle, or security, and the workload for which it is appropriate.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q22. What is Block Public Access?

**💡 Answer:** S3 Block Public Access provides account- and bucket-level controls that prevent common configurations from making S3 data publicly accessible. It is a defense-in-depth control, not a replacement for correct bucket policies.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q23. What is S3 Object Ownership?

**💡 Answer:** S3 Object Ownership controls ownership of objects written to a bucket. Bucket owner enforced disables ACLs for the bucket and makes the bucket owner the owner of objects, simplifying access control.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q24. What is SSE-S3?

**💡 Answer:** SSE-S3 provides server-side encryption using S3-managed encryption keys. S3 performs the encryption and key management for the customer.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q25. What is SSE-KMS?

**💡 Answer:** SSE-KMS encrypts S3 objects using AWS KMS keys. It provides additional key-control, audit, and authorization capabilities compared with SSE-S3.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q26. What is client-side encryption?

**💡 Answer:** Explain the S3 bucket/object model, the feature's effect on storage, access, durability, lifecycle, or security, and the workload for which it is appropriate.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Features

#### Q27. What is S3 Versioning?

**💡 Answer:** S3 Versioning keeps multiple versions of an object under the same key. It helps recover from accidental deletion or overwrite and is commonly combined with lifecycle rules to manage noncurrent versions.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q28. What is S3 Object Lock?

**💡 Answer:** S3 Object Lock provides WORM-style retention controls to help prevent objects from being deleted or overwritten for a defined retention period or legal hold.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q29. What is S3 replication?

**💡 Answer:** S3 replication automatically copies eligible objects between buckets. Cross-Region Replication is commonly used for disaster recovery or geographic distribution; Same-Region Replication is useful for compliance or separation requirements.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q30. What is Cross-Region Replication?

**💡 Answer:** S3 replication automatically copies eligible objects between buckets. Cross-Region Replication is commonly used for disaster recovery or geographic distribution; Same-Region Replication is useful for compliance or separation requirements.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q31. What is Same-Region Replication?

**💡 Answer:** S3 replication automatically copies eligible objects between buckets. Cross-Region Replication is commonly used for disaster recovery or geographic distribution; Same-Region Replication is useful for compliance or separation requirements.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q32. What is S3 Transfer Acceleration?

**💡 Answer:** S3 Transfer Acceleration uses AWS edge locations to accelerate uploads to S3 over long-distance networks.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q33. What are S3 presigned URLs?

**💡 Answer:** An S3 presigned URL grants time-limited access to a specific object or operation without requiring the recipient to have AWS credentials. The permissions are derived from the signing principal.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q34. What are S3 event notifications?

**💡 Answer:** Explain the S3 bucket/object model, the feature's effect on storage, access, durability, lifecycle, or security, and the workload for which it is appropriate.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q35. How can S3 integrate with EventBridge, SQS, SNS, and Lambda?

**💡 Answer:** AWS Lambda runs code without requiring you to provision or manage servers. AWS handles the underlying compute infrastructure and scales execution according to incoming requests/events.

**🔑 Keywords:** `S3` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

## 🚀 Last-Minute Revision

> 🧠 **Remember:** **S3 = objects in buckets**. Lifecycle = automatic movement/deletion; Versioning = recovery from overwrite/delete.

[⬆️ Back to top](#s3)

[⬅️ Back to AWS Topics](../README.md)