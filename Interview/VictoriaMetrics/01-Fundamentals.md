# VictoriaMetrics — Part 1: Fundamentals

## 1. What is VictoriaMetrics?

**VictoriaMetrics (VM)** is a time-series database and monitoring solution designed to store, process, and query large amounts of **time-series data** efficiently.

It is commonly used with monitoring systems such as Prometheus and visualization tools such as Grafana.

At a high level:

```text
Application / Infrastructure
          |
          | Metrics
          v
   Prometheus / vmagent
          |
          | Remote Write
          v
   VictoriaMetrics
          |
          | Query
          v
       Grafana
```

VictoriaMetrics can be used as a long-term storage backend for Prometheus, but it can also perform ingestion and querying itself. VictoriaMetrics provides both **single-node** and **cluster** deployment models.

---

# 2. Why Do We Need VictoriaMetrics?

Before understanding VictoriaMetrics, we need to understand the problem it solves.

A monitoring system continuously generates metrics.

For example:

```text
CPU usage
Memory usage
Disk usage
HTTP requests
HTTP errors
Network traffic
Database connections
Application latency
Kubernetes pod metrics
Container metrics
```

Suppose we monitor:

```text
100 servers
```

and each server exposes:

```text
2,000 time series
```

Then approximately:

```text
100 × 2,000
= 200,000 time series
```

And every time series may receive new samples every few seconds.

Therefore, a monitoring system needs to handle:

- Large numbers of time series
- Continuous ingestion
- High write rates
- Long-term storage
- Efficient queries
- Large retention periods
- Increasing cardinality

This is where a specialized **time-series database (TSDB)** becomes important.

VictoriaMetrics is designed specifically for storing and querying this kind of data.

---

# 3. What is Time-Series Data?

A **time series** is a sequence of values recorded over time.

For example:

```text
Timestamp              CPU Usage

10:00:00               42%
10:00:15               45%
10:00:30               51%
10:00:45               48%
10:01:00               55%
```

The important characteristic is that every value is associated with a **timestamp**.

A monitoring sample can be thought of as:

```text
Metric + Labels + Timestamp + Value
```

For example:

```text
http_requests_total{
    method="GET",
    status="200",
    service="employee-api"
}

Timestamp: 10:00:00
Value:     15342
```

So the database is not simply storing:

```text
15342
```

It is storing a value that belongs to a particular metric, particular label set, and particular point in time.

---

# 4. Understanding a Metric

Consider this metric:

```text
http_requests_total{
    method="GET",
    status="200",
    service="employee-api"
} 15342
```

Let's break it down.

| Component | Meaning |
|---|---|
| `http_requests_total` | Metric name |
| `method="GET"` | HTTP method |
| `status="200"` | HTTP response status |
| `service="employee-api"` | Application/service |
| `15342` | Current sample value |
| Timestamp | When the sample was recorded |

The metric name tells us **what is being measured**.

The labels provide **context** about that measurement.

For example:

```text
http_requests_total
```

alone tells us very little.

But:

```text
http_requests_total{
    service="employee-api",
    method="GET",
    status="500"
}
```

tells us:

> The number of HTTP GET requests returning HTTP 500 from employee-api.

VictoriaMetrics uses a label-based, schemaless data model. Metric names themselves are represented internally as the special `__name__` label.

---

# 5. What Exactly is a Time Series?

This is one of the most important concepts in VictoriaMetrics.

A **time series is uniquely identified by its metric name and complete set of label values**.

For example:

```text
http_requests_total{
    method="GET",
    status="200"
}
```

and:

```text
http_requests_total{
    method="GET",
    status="500"
}
```

are **two different time series**.

Why?

Because their label sets are different.

Conceptually:

```text
Metric name
    +
Complete label set
    =
Unique time series
```

So:

```text
http_requests_total{status="200"}
```

is one series.

```text
http_requests_total{status="500"}
```

is another series.

