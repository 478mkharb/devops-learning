# VictoriaMetrics — Interview Questions & Answers

## 01. What is VictoriaMetrics?
**Answer:** VictoriaMetrics is a time-series database and monitoring platform designed to store and query metrics efficiently. It is highly compatible with the Prometheus ecosystem and supports Prometheus Remote Write, Prometheus-compatible scraping, Grafana, and Kubernetes.

## 02. Why is VictoriaMetrics used?
**Answer:** It is used for efficient metric storage and querying, long-term retention, centralized monitoring, scalable metric ingestion, and deployments ranging from simple single-node setups to horizontally scalable clusters.

## 03. What is a time series?
**Answer:** A time series is a sequence of timestamped values identified by a metric name and a unique set of labels.

Example:
```text
http_requests_total{service="employee-api",status="200"}
```

## 04. What are metrics, labels, and series?
**Answer:** A metric represents a measurable value. Labels add dimensions to a metric. A time series is one unique combination of metric name and label values.

## 05. What is cardinality?
**Answer:** Cardinality is the number of unique time series. Highly dynamic labels such as request IDs can create very high cardinality.

## 06. What is churn?
**Answer:** Churn is how frequently time series are created and removed. Cardinality asks “how many series exist?” while churn asks “how frequently do series change?”

## 07. What are the deployment models of VictoriaMetrics?
**Answer:** The major models are Single-Node VictoriaMetrics and VictoriaMetrics Cluster. Single-node is simpler; cluster separates ingestion, storage, and querying.

## 08. What is Single-Node VictoriaMetrics?
**Answer:** A single VictoriaMetrics process handles ingestion, storage, and querying. A common HTTP port is `8428`.

## 09. What is VictoriaMetrics Cluster?
**Answer:** The cluster separates responsibilities into `vminsert` for ingestion, `vmstorage` for storage, and `vmselect` for querying. This allows independent horizontal scaling.

## 10. What is vminsert?
**Answer:** `vminsert` is the ingestion layer. It receives incoming metrics and distributes them to `vmstorage` nodes. Memory trick: **vminsert = WRITE**.

## 11. What is vmstorage?
**Answer:** `vmstorage` is the storage layer. It stores time-series data and serves data required by queries. Memory trick: **vmstorage = STORE**.

## 12. What is vmselect?
**Answer:** `vmselect` is the query layer. It receives queries, queries the required `vmstorage` nodes, processes the results, and returns them. Memory trick: **vmselect = READ / QUERY**.

## 13. Explain the cluster write path.
**Answer:**
```text
Prometheus / vmagent
        |
        v
     vminsert
        |
   +----+----+
   |    |    |
   v    v    v
 vmstorage nodes
```

## 14. Explain the cluster read path.
**Answer:**
```text
Grafana
   |
   v
vmselect
   |
   +--> vmstorage-1
   +--> vmstorage-2
   +--> vmstorage-3
   |
   v
Result
```

## 15. What is vmagent?
**Answer:** `vmagent` is a lightweight metrics collection, processing, buffering, and forwarding agent. It can scrape targets, receive supported metrics, relabel/filter data, apply cardinality controls, perform stream aggregation, buffer data, and forward metrics.

## 16. What is scraping?
**Answer:** Scraping is collecting metrics from a target, commonly through its `/metrics` HTTP endpoint.
```text
vmagent --GET /metrics--> Node Exporter
```

## 17. What is relabeling?
**Answer:** Relabeling modifies or filters target information and metric labels. It can add, change, remove, or rewrite labels and can drop targets before scraping.

## 18. What is metric filtering?
**Answer:** Filtering selects which targets, metrics, or labels continue through the pipeline. It can reduce unnecessary ingestion, storage, and cardinality.

## 19. What are cardinality limits in vmagent?
**Answer:** vmagent provides limits that protect against excessive series creation, including `series_limit`, `-promscrape.seriesLimitPerTarget`, `__series_limit__`, `-remoteWrite.maxHourlySeries`, and `-remoteWrite.maxDailySeries`.

