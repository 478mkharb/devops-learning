# Redis Interview Notes

## Table of Contents

1. Introduction
2. What is Redis?
3. What Type of Server is Redis?
4. Is Redis SQL or NoSQL?
5. Redis Architecture
6. Redis as Cache
7. How Data is Cached in Redis
8. Cache Hit vs Cache Miss
9. Why is Redis Fast?
10. Is Redis Single-Threaded or Multi-Threaded?
11. Redis Persistence
12. What Happens if Redis Goes Down?
13. Redis Data Structures
14. Redis Use Cases
15. Common Redis Commands
16. Frequently Asked Interview Questions

---

# 1. Introduction

**Redis (Remote Dictionary Server)** is an **open-source, in-memory NoSQL database** designed for ultra-fast data access.

Unlike traditional databases that primarily store data on disk, Redis stores data **in RAM**, making read and write operations extremely fast.

Redis is widely used in modern applications for:

* Caching
* Session Management
* Message Broker
* Pub/Sub Messaging
* Leaderboards
* Rate Limiting
* Distributed Locking
* Real-time Analytics
* Streaming

---

# 2. What is Redis?

Redis is an **In-Memory Data Structure Store**.

It stores data in memory using **key-value pairs** and supports multiple data structures.

```
Key           Value

user:101  ->  Mukesh
salary:101 -> 85000
session:xyz -> JWT Token
```

Redis is commonly deployed **between the application and the database**.

```
Client
   │
   ▼
Application
   │
   ▼
Redis
   │
   ▼
Database
```

---

# 3. What Type of Server is Redis?

Redis is an **In-Memory Data Structure Server**.

It stores all active data in RAM for extremely low latency.

Redis can act as:

* Database
* Cache
* Message Broker
* Streaming Engine
* Session Store

---

# 4. Is Redis SQL or NoSQL?

Redis is a **NoSQL Database**.

It does **not** use:

* Tables
* Rows
* Columns
* SQL Queries
* Joins

Instead, Redis stores data as:

```
Key → Value
```

Example:

```
SET employee:101 "Mukesh"

GET employee:101
```

---

# 5. What Type of NoSQL Database is Redis?

Redis belongs to the **Key-Value Database** category.

Supported Data Structures:

* Strings
* Hashes
* Lists
* Sets
* Sorted Sets
* Streams
* Bitmaps
* HyperLogLogs
* Geospatial Indexes

---

# 6. Why Redis is Used as Cache

Most enterprise applications keep the **primary data** inside databases like:

* MySQL
* PostgreSQL
* ScyllaDB
* Cassandra
* MongoDB

Redis stores **frequently accessed data** temporarily in memory.

```
                 Client
                    │
                    ▼
              Application
                    │
                    ▼
                 Redis Cache
                    │
         Cache Miss │ Cache Hit
                    ▼
               Database
```

Benefits

* Faster response time
* Reduced database load
* Lower latency
* Higher throughput

---

# 7. How Data is Cached in Redis

## Step 1

Client requests Employee 101

```
GET /employee/101
```

↓

Application checks Redis

```
GET employee:101
```

---

## Scenario 1 : Cache Hit

Redis contains the key.

```
Redis

employee:101

↓

Return data immediately
```

Database is **not accessed**.

---

## Scenario 2 : Cache Miss

Redis doesn't contain the key.

```
Redis

↓

Not Found

↓

Application queries Database

↓

Employee Found

↓

SET employee:101

↓

Return Response
```

Future requests are served directly from Redis.

---

## Cache Flow Diagram

```
            Client
               │
               ▼
        Application
               │
               ▼
       Check Redis Cache
               │
      ┌────────┴────────┐
      │                 │
 Cache Hit         Cache Miss
      │                 │
      ▼                 ▼
 Return Data      Query Database
                        │
                        ▼
                 Fetch Data
                        │
                        ▼
                Store in Redis
                        │
                        ▼
                 Return Response
```

---

# 8. Cache Hit vs Cache Miss

## Cache Hit

Data already exists in Redis.

```
Application

↓

Redis

↓

Found

↓

Response
```

Advantages

* Very fast
* No database query
* Low latency

---

## Cache Miss

Data does not exist.

```
Application

↓

Redis

↓

Not Found

↓

Database

↓

Redis

↓

Response
```

---

