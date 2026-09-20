# 02 — VictoriaMetrics Architecture

## 1. How VictoriaMetrics is structured internally ?

The most important architectural idea is:

```text
Single Node
    ↓
One VictoriaMetrics process handles the main responsibilities

Cluster
    ↓
Responsibilities are separated into:
vminsert
vmstorage
vmselect
```

In cluster mode:

- **vminsert** handles the write/ingestion path.
- **vmstorage** stores time-series data and serves data to queries.
- **vmselect** handles the read/query path.

VictoriaMetrics documents these as the three core services of the cluster architecture. Each service can scale independently. [Official documentation](https://docs.victoriametrics.com/victoriametrics/cluster-victoriametrics/)

---

# 2. Single-Node Architecture

A simple setup can look like this:

```text
Application
     |
  /metrics
     |
     v
Prometheus / vmagent
     |
 Remote Write
     |
     v
+----------------------+
|  VictoriaMetrics     |
|                      |
|  Ingestion           |
|  Storage             |
|  Querying            |
+----------------------+
     |
     v
  Grafana
```

The important point is that there is **one VictoriaMetrics process**.

Conceptually, that process is responsible for:

```text
Receive data
     ↓
Store data
     ↓
Process queries
     ↓
Return results
```

Single-node VictoriaMetrics is simpler to configure and operate than the cluster version and can scale vertically using more CPU, RAM, and storage capacity. [Official documentation](https://docs.victoriametrics.com/victoriametrics/cluster-victoriametrics/)

---

# 3. Why Does VictoriaMetrics Have a Cluster Architecture?

A single machine eventually has physical limits.

For example, suppose a monitoring system becomes very large:

```text
Millions of time series
High ingestion rate
Large retention period
Many simultaneous queries
```

If everything is handled by one machine, all workloads compete for the same resources:

```text
              One Machine
                   |
       +-----------+-----------+
       |           |           |
    Ingestion   Storage      Queries
       |           |           |
       +-----------+-----------+
                   |
               CPU / RAM
                   |
                  Disk
```

This creates a scaling problem.

A cluster separates these responsibilities:

```text
             Write Path
                 |
                 v
             vminsert
                 |
                 v
             vmstorage
                 ^
                 |
             vmselect
                 ^
                 |
             Read Path
```

Now each layer can be scaled according to its workload.

---

# 4. Cluster Architecture

The basic VictoriaMetrics cluster looks like this:

```text
                    Writers
                       |
                       v
                 +-----------+
                 | vminsert  |
                 +-----+-----+
                       |
             +---------+---------+
             |         |         |
             v         v         v
        vmstorage  vmstorage  vmstorage
             |         |         |
             +---------+---------+
                       ^
                       |
                 +-----+-----+
                 | vmselect  |
                 +-----+-----+
                       ^
                       |
                    Grafana
```

The three components have different responsibilities:

```text
vminsert
    ↓
Receives and distributes incoming metrics

vmstorage
    ↓
Stores time-series data

vmselect
    ↓
Receives queries and fetches required data
```

VictoriaMetrics uses a **shared-nothing architecture** for its storage nodes: `vmstorage` nodes don't communicate with each other or share data directly. [Official documentation](https://docs.victoriametrics.com/victoriametrics/cluster-victoriametrics/)

---

# 5. The Three Main Components

## 5.1 vminsert

`vminsert` is the **ingestion/distribution layer**.

Its job is to receive incoming metric data and distribute it among `vmstorage` nodes.

Conceptually:

```text
Prometheus / vmagent
        |
        | metrics
        v
   +-----------+
   | vminsert  |
   +-----+-----+
         |
         +------------+
         |            |
         v            v
   vmstorage-1   vmstorage-2
```

VictoriaMetrics distributes data across `vmstorage` nodes using consistent hashing based on the metric name and all its labels. [Official documentation](https://docs.victoriametrics.com/victoriametrics/cluster-victoriametrics/)

### Think of vminsert as:

> **The traffic distributor for incoming metrics.**

It does not act as the main long-term storage layer.

---

# 6. Why Does vminsert Need to Distribute Data?

Suppose we have:

```text
vmstorage-1
vmstorage-2
vmstorage-3
```

and incoming data contains:

```text
employee_cpu_usage
employee_memory_usage
http_requests_total
http_errors_total
```

Instead of sending every sample to one storage node:

```text
             vminsert
                 |
                 v
          vmstorage-1
```

the cluster distributes the data:

```text
             vminsert
          /      |      \
         v       v       v
      storage storage storage
         1       2       3
```

This allows the storage workload to be spread across multiple nodes.

---

# 7. vmstorage

`vmstorage` is the **storage layer**.

It receives data from `vminsert`, stores the time-series data, and returns requested data to `vmselect`.

Conceptually:

```text
vminsert
    |
    v
+----------------+
|   vmstorage    |
|                |
|  Indexes       |
|  Time-series   |
|  Data          |
|  Compressed    |
|  Storage       |
+----------------+
    |
   Disk
```

Each `vmstorage` node has its own storage.

The storage nodes do not directly share their data with each other. This is part of VictoriaMetrics' shared-nothing cluster architecture. [Official documentation](https://docs.victoriametrics.com/victoriametrics/cluster-victoriametrics/)

### Think of vmstorage as:

> **The place where the actual metric data lives.**

---

# 8. vmselect

`vmselect` is the **query/read layer**.

Grafana or another query client sends a query to `vmselect`.

For example:

```promql
rate(http_requests_total[5m])
```

Conceptually:

```text
Grafana
   |
   | Query
   v
+-----------+
| vmselect  |
+-----+-----+
      |
      +----------+----------+
      |          |          |
      v          v          v
  storage-1  storage-2  storage-3
      |          |          |
      +----------+----------+
                 |
          Query results
                 |
                 v
             vmselect
                 |
                 v
              Grafana
```

`vmselect` fetches the required data from the configured `vmstorage` nodes and processes the query result. [Official documentation](https://docs.victoriametrics.com/victoriametrics/cluster-victoriametrics/)

### Think of vmselect as:

> **The query coordinator.**


---

# 9. Write Path

The write path is the route taken by incoming metrics.

A simplified flow is:

```text
Application
     |
     | /metrics
     v
Prometheus / vmagent
     |
     | Remote Write
     v
vminsert
     |
     | distribute
     v
vmstorage
     |
     v
Disk
```

Let's understand each step.

### Step 1 — Application exposes metrics

For example:

```text
http_requests_total{status="200"} 1500
```

### Step 2 — Prometheus or vmagent collects the metric

```text
Application
     ↓
/metrics
     ↓
Prometheus / vmagent
```

### Step 3 — Data is sent to vminsert

```text
Prometheus / vmagent
          ↓
      vminsert
```

### Step 4 — vminsert distributes the data

```text
             vminsert
          /      |      \
         ↓       ↓       ↓
       VM-1    VM-2     VM-3
```

### Step 5 — vmstorage persists the data

```text
vmstorage
    ↓
Storage
```

This is the **write path**.

---

# 10. Read Path

The read path is the opposite direction.

```text
Grafana
   |
   | Query
   v
vmselect
   |
   +-------> vmstorage-1
   |
   +-------> vmstorage-2
   |
   +-------> vmstorage-3
   |
   v
Combine/process results
   |
   v
Grafana
```

For example:

```promql
sum(rate(http_requests_total[5m]))
```

The conceptual process is:

```text
Grafana
   ↓
Query
   ↓
vmselect
   ↓
Find required data
   ↓
Query vmstorage nodes
   ↓
Receive partial data
   ↓
Process/merge result
   ↓
Return result
   ↓
Grafana
```

---

# 11. Why Do We Separate Write and Read Paths?

This is one of the most important architectural ideas.

Imagine:

```text
100,000 metric samples/sec
```

at the same time as:

```text
5,000 dashboard queries
```

If ingestion and querying are tightly coupled inside one service, both workloads compete for resources.

VictoriaMetrics separates them:

```text
                VictoriaMetrics Cluster

              +----------------+
Writes -----> |   vminsert     |
              +-------+--------+
                      |
                      v
              +----------------+
              |   vmstorage    |
              +----------------+
                      ^
                      |
              +-------+--------+
Queries ----> |   vmselect     |
              +----------------+
```

This separation allows the different workloads to scale independently.

---

# 12. Independent Scaling

This is one of the biggest advantages of the cluster architecture.

Suppose ingestion becomes the bottleneck.

You can add more:

```text
vminsert
```

Conceptually:

```text
             Load Balancer
                  |
        +---------+---------+
        |         |         |
        v         v         v
    vminsert  vminsert  vminsert
```

If storage capacity becomes the bottleneck:

```text
Add vmstorage nodes
```

If query traffic becomes the bottleneck:

```text
Add vmselect nodes
```

VictoriaMetrics explicitly documents independent scaling for `vminsert`, `vmselect`, and `vmstorage`. [Official documentation](https://docs.victoriametrics.com/victoriametrics/cluster-victoriametrics/)

So remember:

```text
High ingestion
      ↓
Add vminsert

More storage
      ↓
Add vmstorage

More query load
      ↓
Add vmselect
```

---

# 13. What Does "Shared Nothing" Mean?

VictoriaMetrics uses a shared-nothing architecture for its storage nodes.

This means:

```text
vmstorage-1
     X
     X  direct shared storage
     X
vmstorage-2
```

They don't directly share the same data.

Instead:

```text
                 vminsert
                /        \
               v          v
         vmstorage-1   vmstorage-2
```

Each storage node manages its own local data.

This approach helps simplify horizontal scaling and maintenance. [Official documentation](https://docs.victoriametrics.com/victoriametrics/cluster-victoriametrics/)

---

# 14. What Happens When a Query Needs Data From Multiple Nodes?

Suppose a query needs data distributed across:

```text
vmstorage-1
vmstorage-2
vmstorage-3
```

`vmselect` sends the appropriate query work to the storage nodes.

Conceptually:

```text
                    Query
                      |
                      v
                  vmselect
                /    |    \
               /     |     \
              v      v      v
            VM-1   VM-2   VM-3
              |      |      |
              +------+------+
                     |
               Partial results
                     |
                     v
                 vmselect
                     |
               Final result
                     |
                     v
                  Grafana
```

This is why `vmselect` is not simply another storage component.

It acts as the **read-side coordination layer**.

---

# 15. Does vmstorage Know About Other vmstorage Nodes?

No.

In the standard cluster architecture:

```text
vmstorage-1  ──X──  vmstorage-2
      │                  │
      └───────X──────────┘
```

`vmstorage` nodes don't communicate directly with one another.

Instead:

```text
vminsert
   ↓
vmstorage nodes

vmselect
   ↓
vmstorage nodes
```

This is the shared-nothing design.

---

# 16. How Does Data Get Distributed?

VictoriaMetrics uses **consistent hashing** for distributing incoming data among `vmstorage` nodes.

The hash is based on:

```text
Metric name
+
All labels
```

Conceptually:

```text
Metric + Labels
       |
       v
   Hash function
       |
       v
Choose storage node
```

Example:

```text
http_requests_total{
    service="employee-api",
    status="200"
}
```

The metric identity is used to determine where the data should be stored.

The important interview point is:

> `vminsert` distributes incoming data across `vmstorage` nodes using consistent hashing based on metric name and labels.

[Official documentation](https://docs.victoriametrics.com/victoriametrics/cluster-victoriametrics/)

---

# 17. What Happens When We Add Another vmstorage Node?

Suppose we start with:

```text
vmstorage-1
vmstorage-2
vmstorage-3
```

Then we add:

```text
vmstorage-4
```

The cluster can expand its storage capacity.

However, adding a storage node is not simply "start the container and forget it." VictoriaMetrics documentation describes updating the `vmselect` and `vminsert` configurations so they know about the new storage node. [Official documentation](https://docs.victoriametrics.com/victoriametrics/cluster-victoriametrics/)

Conceptually:

```text
Before

vminsert
  |
  +--> VM1
  +--> VM2
  +--> VM3


After

vminsert
  |
  +--> VM1
  +--> VM2
  +--> VM3
  +--> VM4
```

The new node increases the available storage/processing capacity.

---

# 18. Can We Add Multiple vminsert Nodes?

Yes.

For example:

```text
                Load Balancer
                 /    |    \
                v     v     v
             Insert Insert Insert
                |     |     |
                +-----+-----+
                      |
                 vmstorage
```

Multiple `vminsert` nodes allow incoming ingestion traffic to be distributed across the ingestion layer.

VictoriaMetrics documents that adding `vminsert` nodes can increase maximum ingestion speed because the workload can be split across more ingestion nodes. [Official documentation](https://docs.victoriametrics.com/victoriametrics/cluster-victoriametrics/)

---

# 19. Can We Add Multiple vmselect Nodes?

Yes.

For example:

```text
                 Load Balancer
                 /    |    \
                v     v     v
             select select select
                |     |     |
                +-----+-----+
                      |
                  vmstorage
```

Multiple `vmselect` nodes allow concurrent query traffic to be distributed across the query layer.

VictoriaMetrics documents that adding `vmselect` nodes can increase the maximum query rate by distributing incoming concurrent requests across more `vmselect` nodes. [Official documentation](https://docs.victoriametrics.com/victoriametrics/cluster-victoriametrics/)

---

# 20. Load Balancer in Front of the Cluster

In a production cluster with multiple `vminsert` or `vmselect` nodes, a load balancer can distribute traffic across those nodes.

Conceptually:

```text
                     Clients
                        |
                        v
                 +-------------+
                 |Load Balancer|
                 +------+------+
                        |
             +----------+----------+
             |                     |
             v                     v
         vminsert              vmselect
             |                     |
             v                     v
        vmstorage nodes      vmstorage nodes
```

VictoriaMetrics documentation recommends a load balancer such as `vmauth` or NGINX when running multiple `vminsert` or `vmselect` nodes. [Official documentation](https://docs.victoriametrics.com/victoriametrics/cluster-victoriametrics/)

---

# 21. Replication — High-Level Concept

Replication is separate from simply having multiple storage nodes.

Without application-level replication, data is distributed across storage nodes.

With:

```text
-replicationFactor=N
```

`vminsert` can store multiple copies of incoming samples on distinct `vmstorage` nodes.

For example, with:

```text
replicationFactor = 2
```

conceptually:

```text
             vminsert
              /    \
             v      v
           VM-1   VM-2
             ^      ^
             |      |
          Copy 1  Copy 2
```

This improves availability if a storage node becomes unavailable, at the cost of additional resource usage.

VictoriaMetrics documents that replication increases CPU, RAM, disk-space, and network requirements because multiple copies are stored. [Official documentation](https://docs.victoriametrics.com/victoriametrics/cluster-victoriametrics/)

Do not confuse:

```text
More vmstorage nodes
```

with:

```text
Replication
```

More nodes provide more places to distribute data.

Replication means **multiple copies of data**.

---

# 22. Cluster Availability

The cluster is designed to remain available when individual components become unavailable, provided the remaining components have sufficient capacity.

For example:

```text
vminsert-1    UP
vminsert-2    DOWN
vminsert-3    UP
```

A load balancer can stop sending traffic to the unavailable node.

Similarly, if a storage node is unavailable, `vminsert` can reroute newly ingested data to healthy storage nodes under the documented cluster behavior.

Exact availability depends on the cluster configuration, remaining capacity, and replication settings. [Official documentation](https://docs.victoriametrics.com/victoriametrics/cluster-victoriametrics/)

---

# 23. End-to-End Write Example

Suppose the application exposes:

```text
http_requests_total{
    service="employee-api",
    status="200"
} 15342
```

The flow is:

```text
1. Application
       |
       | /metrics
       v

2. Prometheus / vmagent
       |
       | Remote Write
       v

3. vminsert
       |
       | consistent hashing
       v

4. vmstorage
       |
       | persist
       v

5. Disk
```

The important thing is that the application does not need to know which `vmstorage` node will ultimately hold the data.

The ingestion layer handles the distribution.

---

# 24. End-to-End Query Example

Suppose Grafana asks:

```promql
rate(http_requests_total[5m])
```

The flow is:

```text
1. Grafana
       |
       v

2. vmselect
       |
       +----------+----------+
       |          |          |
       v          v          v
     VM-1       VM-2       VM-3
       |          |          |
       +----------+----------+
                  |
                  v

3. vmselect processes/combines results
                  |
                  v

4. Grafana
```

So:

```text
Write path:
Client → vminsert → vmstorage

Read path:
Client → vmselect → vmstorage
```

This is the single most important architecture distinction in this README.

---

# 25. Single Node vs Cluster

| Feature | Single Node | Cluster |
|---|---|---|
| Main components | One VM process | vminsert + vmstorage + vmselect |
| Ingestion | VM process | vminsert |
| Storage | VM process | vmstorage |
| Querying | VM process | vmselect |
| Architecture | Simple | Distributed |
| Scaling | Mainly vertical | Horizontal / independent by layer |
| Operational complexity | Lower | Higher |
| Small deployment | Often suitable | Usually unnecessary |
| Large deployment | Limited by one node's resources | Designed for distributed workloads |

VictoriaMetrics itself recommends considering the simpler single-node version carefully before choosing cluster mode, because cluster mode adds operational complexity. [Official documentation](https://docs.victoriametrics.com/victoriametrics/cluster-victoriametrics/)

---

# 26. Single Node vs Cluster — Mental Picture

### Single Node

```text
             VictoriaMetrics
          +------------------+
          |                  |
Writes -->| Ingestion        |
Queries ->| Storage          |
          | Querying         |
          +------------------+
```

One process does the main work.

### Cluster

```text
Writes
  |
  v
vminsert
  |
  v
vmstorage
  ^
  |
vmselect
  ^
  |
Queries
```

Responsibilities are separated.

---

# 27. When Would You Choose Each?

## Single Node

Think about:

```text
Small / medium deployment
Simple architecture
Lower operational overhead
One machine has enough resources
```

Example:

```text
Small Kubernetes cluster
        ↓
vmagent
        ↓
VictoriaMetrics single
        ↓
Grafana
```

---

## Cluster

Think about:

```text
Large metric volume
Large retention requirements
High ingestion
High query concurrency
Need for independent scaling
Need for distributed storage
```

Example:

```text
Many Kubernetes clusters
        ↓
     vmagent
        ↓
    vminsert
        ↓
+-------+-------+
|       |       |
VM1    VM2     VM3
+-------+-------+
        ^
        |
    vmselect
        ^
        |
     Grafana
```

The exact point at which a deployment should move from single-node to cluster depends on workload, capacity, availability, and operational requirements; it should not be decided purely by metric count.

---

# 28. Where This Fits With the Other READMEs

You now have a logical learning sequence:

```text
01 — Fundamentals
        |
        v
What is a time series?
Cardinality?
Churn?
        |
        v
02 — Architecture
        |
        +--> Single Node
        |
        +--> vminsert
        |
        +--> vmstorage
        |
        +--> vmselect
        |
        v
03 — vmagent
        |
        v
How metrics are collected
        |
        v
04 — Storage
        |
        v
How vmstorage stores data
        |
        v
05 — Cardinality & Churn
        |
        v
Why series growth matters
        |
        v
06 — Query Engine
        |
        v
How vmselect processes queries
```

This prevents the topics from becoming mixed together.

---

# 29. Important Interview Questions

## Q1. What are the three main components of VictoriaMetrics Cluster?

**Answer:**

> `vminsert`, `vmstorage`, and `vmselect`.

- `vminsert` handles ingestion and distributes data.
- `vmstorage` stores time-series data.
- `vmselect` handles queries and fetches data from storage nodes.

---

## Q2. What is vminsert?

> vminsert is the ingestion and distribution component of VictoriaMetrics Cluster. It accepts incoming metrics and distributes them across vmstorage nodes using consistent hashing based on metric name and labels.

---

## Q3. What is vmstorage?

> vmstorage is the storage component. It stores time-series data and serves the data requested by vmselect.

---

## Q4. What is vmselect?

> vmselect is the query component. It receives queries, fetches the required data from vmstorage nodes, processes the results, and returns the response to the client.

---

## Q5. What is the write path?

```text
Prometheus / vmagent
        ↓
    vminsert
        ↓
    vmstorage
```

---

## Q6. What is the read path?

```text
Grafana
   ↓
vmselect
   ↓
vmstorage
   ↓
vmselect
   ↓
Grafana
```

---

## Q7. Why are vminsert, vmselect, and vmstorage separated?

> To separate ingestion, storage, and query workloads so each layer can be scaled independently.

---

## Q8. What does shared-nothing mean?

> vmstorage nodes maintain their own local data and do not directly share storage or communicate with one another as peers for normal data handling.

---

## Q9. How does vminsert decide where to send data?

> It uses consistent hashing based on the metric name and all labels to distribute incoming data across vmstorage nodes.

---

## Q10. Can vminsert be scaled independently?

Yes.

```text
More ingestion
     ↓
More vminsert nodes
```

---

## Q11. Can vmselect be scaled independently?

Yes.

```text
More query traffic
       ↓
More vmselect nodes
```

---

## Q12. Can vmstorage be scaled independently?

Yes.

```text
More storage/workload
        ↓
More vmstorage nodes
```

---

# 30. Interview Answer — Short Version

> VictoriaMetrics supports both single-node and cluster architectures. In single-node mode, one VictoriaMetrics process handles ingestion, storage, and querying. In cluster mode, these responsibilities are separated into `vminsert`, `vmstorage`, and `vmselect`. `vminsert` receives incoming metrics and distributes them across `vmstorage` nodes, `vmstorage` stores the time-series data, and `vmselect` handles queries by fetching the required data from the storage nodes. Because these layers are separated, they can be scaled independently according to ingestion, storage, or query requirements.

---

# 31. Revision Notes

## Architecture in 30 Seconds

```text
                 WRITE
                   |
                   v
              vminsert
                   |
                   v
              vmstorage
                   ^
                   |
              vmselect
                   ^
                   |
                  READ
```

### vminsert

```text
Purpose:
Ingestion + distribution
```

Remember:

> **INSERT = Write**

---

### vmstorage

```text
Purpose:
Store time-series data
```

Remember:

> **STORAGE = Data**

---

### vmselect

```text
Purpose:
Query/read path
```

Remember:

> **SELECT = Read**

---

## Write Path

```text
Prometheus / vmagent
        ↓
    vminsert
        ↓
    vmstorage
```

## Read Path

```text
Grafana
   ↓
vmselect
   ↓
vmstorage
   ↓
vmselect
   ↓
Grafana
```

---

## Scaling

```text
Ingestion bottleneck
        ↓
Add vminsert

Storage bottleneck
        ↓
Add vmstorage

Query bottleneck
        ↓
Add vmselect
```

---

## Single Node

```text
One process
   ↓
Ingestion
Storage
Querying
```

## Cluster

```text
vminsert
    ↓
vmstorage
    ↑
vmselect
```

---

## Shared Nothing

```text
VM-1     VM-2     VM-3
 |        |        |
Own      Own      Own
data     data     data
```

Storage nodes don't directly share their data.

---

## Most Important Mental Model

```text
vminsert
    =
WRITE

vmstorage
    =
STORE

vmselect
    =
READ
```

If you remember only one thing from this README:

> **vminsert writes, vmstorage stores, vmselect reads.**
