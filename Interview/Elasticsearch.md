# Elasticsearch Interview Notes

## Table of Contents

1. [Introduction](#1-introduction)
2. [What is Elasticsearch?](#2-what-is-elasticsearch)
3. [What Type of Server is Elasticsearch?](#3-what-type-of-server-is-elasticsearch)
4. [Is Elasticsearch SQL or NoSQL?](#4-is-elasticsearch-sql-or-nosql)
5. [Elasticsearch Architecture](#5-elasticsearch-architecture)
6. [Elasticsearch Data Model](#6-elasticsearch-data-model)
7. [Document, Index, Shard, and Replica](#7-document-index-shard-and-replica)
8. [How Elasticsearch Stores Data](#8-how-elasticsearch-stores-data)
9. [Why is Elasticsearch Fast?](#9-why-is-elasticsearch-fast)
10. [Inverted Index](#10-inverted-index)
11. [Indexing vs Searching](#11-indexing-vs-searching)
12. [Shards and Replicas](#12-shards-and-replicas)
13. [Elasticsearch Persistence and Recovery](#13-elasticsearch-persistence-and-recovery)
14. [What Happens if an Elasticsearch Node Goes Down?](#14-what-happens-if-an-elasticsearch-node-goes-down)
15. [Elasticsearch Use Cases](#15-elasticsearch-use-cases)
16. [Common Elasticsearch Commands](#16-common-elasticsearch-commands)
17. [Frequently Asked Interview Questions](#17-frequently-asked-interview-questions)
18. [Elasticsearch in OT-Microservices](#18-elasticsearch-in-ot-microservices)

---

# 1. Introduction

**Elasticsearch** is an open-source, distributed search and analytics engine designed for fast full-text search, filtering, aggregation, and analysis of large volumes of data.

Elasticsearch is built on top of **Apache Lucene**.

It is commonly used for:

* Full-text search
* Application search
* Log analytics
* Observability
* Metrics analysis
* Security analytics
* Real-time dashboards
* Event search
* Product search
* Centralized logging

Elasticsearch is often used together with:

* Logstash
* Beats
* Kibana
* Elastic Agent

This ecosystem is commonly called the **Elastic Stack**.

---

# 2. What is Elasticsearch?

Elasticsearch is a **distributed search and analytics engine**.

Instead of querying relational tables using traditional SQL joins, Elasticsearch stores data as **JSON documents** inside indexes.

Example document:

```json
{
  "employee_id": 101,
  "name": "Mukesh",
  "department": "DevOps",
  "salary": 85000
}
```

The document can be stored in an index:

```text
employee_index
```

Simplified architecture:

```text
Client
   │
   ▼
Elasticsearch
   │
   ▼
Index
   │
   ▼
JSON Documents
```

Applications communicate with Elasticsearch primarily through its **REST APIs**.

---

# 3. What Type of Server is Elasticsearch?

Elasticsearch is a:

* Distributed search engine
* Distributed analytics engine
* NoSQL document-oriented data store
* RESTful server
* Full-text search engine

Elasticsearch is especially useful when the application needs:

* Fast text search
* Filtering
* Sorting
* Aggregations
* Search across large datasets

---

# 4. Is Elasticsearch SQL or NoSQL?

Elasticsearch is primarily considered a **NoSQL document-oriented system**.

It does not use the traditional relational model of:

```text
Database
   │
   ▼
Tables
   │
   ▼
Rows
   │
   ▼
Columns
```

Instead, Elasticsearch uses:

```text
Cluster
   │
   ▼
Index
   │
   ▼
Documents
   │
   ▼
Fields
```

Example:

```json
{
  "employee_id": 101,
  "name": "Mukesh",
  "department": "DevOps"
}
```

Elasticsearch also provides an SQL interface in supported versions, but its primary native data model and query APIs are document-oriented.

---

# 5. Elasticsearch Architecture

## Cluster

A cluster is a group of Elasticsearch nodes working together.

```text
                Elasticsearch Cluster
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
      Node 1         Node 2         Node 3
```

The cluster provides:

* Data distribution
* Search distribution
* Replication
* Fault tolerance
* Horizontal scaling

## Node

A node is an Elasticsearch server instance.

A node can:

* Store shards
* Execute searches
* Index documents
* Participate in cluster coordination

## Coordinating Node

A node receiving a client request can act as the coordinator.

Example:

```text
Client
   │
   ▼
Coordinating Node
   │
   ├── Shard 1
   ├── Shard 2
   └── Shard 3
```

The coordinating node distributes the request and combines responses.

## Node Roles

Modern Elasticsearch nodes can have different roles, including:

* Master-eligible
* Data
* Ingest
* Coordinating

Depending on the deployment, roles may be combined or separated.

---

# 6. Elasticsearch Data Model

Elasticsearch uses a document-oriented model.

```text
Cluster
   │
   ▼
Index
   │
   ▼
Documents
   │
   ▼
Fields
```

## Document

A document is a JSON object representing one logical record.

Example:

```json
{
  "employee_id": 101,
  "name": "Mukesh",
  "department": "DevOps"
}
```

## Index

An index is a logical collection of documents with similar purposes.

Example:

```text
employee_index
```

It is conceptually similar to a database table, but it is not exactly the same thing.

## Field

A field is an individual attribute inside a document.

```json
{
  "name": "Mukesh"
}
```

Here:

```text
name = field
Mukesh = field value
```

---

# 7. Document, Index, Shard, and Replica

These four concepts are extremely important for Elasticsearch interviews.

## Document

The actual JSON record.

```json
{
  "employee_id": 101,
  "name": "Mukesh"
}
```

## Index

A logical collection of documents.

```text
employee_index
```

## Primary Shard

An index is divided into shards.

```text
employee_index
   │
   ├── Primary Shard 0
   ├── Primary Shard 1
   └── Primary Shard 2
```

Each primary shard contains a portion of the index's documents.

## Replica Shard

A replica is a copy of a primary shard.

```text
Primary Shard 0
       │
       ▼
Replica Shard 0
```

Replicas provide:

* High availability
* Fault tolerance
* Additional search capacity

---

# 8. How Elasticsearch Stores Data

When a document is indexed, Elasticsearch performs several important operations.

Simplified flow:

```text
Client
  │
  ▼
Elasticsearch
  │
  ▼
Select Primary Shard
  │
  ▼
Index Document
  │
  ├── In-memory structures
  │
  └── Translog
        │
        ▼
      Refresh
        │
        ▼
   Searchable Segment
```

Elasticsearch uses Lucene underneath.

Lucene stores indexed data in **immutable segments**.

Segments are periodically merged using segment merging.

---

# 9. Why is Elasticsearch Fast?

Elasticsearch is fast because it uses several techniques.

### Distributed Architecture

Work can be distributed across nodes and shards.

```text
Query
  │
  ├── Shard 1
  ├── Shard 2
  └── Shard 3
```

### Inverted Index

Text fields can be searched efficiently using inverted indexes.

### Query Distribution

Search requests can be executed against multiple shards in parallel.

### Caching

Elasticsearch uses several caching mechanisms to improve repeated operations.

### Near Real-Time Search

Documents become searchable shortly after indexing instead of requiring a full database scan.

---

# 10. Inverted Index

The **inverted index** is one of the most important Elasticsearch concepts.

Suppose we have:

```text
Document 1:
"employee works in DevOps"

Document 2:
"employee works in Finance"

Document 3:
"Mukesh works in DevOps"
```

A simplified inverted index looks like:

```text
Term       Documents
-------------------------
employee   1, 2
works      1, 2, 3
DevOps     1, 3
Finance    2
Mukesh     3
```

Now when we search:

```text
DevOps
```

Elasticsearch can quickly find:

```text
Document 1
Document 3
```

Instead of scanning every document from beginning to end.

This is a major reason Elasticsearch is effective for full-text search.

---

# 11. Indexing vs Searching

## Indexing

Indexing means adding or updating a document in Elasticsearch.

Example:

```bash
curl -X PUT "http://localhost:9200/employee_index/_doc/101" \
-H "Content-Type: application/json" \
-d '{
  "employee_id": 101,
  "name": "Mukesh",
  "department": "DevOps"
}'
```

Simplified flow:

```text
Application
     │
     ▼
Elasticsearch
     │
     ▼
Primary Shard
     │
     ▼
Document Indexed
```

## Searching

Searching retrieves matching documents.

Example:

```bash
curl -X GET "http://localhost:9200/employee_index/_search" \
-H "Content-Type: application/json" \
-d '{
  "query": {
    "match": {
      "department": "DevOps"
    }
  }
}'
```

Simplified flow:

```text
Client
   │
   ▼
Search Request
   │
   ▼
Multiple Shards
   │
   ▼
Collect Results
   │
   ▼
Return Response
```

---

# 12. Shards and Replicas

Suppose:

```text
Index = employee_index
Primary Shards = 3
Replicas = 1
```

The cluster may look conceptually like:

```text
Node 1
├── Primary 0
└── Replica 1

Node 2
├── Primary 1
└── Replica 2

Node 3
├── Primary 2
└── Replica 0
```

The exact placement is controlled by Elasticsearch.

## Primary Shard

Stores the original shard data.

## Replica Shard

Copies the data from the primary shard.

### Why replicas?

If a node fails:

```text
Primary Shard
     │
     ▼
Node Failure
     │
     ▼
Replica promoted
     │
     ▼
Search continues
```

Replicas can also increase search throughput because searches can be served by different shard copies.

---

# 13. Elasticsearch Persistence and Recovery

Elasticsearch persists indexed data to disk.

Important concepts include:

* Lucene segments
* Translog
* Refresh
* Flush
* Merge
* Snapshots

## Translog

The transaction log records operations so Elasticsearch can recover acknowledged changes after a failure before those changes are safely represented in Lucene segments.

Simplified:

```text
Write Request
     │
     ▼
Translog
     │
     ▼
In-memory indexing structures
     │
     ▼
Refresh
     │
     ▼
Lucene Segment
```

## Refresh

A refresh makes recently indexed documents available for search.

This is why Elasticsearch is commonly described as **near real-time**, rather than strictly real-time.

## Flush

A flush helps persist buffered indexing state and manage the translog lifecycle.

## Snapshot

Elasticsearch supports snapshots for backup and disaster recovery.

```text
Elasticsearch
      │
      ▼
Snapshot
      │
      ▼
Snapshot Repository
```

Snapshots are the supported mechanism for backing up an Elasticsearch cluster.

---

# 14. What Happens if an Elasticsearch Node Goes Down?

Suppose:

```text
Primary Shard
     │
     ▼
Node 1
     │
     X
   DOWN
```

If a replica exists on another node:

```text
Node 2
   │
   ▼
Replica Shard
```

Elasticsearch can promote the replica to primary.

Simplified flow:

```text
Node Failure
     │
     ▼
Cluster Detects Failure
     │
     ▼
Replica Promoted
     │
     ▼
New Primary
     │
     ▼
Cluster Recovers
```

The exact recovery depends on cluster health, shard allocation, replica availability, and node roles.

---

# 15. Elasticsearch Use Cases

| Use Case | Description |
|---|---|
| Full-Text Search | Search large amounts of text efficiently |
| Application Search | Add search functionality to applications |
| Log Analytics | Search and analyze application logs |
| Observability | Analyze logs, metrics, and traces |
| Security Analytics | Search security events |
| Product Search | Search e-commerce product catalogs |
| Real-Time Dashboards | Aggregate and visualize incoming data |
| Event Search | Search application or business events |
| Monitoring | Analyze operational data |
| Recommendation Systems | Support search-oriented recommendation workflows |

---

# 16. Common Elasticsearch Commands

## Check Elasticsearch

```bash
curl http://localhost:9200
```

## Cluster Health

```bash
curl "http://localhost:9200/_cluster/health?pretty"
```

## Cluster State

```bash
curl "http://localhost:9200/_cluster/state?pretty"
```

## Node Information

```bash
curl "http://localhost:9200/_nodes?pretty"
```

## List Indices

```bash
curl "http://localhost:9200/_cat/indices?v"
```

## Create an Index

```bash
curl -X PUT "http://localhost:9200/employee_index"
```

## Insert a Document

```bash
curl -X PUT "http://localhost:9200/employee_index/_doc/101" \
-H "Content-Type: application/json" \
-d '{
  "employee_id": 101,
  "name": "Mukesh",
  "department": "DevOps"
}'
```

## Get a Document

```bash
curl "http://localhost:9200/employee_index/_doc/101"
```

## Search Documents

```bash
curl -X GET "http://localhost:9200/employee_index/_search" \
-H "Content-Type: application/json" \
-d '{
  "query": {
    "match": {
      "department": "DevOps"
    }
  }
}'
```

## Match All

```bash
curl -X GET "http://localhost:9200/employee_index/_search" \
-H "Content-Type: application/json" \
-d '{
  "query": {
    "match_all": {}
  }
}'
```

## Count Documents

```bash
curl "http://localhost:9200/employee_index/_count"
```

## Delete a Document

```bash
curl -X DELETE \
"http://localhost:9200/employee_index/_doc/101"
```

## Delete an Index

```bash
curl -X DELETE \
"http://localhost:9200/employee_index"
```

## Shard Allocation

```bash
curl "http://localhost:9200/_cat/shards?v"
```

## Nodes

```bash
curl "http://localhost:9200/_cat/nodes?v"
```

## Recovery

```bash
curl "http://localhost:9200/_cat/recovery?v"
```

---

# 17. Frequently Asked Interview Questions

| Question | Detailed Answer |
|---|---|
| **What is Elasticsearch?** | Elasticsearch is an open-source, distributed search and analytics engine built on Apache Lucene. It is designed for full-text search, filtering, aggregations, and analysis of large datasets. |
| **Is Elasticsearch SQL or NoSQL?** | Elasticsearch is primarily a NoSQL, document-oriented system. It stores JSON documents in indexes. |
| **What is an index?** | An index is a logical collection of documents that are indexed and searched together. |
| **What is a document?** | A document is a JSON object representing one logical record. |
| **What is a shard?** | A shard is a physical Lucene-based partition of an Elasticsearch index. It allows data and work to be distributed across nodes. |
| **What is a replica?** | A replica is a copy of a primary shard used for high availability and additional search capacity. |
| **What is an inverted index?** | An inverted index maps terms to the documents containing them, allowing efficient text search. |
| **Why is Elasticsearch fast?** | Elasticsearch uses inverted indexes, distributed shards, parallel execution, caching, and Lucene's optimized storage and search capabilities. |
| **What is near real-time search?** | Newly indexed documents become searchable after a refresh, so search visibility is typically near real-time rather than instantaneous. |
| **What is a translog?** | The translog records indexing operations to help recover acknowledged changes before they are safely represented in Lucene segments. |
| **What happens when a node fails?** | If replicas are available, Elasticsearch can promote replica shards and recover the affected shard allocation on other nodes. |
| **Can Elasticsearch replace PostgreSQL?** | Usually not. PostgreSQL is a relational transactional database and often acts as the source of truth, while Elasticsearch is commonly used for search and analytics. |
| **Can Elasticsearch replace Redis?** | They solve different problems. Redis is an in-memory data structure store commonly used for caching and low-latency operations, while Elasticsearch specializes in search and analytics. |
| **What is a mapping?** | A mapping defines how fields in Elasticsearch documents are interpreted and indexed, including their data types and indexing behavior. |
| **What is an aggregation?** | An aggregation summarizes or analyzes data, such as counts, averages, sums, buckets, and other metrics. |
| **What is a snapshot?** | A snapshot is a backup of Elasticsearch indices and cluster state stored in a configured snapshot repository. |
| **What is Kibana?** | Kibana is a visualization and user-interface layer used to explore Elasticsearch data, build dashboards, and analyze information. |
| **What is Apache Lucene?** | Lucene is the search library underneath Elasticsearch that provides indexing and search capabilities. |
| **What is cluster health?** | Cluster health indicates the state of the Elasticsearch cluster, commonly shown as green, yellow, or red. |
| **What do green, yellow, and red mean?** | Green means all primary and replica shards are assigned, yellow means all primaries are assigned but some replicas are not, and red means at least one primary shard is unavailable. |

---

# 18. Elasticsearch in OT-Microservices

In the OT-Microservices project:

* **Salary API (Java)** stores salary data in ScyllaDB.
* Salary information can be indexed into Elasticsearch for search and processing.
* The **Notification Worker** reads pending salary records from Elasticsearch.
* Elasticsearch acts as a **search/indexing layer**, not the primary source of truth.

## Data Flow

```text
Salary API
    │
    ▼
 ScyllaDB
    │
    │ Index / Mirror
    ▼
Elasticsearch
    │
    ▼
Notification Worker
    │
    ├── Find pending salary record
    │
    ▼
Generate PDF
    │
    ▼
Send Email
```

## Source of Truth

The architecture should treat:

```text
ScyllaDB
    │
    ▼
SOURCE OF TRUTH
```

while:

```text
Elasticsearch
    │
    ▼
SEARCH / INDEX / PROCESSING
```

If Elasticsearch becomes unavailable, the primary salary data remains in ScyllaDB.

The Elasticsearch index can be rebuilt or re-indexed from the source data when required.

## Example Index

The OT-Microservices environment can use an index such as:

```text
salary_records
```

or, for employee information:

```text
employee_index
```

Example salary document:

```json
{
  "employee_id": "101",
  "employee_name": "Mukesh",
  "month": "September",
  "year": 2026,
  "amount": 85000,
  "status": "PENDING"
}
```

The notification worker can search for pending records:

```json
{
  "query": {
    "match": {
      "status": "PENDING"
    }
  }
}
```

Then:

```text
Elasticsearch
     │
     ▼
Find PENDING salary
     │
     ▼
Notification Worker
     │
     ▼
Generate Salary Slip PDF
     │
     ▼
Send Email
```

This architecture keeps **ScyllaDB as the source of truth** while using Elasticsearch for efficient search and downstream processing.