# 9. Why Redis is Fast?

Redis is fast because:

* Entire dataset is stored in RAM
* No disk I/O for normal operations
* Single-threaded command execution
* Efficient event loop
* Optimized data structures
* Low network overhead

Typical latency:

```
Redis

≈ 0.2 ms

Database

≈ 5–50 ms
```

---

# 10. Is Redis Single-Threaded or Multi-Threaded?

## Redis before Version 6

Command execution:

```
Client

↓

Redis

↓

Single Thread
```

Commands execute sequentially.

Example

```
SET A

↓

GET A

↓

DEL A
```

---

## Redis 6+

Redis introduced multiple threads for:

* Network I/O
* Reading requests
* Writing responses
* Background operations

However,

**Command execution is still single-threaded.**

```
          Clients
             │
             ▼
      Multiple I/O Threads
             │
             ▼
      Main Redis Thread
      Executes Commands
```

Background threads perform:

* RDB Snapshot
* AOF Rewrite
* Replication
* Lazy Free
* Disk I/O

---

# 11. Redis Persistence

Although Redis stores data in memory, it supports persistence.

## RDB Snapshot

```
Memory

↓

Snapshot

↓

dump.rdb
```

Advantages

* Small file
* Fast recovery
* Good backups

Disadvantage

Recent writes between snapshots may be lost.

---

## AOF (Append Only File)

Every write command is recorded.

```
SET employee 101

↓

appendonly.aof
```

During restart:

```
Replay Commands

↓

Restore Memory
```

Advantages

* Better durability
* Minimal data loss

---

# 12. What Happens if Redis Goes Down?

Redis is usually **not the Source of Truth**.

Primary data is stored in databases.

Example:

```
Employee API

↓

Redis Down

↓

ScyllaDB

↓

Employee Data

↓

Redis Restarts

↓

Cache Rebuilt
```

Application continues working.

Only response time becomes slower.

---

# 13. Redis Data Structures

| Data Structure | Description          | Example         |
| -------------- | -------------------- | --------------- |
| String         | Text, Number         | User Name       |
| Hash           | Object               | Employee Record |
| List           | Ordered Collection   | Notifications   |
| Set            | Unique Values        | Roles           |
| Sorted Set     | Ranked Data          | Leaderboard     |
| Stream         | Event Stream         | Logs            |
| Bitmap         | Bit Operations       | User Activity   |
| HyperLogLog    | Approximate Counting | Unique Visitors |
| Geo            | Geographic Data      | Nearby Stores   |

---

# 14. Redis Use Cases Beyond Caching

Redis is much more than a cache.

| Use Case            | Description                    |
| ------------------- | ------------------------------ |
| Cache               | Store frequently accessed data |
| Session Store       | Store login sessions           |
| Message Broker      | Pub/Sub communication          |
| Distributed Locking | Prevent concurrent updates     |
| Rate Limiting       | Limit API requests             |
| Leaderboards        | Gaming rankings                |
| Shopping Cart       | Temporary cart storage         |
| Real-time Analytics | Live dashboards                |
| Queue System        | Task queues                    |
| Notification System | Event processing               |
| Streaming           | Event streams                  |
| Token Store         | Store JWT or OAuth tokens      |

---

# 15. Common Redis Commands

## Connection

```bash
redis-cli

PING
```

---

## String

```bash
SET name Mukesh

GET name

DEL name

EXISTS name

INCR counter

DECR counter
```

---

## Expiration

```bash
EXPIRE otp 60

TTL otp

PERSIST otp
```

---

## Hash

```bash
HSET employee id 101

HSET employee name Mukesh

HGET employee name

HGETALL employee
```

---

## List

```bash
LPUSH employees Mukesh

RPUSH employees Rahul

LRANGE employees 0 -1
```

---

## Set

```bash
SADD devops Mukesh

SMEMBERS devops
```

---

## Sorted Set

```bash
ZADD leaderboard 100 Mukesh

ZRANGE leaderboard 0 -1
```

---

## Transaction

```bash
MULTI

EXEC

WATCH

DISCARD
```

---

## Persistence

```bash
SAVE

BGSAVE
```

---

## Server

```bash
INFO

DBSIZE

FLUSHDB

FLUSHALL
```

---

# 16. Frequently Asked Interview Questions