VictoriaMetrics documentation defines a time series as the combination of a metric name and its labels.

---

# 6. A Practical Example of Multiple Time Series

Suppose our application exposes:

```text
http_requests_total{
    service="employee-api",
    method="GET",
    status="200"
}

http_requests_total{
    service="employee-api",
    method="GET",
    status="500"
}

http_requests_total{
    service="employee-api",
    method="POST",
    status="200"
}

http_requests_total{
    service="employee-api",
    method="POST",
    status="500"
}
```

Although all four metrics have the same metric name:

```text
http_requests_total
```

they represent **four different time series**.

We can visualize them as:

```text
                    http_requests_total
                            |
              +-------------+-------------+
              |                           |
           GET                           POST
              |                           |
         +----+----+                 +----+----+
         |         |                 |         |
       200       500               200       500
```

Therefore:

```text
1 metric name
+
2 methods
+
2 status codes
=
4 unique time series
```

This is the foundation for understanding **cardinality**.

---

# 7. What is Cardinality?

**Cardinality is the number of unique time series.**

For example, if VictoriaMetrics contains:

```text
1,000 unique time series
```

then its cardinality is approximately:

```text
1,000
```

If it contains:

```text
1,000,000 unique time series
```

then cardinality is:

```text
1 million
```

VictoriaMetrics identifies high cardinality as having a large number of active time series, which can increase resource usage.

---

# 8. How Labels Increase Cardinality

Suppose we have:

```text
http_requests_total{
    method="GET"
}
```

and only two methods:

```text
GET
POST
```

Then:

```text
2 possible series
```

Now add:

```text
status
```

with:

```text
200
404
500
```

Now:

```text
2 methods × 3 statuses
= 6 series
```

Add:

```text
environment
```

with:

```text
dev
staging
prod
```

Now:

```text
2 × 3 × 3
= 18 series
```

The important idea is:

```text
More unique label combinations
             ↓
More unique time series
             ↓
Higher cardinality
```

---

# 9. The Dangerous Cardinality Example

Consider this:

```text
http_requests_total{
    user_id="100001"
}
```

Suppose the application has:

```text
1,000,000 users
```

Then this one label could create approximately:

```text
1,000,000 unique series
```

Now imagine additional labels:

```text
method
status
service
region
user_id
```

The number of possible combinations can grow very quickly.

This is why labels such as:

```text
user_id
request_id
session_id
UUID
IP address
highly-variable URL
```

can become dangerous when used without careful consideration.

VictoriaMetrics specifically identifies labels such as `user_id`, `url`, and `ip` as common sources of high cardinality.

---

# 10. Cardinality vs Number of Samples

These two concepts are different.

### Cardinality

Number of **unique time series**.

### Samples

Number of individual metric values stored over time.

Example:

```text
10,000 time series
```

If each series receives one sample every:

```text
15 seconds
```

then each series produces:

```text
4 samples/second
```

Therefore:

```text
10,000 × 4
=
40,000 samples/second
```

So we have:

```text
Cardinality:
10,000 series

Ingestion rate:
40,000 samples/second
```

They are related, but they are **not the same thing**.

---

# 11. What is Churn?

Another important concept is **time-series churn**.

Churn describes **how frequently new time series are created**.

For example, imagine Kubernetes.

A Pod starts:

```text
pod="employee-api-abc123"
```

Later it is deleted.

A new Pod starts:

```text
pod="employee-api-def456"
```

Even if the application is conceptually the same, the label value changed.

That can create a new time series.

So:

```text
Old Pod
   ↓
employee-api-abc123

New Pod
   ↓
employee-api-def456
```

This creates series churn.

VictoriaMetrics defines churn as how frequently a new time series is created and notes that changing Pod names in Kubernetes is a common source of churn.

---

# 12. Cardinality vs Churn

These concepts are often confused.

### Cardinality

Answers:

> How many unique time series do I have?

