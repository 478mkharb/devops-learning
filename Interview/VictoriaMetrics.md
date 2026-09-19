# VictoriaMetrics — Detailed DevOps Guide

VictoriaMetrics is a time-series database (TSDB) and monitoring platform designed for storing and querying metrics efficiently. It is highly compatible with the Prometheus ecosystem and supports Prometheus Remote Write, Prometheus-compatible scraping, Grafana, Kubernetes, and related observability tooling.

---

## 1. What is VictoriaMetrics?

VictoriaMetrics stores time-series metrics such as:

- CPU usage
- Memory usage
- Disk usage
- HTTP request rate
- HTTP latency
- Error counts
- Kubernetes metrics
- Application metrics

A metric can look like:

```text
http_requests_total{method="GET",status="200",service="employee-api"} 1520
```

Conceptually:

```text
Metric
  |
  +-- Name: http_requests_total
  +-- Labels: method, status, service
  +-- Timestamp
  +-- Value
```

---

# 2. Why VictoriaMetrics?

A basic Prometheus architecture is:

```text
Targets
   |
   v
Prometheus
   |
   v
Local TSDB
```

For larger environments you may need:

- Longer retention
- Centralized metrics storage
- Multiple collectors
- Horizontal scaling
- Multi-tenancy
- High availability
- Efficient ingestion

VictoriaMetrics can act as a centralized metrics backend.

```text
Prometheus / vmagent
          |
          | Remote Write
          v
   VictoriaMetrics
          |
          v
        Storage
          |
          v
       Grafana
```

---

# 3. VictoriaMetrics Deployment Models

The two major deployment models are:

1. Single-node VictoriaMetrics
2. VictoriaMetrics Cluster

## Single-node

One main VictoriaMetrics process handles ingestion, storage, and querying.

```text
Prometheus / vmagent
        |
        v
+---------------------+
| VictoriaMetrics     |
| Single Node         |
|                     |
| Ingest + Store      |
| + Query             |
+---------------------+
        |
        v
      Disk
```

It is simpler to operate and is suitable for many smaller and medium-sized deployments.

## Cluster

The cluster separates responsibilities into:

```text
vminsert
vmstorage
vmselect
```

```text
                 Clients
                    |
          +---------+---------+
          |                   |
          v                   v
      vminsert            vmselect
          |                   |
          v                   v
      vmstorage          vmstorage
          |                   |
          +---------+---------+
                    |
                   Disk
```

---

# 4. Single-Node VictoriaMetrics

The single-node version provides:

- Metric ingestion
- Metric storage
- Querying
- Prometheus-compatible APIs
- Web UI

A common HTTP port is:

```text
8428
```

Example:

```text
http://localhost:8428
```

VMUI is available from the VictoriaMetrics HTTP interface.

---

# 5. VictoriaMetrics Cluster

The core cluster components are:

```text
vminsert
vmstorage
vmselect
```

## 5.1 vminsert

`vminsert` is the write/ingestion layer.

```text
Prometheus
    |
    | remote_write
    v
vminsert
```

It accepts incoming metrics and distributes them among `vmstorage` nodes.

**Memory trick:**

```text
vminsert = WRITE
```

---

## 5.2 vmstorage

`vmstorage` is the storage layer.

```text
vminsert
    |
    v
vmstorage
    |
    v
Disk
```

It stores time-series data and returns data needed by queries.

**Memory trick:**

```text
vmstorage = STORE
```

---

## 5.3 vmselect

`vmselect` is the query/read layer.

```text
Grafana
   |
   | Query
   v
vmselect
   |
   +----> vmstorage-1
   +----> vmstorage-2
   +----> vmstorage-3
```

It queries the required storage nodes and returns the result.

**Memory trick:**

```text
vmselect = READ / QUERY
```

---

# 6. Cluster Write Path

```text
Prometheus / vmagent
        |
        | Remote Write
        v
     vminsert
        |
        | distribute
        v
+-------+-------+-------+
|               |       |
v               v       v
vmstorage-1  vmstorage-2 vmstorage-3
```