## 20. What is deduplication?
**Answer:** Deduplication removes duplicate samples representing the same data. It is different from aggregation: deduplication removes duplicates, while aggregation calculates a result from samples.

## 21. What is stream aggregation?
**Answer:** Stream aggregation calculates aggregated metric results while samples are flowing through vmagent or VictoriaMetrics, before the resulting data is written to storage. It can reduce stored samples or series and can perform operations such as sums, counts, quantiles, and histogram-related aggregation.

```text
Incoming Metrics
      |
    vmagent
      |
Stream Aggregation
      |
Aggregated Metrics
      |
VictoriaMetrics
```

## 22. What are `by` and `without` in stream aggregation?
**Answer:** `by` specifies labels used to group the aggregation. `without` specifies labels excluded from the grouping. For example, `without: [instance]` can combine data across instances while retaining other grouping dimensions.

## 23. Stream aggregation vs recording rules?
**Answer:** Stream aggregation processes incoming samples before storage. Recording rules evaluate queryable data and create precomputed metrics.
```text
Stream: Incoming -> Aggregate -> Storage
Rule:   Storage -> Query/Rule -> Recorded metric
```

## 24. What is a persistent queue?
**Answer:** A persistent queue allows vmagent to buffer data on disk when remote storage is unavailable or slower than the incoming rate. This helps absorb temporary remote-storage problems.

## 25. What is Prometheus Remote Write?
**Answer:** It is a protocol used to send metrics from Prometheus-compatible collectors to remote metrics storage.
```yaml
remote_write:
  - url: http://victoriametrics:8428/api/v1/write
```
For a cluster, the destination normally targets `vminsert`.

## 26. What is VictoriaMetrics storage?
**Answer:** VictoriaMetrics stores time-series samples on disk using a time-series-optimized storage engine. Storage includes time-series data, indexes, metadata, and compressed structures.

## 27. What is indexing?
**Answer:** Indexing allows VictoriaMetrics to efficiently identify time series matching metric and label selectors before retrieving their samples.

## 28. How does compression help?
**Answer:** Compression reduces disk usage and disk I/O. Time-series data often contains patterns that can be compressed efficiently.

## 29. What is retention?
**Answer:** Retention defines how long metric data is kept. For example, `-retentionPeriod=30d` configures a 30-day retention period. Longer retention generally requires more storage.

## 30. What is downsampling?
**Answer:** Downsampling reduces the resolution of retained data by keeping less-frequent or aggregated samples. Retention removes old data; downsampling reduces the resolution of data that remains.

## 31. What is MetricsQL?
**Answer:** MetricsQL is VictoriaMetrics' query language. It is compatible with PromQL and provides additional query capabilities.
```promql
rate(http_requests_total[5m])
```

## 32. What is VMUI?
**Answer:** VMUI is VictoriaMetrics' built-in web interface for running queries, exploring metrics, inspecting time series, and troubleshooting.

## 33. What is vmalert?
**Answer:** `vmalert` evaluates Prometheus-compatible alerting and recording rules against a metrics datasource. It can generate alerts and recording-rule results.

## 34. What are recording rules?
**Answer:** Recording rules precompute frequently used queries and store the results as new time series.
```yaml
- record: api:http_requests_rate5m
  expr: sum(rate(http_requests_total[5m]))
```

## 35. What are alerting rules?
**Answer:** Alerting rules evaluate an expression and generate an alert when a condition remains true for the configured duration.
```yaml
- alert: HighCPU
  expr: cpu_usage > 80
  for: 5m
```

## 36. What is the VictoriaMetrics Operator?
**Answer:** It is a Kubernetes operator used to manage VictoriaMetrics components and monitoring resources declaratively. Common resources include `VMCluster`, `VMSingle`, `VMAgent`, `VMServiceScrape`, `VMPodScrape`, `VMRule`, and `VMAlert`.

## 37. What is VMCluster?
**Answer:** `VMCluster` is a Kubernetes custom resource used by the VictoriaMetrics Operator to manage a VictoriaMetrics cluster built around `vminsert`, `vmstorage`, and `vmselect`.