### Churn

Answers:

> How quickly am I creating new time series?

Example:

```text
System A

1,000,000 stable time series
```

It may have:

```text
High cardinality
Low churn
```

Another system might have:

```text
100,000 active series
```

but continuously create and remove series.

It could have:

```text
Lower cardinality
High churn
```

Therefore:

```text
Cardinality = How many?

Churn = How frequently are new ones created?
```

Churn affects resource requirements, caching efficiency, query performance, and on-disk compression.

---

# 13. Why VictoriaMetrics is Useful

VictoriaMetrics is designed for workloads where the amount of time-series data can become large.

Important characteristics include:

- Efficient time-series storage
- High ingestion performance
- Efficient querying
- Long-term metric retention
- Prometheus-compatible querying
- Single-node deployment
- Cluster deployment
- Support for high-cardinality workloads

VictoriaMetrics can also act as a Prometheus-compatible storage and querying backend.

---

# 14. VictoriaMetrics and Prometheus

Prometheus and VictoriaMetrics are **not necessarily competitors that must be used separately**.

They can work together.

A common architecture is:

```text
                 Application
                     |
                  /metrics
                     |
                     v
                Prometheus
                     |
               Remote Write
                     |
                     v
             VictoriaMetrics
                     |
                   Query
                     |
                     v
                  Grafana
```

Here:

### Prometheus

Primarily handles:

- Service discovery
- Scraping
- Metric collection
- Local metric storage
- PromQL querying
- Recording rules
- Alerting rules

### VictoriaMetrics

Can provide:

- Metric ingestion
- Time-series storage
- Long-term retention
- Querying
- Prometheus-compatible APIs
- Large-scale metric storage

VictoriaMetrics also provides `vmagent` for Prometheus-compatible scraping and `vmalert` for Prometheus-compatible alerting and recording rules.

---

# 15. Prometheus → VictoriaMetrics

The most important concept to understand is:

> Prometheus can collect the metrics, while VictoriaMetrics can provide the storage backend.

For example:

```text
                TARGETS
                   |
                   | scrape
                   v
             +-----------+
             | Prometheus |
             +-----------+
                   |
                   | Remote Write
                   v
          +-------------------+
          | VictoriaMetrics   |
          +-------------------+
                   |
                   | PromQL / MetricsQL
                   v
               Grafana
```

This means you do **not** necessarily need to replace Prometheus to introduce VictoriaMetrics.

---

# 16. Where Does vmagent Fit?

`vmagent` is VictoriaMetrics' lightweight metric collection and forwarding component.

A conceptual architecture is:

```text
Application
    |
    | /metrics
    v
 vmagent
    |
    | Remote Write
    v
VictoriaMetrics
    |
    v
 Grafana
```

So instead of:

```text
Application
     ↓
Prometheus
     ↓
VictoriaMetrics
```

you can have:

```text
Application
     ↓
vmagent
     ↓
VictoriaMetrics
```

The detailed behavior and architecture of `vmagent` will be covered in **Part 3**.

---

# 17. Single-Node VictoriaMetrics

VictoriaMetrics has a **single-node** deployment.

Conceptually:

```text
+--------------------------------+
|     VictoriaMetrics Node       |
|                                |
|  Ingestion                     |
|  Storage                       |
|  Querying                      |
|  APIs                          |
+--------------------------------+
              |
              v
            Disk
```

A single-node deployment runs as one process responsible for ingestion, storage, and querying.

It is particularly useful when you want:

- Simple deployment
- Simple architecture
- Lower operational complexity
- Vertical scaling

VictoriaMetrics documentation notes that the single-node version scales vertically by adding CPU, RAM, disk I/O and disk capacity.

---

# 18. VictoriaMetrics Cluster

For larger deployments, VictoriaMetrics also provides a cluster architecture.

At a high level:

```text
                 Clients
                    |
          +---------+---------+
          |                   |
      vminsert             vmselect
          |                   |
          v                   v
       vmstorage          Query results
          |
       Storage
```

The cluster separates responsibilities into components such as:

```text
vminsert
vmselect
vmstorage
```

The detailed architecture will be covered in **Part 2 — Architecture**.

For now, remember:

```text
Single Node
    =
One VictoriaMetrics process

Cluster
    =
Multiple specialized components
```

---

# 19. VictoriaMetrics in an Observability Stack

VictoriaMetrics is responsible for **metrics**.

A modern observability stack can therefore look like:

```text
                    Observability
                         |
        +----------------+----------------+
        |                |                |
      Metrics           Logs            Traces
        |                |                |
        v                v                v
VictoriaMetrics        Loki             Tempo
        |                |                |
        +----------------+----------------+
                         |
                         v
                      Grafana
```

So:

```text
VictoriaMetrics → Metrics
Loki             → Logs
Tempo            → Traces
Grafana          → Visualization
```

This is an important distinction because VictoriaMetrics is a **time-series/metrics database**, not your general-purpose logs or traces database.

---

# 20. What Happens When a Metric Enters VictoriaMetrics?

Conceptually:

```text
Metric Sample
     |
     v
Metric + Labels
     |
     v
Identify the Time Series
     |
     v
Store the Sample
     |
     v
Persist Time-Series Data
     |
     v
Query Later
```

For example:

```text
http_requests_total{
    service="employee-api",
    status="200"
} 15342
```

VictoriaMetrics needs to associate this sample with the correct time series.

Conceptually:

```text
Metric name
     +
Label set
     |
     v
Unique Time Series
     |
     v
Stored samples over time
```

The detailed internal storage mechanisms are intentionally left for **Part 4 — Storage**.

---

# 21. What Happens When Grafana Queries VictoriaMetrics?

Suppose Grafana asks:

```promql
rate(http_requests_total[5m])
```

Conceptually:

```text
Grafana
   |
   | Query
   v
VictoriaMetrics
   |
   | Find matching series
   v
Read required samples
   |
   v
Calculate query
   |
   v
Return result
   |
   v
Grafana
```

The detailed query execution process will be covered in **Part 6 — Query Engine**.

---

# 22. Why Cardinality Matters to VictoriaMetrics

Imagine two systems.

### System A

```text
100,000 time series
```

with relatively stable labels.

### System B

```text
100,000 time series
```

but continuously creating and removing series.

The second system can be more challenging because churn affects:

- Active series
- Memory usage
- Cache efficiency
- Query performance
- Storage compression

VictoriaMetrics' capacity planning documentation explicitly considers **active time series, ingestion rate, churn rate, query rate, and retention period** when sizing a deployment.

Therefore, when evaluating a VictoriaMetrics deployment, don't look only at:

```text
samples/second
```

Also think about:

```text
Active series
Cardinality
Churn
Query workload
Retention
```

---

# 23. A Complete Mental Model

At this point, you should be able to visualize VictoriaMetrics like this:

```text
                    APPLICATIONS
                         |
                         | Metrics
                         v
                +------------------+
                | Prometheus       |
                | or vmagent       |
                +------------------+
                         |
                         | Remote Write
                         v
                +------------------+
                | VictoriaMetrics  |
                |                  |
                | Ingestion        |
                | Storage          |
                | Querying         |
                +------------------+
                         |
                         | Query
                         v
                     Grafana
```

And conceptually inside the metrics:

```text
Metric
  |
  +---- Labels
  |
  +---- Timestamp
  |
  +---- Value
  |
  v
Time Series
  |
  v
Cardinality
  |
  v
Storage
  |
  v
Query
```

---

# 24. Important Distinctions

## Metric vs Time Series

A metric name:

```text
http_requests_total
```

is not necessarily one time series.

These are different series:

```text
http_requests_total{status="200"}

http_requests_total{status="500"}
```

because the label sets differ.