# 7. Cluster Read Path

```text
Grafana
   |
   | Query
   v
vmselect
   |
   +----> vmstorage-1
   +----> vmstorage-2
   +----> vmstorage-3
   |
   v
Result
   |
   v
Grafana
```

---

# 8. What is vmagent?

`vmagent` is a lightweight metrics collection and forwarding agent.

It can:

- Scrape Prometheus-compatible targets
- Receive supported push-based metrics
- Relabel metrics
- Filter metrics
- Buffer data on disk
- Forward metrics to VictoriaMetrics
- Forward metrics to other Prometheus-compatible remote storage

Typical flow:

```text
Node Exporter
     |
     | /metrics
     v
   vmagent
     |
     | remote_write
     v
VictoriaMetrics
```

---

# 9. vmagent vs Prometheus

Prometheus is a complete monitoring server with scraping, local storage, querying, and rule evaluation.

`vmagent` is designed primarily as a lightweight collection, processing, buffering, and forwarding layer.

Example:

```text
Target
  |
  v
vmagent
  |
  v
VictoriaMetrics
```

This is useful when you want to separate metric collection from long-term storage.

---

# 10. Pull Model and Push Model

## Pull

A collector scrapes a target:

```text
vmagent
   |
   | GET /metrics
   v
Node Exporter
```

## Push

A source sends metrics to an endpoint:

```text
Application
    |
    | push
    v
vmagent / VictoriaMetrics
```

VictoriaMetrics ecosystem supports both models through its components and supported protocols.

---

# 11. vmagent Scrape Configuration

Example:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: node
    static_configs:
      - targets:
          - node-exporter:9100
```

Start vmagent:

```bash
./vmagent   -promscrape.config=prometheus.yml   -remoteWrite.url=http://victoriametrics:8428/api/v1/write
```

Flow:

```text
node-exporter
      |
      | scrape
      v
   vmagent
      |
      | remote write
      v
VictoriaMetrics
```

---

# 12. Prometheus Remote Write

Prometheus can send a copy of its metrics to VictoriaMetrics using Remote Write.

Example:

```yaml
remote_write:
  - url: http://victoriametrics:8428/api/v1/write
```

Flow:

```text
Prometheus
    |
    | remote_write
    v
VictoriaMetrics
```

For a cluster, the URL normally targets `vminsert` and includes the tenant path:

```yaml
remote_write:
  - url: http://vminsert:8480/insert/0/prometheus/api/v1/write
```

The exact URL depends on the cluster topology and tenant configuration.

---

# 13. MetricsQL

VictoriaMetrics provides **MetricsQL**, which is compatible with PromQL and adds additional query capabilities.

Example:

```promql
rate(http_requests_total[5m])
```

Request rate grouped by service:

```promql
sum by (service) (
  rate(http_requests_total[5m])
)
```

Conceptually:

```text
MetricsQL
   |
   +-- PromQL-compatible queries
   +-- VictoriaMetrics extensions
```

---

# 14. Labels and Time Series

Example:

```text
http_requests_total{
  service="employee-api",
  method="GET",
  status="200"
}
```

Each unique combination of metric name and labels represents a distinct time series.

For example:

```text
http_requests_total{service="employee-api",status="200"}
http_requests_total{service="employee-api",status="500"}
http_requests_total{service="salary-api",status="200"}
```

These are different time series.

---

# 15. Cardinality

Cardinality is the number of unique time series.

Bad example:

```text
http_requests_total{
  request_id="8f7a..."
}
```

If every request gets a unique `request_id`, the number of time series can grow rapidly.

Better labels:

```text
service="employee-api"
method="GET"
status="200"
```

Avoid highly dynamic values as labels unless you have a specific reason.

---

# 16. Retention

Retention determines how long metrics are kept.

For example, a deployment may be configured with a retention period such as:

```bash
-retentionPeriod=30d
```

Conceptually:

```text
Metrics
 |
 +-- 1 day
 +-- 7 days
 +-- 30 days
 +-- 12 months
