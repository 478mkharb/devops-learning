# 🧱 ScyllaDB Architecture

> ⚠️ **Important Distinction**
>
> * **Architecture = HOW Scylla processes data**
> * **Ring Architecture = WHERE data is stored**
>
> Both are related but serve different purposes.

---

## 📌 Overview

ScyllaDB follows a **distributed, peer-to-peer architecture** optimized for modern hardware. It is built using C++ and the Seastar framework to achieve high throughput and low latency.

---

## 🏗️ High-Level Architecture (Processing View)

👉 Focus: *How requests are executed inside the system*

---
><img width="1536" height="1024" alt="ChatGPT Image May 7, 2026, 12_01_37 PM" src="https://github.com/user-attachments/assets/01f682bb-ceff-436a-a7b9-10171bc4edaa" />


---

## 🔑 Core Concepts (Execution Layer)

### 1. Peer-to-Peer Cluster

* No master node
* All nodes are equal
* Any node can act as a **coordinator**

---

### 2. Shard-per-Core Architecture

* Each CPU core = one shard
* Each shard:

  * Handles its own data
  * Processes requests independently
* Eliminates locking and thread contention

```
Node
 ├── Shard 1 (Core 1)
 ├── Shard 2 (Core 2)
 ├── Shard 3 (Core 3)
 └── ...
```

---

### 3. Token Ring Distribution (Connection to Ring Architecture)

👉 This is where architecture connects to **ring topology (data placement)**

* Partition key → hashed → token
* Token decides **which node owns data**
* Architecture then processes request inside that node

```text
Partition Key → Hash → Token → Node (Ring decides)
                          ↓
                    Node processes request (Architecture)
```

---

* Data distributed using **consistent hashing**
* Each node owns a **token range**

```
Partition Key → Hash → Token → Node
```

---

### 4. Coordinator Node

* Receives client request
* Determines target node using token
* Forwards request if needed

---

### 5. Driver Role

* Token-aware routing
* Shard-aware routing
* Load balancing
* Retry & failover handling

---

## 🔄 Request Flow (End-to-End Execution)

### Write Flow

```
Client → Driver → Coordinator → Commit Log → Memtable → SSTable
```

### Read Flow

```
Client → Driver → Coordinator → Cache → Memtable → SSTable
```

---

## ⚙️ Internal Components (Processing Engine)

| Component      | Role                         |
| -------------- | ---------------------------- |
| Seastar Engine | Async execution framework    |
| Shards         | Per-core processing units    |
| Token Ring     | Data distribution mechanism  |
| Coordinator    | Request routing              |
| Storage Engine | Handles memtables & SSTables |
| Raft           | Cluster state consistency    |

---

## 🚀 Key Benefits

* Linear scalability
* Low latency
* High throughput
* No single point of failure
* Efficient CPU utilization

---

# 🔁 Ring vs Architecture (Clear Comparison)

| Aspect           | Architecture                      | Ring Architecture      |
| ---------------- | --------------------------------- | ---------------------- |
| Purpose          | How data is processed             | Where data is stored   |
| Scope            | Full system                       | Data distribution only |
| Focus            | Execution, storage, performance   | Token placement        |
| Components       | Shards, memtables, SSTables, Raft | Tokens, nodes, ranges  |
| Trigger Question | "How does it work?"               | "Where is my data?"    |

---

## 🧠 Combined Flow (Most Important)

```text
Client Request
   ↓
Driver (token-aware)
   ↓
Partition Key → Hash → Token
   ↓
Ring decides Node (WHERE)
   ↓
Architecture processes request (HOW)
   ↓
Shard → Memtable → SSTable
```

---

* Linear scalability
* Low latency
* High throughput
* No single point of failure
* Efficient CPU utilization

---

## 🔑 Summary

ScyllaDB architecture defines **how data is processed inside nodes**, while the ring architecture defines **where data is placed across the cluster**. Together, they enable a scalable and high-performance distributed system.
ScyllaDB architecture is designed to fully utilize modern hardware by combining shard-per-core execution, token-based distribution, and a peer-to-peer model. This results in predictable performance and seamless scalability for distributed workloads.