---

## Cardinality vs Churn

```text
Cardinality
= Number of unique time series

Churn
= Rate at which new time series are created
```

---

## Samples vs Series

```text
Series
= unique metric + label combination

Sample
= value recorded for a series at a particular timestamp
```

---

## Prometheus vs VictoriaMetrics

A simplified mental model:

```text
Prometheus
    =
Collect + Scrape + Evaluate + Query

VictoriaMetrics
    =
Ingest + Store + Query + Retain
```

This is a conceptual simplification, not a strict limitation of either product. VictoriaMetrics can itself provide scraping and alerting-related components through `vmagent` and `vmalert`.

---

# 25. What You Should Understand Before Moving On

You should be comfortable answering these questions:

### Q1. What is VictoriaMetrics?

A time-series database and monitoring solution designed to efficiently ingest, store, and query time-series metrics.

### Q2. What is a time series?

A unique combination of metric name and label set, with samples recorded over time.

### Q3. What is cardinality?

The number of unique time series.

### Q4. What is churn?

The rate at which new time series are created.

### Q5. Why can `user_id` be a dangerous label?

Because a large number of unique user IDs can create a large number of unique time series.

### Q6. Can Prometheus and VictoriaMetrics work together?

Yes. Prometheus can scrape metrics and remote-write them to VictoriaMetrics, which can provide scalable storage and querying.

### Q7. What does Grafana do?

Grafana queries systems such as VictoriaMetrics and visualizes the returned metrics.

### Q8. What is the difference between single-node and cluster VictoriaMetrics?

Single-node runs the core functionality in one process and scales vertically; cluster deployment distributes responsibilities across components and supports horizontal scaling.

---

# 26. Revision Notes

## VictoriaMetrics — One-Minute Revision

### Definition

> **VictoriaMetrics is a time-series database and monitoring solution used to ingest, store, and query large amounts of metric data efficiently.**

---

### Time Series

```text
Metric name + Complete label set
             ↓
        Time Series
```

Example:

```text
http_requests_total{status="200"}
```

is different from:

```text
http_requests_total{status="500"}
```

---

### Metric Sample

```text
Metric
  +
Labels
  +
Timestamp
  +
Value
```

---

### Cardinality

```text
Number of unique time series
```

High-cardinality examples:

```text
user_id
request_id
session_id
UUID
IP
highly-variable URL
```

---

### Churn

```text
How frequently new time series are created
```

Common Kubernetes example:

```text
Pod A → pod="employee-api-abc123"

Pod A deleted

Pod B → pod="employee-api-def456"

New series created
```

---

### Prometheus + VictoriaMetrics

```text
Application
     ↓
Prometheus / vmagent
     ↓
Remote Write
     ↓
VictoriaMetrics
     ↓
Grafana
```

---

### Single Node

```text
One process
    ↓
Ingest
Store
Query
```

---

### Cluster

```text
vminsert
    ↓
vmstorage

vmselect
    ↓
Query
```

---

### Observability Stack

```text
Metrics → VictoriaMetrics
Logs    → Loki
Traces  → Tempo
             ↓
          Grafana
```

---

# 27. Final Mental Model

Remember these six words:

```text
COLLECT
   ↓
INGEST
   ↓
IDENTIFY
   ↓
STORE
   ↓
QUERY
   ↓
VISUALIZE
```

In a typical setup:

```text
Prometheus / vmagent
        ↓
     COLLECT
        ↓
    VictoriaMetrics
        ↓
      INGEST
        ↓
   Identify Series
        ↓
      STORE
        ↓
      QUERY
        ↓
      Grafana
        ↓
    VISUALIZE
```

And the three concepts you absolutely must remember before moving to the next README are:

```text
TIME SERIES
     ↓
CARDINALITY
     ↓
CHURN
```

These concepts are the foundation for understanding **VictoriaMetrics architecture, vmagent, storage, and query performance**.