## 38. What is VMAgent?
**Answer:** `VMAgent` is the Operator resource used to configure and manage a vmagent deployment in Kubernetes, including metric discovery, scraping, processing, and forwarding.

## 39. What is VMServiceScrape?
**Answer:** `VMServiceScrape` defines how Kubernetes Services should be scraped by VMAgent.

## 40. What is VMPodScrape?
**Answer:** `VMPodScrape` defines scraping configuration for Kubernetes Pods directly, based on Pod metadata, ports, paths, and related settings.

## 41. What is multitenancy?
**Answer:** VictoriaMetrics Cluster supports multitenancy using account or tenant identifiers. A write URL can contain an account ID, for example:
```text
/insert/0/prometheus/api/v1/write
```

## 42. What is vmauth?
**Answer:** `vmauth` is an authorization proxy and load balancer for VictoriaMetrics components. It can provide authentication, authorization, request routing, and load balancing.

## 43. How are VictoriaMetrics backups performed?
**Answer:** VictoriaMetrics provides `vmbackup`, `vmrestore`, and `vmbackupmanager`. Backups should be validated by testing restoration, not only by checking whether backup files exist.

## 44. What is vmctl?
**Answer:** `vmctl` is a utility for migrating or copying metrics between supported storage systems.

## 45. What happens when a VictoriaMetrics component fails?
**Answer:** Failure behavior depends on the deployment and configuration. Cluster components can be deployed as multiple instances, but availability depends on replication, routing, and which component failed.

## 46. How does VictoriaMetrics provide high availability?
**Answer:** High availability can be achieved by running multiple instances of relevant cluster components and configuring appropriate replication and routing. Multiple `vminsert`, `vmselect`, and storage instances can be used according to the required architecture.

## 47. What is a multi-cluster monitoring architecture?
**Answer:** It centralizes metrics from multiple Kubernetes or infrastructure environments into a common VictoriaMetrics deployment.
```text
Cluster A -> vmagent --+
Cluster B -> vmagent --+--> Central VictoriaMetrics
Cluster C -> vmagent --+
```

# 🔥 Important Interview Questions

## 48. Cardinality vs churn?
**Answer:**
```text
Cardinality = number of unique time series
Churn       = rate at which time series are created and removed
```
A system can have moderate cardinality but high churn, for example when Kubernetes workloads frequently create and delete Pods.

## 49. Deduplication vs stream aggregation?
**Answer:**
```text
Deduplication -> removes duplicate samples
Aggregation   -> calculates new results from samples
```

Example:
```text
Deduplication:
A A B B -> A B

Aggregation:
100 200 300 -> SUM = 600
```

## 50. Why use stream aggregation?
**Answer:** It can reduce the amount of data written to storage by calculating useful aggregated results before storage. It is useful for reducing stored samples/series and for precomputing common aggregates.

## 51. Stream aggregation vs recording rules?
**Answer:**
```text
Stream Aggregation:
Incoming data -> Aggregate -> Storage

Recording Rule:
Stored/queryable data -> Rule evaluation -> Recorded metric
```

## 52. Why is high cardinality dangerous?
**Answer:** A large number of unique series increases indexing, memory, storage, and processing requirements. Highly dynamic labels such as request IDs are a common source of excessive cardinality.

## 53. Explain the complete VictoriaMetrics data flow.
**Answer:**
```text
Application
     |
   /metrics
     v
  vmagent
     |
     +--> Relabeling
     +--> Filtering
     +--> Cardinality Limits
     +--> Deduplication
     +--> Stream Aggregation
     +--> Persistent Queue
     |
 Remote Write
     |
  vminsert
     |
 vmstorage
     |
 vmselect
     |
 MetricsQL
     |
 Grafana
```

## 54. How do you remember the main VictoriaMetrics components?
**Answer:**
```text
vmagent   -> COLLECT / PROCESS
vminsert  -> WRITE
vmstorage -> STORE
vmselect  -> READ / QUERY
vmalert   -> ALERT / RECORD
vmauth    -> AUTH / ROUTE
vmctl     -> MIGRATE
vmbackup  -> BACKUP
vmrestore -> RESTORE
```