```

Longer retention requires more storage capacity.

---

# 17. Storage

VictoriaMetrics stores time-series data on disk.

Example:

```text
VictoriaMetrics
      |
      v
/var/lib/victoria-metrics
      |
      +-- time-series data
      +-- indexes
      +-- metadata
```

Production planning should consider:

- Disk capacity
- Retention
- Ingestion rate
- Query load
- Backup requirements
- Availability requirements

---

# 18. Grafana Integration

Grafana can use VictoriaMetrics as a metrics data source.

```text
                    +-------------+
                    |   Grafana   |
                    +------+------+
                           |
                           | Query
                           v
                    +-------------+
                    | Victoria    |
                    | Metrics     |
                    +------+------+
                           |
                           v
                         Data
```

For a cluster, Grafana normally queries through `vmselect`.

A common cluster query URL is:

```text
http://vmselect:8481/select/0/prometheus
```

---

# 19. VMUI

VictoriaMetrics includes a built-in UI called **VMUI**.

It can be used to:

- Run queries
- Explore metrics
- Inspect time series
- Troubleshoot metric data

A common single-node URL is:

```text
http://localhost:8428/vmui
```

---

# 20. Docker — Single Node

Example:

```bash
docker run -d   --name victoriametrics   -p 8428:8428   -v vmdata:/victoria-metrics-data   victoriametrics/victoria-metrics
```

Check:

```bash
docker ps
```

Open:

```text
http://localhost:8428
```

---

# 21. Docker Compose Example

```yaml
services:

  victoriametrics:
    image: victoriametrics/victoria-metrics:latest
    container_name: victoriametrics
    ports:
      - "8428:8428"
    volumes:
      - vmdata:/victoria-metrics-data
    command:
      - "-storageDataPath=/victoria-metrics-data"
      - "-retentionPeriod=30d"

volumes:
  vmdata:
```

Start:

```bash
docker compose up -d
```

Check:

```bash
docker compose ps
```

Logs:

```bash
docker compose logs -f victoriametrics
```

---

# 22. Prometheus + VictoriaMetrics

Architecture:

```text
             +----------------+
             | Node Exporter  |
             +-------+--------+
                     |
                     v
             +----------------+
             |  Prometheus    |
             +-------+--------+
                     |
                     | remote_write
                     v
             +----------------+
             | VictoriaMetrics|
             +----------------+
                     |
                     v
                   Disk
```

Prometheus:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: node
    static_configs:
      - targets:
          - node-exporter:9100

remote_write:
  - url: http://victoriametrics:8428/api/v1/write
```

---

# 23. vmagent + VictoriaMetrics

Lightweight collection architecture:

```text
                 +----------------+
                 | Node Exporter  |
                 +-------+--------+
                         |
                         v
                  +-------------+
                  |   vmagent   |
                  +------+------+
                         |
                         | remote_write
                         v
                  +-------------+
                  | Victoria    |
                  | Metrics     |
                  +-------------+
```

---

# 24. Kubernetes Architecture

A common Kubernetes architecture is:

```text
                  Kubernetes Cluster
                         |
              +----------+----------+
              |                     |
              v                     v
          Applications          Node Exporter
              |                     |
              +----------+----------+
                         |
                         v
                      vmagent
                         |
                         | remote write
                         v
                     vminsert
                         |
              +----------+----------+
              |                     |
              v                     v
          vmstorage-0          vmstorage-1

Grafana
   |
   v
vmselect
   |
   +----> vmstorage
```

---

# 25. VictoriaMetrics Operator

The VictoriaMetrics Operator provides Kubernetes resources for managing VictoriaMetrics components.

Common resources include:

- `VMCluster`
- `VMSingle`
- `VMAgent`
- `VMServiceScrape`
- `VMPodScrape`
- `VMRule`
- `VMAlert`

Example:

```yaml
apiVersion: operator.victoriametrics.com/v1beta1
kind: VMCluster
metadata:
  name: example
spec:
  vmstorage:
    replicaCount: 2

  vminsert:
    replicaCount: 2

  vmselect:
    replicaCount: 2
```

