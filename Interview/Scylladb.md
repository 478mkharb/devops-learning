# ScyllaDB Interview Notes

## Table of Contents

1. [Introduction](#1-introduction)
2. [What is ScyllaDB?](#2-what-is-scylladb)
3. [What Type of Database is ScyllaDB?](#3-what-type-of-database-is-scylladb)
4. [Is ScyllaDB SQL or NoSQL?](#4-is-scylladb-sql-or-nosql)
5. [ScyllaDB Architecture](#5-scylladb-architecture)
6. [ScyllaDB Data Model](#6-scylladb-data-model)
7. [Keyspace, Table, Partition Key, and Clustering Key](#7-keyspace-table-partition-key-and-clustering-key)
8. [How Data is Stored](#8-how-data-is-stored)
9. [Why is ScyllaDB Fast?](#9-why-is-scylladb-fast)
10. [Replication and Consistency](#10-replication-and-consistency)
11. [ScyllaDB Persistence](#11-scylladb-persistence)
12. [What Happens if a Node Goes Down?](#12-what-happens-if-a-node-goes-down)
13. [ScyllaDB Use Cases](#13-scylladb-use-cases)
14. [Common CQL Commands](#14-common-cql-commands)
15. [Frequently Asked Interview Questions](#15-frequently-asked-interview-questions)
16. [ScyllaDB in OT-Microservices](#16-scylladb-in-ot-microservices)

---

# 1. Introduction

**ScyllaDB** is an open-source, distributed NoSQL database designed for high throughput, low latency, and horizontal scalability.

It is compatible with the Cassandra Query Language, commonly called **CQL**.

ScyllaDB is commonly used for:

* Large-scale applications
* Real-time data processing
* User and employee profiles
* IoT workloads
* Time-series data
* Messaging systems
* Recommendation systems
* High-volume transactional workloads

---

# 2. What is ScyllaDB?

ScyllaDB is a **distributed wide-column NoSQL database**.

Unlike traditional relational databases, ScyllaDB does not primarily organize data using joins between normalized tables.

It distributes data across multiple nodes.

```text
Client
   │
   ▼
ScyllaDB Cluster
   │
   ├── Node 1
   ├── Node 2
   └── Node 3
```

ScyllaDB is designed to continue serving requests while the cluster scales or individual nodes experience failures.

---

# 3. What Type of Database is ScyllaDB?

ScyllaDB is a:

* NoSQL database
* Distributed database
* Wide-column database
* Shared-nothing database
* Horizontally scalable database
* Cassandra-compatible database

It is optimized for workloads requiring predictable low latency and high write throughput.

---

# 4. Is ScyllaDB SQL or NoSQL?

ScyllaDB is a **NoSQL database**.

It does not use traditional relational concepts such as:

* Foreign-key joins
* Relational normalization as the primary design model
* Arbitrary SQL joins
* A single centralized database server

Instead, ScyllaDB uses:

```text
Keyspace
   │
   ▼
Table
   │
   ▼
Partition
   │
   ▼
Rows and Columns
```

ScyllaDB uses **CQL**, which looks similar to SQL.

Example:

```sql
SELECT *
FROM employee.employee_info
WHERE employee_id = 101;
```

Although the syntax resembles SQL, query behavior is governed by the partition-key and clustering-key design.

---

# 5. ScyllaDB Architecture

## Cluster

A ScyllaDB cluster is a group of nodes working together.

```text
                 Client
                    │
                    ▼
             ScyllaDB Cluster
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
    Node 1       Node 2       Node 3
       │            │            │
       └────────────┴────────────┘
```

## Node

A node is a ScyllaDB server instance.

Each node:

* Stores a portion of the data
* Accepts client requests
* Participates in replication
* Performs reads and writes
* Can act as a coordinator for requests

## Coordinator Node

The coordinator is the node that receives a client request.

It determines which replicas should handle the request.

```text
Client
   │
   ▼
Coordinator Node
   │
   ├── Replica Node 1
   ├── Replica Node 2
   └── Replica Node 3
```

The coordinator may be any node in the cluster.

---

# 6. ScyllaDB Data Model

ScyllaDB uses a hierarchy similar to:

```text
Cluster
   │
   ▼
Keyspace
   │
   ▼
Table
   │
   ▼
Partition
   │
   ▼
Rows
   │
   ▼
Columns
```

Example:

```text
Cluster
└── employee_keyspace
    └── employee_info
        ├── Partition: employee_id = 101
        ├── Partition: employee_id = 102
        └── Partition: employee_id = 103
```

A table is divided into partitions based on the partition key.

---

# 7. Keyspace, Table, Partition Key, and Clustering Key

## Keyspace

A keyspace is a namespace that contains tables and defines important settings such as replication.

```sql
CREATE KEYSPACE employee
WITH replication = {
    'class': 'NetworkTopologyStrategy',
    'replication_factor': 3
};
```

## Table

A table stores rows and columns.

```sql
CREATE TABLE employee.employee_info (
    employee_id UUID PRIMARY KEY,
    name TEXT,
    department TEXT,
    salary DECIMAL
);
```

## Partition Key

The partition key determines which partition stores a row.

```sql
PRIMARY KEY (employee_id)
```

Here, `employee_id` is the partition key.

## Clustering Key

A clustering key determines the ordering of rows inside a partition.

```sql
PRIMARY KEY ((employee_id), event_time)
```

Here:

* `employee_id` is the partition key.
* `event_time` is the clustering key.

Example:

```sql
CREATE TABLE employee.attendance (
    employee_id UUID,
    event_time TIMESTAMP,
    status TEXT,
    PRIMARY KEY ((employee_id), event_time)
);
```

Rows for the same employee are stored in the same partition and ordered by `event_time`.

---

# 8. How Data is Stored

ScyllaDB uses a distributed storage architecture.

Data is distributed across nodes using partitioning.

```text
Employee 101
     │
     ▼
Partition Hash
     │
     ▼
Token Ring
     │
     ▼
Replica Nodes
```

ScyllaDB uses a token-based distribution model.

The partition key is hashed to determine the token, and the token determines the replica placement.

## Important Design Rule

Queries should normally include the partition key.

Good query:

```sql
SELECT *
FROM employee.employee_info
WHERE employee_id = 101;
```

Potentially expensive or invalid query:

```sql
SELECT *
FROM employee.employee_info
WHERE department = 'DevOps';
```

The second query is not supported efficiently unless the table is designed for it, for example with an appropriate primary key or index.

---

# 9. Why is ScyllaDB Fast?

ScyllaDB is fast because of:

* Shared-nothing architecture
* Horizontal scaling
* Efficient partitioning
* Shard-per-core architecture
* Asynchronous I/O
* Low coordination overhead
* Efficient memory and storage management
* Distributed request processing

## Shard-per-Core Architecture

ScyllaDB is designed to use CPU cores efficiently.

```text
CPU
├── Core 1 → Shard 1
├── Core 2 → Shard 2
├── Core 3 → Shard 3
└── Core 4 → Shard 4
```

A shard generally owns a portion of the data and processing work.

This reduces unnecessary cross-core communication.

---

# 10. Replication and Consistency

## Replication Factor

Replication factor specifies how many copies of data are stored.

```text
Replication Factor = 3
```

means three replicas are maintained for a partition.

```text
Partition A
   ├── Node 1
   ├── Node 2
   └── Node 3
```

## Consistency Level

Consistency level determines how many replicas must respond for an operation to be considered successful.

Common consistency levels include:

* ONE
* QUORUM
* LOCAL_QUORUM
* ALL
* ANY for certain write operations

For a replication factor of 3:

```text
QUORUM = 2 replicas
```

A quorum requires a majority of replicas.

## Stronger vs Lower Consistency

Higher consistency can provide stronger read guarantees but may increase latency or reduce availability during failures.

The correct consistency level depends on the application.

---

# 11. ScyllaDB Persistence

ScyllaDB is a persistent database.

It stores data on disk and uses memory for caching and efficient processing.

Important storage concepts include:

* Commit log
* Memtables
* SSTables
* Compaction

## Write Flow

```text
Client
   │
   ▼
Commit Log
   │
   ▼
Memtable
   │
   ▼
SSTable
```

## Commit Log

The commit log helps protect writes against process or node failure before the data is flushed into SSTables.

## Memtable

A memtable is an in-memory structure holding recently written data.

## SSTable

An SSTable is an immutable on-disk data file.

## Compaction

Compaction merges SSTables and removes obsolete or overwritten data when appropriate.

---

# 12. What Happens if a Node Goes Down?

ScyllaDB is designed for fault tolerance through replication.

Example:

```text
Replication Factor = 3

Partition A
   ├── Node 1
   ├── Node 2
   └── Node 3
```

If Node 2 goes down:

```text
Node 1 → Available
Node 2 → Down
Node 3 → Available
```

The cluster may continue serving requests if the configured consistency level can be satisfied by the remaining replicas.

ScyllaDB can use mechanisms such as:

* Replication
* Hinted handoff
* Repair
* Failure detection
* Consistency levels

The exact behavior depends on the cluster configuration and consistency level.

---

# 13. ScyllaDB Use Cases

| Use Case | Description |
|---|---|
| Employee Management | Store employee profiles |
| Salary Systems | Store salary records |
| IoT | Store high-volume device data |
| Time-Series Data | Store events over time |
| Messaging | Store messages and conversations |
| Recommendation Systems | Store user preferences |
| Real-Time Applications | Support low-latency reads and writes |
| Large-Scale APIs | Handle high request volume |
| Event Storage | Store application events |
| User Profiles | Store account and profile information |

---

# 14. Common CQL Commands

## Connect to ScyllaDB

```bash
cqlsh
```

Connect to a remote node:

```bash
cqlsh <HOST> 9042
```

## Keyspace

```sql
CREATE KEYSPACE employee
WITH replication = {
    'class': 'NetworkTopologyStrategy',
    'replication_factor': 1
};
```

List keyspaces:

```sql
DESCRIBE KEYSPACES;
```

Use a keyspace:

```sql
USE employee;
```

Drop a keyspace:

```sql
DROP KEYSPACE employee;
```

## Table

```sql
CREATE TABLE employee_info (
    employee_id UUID PRIMARY KEY,
    name TEXT,
    department TEXT
);
```

Describe a table:

```sql
DESCRIBE TABLE employee_info;
```

Drop a table:

```sql
DROP TABLE employee_info;
```

## Insert

```sql
INSERT INTO employee_info (
    employee_id,
    name,
    department
)
VALUES (
    00000000-0000-0000-0000-000000000101,
    'Mukesh',
    'DevOps'
);
```

## Read

```sql
SELECT *
FROM employee_info;
```

Read a specific employee:

```sql
SELECT *
FROM employee_info
WHERE employee_id =
00000000-0000-0000-0000-000000000101;
```

## Update

```sql
UPDATE employee_info
SET department = 'Cloud DevOps'
WHERE employee_id =
00000000-0000-0000-0000-000000000101;
```

## Delete

```sql
DELETE FROM employee_info
WHERE employee_id =
00000000-0000-0000-0000-000000000101;
```

## Server Information

```bash
nodetool status
```

```bash
nodetool describecluster
```

```bash
nodetool ring
```

---

# 15. Frequently Asked Interview Questions

| Question | Detailed Answer |
|---|---|
| **What is ScyllaDB?** | ScyllaDB is an open-source, distributed wide-column NoSQL database designed for high throughput, low latency, and horizontal scalability. |
| **Is ScyllaDB SQL or NoSQL?** | ScyllaDB is NoSQL. It uses CQL, which resembles SQL but follows a partition-oriented data model. |
| **What is a keyspace?** | A keyspace is a namespace containing tables and replication configuration. |
| **What is a partition key?** | The partition key determines the partition in which a row is stored and helps distribute data across nodes. |
| **What is a clustering key?** | A clustering key orders rows within a partition. |
| **What is replication factor?** | Replication factor specifies how many copies of each partition are maintained. |
| **Why is ScyllaDB fast?** | ScyllaDB uses shared-nothing architecture, shard-per-core processing, efficient partitioning, asynchronous I/O, and horizontal scaling. |
| **What happens if a node fails?** | Replication allows the cluster to continue serving requests when the configured consistency level can be satisfied by remaining replicas. |
| **What is an SSTable?** | An SSTable is an immutable on-disk file containing persisted database data. |
| **What is compaction?** | Compaction merges SSTables and manages obsolete data to improve storage and read efficiency. |
| **Can ScyllaDB perform joins like PostgreSQL?** | No. ScyllaDB is designed around query-driven tables and does not provide traditional relational joins. |
| **What is CQL?** | CQL stands for Cassandra Query Language. It is used to interact with ScyllaDB. |
| **Can ScyllaDB scale horizontally?** | Yes. Nodes can be added to distribute data and workload across the cluster. |
| **What is a consistency level?** | It defines how many replicas must acknowledge a read or write operation. |
| **What is the difference between ScyllaDB and Redis?** | ScyllaDB is a persistent distributed primary database, while Redis is commonly used as an in-memory cache, key-value store, or low-latency data service. |

---

# 16. ScyllaDB in OT-Microservices

In the OT-Microservices project:

* **Employee API (Go)** uses ScyllaDB for employee information.
* **Salary API (Java)** uses ScyllaDB for salary records.
* Redis may be used as a cache.
* Elasticsearch may be used as a search or indexing layer.

## Data Ownership

```text
Employee API
     │
     ▼
 ScyllaDB
     │
     ▼
Employee Data
```

```text
Salary API
     │
     ▼
 ScyllaDB
     │
     ▼
Salary Records
```

ScyllaDB should remain the **source of truth** for employee and salary data.

Redis can cache frequently accessed data, while Elasticsearch can maintain a searchable mirror or index.

## Request Flow

```text
Client
   │
   ▼
NGINX / Frontend
   │
   ▼
Employee API / Salary API
   │
   ▼
ScyllaDB
   │
   ▼
Response
```

This architecture provides persistent storage, horizontal scalability, and low-latency access for employee and salary workloads.
