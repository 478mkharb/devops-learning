# S3

### Q1. What is Amazon S3?

**Answer:** Amazon S3 is an object-storage service. Data is stored as objects in buckets and can be protected, versioned, replicated, transitioned between storage classes, and accessed through APIs.

---

### Q2. What is a bucket?

**Answer:** An S3 bucket is a container for objects. A bucket has a globally unique name within the relevant namespace and is created in a chosen AWS Region.

---

### Q3. What is an object?

**Answer:** An S3 object is a unit of stored data consisting of the object data, key, metadata, and related attributes. Objects are stored inside buckets.

---

### Q4. What is an object key?

**Answer:** An S3 object key is the unique identifier of an object within a bucket. Prefixes are part of the key name and are commonly used to organize objects logically.

---

### Q5. What is S3 storage class?

**Answer:** An S3 storage class defines how S3 stores and prices objects based on access frequency, availability characteristics, retrieval behavior, and storage cost.

---

### Q6. Is S3 regional or global?

**Answer:** S3 is a regional service: each bucket is created in a specific AWS Region and its objects are stored there unless another feature, such as replication, copies them elsewhere. The S3 service itself is globally available, but bucket data is Region-specific.

---

### Q7. Explain S3 Standard.

**Answer:** S3 Standard is designed for frequently accessed data and provides high durability and availability with low-latency access.

---

### Q8. Explain S3 Intelligent-Tiering.

**Answer:** S3 Intelligent-Tiering automatically moves objects among access tiers based on changing access patterns, helping reduce storage cost without requiring you to predict when access frequency will change.

---

### Q9. Explain S3 Standard-IA.

**Answer:** S3 Standard-Infrequent Access is designed for data accessed less frequently but requiring rapid access when requested. It has lower storage cost and higher access charges than S3 Standard.

---

### Q10. Explain S3 One Zone-IA.

**Answer:** S3 One Zone-Infrequent Access stores data in a single Availability Zone and costs less than multi-AZ storage classes. It is suitable for re-creatable or secondary data where single-AZ storage is acceptable.

---

### Q11. Explain S3 Glacier Instant Retrieval.

**Answer:** S3 Glacier Instant Retrieval is designed for rarely accessed archive data that still needs millisecond access when retrieved.

---

### Q12. Explain S3 Glacier Flexible Retrieval.

**Answer:** S3 Glacier Flexible Retrieval is designed for archive data where retrieval can take minutes to hours in exchange for lower storage cost.

---

### Q13. Explain S3 Glacier Deep Archive.

**Answer:** S3 Glacier Deep Archive is designed for long-term archival data that is rarely accessed and can tolerate long retrieval times. It is optimized for very low storage cost.

---

### Q14. What is an S3 Lifecycle configuration?

**Answer:** An S3 Lifecycle configuration contains rules that automatically transition objects between storage classes or expire them. Rules can be scoped using prefixes, tags, and supported object-size conditions.

---

### Q15. What is a transition rule?

**Answer:** An S3 Lifecycle transition rule automatically moves objects to another storage class after a specified period or when conditions are met.

---

### Q16. What is an expiration rule?

**Answer:** An S3 Lifecycle expiration rule automatically deletes objects or noncurrent object versions when the configured conditions are met.

---

### Q17. How can lifecycle rules reduce storage cost?

**Answer:** Lifecycle rules reduce cost by automatically transitioning objects to cheaper storage classes as they become less frequently accessed and expiring objects or old versions when retention is complete. This avoids manual object management and aligns storage cost with the data lifecycle.

---

### Q18. Can lifecycle rules target object prefixes?

**Answer:** Yes. A Lifecycle rule can use a prefix filter so that transitions or expiration apply only to objects whose keys begin with that prefix, such as `logs/` or `archive/`.

---

### Q19. Can lifecycle rules use object tags?

**Answer:** Yes. Lifecycle rules can use object tags as filters. This lets you apply different retention or transition behavior to objects based on application-defined classifications.

---

### Q20. How would you move old objects to Glacier classes automatically?

**Answer:** Create an S3 Lifecycle rule that matches the required objects and add a transition action after a defined number of days. For example, objects can transition from Standard to Glacier Flexible Retrieval or Deep Archive when they become long-term archival data.

---

### Q21. What is an S3 bucket policy?

**Answer:** An S3 bucket policy is a resource-based JSON policy attached to a bucket. It can control access by principals, actions, resources, and conditions.

---

### Q22. What is Block Public Access?

**Answer:** S3 Block Public Access provides account- and bucket-level controls that prevent common configurations from exposing S3 data publicly. It is a defense-in-depth control.

---

### Q23. What is S3 Object Ownership?

**Answer:** S3 Object Ownership controls object ownership and ACL behavior. With Bucket owner enforced, ACLs are disabled and the bucket owner owns newly written objects.

---

### Q24. What is SSE-S3?

**Answer:** SSE-S3 provides server-side encryption using S3-managed encryption keys. S3 handles the encryption-key management for the customer.

---

### Q25. What is SSE-KMS?

**Answer:** SSE-KMS encrypts S3 objects using AWS KMS keys. It provides additional key-management, authorization, and audit controls.

---

### Q26. What is client-side encryption?

**Answer:** Client-side encryption encrypts data before it is uploaded to S3. The customer controls the encryption process and keys, while S3 stores the resulting ciphertext.

---

### Q27. What is S3 Versioning?

**Answer:** S3 Versioning keeps multiple versions of an object under the same key. It helps recover from accidental overwrite or deletion and can be combined with lifecycle rules for old versions.

---

### Q28. What is S3 Object Lock?

**Answer:** S3 Object Lock provides WORM-style retention controls that help prevent objects from being deleted or overwritten for a configured retention period or legal hold.

---

### Q29. What is S3 replication?

**Answer:** S3 replication automatically copies eligible objects from a source bucket to destination buckets. It can be configured across Regions or within the same Region and is used for resilience, compliance, data distribution, and other replication requirements.

---

### Q30. What is Cross-Region Replication?

**Answer:** S3 Cross-Region Replication automatically replicates eligible objects from a source bucket to a destination bucket in another AWS Region. It is useful for disaster recovery, compliance, and geographic distribution.

---

### Q31. What is Same-Region Replication?

**Answer:** S3 Same-Region Replication automatically replicates eligible objects to another bucket in the same AWS Region. It can support compliance, separation, and replication workflows.

---

### Q32. What is S3 Transfer Acceleration?

**Answer:** S3 Transfer Acceleration uses AWS edge locations to accelerate uploads to S3 over long-distance networks.

---

### Q33. What are S3 presigned URLs?

**Answer:** An S3 presigned URL grants time-limited access to a specific S3 object or operation without requiring the recipient to have AWS credentials. The permissions are derived from the principal that created the URL.

---

### Q34. What are S3 event notifications?

**Answer:** S3 event notifications publish events when supported bucket/object actions occur, such as object creation or deletion. They can integrate with destinations including SNS, SQS, Lambda, and EventBridge for event-driven processing.

---

### Q35. How can S3 integrate with EventBridge, SQS, SNS, and Lambda?

**Answer:** AWS Lambda is a serverless compute service that runs code in response to events without requiring the customer to manage servers. AWS provisions and scales the execution environment.

---