Always check the installed Operator version before applying a manifest because CRD fields can change between releases.

---

# 26. VMServiceScrape

`VMServiceScrape` defines how services can be scraped by VMAgent.

Example:

```yaml
apiVersion: operator.victoriametrics.com/v1beta1
kind: VMServiceScrape
metadata:
  name: employee-api
spec:
  selector:
    matchLabels:
      app: employee-api

  endpoints:
    - port: metrics
      path: /metrics
```

Conceptually:

```text
Service
   |
   | discovered
   v
VMAgent
   |
   | remote write
   v
VictoriaMetrics
```

---

# 27. VMPodScrape

`VMPodScrape` is used for Pod-based scraping.

Conceptually:

```text
Pod
 |
 | /metrics
 v
VMAgent
 |
 v
VictoriaMetrics
```

It is useful when metrics are exposed directly by Pods rather than through a Service.

---

# 28. vmalert

`vmalert` evaluates Prometheus-compatible alerting and recording rules.

Architecture:

```text
VictoriaMetrics
      ^
      | Query
      |
   vmalert
      |
      | Alert
      v
Alertmanager
```

Example:

```yaml
groups:
  - name: infrastructure
    rules:

      - alert: HighCPU
        expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage"
```

---

# 29. Recording Rules

Recording rules precompute frequently used queries.

Example:

```yaml
groups:
  - name: api
    rules:
      - record: api:http_requests_rate5m
        expr: sum(rate(http_requests_total[5m]))
```

Then query:

```promql
api:http_requests_rate5m
```

Instead of repeatedly calculating the full expression.

---

# 30. Alerting vs Recording Rules

| Rule | Purpose |
|---|---|
| Alerting rule | Detect a condition and generate an alert |
| Recording rule | Precompute a query result |

Example:

```text
Alerting:
CPU > 80% → ALERT

Recording:
Request rate → calculated metric
```

---

# 31. Multitenancy

VictoriaMetrics Cluster supports multitenancy.

A common write URL contains an account/tenant identifier:

```text
/insert/<accountID>/prometheus/api/v1/write
```

Example:

```text
/insert/0/prometheus/api/v1/write
```

Another tenant could use:

```text
/insert/1/prometheus/api/v1/write
```

Conceptually:

```text
Tenant A ──┐
Tenant B ──+──> vminsert ──> vmstorage
Tenant C ──┘
```

The single-node version does not provide the same cluster multitenancy model.

---

# 32. vmauth

`vmauth` is an authorization proxy and load balancer for VictoriaMetrics components.

```text
Client
  |
  v
vmauth
  |
  +----> vminsert
  |
  +----> vmselect
```

It can be used for:

- Authentication
- Authorization
- Request routing
- Load balancing

---

# 33. Backups

VictoriaMetrics provides:

```text
vmbackup
vmrestore
vmbackupmanager
```

Typical concept:

```text
VictoriaMetrics
      |
      | vmbackup
      v
Backup Storage
      |
      | vmrestore
      v
VictoriaMetrics
```

Backups should be tested with restoration procedures rather than only checking that backup files exist.

---

# 34. vmctl

`vmctl` is used for migrating or copying metrics between storage systems.

```text
Existing TSDB
      |
      | vmctl
      v
VictoriaMetrics
```

This can be useful during monitoring-stack migration.

---

# 35. Useful HTTP APIs

## Prometheus Remote Write

```text
POST /api/v1/write
```

Example Prometheus configuration:

```yaml
remote_write:
  - url: http://victoriametrics:8428/api/v1/write
```

## Prometheus exposition import

```text
POST /api/v1/import/prometheus
```

Example:

```bash
curl -d 'demo_metric{service="employee-api"} 100'   http://localhost:8428/api/v1/import/prometheus
```

## Query

VictoriaMetrics exposes Prometheus-compatible query APIs for reading metrics.

Example:

```bash
curl 'http://localhost:8428/api/v1/query?query=up'
```

---

