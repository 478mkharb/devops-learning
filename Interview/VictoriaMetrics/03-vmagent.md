# 03 — vmagent

## 1. What is vmagent?

`vmagent` is a lightweight metrics collection, processing, and forwarding agent from VictoriaMetrics.

Its main job is to sit between metric sources and remote metric storage.

A simple mental model is:

```text
Targets
   |
   | scrape / push
   v
vmagent
   |
   | process
   | relabel
   | filter
   | limit
   | buffer
   v
Remote Storage
```

A common setup is:

```text
Application
    |
 /metrics
    |
    v
 vmagent
    |
Remote Write
    |
    v
VictoriaMetrics
    |
    v
 Grafana
```

`vmagent` can discover and scrape Prometheus-compatible targets, process the collected samples, and send them to VictoriaMetrics or other systems that support the Prometheus `remote_write` protocol. It is designed to use relatively little RAM and CPU compared with a full Prometheus server in many scraping workloads. [Official documentation](https://docs.victoriametrics.com/vmagent/)

---

# 2. Why Does vmagent Exist?

Prometheus is very good at:

```text
Service discovery
        +
Scraping
        +
Local storage
        +
PromQL
        +
Recording rules
        +
Alerting
```

But sometimes we don't want every monitoring node to maintain a large local Prometheus database.

For example:

```text
100 Kubernetes nodes
        |
        v
Each node collects metrics
        |
        v
Central VictoriaMetrics
```

We may want a small component whose primary job is:

```text
Discover
   ↓
Scrape
   ↓
Process
   ↓
Buffer
   ↓
Forward
```

That is where `vmagent` fits.

VictoriaMetrics describes `vmagent` as a lightweight agent that can be used as a Prometheus-compatible scraper and flexible metrics relay. [Official documentation](https://docs.victoriametrics.com/vmagent/)

---

# 3. What Problems Does vmagent Solve?

`vmagent` provides several capabilities in one component.

### 1. Target discovery

It can discover Prometheus-compatible targets.

### 2. Scraping

It can scrape `/metrics` endpoints.

### 3. Relabeling

It can add, remove, or modify labels.

### 4. Filtering

It can drop unwanted metrics or targets.

### 5. Cardinality control

It can limit the number of series produced by targets and written to remote storage.

### 6. Buffering

It can persist unsent data to disk when remote storage is temporarily unavailable.

### 7. Remote Write

It can forward processed metrics to VictoriaMetrics or other compatible remote storage systems.

### 8. Sharding and replication

It can distribute or replicate data across configured remote storage destinations depending on configuration.

These capabilities are part of the reason `vmagent` is useful as a metrics collection and forwarding layer. [Official documentation](https://docs.victoriametrics.com/vmagent/)

---

# 4. Basic Architecture

The simplest architecture is:

```text
+-------------+
| Application |
+------+------+
       |
       | /metrics
       v
+-------------+
|   vmagent   |
+------+------+
       |
       | Remote Write
       v
+----------------------+
| VictoriaMetrics      |
+----------+-----------+
           |
           v
        Grafana
```

Let's understand the responsibilities.

```text
Application
    ↓
Exposes metrics

vmagent
    ↓
Collects + processes + forwards

VictoriaMetrics
    ↓
Stores + queries

Grafana
    ↓
Visualizes
```

This gives us a clean separation:

```text
Collection / Processing
          ↓
       vmagent

Storage / Querying
          ↓
   VictoriaMetrics

Visualization
          ↓
       Grafana
```

---

# 5. vmagent vs Prometheus

This is an important distinction.

## Prometheus

A simplified Prometheus architecture is:

```text
Targets
   |
   v
Prometheus
   |
   +--> Local TSDB
   |
   +--> Remote Write
   |
   +--> Querying
   |
   +--> Rules
   |
   +--> Alerting
```

Prometheus is a complete monitoring server.

---

## vmagent

A simplified vmagent architecture is:

```text
Targets
   |
   v
vmagent
   |
   +--> Relabel
   +--> Filter
   +--> Cardinality control
   +--> Buffer
   |
   v
Remote Storage
```

`vmagent` is primarily designed as a **collection and forwarding layer**, rather than as the full monitoring server that Prometheus is.

---

# 6. Can vmagent Replace Prometheus?

For some scraping/forwarding workloads, yes.

VictoriaMetrics documents `vmagent` as a drop-in replacement for Prometheus for discovering and scraping Prometheus-compatible targets.

However, that does **not** mean that every Prometheus feature has simply moved into vmagent.

Think of the roles like this:

```text
Prometheus
=
Scrape
+ Local Storage
+ Query
+ Rules
+ Alerting

vmagent
=
Scrape
+ Process
+ Buffer
+ Forward
```

For example, if your architecture is:

```text
Targets
   ↓
Prometheus
   ↓
VictoriaMetrics
```

you can potentially use:

```text
Targets
   ↓
vmagent
   ↓
VictoriaMetrics
```

when you want a dedicated collection/forwarding layer.

[Official documentation](https://docs.victoriametrics.com/vmagent/)

---

# 7. Scraping

One of vmagent's main jobs is scraping metrics.

Suppose an application exposes:

```text
http://employee-api:8080/metrics
```

The endpoint might return:

```text
http_requests_total{method="GET",status="200"} 1500
http_requests_total{method="GET",status="500"} 12
process_cpu_seconds_total 345.2
```

vmagent periodically requests this endpoint.

Conceptually:

```text
              HTTP GET
vmagent ------------------> employee-api:8080/metrics
       <------------------
          metric samples
```

Then:

```text
vmagent
   |
   v
Process samples
   |
   v
Remote Write
```

---

# 8. Scrape Configuration

A basic Prometheus-compatible scrape configuration can look like:

```yaml
scrape_configs:
  - job_name: employee-api
    static_configs:
      - targets:
          - employee-api:8080
```

The important part is:

```yaml
targets:
  - employee-api:8080
```

vmagent will scrape the target's Prometheus-compatible metrics endpoint.

Conceptually:

```text
job_name
   |
   v
employee-api
   |
   v
employee-api:8080/metrics
```

---

# 9. Static Targets vs Service Discovery

Targets don't always need to be manually written.

In a small environment:

```yaml
static_configs:
  - targets:
      - employee-api:8080
```

may be enough.

In a dynamic environment such as Kubernetes, targets can be discovered automatically.

Conceptually:

```text
Kubernetes API
       |
       | discover
       v
    vmagent
       |
       | scrape
       v
Pods / Services
```

This is one of the important reasons an agent is useful in dynamic environments.

---

# 10. The vmagent Processing Pipeline

A very important concept is that vmagent doesn't simply do:

```text
scrape → send
```

There can be several processing stages.

A simplified pipeline is:

```text
Pushed / Scraped Samples
          |
          v
Ingestion Rate Limiting
          |
          v
Global Relabeling
          |
          v
Complexity Limits
          |
          v
Cardinality Limits
          |
          v
Stream Aggregation
          |
          v
Per-URL Processing
          |
          v
Persistent Queue
          |
          v
Remote Storage
```

VictoriaMetrics documents this as the sample-processing pipeline, with additional scraping-specific stages such as service-discovery relabeling, scrape relabeling, `sample_limit`, and `series_limit`. [Official documentation](https://docs.victoriametrics.com/vmagent/)

The exact pipeline contains more options than this simplified diagram, but this model is useful for understanding where each feature belongs.

---

# 11. Relabeling

Relabeling means changing or filtering metric labels.

Suppose we receive:

```text
http_requests_total{
    service="employee-api",
    environment="prod",
    request_id="abc123"
}
```

Maybe `request_id` is not useful for long-term monitoring.

We can remove it before sending the data onward.

```text
Before
--------------------------------
http_requests_total{
  service="employee-api",
  environment="prod",
  request_id="abc123"
}

                ↓ relabel

After
--------------------------------
http_requests_total{
  service="employee-api",
  environment="prod"
}
```

This can reduce unnecessary cardinality.

---

# 12. metric_relabel_configs

A scrape configuration can contain:

```yaml
metric_relabel_configs:
  - action: labeldrop
    regex: "request_id|session_id"
```

This means:

```text
For scraped metrics
       ↓
Find labels matching:
request_id
session_id
       ↓
Remove them
```

VictoriaMetrics documents `metric_relabel_configs` as the stage that operates on individual scraped metrics after they have been scraped. [Official documentation](https://docs.victoriametrics.com/victoriametrics/relabeling/)

---

# 13. relabel_configs vs metric_relabel_configs

This distinction is extremely important.

## relabel_configs

Primarily works with **targets before scraping**.

Conceptually:

```text
Discover targets
      ↓
relabel_configs
      ↓
Decide which targets / labels to use
      ↓
Scrape
```

## metric_relabel_configs

Works with **individual samples after scraping**.

```text
Scrape target
      ↓
Metric samples
      ↓
metric_relabel_configs
      ↓
Keep / modify / drop samples
```

So remember:

```text
relabel_configs
=
TARGET relabeling

metric_relabel_configs
=
METRIC relabeling
```

This distinction is documented in VictoriaMetrics' relabeling guide. [Official documentation](https://docs.victoriametrics.com/victoriametrics/relabeling/)

---

# 14. Dropping Entire Metrics

Suppose your application exposes debugging metrics:

```text
debug_cache_hit_ratio
debug_internal_state
debug_temp_value
```

You don't want to store them.

You can drop them using:

```yaml
metric_relabel_configs:
  - source_labels: [__name__]
    regex: "debug_.*"
    action: drop
```

The flow becomes:

```text
Target
  |
  v
Scrape
  |
  +--> debug_cache_hit_ratio
  +--> debug_internal_state
  +--> http_requests_total
  |
  v
metric_relabel_configs
  |
  +--> DROP debug_*
  |
  v
Remote Write
```

This is useful because unnecessary samples don't need to travel to remote storage.

---

# 15. Dropping Labels vs Dropping Metrics

These are different.

### Drop a label

```yaml
action: labeldrop
```

Example:

```text
Before:
http_requests_total{
    service="api",
    request_id="abc"
}

After:
http_requests_total{
    service="api"
}
```

The metric remains.

---

### Drop a metric

```yaml
action: drop
```

Example:

```text
Before:
debug_internal_state 10

After:
Nothing
```

The entire sample is removed.

Remember:

```text
labeldrop
    ↓
Remove label

drop
    ↓
Remove metric/sample
```

---

# 16. Why Relabeling Matters for Cardinality

Consider:

```text
http_requests_total{
    service="api",
    request_id="abc123"
}
```

Suppose every request has a different ID.

Then:

```text
request_id = abc123
request_id = abc124
request_id = abc125
...
```

can create a large number of unique time series.

If we remove:

```text
request_id
```

then many requests can belong to the same series identity.

Conceptually:

```text
Before

request_id
   ↓
Many unique values
   ↓
Many series


After

request_id removed
   ↓
Fewer unique label combinations
   ↓
Lower cardinality
```

This is one of the most useful ways to control cardinality before data reaches the storage backend.

---

# 17. Cardinality Limits

Relabeling is not the only protection.

vmagent also provides cardinality limiting.

There are several different controls, and they solve slightly different problems.

---

## 17.1 series_limit

`series_limit` can be configured inside a scrape configuration.

It limits the number of unique time series that a particular scrape target can expose.

Conceptually:

```text
Target
  |
  | exposes 50,000 series
  v
series_limit = 10,000
  |
  v
Only allowed series are accepted
```

VictoriaMetrics documents `series_limit` as a per-target limit over the relevant 24-hour window. [Official documentation](https://docs.victoriametrics.com/vmagent/)

---

# 18. prom scrape Series Limit

The global command-line option:

```text
-promscrape.seriesLimitPerTarget
```

can set a default per-target series limit.

A `series_limit` value in a specific scrape configuration can override this default.

Conceptually:

```text
Global default
      |
      v
-promscrape.seriesLimitPerTarget
      |
      +----> Target A
      |
      +----> Target B
      |
      +----> Target C
```

Then an individual target can override the default.

---

# 19. __series_limit__

The special:

```text
__series_limit__
```

label can be used to override the series limit for an individual target through relabeling.

This is useful in dynamic environments such as Kubernetes, where different targets may need different limits.

Conceptually:

```text
Target A → 5,000 series
Target B → 20,000 series
Target C → 2,000 series
```

instead of forcing every target to have the same limit.

VictoriaMetrics documents `__series_limit__` as a per-target override mechanism. [Official documentation](https://docs.victoriametrics.com/vmagent/)

---

# 20. maxHourlySeries

The flag:

```text
-remoteWrite.maxHourlySeries
```

limits the number of unique series that vmagent can send to remote storage during the last hour.

This is useful for controlling **series cardinality growth**.

Conceptually:

```text
Remote storage
      ^
      |
  vmagent
      |
      | maxHourlySeries
      |
      v
Limit new unique series
```

When the limit is reached, samples belonging to new series are dropped.

This is a different control from a per-target `series_limit`.

---

# 21. maxDailySeries

The flag:

```text
-remoteWrite.maxDailySeries
```

limits the number of unique series written to remote storage during the last 24 hours.

This is particularly useful for controlling **series churn**.

Think of it like:

```text
maxHourlySeries
      ↓
Protect hourly cardinality

maxDailySeries
      ↓
Protect daily series churn
```

VictoriaMetrics explicitly describes `maxHourlySeries` as useful for limiting active-series cardinality and `maxDailySeries` as useful for limiting daily churn. [Official documentation](https://docs.victoriametrics.com/vmagent/)

---

# 22. Important Correction About "Hourly" and "Daily"

It is tempting to think:

```text
Hourly = active series
Daily = churn
```

as an absolute rule.

A better mental model is:

```text
maxHourlySeries
=
limit unique series registered/written over the last hour

maxDailySeries
=
limit unique series registered/written over the last 24 hours
```

The documentation describes the first as useful for cardinality protection and the second as useful for churn protection.

Both are **series limits**, not general-purpose sample-rate limits.

---

# 23. How the Different Limits Fit Together

Think of the protections at different levels:

```text
                    vmagent
                       |
        +--------------+--------------+
        |                             |
   Per-target                     Global/output
        |                             |
        v                             v
  series_limit              maxHourlySeries
                                  +
                           maxDailySeries
```

Example:

```text
Kubernetes target
       |
       | exposes too many series
       v
series_limit
```

Then:

```text
All targets combined
       |
       v
maxHourlySeries
```

and:

```text
New series created over the day
       |
       v
maxDailySeries
```

---

# 24. Monitoring Cardinality Limits

vmagent exposes metrics that allow you to monitor these limits.

For example:

```text
vmagent_hourly_series_limit_current_series
vmagent_hourly_series_limit_max_series
vmagent_hourly_series_limit_rows_dropped_total
```

and:

```text
vmagent_daily_series_limit_current_series
vmagent_daily_series_limit_max_series
vmagent_daily_series_limit_rows_dropped_total
```

This means you can monitor the protection mechanisms themselves.

For example:

```text
Current series
      /
Maximum series
      =
Utilization
```

You can then alert before the limit becomes a serious problem.

VictoriaMetrics documents these metrics as part of the vmagent cardinality limiter. [Official documentation](https://docs.victoriametrics.com/vmagent/)

---

# 25. Persistent Queue

One of the most useful vmagent features is its **on-disk persistence queue**.

Imagine:

```text
vmagent
   |
   | Remote Write
   X
VictoriaMetrics unavailable
```

Without buffering, data could be lost.

With vmagent's persistent queue:

```text
vmagent
   |
   X remote storage unavailable
   |
   v
Local Disk
   |
   | pending data
   |
   v
Queue
```

When the destination becomes available again:

```text
Local Queue
     |
     v
VictoriaMetrics
```

VictoriaMetrics documents that vmagent stores pending data on disk until it can send the data to the configured remote storage. [Official documentation](https://docs.victoriametrics.com/vmagent/)

---

# 26. Why Is the Queue on Disk?

Suppose:

```text
10 GB
```

of metrics are waiting.

Keeping everything in RAM would be risky.

A process restart could lose the buffered data.

Instead:

```text
Memory
   ↓
Disk
   ↓
Persistent queue
```

The queue therefore provides persistence across vmagent restarts, subject to the configured disk capacity and normal system behavior.

---

# 27. Persistent Queue Location

The location is controlled using:

```text
-remoteWrite.tmpDataPath
```

Conceptually:

```text
vmagent
   |
   v
/tmp or configured path
   |
   v
persistent-queue
```

For example:

```text
/var/lib/vmagent/
```

could be used as the data path in an actual deployment.

The exact directory is a deployment choice.

---

# 28. Limiting Queue Disk Usage

The important flag is:

```text
-remoteWrite.maxDiskUsagePerURL
```

It controls the maximum file-based buffer size for each configured remote-write destination.

Conceptually:

```text
vmagent
   |
   v
Persistent Queue
   |
   | maxDiskUsagePerURL
   v
Disk limit
```

If the queue reaches the configured maximum, vmagent drops the oldest buffered data to make room for newly ingested data.

This means the persistent queue is **not infinite**.

VictoriaMetrics documents this behavior explicitly. [Official documentation](https://docs.victoriametrics.com/vmagent/)

---

# 29. Queue Recovery

Suppose VictoriaMetrics is down:

```text
10:00
vmagent → queue

10:05
vmagent → queue

10:10
vmagent → queue
```

Then VictoriaMetrics comes back:

```text
10:15
queue → VictoriaMetrics
```

The buffered data is sent to the remote storage.

By default, vmagent processes the persistent queue in FIFO order, meaning older queued data is processed before newer data for that destination.

This helps preserve the ordering of buffered data. [Official documentation](https://docs.victoriametrics.com/vmagent/)

---

# 30. What If Remote Storage Is Down for Too Long?

Suppose:

```text
Remote storage unavailable
        |
        v
Queue grows
        |
        v
Disk usage increases
        |
        v
maxDiskUsagePerURL reached
```

At this point vmagent must make room for newly ingested data.

The documented behavior is:

```text
Oldest buffered data
        ↓
Dropped
```

Therefore, the persistent queue protects against **temporary** remote-storage failures, but it does not guarantee unlimited buffering.

---

# 31. Remote Write

After vmagent processes the data:

```text
Scrape
  ↓
Relabel
  ↓
Filter
  ↓
Cardinality controls
  ↓
Queue
  ↓
Remote Write
```

it sends the data to a configured remote storage URL.

For example:

```text
vmagent
   |
   | HTTP Remote Write
   v
VictoriaMetrics
```

The `-remoteWrite.url` flag specifies the destination.

VictoriaMetrics documents that vmagent can send using the Prometheus Remote Write protocol or the VictoriaMetrics Remote Write protocol. [Official documentation](https://docs.victoriametrics.com/vmagent/)

---

# 32. Remote Write to Single-Node VictoriaMetrics

A conceptual setup is:

```text
vmagent
   |
   | Remote Write
   v
VictoriaMetrics
```

The destination may expose a Prometheus-compatible write endpoint.

The exact URL depends on how VictoriaMetrics is deployed.

The important concept is:

```text
vmagent
    =
Producer

VictoriaMetrics
    =
Remote storage consumer
```

---

# 33. Remote Write to VictoriaMetrics Cluster

For a VictoriaMetrics cluster, the destination is normally the **vminsert** layer.

Conceptually:

```text
vmagent
    |
    | Remote Write
    v
vminsert
    |
    v
vmstorage
```

This connects Part 3 with Part 2:

```text
Part 3
vmagent
   ↓
Remote Write
   ↓
Part 2
vminsert
   ↓
vmstorage
```

That is the architecture you should remember.

---

# 34. Multiple Remote Write Destinations

vmagent can be configured with multiple remote-write URLs.

Conceptually:

```text
                 vmagent
                /       \
               /         \
              v           v
         VictoriaVM-1  VictoriaVM-2
```

By default, vmagent can replicate collected data to multiple configured remote storage destinations.

It can also shard outgoing series among configured remote storage systems when `-remoteWrite.shardByURL` is enabled.

[Official documentation](https://docs.victoriametrics.com/vmagent/)

---

# 35. Replication vs Sharding

These are important to distinguish.

## Replication

Same data goes to multiple destinations:

```text
                 vmagent
                /       \
               v         v
             VM-A      VM-B
               \         /
                same data
```

Purpose:

```text
Redundancy / availability
```

---

## Sharding

Different series are distributed across destinations:

```text
                 vmagent
                /       \
               v         v
            VM-A        VM-B

          Series A      Series B
          Series C      Series D
```

Purpose:

```text
Distribute workload
```

VictoriaMetrics supports both behaviors through its remote-write configuration. [Official documentation](https://docs.victoriametrics.com/vmagent/)

---

# 36. End-to-End Example

Suppose we have:

```text
employee-api
salary-api
attendance-api
```

Each exposes:

```text
/metrics
```

Architecture:

```text
 employee-api ----\
                   \
 salary-api -------+--> vmagent
                   /
 attendance-api --/
                       |
                       | scrape
                       v
                  vmagent
                       |
               relabel/filter
                       |
              cardinality limits
                       |
                 persistent queue
                       |
                       | Remote Write
                       v
               VictoriaMetrics
                       |
                       v
                    Grafana
```

Now suppose `employee-api` exposes:

```text
http_requests_total{
    service="employee-api",
    request_id="abc123"
}
```

and:

```text
request_id
```

has huge cardinality.

vmagent can remove it:

```text
Before
--------------------------------
http_requests_total{
  service="employee-api",
  request_id="abc123"
}

After
--------------------------------
http_requests_total{
  service="employee-api"
}
```

Then the cleaned metric is sent to VictoriaMetrics.

---

# 37. vmagent in a Kubernetes Environment

A common architecture is:

```text
                 Kubernetes Cluster
                        |
       +----------------+----------------+
       |                |                |
       v                v                v
    Pod A             Pod B            Pod C
       |                |                |
       +----------------+----------------+
                        |
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

vmagent can use service discovery to find changing targets.

This is particularly useful because Kubernetes workloads are dynamic:

```text
Pod created
Pod deleted
Pod recreated
Pod scaled
Pod moved
```

The monitoring system therefore needs dynamic discovery rather than a permanently hard-coded list of targets.

---

# 38. Why vmagent Is Useful for Kubernetes

Kubernetes creates two major monitoring challenges:

### Dynamic targets

Pods and services change.

### High cardinality

Labels can include:

```text
namespace
pod
container
deployment
service
node
instance
```

and application metrics can introduce additional labels.

vmagent can help by combining:

```text
Service discovery
      +
Scraping
      +
Relabeling
      +
Filtering
      +
Series limits
      +
Persistent queue
```

before sending the metrics to centralized storage.

---

# 39. vmagent and Cardinality Optimization

A practical optimization flow is:

```text
Application
     |
     | /metrics
     v
  vmagent
     |
     | Remove unnecessary labels
     | Drop unnecessary metrics
     | Apply series limits
     v
VictoriaMetrics
```

This is better than allowing obviously unnecessary high-cardinality data to travel all the way into the storage backend.

However, cardinality should ideally also be controlled at the **source/application instrumentation level**.

For example:

```text
Bad design:
user_id on every metric

Better:
aggregate by meaningful dimensions
```

vmagent provides protection and processing, but it should not be treated as an excuse to instrument every possible dynamic value as a label.

---

# 40. Important vmagent Metrics

vmagent exposes its own metrics.

You can monitor things such as:

```text
vmagent_hourly_series_limit_current_series
vmagent_hourly_series_limit_rows_dropped_total
vmagent_daily_series_limit_current_series
vmagent_daily_series_limit_rows_dropped_total
```

You can also monitor remote-write activity and queue behavior.

A useful mental model is:

```text
Don't only monitor your applications.

Monitor vmagent itself.
```

Because vmagent is part of the monitoring pipeline.

If vmagent fails:

```text
Application metrics
       ↓
      X
VictoriaMetrics
```

Your monitoring pipeline may develop gaps.

---

# 41. A Complete vmagent Mental Model

At this point, visualize vmagent as:

```text
                    TARGETS
                       |
                       v
                +-------------+
                |   vmagent   |
                +-------------+
                       |
          +------------+------------+
          |            |            |
          v            v            v
       Discover      Scrape      Receive Push
          |            |            |
          +------------+------------+
                       |
                       v
                 Relabel / Filter
                       |
                       v
               Cardinality Limits
                       |
                       v
               Stream Processing
                       |
                       v
                Persistent Queue
                       |
                       v
                  Remote Write
                       |
                       v
              VictoriaMetrics
```

The core idea is:

> **vmagent is the collection and processing pipeline that prepares metrics before they reach remote storage.**

---

# 42. vmagent vs Prometheus — Detailed Comparison

| Area | Prometheus | vmagent |
|---|---|---|
| Scraping | Yes | Yes |
| Service discovery | Yes | Yes |
| Relabeling | Yes | Yes |
| Metric filtering | Yes | Yes |
| Remote Write | Yes | Yes |
| Local TSDB | Yes | No local Prometheus-style TSDB |
| Query engine | PromQL | Not its primary role |
| Recording rules | Yes | No, use vmalert for this role |
| Alerting rules | Yes | No, use vmalert for this role |
| Persistent remote-write queue | Limited by architecture/use case | Core feature |
| Lightweight collection agent | No | Yes |
| Remote storage relay | Possible | Core use case |

The comparison is intentionally role-based. `vmagent` is not simply "Prometheus with fewer features"; it is designed around efficient collection, processing, buffering, and forwarding. [Official documentation](https://docs.victoriametrics.com/vmagent/)

---

# 43. Important Distinctions

## vmagent vs VictoriaMetrics

```text
vmagent
=
Collect + Process + Forward

VictoriaMetrics
=
Store + Query
```

---

## vmagent vs vminsert

```text
vmagent
=
Collection/processing agent

vminsert
=
VictoriaMetrics cluster ingestion layer
```

Typical flow:

```text
vmagent
   ↓
vminsert
   ↓
vmstorage
```

---

## Relabeling vs Cardinality Limit

```text
Relabeling
=
Change/filter labels and samples

Cardinality limit
=
Protect against too many unique series
```

---

## Cardinality vs Churn

```text
Cardinality
=
How many unique series?

Churn
=
How quickly are new series created?
```

---

# 44. Interview Questions

## Q1. What is vmagent?

> vmagent is a lightweight metrics collection and processing agent that can discover and scrape Prometheus-compatible targets, relabel and filter metrics, apply cardinality limits, buffer unsent data on disk, and forward metrics to VictoriaMetrics or other Remote Write-compatible systems.

---

## Q2. What is the difference between Prometheus and vmagent?

> Prometheus is a complete monitoring server with local TSDB storage, querying, recording rules, and alerting. vmagent is primarily a lightweight collection, processing, buffering, and forwarding component.

---

## Q3. How does vmagent send metrics to VictoriaMetrics?

```text
Target
  ↓
vmagent
  ↓
Remote Write
  ↓
VictoriaMetrics
```

For a VictoriaMetrics cluster:

```text
vmagent
   ↓
Remote Write
   ↓
vminsert
   ↓
vmstorage
```

---

## Q4. What is relabeling?

> Relabeling changes, adds, removes, or filters labels and metric samples before they are sent to remote storage.

---

## Q5. What is metric_relabel_configs?

> It is a relabeling stage that operates on individual metrics after they have been scraped from a target.

---

## Q6. What is the difference between relabel_configs and metric_relabel_configs?

> `relabel_configs` primarily processes discovered targets before scraping, while `metric_relabel_configs` processes the scraped metric samples after the scrape.

---

## Q7. How can vmagent reduce cardinality?

> It can drop unnecessary labels or metrics through relabeling and can enforce series limits such as per-target `series_limit`, `-promscrape.seriesLimitPerTarget`, `-remoteWrite.maxHourlySeries`, and `-remoteWrite.maxDailySeries`.

---

## Q8. What is maxHourlySeries?

> It limits the number of unique series vmagent can send to remote storage during the last hour and is useful for controlling series cardinality.

---

## Q9. What is maxDailySeries?

> It limits the number of unique series sent to remote storage during the last 24 hours and is useful for controlling series churn.

---

## Q10. What happens if VictoriaMetrics becomes unavailable?

> vmagent can buffer pending remote-write data on disk and send it to the remote storage when it becomes available again.

---

## Q11. What controls the persistent queue size?

> `-remoteWrite.maxDiskUsagePerURL` controls the maximum file-based buffer size for each remote-write destination.

---

## Q12. What happens when the queue reaches its limit?

> vmagent drops the oldest buffered data to make room for newly ingested data.

---

## Q13. Can vmagent send to multiple destinations?

> Yes. It can replicate data to multiple remote-write destinations, and it can also shard outgoing series across configured destinations when `-remoteWrite.shardByURL` is enabled.

---

# 45. Interview Answer — Short Version

> `vmagent` is a lightweight metrics collection and processing agent from VictoriaMetrics. It can discover and scrape Prometheus-compatible targets, apply relabeling and filtering, enforce per-target and remote-write series limits, buffer unsent metrics on disk, and forward the processed metrics using Remote Write. In a typical architecture, vmagent sits between applications or exporters and VictoriaMetrics. For a VictoriaMetrics cluster, the flow is usually `targets → vmagent → vminsert → vmstorage`, while queries are handled separately through `vmselect`.

---

# 46. Revision Notes

## vmagent in One Line

> **vmagent collects, processes, protects, buffers, and forwards metrics.**

---

## Basic Flow

```text
Targets
   ↓
vmagent
   ↓
Remote Write
   ↓
VictoriaMetrics
```

---

## vmagent Responsibilities

```text
Discover
Scrape
Relabel
Filter
Limit
Buffer
Forward
```

---

## Scraping

```text
vmagent
   ↓
GET /metrics
   ↓
Target
   ↓
Metric samples
```

---

## Relabeling

```text
Before:
metric + unnecessary labels

        ↓

Relabel

        ↓

After:
cleaner metric
```

---

## Important Relabeling Difference

```text
relabel_configs
      =
TARGET processing

metric_relabel_configs
      =
METRIC processing
```

---

## Drop Label

```yaml
action: labeldrop
```

Removes a label but keeps the metric.

---

## Drop Metric

```yaml
action: drop
```

Removes the sample/metric entirely.

---

## Cardinality Controls

```text
series_limit
      ↓
Per-target series protection

-promscrape.seriesLimitPerTarget
      ↓
Default per-target series protection

-remoteWrite.maxHourlySeries
      ↓
Hourly unique-series protection

-remoteWrite.maxDailySeries
      ↓
Daily unique-series / churn protection
```

---

## Persistent Queue

```text
vmagent
   |
   X
remote storage down
   |
   v
disk queue
   |
   ↓
remote storage recovers
   |
   v
send queued data
```

Main flags:

```text
-remoteWrite.tmpDataPath
-remoteWrite.maxDiskUsagePerURL
```

---

## Cluster Integration

```text
vmagent
    ↓
Remote Write
    ↓
vminsert
    ↓
vmstorage
```

---

## Replication vs Sharding

```text
Replication
=
Same data → multiple destinations

Sharding
=
Different series → different destinations
```

---

## Most Important Mental Model

```text
vmagent
   |
   +--> Discover
   |
   +--> Scrape
   |
   +--> Relabel
   |
   +--> Filter
   |
   +--> Limit cardinality
   |
   +--> Buffer
   |
   +--> Remote Write
   |
   v
VictoriaMetrics
```

### Remember:

> **vmagent is not the database. It is the metric collection and processing layer in front of the database.**
