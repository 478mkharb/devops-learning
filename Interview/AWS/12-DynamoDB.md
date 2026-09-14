# DynamoDB

### Q1. What is DynamoDB?

**Answer:** Amazon DynamoDB is a fully managed NoSQL key-value and document database designed for low-latency performance at scale. Data modeling centers on partition keys and optional sort keys.

---

### Q2. What is a table?

**Answer:** A DynamoDB table is a collection of items. The table's primary key determines how items are uniquely identified and partitioned.

---

### Q3. What is an item?

**Answer:** A DynamoDB item is a single record in a table, similar conceptually to a row in a relational database, but it can contain flexible attributes.

---

### Q4. What is an attribute?

**Answer:** A DynamoDB attribute is a named value within an item, similar conceptually to a column value but with a flexible schema.

---

### Q5. What is a partition key?

**Answer:** A DynamoDB partition key determines the logical partitioning of items. A well-distributed key is important for spreading traffic and avoiding hot partitions.

---

### Q6. What is a sort key?

**Answer:** A DynamoDB sort key is the second component of a composite primary key. Items sharing the same partition key are ordered by sort key, enabling efficient range and ordered queries.

---

### Q7. What is a composite primary key?

**Answer:** A DynamoDB composite primary key consists of a partition key and sort key. Together they uniquely identify an item and allow multiple related items to share a partition key.

---

### Q8. What is provisioned capacity?

**Answer:** DynamoDB provisioned capacity specifies expected read and write capacity in capacity units. Auto Scaling can adjust provisioned capacity according to utilization.

---

### Q9. What is on-demand capacity?

**Answer:** DynamoDB on-demand capacity automatically handles changing traffic without requiring advance capacity provisioning. It is useful for unpredictable workloads.

---

### Q10. What is DynamoDB auto scaling?

**Answer:** DynamoDB Auto Scaling adjusts provisioned read and write capacity in response to utilization targets. It helps maintain performance while avoiding excessive over-provisioning.

---

### Q11. What is read capacity unit?

**Answer:** One DynamoDB read capacity unit represents one strongly consistent read per second for an item up to 4 KB, or two eventually consistent reads per second for an item up to 4 KB.

---

### Q12. What is write capacity unit?

**Answer:** One DynamoDB write capacity unit represents one write per second for an item up to 1 KB.

---

### Q13. What is a GSI?

**Answer:** A Global Secondary Index (GSI) can use a different partition key and optional sort key from the base table. It can be created after the table exists and has its own throughput/cost considerations.

---

### Q14. What is an LSI?

**Answer:** A Local Secondary Index (LSI) uses the same partition key as the base table but a different sort key. It must be defined when the table is created and shares the base table's partition-key scope.

---

### Q15. How do GSI and LSI differ?

**Answer:** A GSI can use a different partition key from the base table and can be added after table creation. An LSI must use the same partition key as the base table, changes the sort key, and must be defined when the table is created. Their capacity and consistency characteristics also differ.

---

### Q16. When should you use a secondary index?

**Answer:** Use a DynamoDB secondary index when the application needs efficient queries using an access pattern that the table's primary key cannot support. Choose a GSI when the index needs a different partition key; choose an LSI when the same partition key is retained and the alternate sort key is known at table creation.

---

### Q17. What is DynamoDB Streams?

**Answer:** DynamoDB Streams records item-level changes in near real time. It can trigger Lambda and support event-driven processing, auditing, replication, and downstream workflows.

---

### Q18. What is TTL?

**Answer:** DNS TTL specifies how long a recursive DNS resolver can cache an answer before it needs to query an authoritative server again.

---

### Q19. What is point-in-time recovery?

**Answer:** Point-in-time recovery restores an RDS database to a selected time within the available automated-backup retention window.

---

### Q20. What is Global Tables?

**Answer:** DynamoDB Global Tables provide multi-Region, multi-active replication so applications can read and write in multiple Regions while DynamoDB manages cross-Region replication.

---

### Q21. What is DAX?

**Answer:** DynamoDB Accelerator (DAX) is an in-memory cache designed for DynamoDB workloads. It can reduce read latency for applications that use the DAX-compatible API.

---

### Q22. What is conditional write?

**Answer:** A DynamoDB conditional write succeeds only when a specified condition is true. It is useful for enforcing business rules and preventing conflicting updates.

---

### Q23. What is optimistic concurrency in DynamoDB?

**Answer:** A common optimistic-concurrency pattern stores a version number with each item and uses a conditional write requiring the version to match the expected value. If another writer has already changed the item, the condition fails instead of silently overwriting the newer data.

---
