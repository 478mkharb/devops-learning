# DynamoDB

[⬅️ Back to AWS Topics](../README.md)

## 🔑 Keywords

⚡ Table | Item | Partition Key | Sort Key | GSI | LSI | RCU | WCU | Streams | TTL | Global Tables

## 🧠 Core Memory

🧠 **Remember:** **Partition Key decides where data lives; Sort Key organizes items within the same partition key.**

---

## ❓ Interview Questions

### 📌 Core

#### Q1. What is DynamoDB?

**💡 Answer:** Amazon DynamoDB is a fully managed NoSQL key-value and document database designed for low-latency performance at scale. Data modeling centers on partition keys and optional sort keys.

**🔑 Keywords:** `DynamoDB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q2. What is a table?

**💡 Answer:** Explain the DynamoDB data model and access pattern involved, then discuss partitioning, consistency, capacity, scaling, and the main trade-off.

**🔑 Keywords:** `DynamoDB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q3. What is an item?

**💡 Answer:** Explain the DynamoDB data model and access pattern involved, then discuss partitioning, consistency, capacity, scaling, and the main trade-off.

**🔑 Keywords:** `DynamoDB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q4. What is an attribute?

**💡 Answer:** Explain the DynamoDB data model and access pattern involved, then discuss partitioning, consistency, capacity, scaling, and the main trade-off.

**🔑 Keywords:** `DynamoDB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q5. What is a partition key?

**💡 Answer:** A DynamoDB partition key determines the partitioning of items across the table's underlying storage. A well-distributed key is important for avoiding hot partitions.

**🔑 Keywords:** `DynamoDB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q6. What is a sort key?

**💡 Answer:** A DynamoDB sort key is the second component of a composite primary key. Items with the same partition key are ordered by sort key, enabling efficient range and ordered queries.

**🔑 Keywords:** `DynamoDB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q7. What is a composite primary key?

**💡 Answer:** Explain the DynamoDB data model and access pattern involved, then discuss partitioning, consistency, capacity, scaling, and the main trade-off.

**🔑 Keywords:** `DynamoDB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Capacity

#### Q8. What is provisioned capacity?

**💡 Answer:** Provisioned capacity specifies the read and write capacity units you expect a DynamoDB table or index to need. Auto Scaling can adjust provisioned capacity based on utilization.

**🔑 Keywords:** `DynamoDB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q9. What is on-demand capacity?

**💡 Answer:** On-Demand pricing provides flexible pay-as-you-go capacity without a long-term commitment. It is useful for unpredictable workloads, short-lived environments, and workloads where flexibility is more important than the lowest unit cost.

**🔑 Keywords:** `DynamoDB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q10. What is DynamoDB auto scaling?

**💡 Answer:** Amazon DynamoDB is a fully managed NoSQL key-value and document database designed for low-latency performance at scale. Data modeling centers on partition keys and optional sort keys.

**🔑 Keywords:** `DynamoDB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q11. What is read capacity unit?

**💡 Answer:** One read capacity unit represents one strongly consistent read per second for an item up to 4 KB, or two eventually consistent reads per second for an item up to 4 KB.

**🔑 Keywords:** `DynamoDB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q12. What is write capacity unit?

**💡 Answer:** One write capacity unit represents one write per second for an item up to 1 KB.

**🔑 Keywords:** `DynamoDB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Indexes

#### Q13. What is a GSI?

**💡 Answer:** A Global Secondary Index can use a different partition key and optional sort key from the base table. It can be created and managed separately and can span the full table.

**🔑 Keywords:** `DynamoDB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q14. What is an LSI?

**💡 Answer:** A Local Secondary Index uses the same partition key as the base table but a different sort key. It must be defined when the table is created and is limited to the base table's partition-key scope.

**🔑 Keywords:** `DynamoDB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q15. How do GSI and LSI differ?

**💡 Answer:** A Global Secondary Index can use a different partition key and optional sort key from the base table. It can be created and managed separately and can span the full table.

**🔑 Keywords:** `DynamoDB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q16. When should you use a secondary index?

**💡 Answer:** Explain the DynamoDB data model and access pattern involved, then discuss partitioning, consistency, capacity, scaling, and the main trade-off.

**🔑 Keywords:** `DynamoDB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Features

#### Q17. What is DynamoDB Streams?

**💡 Answer:** Amazon DynamoDB is a fully managed NoSQL key-value and document database designed for low-latency performance at scale. Data modeling centers on partition keys and optional sort keys.

**🔑 Keywords:** `DynamoDB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q18. What is TTL?

**💡 Answer:** DNS TTL specifies how long a resolver may cache a DNS answer before querying again. A lower TTL can make changes visible sooner but increases DNS query traffic.

**🔑 Keywords:** `DynamoDB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q19. What is point-in-time recovery?

**💡 Answer:** Point-in-time recovery restores an RDS database to a selected time within the available automated-backup retention window.

**🔑 Keywords:** `DynamoDB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q20. What is Global Tables?

**💡 Answer:** DynamoDB Global Tables provide multi-Region, multi-active replication so applications can read and write in multiple Regions while DynamoDB manages cross-Region replication.

**🔑 Keywords:** `DynamoDB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q21. What is DAX?

**💡 Answer:** DynamoDB Accelerator (DAX) is an in-memory cache designed for DynamoDB workloads. It can reduce read latency for applications that can use the DAX API-compatible interface.

**🔑 Keywords:** `DynamoDB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q22. What is conditional write?

**💡 Answer:** Explain the DynamoDB data model and access pattern involved, then discuss partitioning, consistency, capacity, scaling, and the main trade-off.

**🔑 Keywords:** `DynamoDB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q23. What is optimistic concurrency in DynamoDB?

**💡 Answer:** Amazon DynamoDB is a fully managed NoSQL key-value and document database designed for low-latency performance at scale. Data modeling centers on partition keys and optional sort keys.

**🔑 Keywords:** `DynamoDB` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

## 🚀 Last-Minute Revision

> 🧠 **Remember:** **Partition Key decides where data lives; Sort Key organizes items within the same partition key.**

[⬆️ Back to top](#dynamodb)

[⬅️ Back to AWS Topics](../README.md)