| Question                                              | Detailed Answer                                                                                                                                                                                                                                                                         |
| ----------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **What is Redis?**                                    | Redis (Remote Dictionary Server) is an open-source, in-memory NoSQL key-value database used as a cache, database, message broker, streaming engine, and session store.                                                                                                                  |
| **What type of server is Redis?**                     | Redis is an **In-Memory Data Structure Server** because it stores data primarily in RAM and supports multiple advanced data structures.                                                                                                                                                 |
| **Is Redis SQL or NoSQL?**                            | Redis is a **NoSQL Key-Value Database**. It does not use tables, rows, columns, SQL queries, or joins.                                                                                                                                                                                  |
| **Why is Redis fast?**                                | Redis stores data in RAM, avoids disk I/O for most operations, uses highly optimized data structures, and executes commands through a single-threaded event loop.                                                                                                                       |
| **Is Redis single-threaded?**                         | Command execution is single-threaded. Redis 6+ uses multiple threads only for network I/O and background operations like persistence and replication.                                                                                                                                   |
| **Is Redis persistent?**                              | Yes. Redis supports persistence using **RDB Snapshots** and **Append Only Files (AOF)**.                                                                                                                                                                                                |
| **Can Redis replace MySQL or ScyllaDB?**              | Generally no. Redis is optimized for speed, whereas databases like MySQL, PostgreSQL, ScyllaDB, or Cassandra provide durable storage and remain the source of truth.                                                                                                                    |
| **What happens if Redis goes down?**                  | Applications usually fall back to the primary database, fetch the required data, and repopulate the Redis cache once it becomes available again.                                                                                                                                        |
| **How does Redis cache data?**                        | The application first checks Redis. If the data exists (Cache Hit), Redis returns it immediately. If not (Cache Miss), the application queries the database, stores the result in Redis (often with a TTL), and returns it to the client.                                               |
| **What is Cache Hit?**                                | The requested data is found in Redis, eliminating the need to query the database.                                                                                                                                                                                                       |
| **What is Cache Miss?**                               | The requested data is not found in Redis, so the application retrieves it from the database, caches it, and returns it to the client.                                                                                                                                                   |
| **What is TTL?**                                      | TTL (Time To Live) defines how long a key remains in Redis before it expires automatically.                                                                                                                                                                                             |
| **Can Redis remove expired data automatically?**      | Yes. Redis automatically deletes expired keys based on their configured TTL.                                                                                                                                                                                                            |
| **What data structures does Redis support?**          | Strings, Hashes, Lists, Sets, Sorted Sets, Streams, Bitmaps, HyperLogLogs, and Geospatial indexes.                                                                                                                                                                                      |
| **What are Redis use cases besides caching?**         | Session management, Pub/Sub messaging, distributed locking, leaderboards, API rate limiting, task queues, shopping carts, real-time analytics, streaming, notifications, and token storage.                                                                                             |
| **Does Redis support transactions?**                  | Yes. Redis supports transactions using **MULTI**, **EXEC**, **WATCH**, and **DISCARD**.                                                                                                                                                                                                 |
| **Can Redis be clustered?**                           | Yes. Redis supports Replication, Sentinel for High Availability, and Redis Cluster for horizontal scaling.                                                                                                                                                                              |
| **Why don't companies use only Redis as a database?** | Redis stores data mainly in memory, which is fast but expensive for very large datasets. Primary databases provide stronger durability, richer querying capabilities, and are better suited as the long-term source of truth, while Redis complements them by accelerating data access. |

---

# Redis in OT-Microservices

In the OT-Microservices project:

* **Employee API (Go)** uses Redis to cache employee information.
* **Attendance API (Python)** uses Redis to cache attendance-related data.
* **Salary API (Java)** can use Redis for frequently accessed salary records.
* The **primary databases** remain:

  * **ScyllaDB** for Employee and Salary data.
  * **PostgreSQL** for Attendance data.

### Request Flow

```text
Client
   │
   ▼
Microservice
   │
   ▼
Redis Cache
   │
Cache Hit? ─────► Yes ─► Return Response
   │
   No
   │
   ▼
Primary Database
   │
   ▼
Store Result in Redis
   │
   ▼
Return Response
```

This architecture reduces latency, lowers database load, and improves overall application performance while ensuring that the databases remain the **source of truth**.