# 36. Complete Data Flow Example

Suppose an application exposes `/metrics`.

```text
                  Kubernetes
                      |
                      v
              +---------------+
              | employee-api  |
              |   /metrics    |
              +-------+-------+
                      |
                      | scrape
                      v
                  +-------+
                  |vmagent|
                  +---+---+
                      |
                      | remote write
                      v
                 +---------+
                 |vminsert |
                 +----+----+
                      |
             +--------+--------+
             |                 |
             v                 v
        vmstorage-0      vmstorage-1
             |                 |
             +--------+--------+
                      |
                    query
                      ^
                      |
                  vmselect
                      ^
                      |
                   Grafana
```

The logical path is:

```text
Collection
    ↓
Ingestion
    ↓
Storage
    ↓
Query
    ↓
Visualization
```

---

# 37. Single Node vs Cluster

| Feature | Single Node | Cluster |
|---|---|---|
| Simplicity | High | Lower |
| Main architecture | One main service | vminsert + vmselect + vmstorage |
| Independent scaling | Limited | Yes |
| Cluster multitenancy | No | Yes |
| Operations | Easier | More complex |
| Query layer | Single node | vmselect |
| Write layer | Single node | vminsert |
| Storage layer | Single node | vmstorage |

Do not choose the cluster architecture automatically. VictoriaMetrics documentation explicitly notes that the single-node version is easier to configure and operate and can be sufficient for many deployments.

---

# 38. VictoriaMetrics vs Prometheus

| Feature | Prometheus | VictoriaMetrics |
|---|---|---|
| Metrics collection | Native | vmagent provides collection |
| Local TSDB | Yes | Yes |
| Prometheus Remote Write | Yes | Yes |
| Long-term centralized storage | Often paired with remote storage | Core capability |
| Cluster architecture | Different ecosystem | vminsert/vmselect/vmstorage |
| Query language | PromQL | MetricsQL / PromQL-compatible |
| Grafana | Yes | Yes |
| Kubernetes integration | Strong | Strong |

They can also be used together:

```text
Targets
   |
   v
Prometheus
   |
   | remote_write
   v
VictoriaMetrics
   |
   v
Grafana
```

---

# 39. Common Ports

Common default ports include:

| Component | Common Port |
|---|---:|
| VictoriaMetrics single node | `8428` |
| vmagent | `8429` |
| vminsert | `8480` |
| vmselect | `8481` |
| vmstorage | `8482` |

Always verify the actual port in your deployment/version.

---

# 40. Useful Commands

Docker:

```bash
docker ps
docker logs victoriametrics
docker logs -f victoriametrics
```

Health:

```bash
curl http://localhost:8428/health
```

Root endpoint:

```bash
curl http://localhost:8428/
```

Query:

```bash
curl 'http://localhost:8428/api/v1/query?query=up'
```

---

# 41. Troubleshooting Checklist

If metrics are missing:

```text
1. Is the target reachable?
2. Does /metrics return data?
3. Is Prometheus/vmagent scraping it?
4. Is remote_write configured?
5. Can the collector reach VictoriaMetrics?
6. Are relabeling rules dropping the metrics?
7. Is Grafana using the correct datasource?
8. Is the query correct?
```

Test a target:

```bash
curl http://target:port/metrics
```

Check collector logs:

```bash
docker logs vmagent
```

Check VictoriaMetrics logs:

```bash
docker logs victoriametrics
```

---

# 42. Common Interview Questions

## Q1. What is VictoriaMetrics?

**Answer:**

VictoriaMetrics is a time-series database and monitoring platform designed to efficiently store and query metrics. It is highly compatible with the Prometheus ecosystem and supports Prometheus Remote Write.

## Q2. What is vmagent?

**Answer:**

vmagent is a lightweight metrics agent that can scrape Prometheus-compatible targets, receive supported metrics, relabel/filter data, buffer it, and forward it to VictoriaMetrics or other remote storage.

## Q3. What are vminsert, vmselect, and vmstorage?

**Answer:**

