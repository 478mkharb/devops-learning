# ScyllaDB Interview Questions & Answers

| **Interview Question**                                        | **Detailed Answer**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **What is ScyllaDB?**                                         | ScyllaDB is an **open-source, distributed, wide-column NoSQL database** designed for extremely high throughput, low latency, and horizontal scalability. It is API-compatible with Apache Cassandra and uses **Cassandra Query Language (CQL)**. It is commonly used in microservices, IoT, financial systems, gaming, and real-time analytics.                                                                                                                                                                          |
| **What type of database is ScyllaDB?**                        | ScyllaDB is a **Wide-Column NoSQL Database**. Unlike relational databases that organize data into tables with relationships, ScyllaDB stores data in partitions and column families. It is optimized for massive datasets distributed across multiple nodes.                                                                                                                                                                                                                                                             |
| **Is ScyllaDB SQL or NoSQL?**                                 | ScyllaDB is a **NoSQL database**. Although it uses **CQL (Cassandra Query Language)**, which looks similar to SQL, it does not support relational features such as JOINs, Foreign Keys, or complex ACID transactions across multiple tables.                                                                                                                                                                                                                                                                             |
| **How is data stored in ScyllaDB?**                           | Data is stored in a sequence of components. When data is written, it is first recorded in the **Commit Log** for durability, then stored in an in-memory **Memtable**. Once the Memtable becomes full, it is flushed to disk as an immutable **SSTable (Sorted String Table)**. During reads, ScyllaDB checks the Memtable first and then the SSTables. Periodically, ScyllaDB performs **Compaction**, which merges multiple SSTables into fewer optimized SSTables to improve read performance and reclaim disk space. |
| **Explain the ScyllaDB write path.**                          | **1. Client sends write request → 2. Commit Log stores the write for durability → 3. Data is written into Memtable (RAM) → 4. When Memtable is full, it is flushed to disk as an SSTable → 5. Compaction later merges SSTables.** This write path provides high write throughput because most writes initially occur in memory.                                                                                                                                                                                          |
| **Explain the ScyllaDB read path.**                           | **1. Client requests data → 2. ScyllaDB checks Memtable → 3. If not found, it checks Bloom Filters to identify relevant SSTables → 4. Reads the required SSTables → 5. Merges the results and returns data to the client.** Bloom Filters help avoid unnecessary disk reads, improving performance.                                                                                                                                                                                                                      |
| **What is a Commit Log?**                                     | The Commit Log is a **durability mechanism**. Every write operation is first appended to the Commit Log before being stored in memory. If the server crashes before the Memtable is flushed, ScyllaDB can recover the data from the Commit Log during restart, preventing data loss.                                                                                                                                                                                                                                     |
| **What is a Memtable?**                                       | A **Memtable** is an **in-memory data structure (RAM)** that temporarily stores incoming writes. Since writing to memory is much faster than writing directly to disk, ScyllaDB achieves very high write performance. Once the Memtable reaches a threshold, it is flushed to disk as an SSTable.                                                                                                                                                                                                                        |
| **What is an SSTable?**                                       | **SSTable (Sorted String Table)** is an immutable file stored on disk containing sorted data. Once created, SSTables are never modified. New updates create new SSTables, and background compaction merges them over time. This design enables fast sequential writes and efficient reads.                                                                                                                                                                                                                               |
| **What is Compaction?**                                       | Compaction is a background process that merges multiple SSTables into fewer larger SSTables. It removes obsolete records, expired tombstones, and duplicate versions of data. Compaction improves read performance and reduces storage fragmentation.                                                                                                                                                                                                                                                                    |
| **What is Ring Topology?**                                    | ScyllaDB uses a **Ring Topology**, where all nodes are arranged logically in a circular ring. Every node is equal (peer-to-peer architecture); there is **no master or slave**. Data is distributed around the ring using **consistent hashing** based on the partition key. This architecture provides automatic load balancing, fault tolerance, and easy horizontal scaling.                                                                                                                                          |
| **How does Ring Topology work?**                              | Every node owns a specific range of **tokens** in the ring. When a row is inserted, ScyllaDB hashes the partition key to generate a token. The token determines which node stores the data. If additional nodes are added, token ranges are redistributed automatically without requiring application changes.                                                                                                                                                                                                           |
| **What is a Partition Key?**                                  | The Partition Key determines **which node in the ring stores the data**. ScyllaDB hashes the Partition Key using consistent hashing to identify the responsible node. A good partition key ensures even data distribution across the cluster.                                                                                                                                                                                                                                                                            |
| **What is a Clustering Column?**                              | Clustering Columns determine the order of rows within a partition. They allow efficient sorting and range queries inside a partition without affecting data distribution across nodes.                                                                                                                                                                                                                                                                                                                                   |
| **How is data distributed across nodes?**                     | ScyllaDB uses **consistent hashing**. The Partition Key is hashed into a token, and that token determines the node responsible for storing the row. Data is automatically balanced across the cluster to prevent hotspots.                                                                                                                                                                                                                                                                                               |
| **What is Replication Factor?**                               | Replication Factor specifies how many copies of each partition are stored across different nodes. For example, a Replication Factor of **3** means every piece of data exists on three different nodes, improving availability and fault tolerance.                                                                                                                                                                                                                                                                      |
| **What happens if a ScyllaDB node fails?**                    | Since data is replicated across multiple nodes, other replicas continue serving read and write requests. Applications continue functioning with minimal disruption, making ScyllaDB highly available.                                                                                                                                                                                                                                                                                                                    |
| **What is Consistency Level?**                                | Consistency Level determines how many replicas must acknowledge a read or write before it is considered successful. Common levels include **ONE**, **QUORUM**, and **ALL**. QUORUM provides a good balance between consistency and availability.                                                                                                                                                                                                                                                                         |
| **Why is ScyllaDB so fast?**                                  | ScyllaDB is written in **C++**, uses a **Shard-per-Core architecture**, asynchronous I/O, lock-free programming, and optimized memory management. Each CPU core manages its own shard independently, eliminating thread contention and maximizing hardware utilization.                                                                                                                                                                                                                                                  |
| **Why is ScyllaDB better than MySQL or PostgreSQL?**          | ScyllaDB is designed for **horizontal scalability**, handling billions of records and millions of operations per second. Adding more nodes increases capacity almost linearly. MySQL and PostgreSQL are excellent relational databases but typically scale vertically and are better suited for transactional workloads requiring joins and ACID guarantees. For high-throughput distributed microservices, ScyllaDB often provides lower latency and better scalability.                                                |
| **When should you choose ScyllaDB over PostgreSQL or MySQL?** | Choose ScyllaDB when you need **very high write throughput, horizontal scaling, distributed architecture, low latency, and high availability**. Choose PostgreSQL or MySQL when you require **complex joins, foreign keys, multi-row ACID transactions, stored procedures, or relational reporting**.                                                                                                                                                                                                                    |
| **Why is ScyllaDB a good choice for Microservices?**          | Each microservice can own its data independently while ScyllaDB provides fast reads, high write throughput, replication, fault tolerance, and horizontal scalability. These characteristics make it well suited for cloud-native distributed systems.                                                                                                                                                                                                                                                                    |
| **How is ScyllaDB used in the OT-Microservices project?**     | In the OT-Microservices architecture, **Employee API** and **Salary API** store their primary data in ScyllaDB. Redis acts as a cache to reduce database load, while Elasticsearch indexes selected data for fast searching. ScyllaDB remains the **source of truth** for Employee and Salary records.                                                                                                                                                                                                                   |
| **What are the advantages of ScyllaDB?**                      | • Horizontal Scaling • High Availability • Fault Tolerance • Very Low Latency • High Throughput • Cassandra Compatibility • No Single Point of Failure • Automatic Data Distribution • Efficient Multi-Core CPU Utilization • Excellent Performance for Distributed Microservices                                                                                                                                                                                                                                        |
| **What is a Bloom Filter?** |	A memory-efficient probabilistic data structure used by ScyllaDB to determine whether a key might exist or definitely does not exist in an SSTable.|
| **Why does ScyllaDB use Bloom Filters?** | 	To avoid unnecessary SSTable disk reads and improve read performance. |

---

# ScyllaDB Internal Write Flow

```text
                Client
                   │
                   ▼
             Write Request
                   │
                   ▼
             Commit Log (Disk)
                   │
                   ▼
             Memtable (RAM)
                   │
        Memtable Full?
           Yes
            │
            ▼
     Flush to SSTable (Disk)
            │
            ▼
     Background Compaction
```

---

# ScyllaDB Read Flow

```text
Client
   │
   ▼
Read Request
   │
   ▼
Check Memtable
   │
Found?
 │     │
Yes    No
 │      │
 ▼      ▼
Return  Bloom Filter
          │
          ▼
     Locate SSTable
          │
          ▼
      Read SSTable
          │
          ▼
    Return Result
```

---

# Ring Topology

```text
           ┌───────────────┐
           │    Node A     │
           └──────┬────────┘
                  │
      ┌───────────┼───────────┐
      │                       │
┌─────▼─────┐           ┌─────▼─────┐
│  Node D   │           │  Node B   │
└─────┬─────┘           └─────┬─────┘
      │                       │
      └───────────┬───────────┘
                  │
           ┌──────▼──────┐
           │   Node C    │
           └─────────────┘

Every node is equal.
No Master.
No Slave.
Data is distributed using Consistent Hashing.
```