- `vminsert` handles ingestion.
- `vmstorage` stores metric data.
- `vmselect` handles queries.

## Q4. How does Prometheus send metrics to VictoriaMetrics?

**Answer:**

Prometheus uses the Prometheus Remote Write protocol.

```yaml
remote_write:
  - url: http://victoriametrics:8428/api/v1/write
```

## Q5. What is MetricsQL?

**Answer:**

MetricsQL is VictoriaMetrics' query language. It is compatible with PromQL and provides additional capabilities.

## Q6. What is cardinality?

**Answer:**

Cardinality is the number of unique time series. Highly dynamic labels such as request IDs can create very high cardinality.

## Q7. Difference between vminsert and vmselect?

```text
vminsert → WRITE
vmselect  → READ / QUERY
```

## Q8. Why use Grafana with VictoriaMetrics?

**Answer:**

VictoriaMetrics provides metrics storage/querying, while Grafana provides visualization and dashboards.

## Q9. Single-node or cluster?

**Answer:**

Single-node is simpler and may be sufficient for many deployments. Cluster is useful when independent horizontal scaling, cluster multitenancy, or larger-scale architecture is required.

---

# 43. Quick Revision

```text
VictoriaMetrics
|
+-- Single Node
|     +-- Ingest
|     +-- Store
|     +-- Query
|
+-- Cluster
|     +-- vminsert
|     |      +-- WRITE
|     |
|     +-- vmstorage
|     |      +-- STORE
|     |
|     +-- vmselect
|            +-- READ / QUERY
|
+-- vmagent
|     +-- Scrape
|     +-- Relabel
|     +-- Filter
|     +-- Buffer
|     +-- Remote Write
|
+-- vmalert
|     +-- Alerting rules
|     +-- Recording rules
|
+-- vmauth
|     +-- Authentication
|     +-- Authorization
|     +-- Routing
|
+-- vmbackup / vmrestore
|     +-- Backup / Restore
|
+-- vmctl
      +-- Migration / Data copy
```

---

# 44. One-Line Memory Tricks

```text
VictoriaMetrics = Metrics storage + query platform

vmagent   = Collect / process / forward
vminsert  = Write
vmstorage = Store
vmselect  = Read / Query
vmalert   = Alert / Record
vmauth    = Authenticate / Authorize / Route
vmctl     = Migrate
vmbackup  = Backup
vmrestore = Restore
```

---

# 45. Typical Production Architecture

```text
                    +----------------+
                    |    Grafana     |
                    +-------+--------+
                            |
                            v
                       +---------+
                       | vmauth   |
                       +----+----+
                            |
                            v
                       +---------+
                       | vmselect|
                       +----+----+
                            |
             +--------------+--------------+
             |              |              |
             v              v              v
        vmstorage-1   vmstorage-2   vmstorage-3
             ^
             |
        +----+-----+
        | vminsert |
        +----+-----+
             ^
             |
       +-----+------+
       |            |
       v            v
    vmagent      vmagent
       ^            ^
       |            |
   Targets       Targets
```

This separates:

```text
Collection → Ingestion → Storage → Query → Visualization
```

---

# 46. Final Interview Answer

> **VictoriaMetrics is a time-series database and monitoring platform that is highly compatible with the Prometheus ecosystem. It can be deployed as a single node or as a cluster. In the cluster architecture, vminsert handles metric ingestion, vmstorage stores the time-series data, and vmselect handles queries. vmagent can be used as a lightweight collection and forwarding layer. Grafana can query VictoriaMetrics for visualization, while vmalert evaluates alerting and recording rules.**

---

# Official Documentation

- VictoriaMetrics documentation: https://docs.victoriametrics.com/
- vmagent: https://docs.victoriametrics.com/vmagent/
- Cluster architecture: https://docs.victoriametrics.com/victoriametrics/cluster-victoriametrics/
- Prometheus integration: https://docs.victoriametrics.com/victoriametrics/integrations/prometheus/
- VictoriaMetrics GitHub: https://github.com/VictoriaMetrics/VictoriaMetrics
