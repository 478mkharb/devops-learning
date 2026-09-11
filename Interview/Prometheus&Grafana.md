# Prometheus and Grafana L2 Interview Questions

This README is organized for **L2 DevOps / SRE / Monitoring interviews**. P1 topics are the core interview areas; P2 topics cover supporting operational skills; P3 covers advanced architecture, integrations and comparison topics.

| Priority | Questions | Main Focus |
|---|---:|---|
| 🔴 P1 - High | 1–65 | Monitoring fundamentals, Prometheus architecture, metrics, labels, scraping, exporters, PromQL, alerting and Grafana |
| 🟠 P2 - Medium | 66–95 | Service discovery, relabeling, recording rules, troubleshooting, dashboard engineering, provisioning and operational practices |
| 🟢 P3 - Low | 96–110 | HA, remote storage, federation, scaling, Thanos/Mimir, observability architecture and interview scenarios |

---

# 1. P1 — Monitoring Fundamentals and Prometheus Architecture

## 1. What is Prometheus and what problem does it solve?

**Priority: 🔴 P1**

Prometheus is an open-source monitoring and alerting system designed around **time-series metrics**.

It is commonly used to monitor:

- Linux servers
- Applications
- Kubernetes clusters
- Databases
- APIs
- Network components
- Cloud infrastructure

Its normal flow is:

```text
Application / Server
        |
        | /metrics
        v
   Prometheus
        |
        +----> Local TSDB
        |
        +----> Alert rules
        |
        v
   Alertmanager
        |
        v
 Email / Slack / PagerDuty

Prometheus <---- PromQL ---- Grafana
```

Prometheus mainly answers:

> What is happening to my systems over time, and when should I be alerted?

Grafana is commonly used on top of Prometheus for visualization and dashboards.

---

## 2. Why is Prometheus called a pull-based monitoring system?

**Priority: 🔴 P1 — Very Important**

Prometheus normally **pulls/scrapes** metrics from monitored targets.

```text
Prometheus ----HTTP GET----> Target
                              /metrics
```

The target exposes metrics, usually through an HTTP endpoint such as:

```text
http://10.0.1.20:9100/metrics
```

Prometheus periodically scrapes that endpoint and stores the returned samples.

### Pull vs push

| Model | Description |
|---|---|
| Pull | Monitoring server fetches metrics |
| Push | Application/agent sends metrics |

Prometheus prefers pull because it can control scrape frequency, detect unreachable targets and keep discovery/target state centrally visible.

### Interview point

Prometheus can receive data through additional mechanisms such as remote write or Pushgateway, but its normal target monitoring model is **pull-based scraping**.

---

## 3. Explain the main components of a Prometheus ecosystem.

**Priority: 🔴 P1**

| Component | Purpose |
|---|---|
| Prometheus Server | Scrapes, stores and queries metrics |
| Exporter | Exposes metrics for systems that do not natively expose Prometheus metrics |
| Service Discovery | Finds monitoring targets dynamically |
| PromQL | Query language for metrics |
| Rule Engine | Evaluates recording and alerting rules |
| Alertmanager | Groups, routes and silences alerts |
| Grafana | Visualization and dashboarding |
| Pushgateway | Temporary push endpoint for suitable short-lived jobs |
| Remote Write | Sends samples to compatible remote storage systems |

Typical architecture:

```text
              +-------------------+
              |    Applications   |
              +---------+---------+
                        |
                   /metrics
                        |
              +---------v---------+
              |    Prometheus     |
              |  scrape + TSDB    |
              +----+----------+---+
                   |          |
                PromQL       Alerts
                   |          |
             +-----v---+  +---v-----------+
             | Grafana |  | Alertmanager   |
             +---------+  +-------+--------+
                                  |
                         Email / Slack / PagerDuty
```

---

## 4. What is a time series in Prometheus?

**Priority: 🔴 P1**

A time series is identified by:

```text
metric name + complete set of labels
```

For example:

```text
http_requests_total{job="api",instance="10.0.1.10:8080",method="GET",status="200"}
```

Each unique label combination represents a different time series.

Conceptually:

```text
http_requests_total{job="api",status="200"}
http_requests_total{job="api",status="500"}
```

are two different series.

### Important L2 point

Adding a high-cardinality label can create thousands or millions of time series. This can increase memory usage, disk usage and query cost.

---

## 5. What are labels in Prometheus?

**Priority: 🔴 P1**

Labels add dimensions to metrics.

Example:

```text
http_requests_total{
  job="api",
  instance="10.0.1.10:8080",
  method="GET",
  status="200"
}
```

Here:

- `job` identifies the scrape job
- `instance` identifies the target
- `method` identifies the HTTP method
- `status` identifies the HTTP status

Labels allow queries such as:

```promql
http_requests_total{status="500"}
```

### Brain teaser

If you add this label:

```text
request_id="a8fd92..."
```

for every request, is that a good Prometheus label?

**No.** Request IDs usually have extremely high cardinality and should generally not be used as metric labels.

---

## 6. What is metric cardinality and why is it dangerous?

**Priority: 🔴 P1 — Very Important**

Cardinality is the number of unique time series generated by a metric and its label combinations.

Suppose:

```text
method = 5 values
status = 10 values
endpoint = 100 values
```

Approximate series count:

```text
5 × 10 × 100 = 5,000
```

If you add:

```text
user_id = 1,000,000 values
```

the theoretical combination count becomes enormous.

### Bad labels

Avoid labels such as:

```text
user_id
request_id
session_id
full_url_with_query_parameters
```

when they can take a very large number of values.

### Better design

Prefer bounded dimensions:

```text
method
status
service
region
environment
```

---

## 7. What are the four common Prometheus metric types?

**Priority: 🔴 P1**

| Type | Meaning | Example |
|---|---|---|
| Counter | Monotonically increasing value | Requests processed |
| Gauge | Value that can increase/decrease | CPU temperature |
| Histogram | Distribution of observations in buckets | Request latency |
| Summary | Client-side quantile/distribution summary | Request latency |

### Counter

```text
http_requests_total 125430
```

Usually query counters with `rate()` or `increase()`.

### Gauge

```text
node_memory_MemAvailable_bytes 8.2e+09
```

Can move up or down.

### Histogram

A histogram exposes bucket metrics such as:

```text
http_request_duration_seconds_bucket
http_request_duration_seconds_sum
http_request_duration_seconds_count
```

### Summary

A summary can expose quantile estimates plus `_sum` and `_count`.

### Interview trap

Do not calculate a counter's value as a rate directly. Use a function such as:

```promql
rate(http_requests_total[5m])
```

---

## 8. Counter vs Gauge — when should each be used?

**Priority: 🔴 P1**

Use a **Counter** when the value should only increase except for process restarts.

Examples:

```text
requests_total
errors_total
transactions_total
```

Use a **Gauge** when the value can move in both directions.

Examples:

```text
memory_available_bytes
active_connections
queue_depth
temperature_celsius
```

Wrong design:

```text
active_users_total
```

if users can log in and out and the value goes both up and down.

Better:

```text
active_users
```

as a gauge.

---

## 9. What is an exporter?

**Priority: 🔴 P1**

An exporter converts metrics from a system into a Prometheus-readable format.

A common example is **Node Exporter** for Linux host metrics.

```text
Linux Kernel / /proc / /sys
            |
            v
      Node Exporter
            |
       /metrics
            |
            v
       Prometheus
```

Examples:

| Exporter | Purpose |
|---|---|
| Node Exporter | Linux host metrics |
| Blackbox Exporter | Probe HTTP, TCP, ICMP and other endpoints |
| MySQL Exporter | MySQL metrics |
| PostgreSQL Exporter | PostgreSQL metrics |
| NGINX Exporter | NGINX metrics |

An exporter is **not the monitoring system itself**. It is a metrics adapter/exposure component.

---

## 10. What is Node Exporter and what does it monitor?

**Priority: 🔴 P1**

Node Exporter exposes hardware and OS-level metrics from Unix-like systems.

Typical metrics include:

```text
node_cpu_seconds_total
node_memory_MemAvailable_bytes
node_filesystem_avail_bytes
node_disk_read_bytes_total
node_network_receive_bytes_total
node_load1
```

Typical architecture:

```text
Ubuntu / RHEL
     |
 Node Exporter :9100
     |
     v
 Prometheus
```

A basic scrape configuration can look like:

```yaml
scrape_configs:
  - job_name: "node"
    static_configs:
      - targets:
          - "10.0.1.10:9100"
          - "10.0.1.11:9100"
```

---

## 11. What is Blackbox Exporter?

**Priority: 🔴 P1**

Blackbox Exporter is used for **black-box probing** of endpoints.

It is useful when you care about what a client can observe rather than internal application instrumentation.

Typical probes include:

- HTTP/HTTPS
- TCP
- ICMP
- DNS

Architecture:

```text
Prometheus
    |
    | probe request
    v
Blackbox Exporter
    |
    +----> https://example.com
    +----> TCP endpoint
    +----> ICMP target
```

Typical metric:

```text
probe_success
```

Example query:

```promql
probe_success == 0
```

A common use case is checking whether an externally visible URL is reachable.

---

## 12. What is a scrape in Prometheus?

**Priority: 🔴 P1**

A scrape is one Prometheus collection operation against a target.

For example, with:

```yaml
global:
  scrape_interval: 15s
```

Prometheus normally attempts a scrape every 15 seconds unless overridden for a job.

Useful built-in metrics include:

```promql
up
scrape_duration_seconds
scrape_samples_scraped
scrape_samples_post_metric_relabeling
```

`up` is especially important:

```promql
up == 0
```

usually means Prometheus could not successfully scrape that target.

---

## 13. What is `up` metric and how do you troubleshoot `up == 0`?

**Priority: 🔴 P1 — Common Scenario**

`up` is a standard Prometheus target health metric.

Typical values:

```text
up = 1  -> scrape succeeded
up = 0  -> scrape failed
```

Troubleshooting flow:

```text
up == 0
   |
   +--> Is target discovered?
   |
   +--> Is target address correct?
   |
   +--> Is port listening?
   |
   +--> Is /metrics reachable?
   |
   +--> Network / firewall / SG?
   |
   +--> TLS/authentication issue?
   |
   +--> Exporter/application healthy?
```

From the Prometheus host:

```bash
curl http://10.0.1.10:9100/metrics
```

Also inspect **Status → Targets** in Prometheus.

### Interview answer

Do not immediately conclude that the application is down. `up == 0` means the **Prometheus scrape failed**, which can also be caused by network, DNS, authentication or endpoint configuration problems.

---

## 14. What is `scrape_interval` and `scrape_timeout`?

**Priority: 🔴 P1**

`scrape_interval` controls how often Prometheus attempts a scrape.

```yaml
global:
  scrape_interval: 15s
```

`scrape_timeout` controls how long Prometheus waits for a scrape response.

```yaml
scrape_configs:
  - job_name: "api"
    scrape_interval: 10s
    scrape_timeout: 5s
```

The timeout must not exceed the scrape interval.

### Interview trap

Reducing the scrape interval from 15 seconds to 1 second is not automatically better. It increases scrape load, samples ingested and storage requirements.

---

## 15. Explain the basic Prometheus configuration file.

**Priority: 🔴 P1**

A basic configuration commonly contains:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "rules/*.yml"

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets:
          - "localhost:9090"

  - job_name: "node"
    static_configs:
      - targets:
          - "10.0.1.10:9100"
          - "10.0.1.11:9100"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - "alertmanager:9093"
```

Important sections:

| Section | Purpose |
|---|---|
| `global` | Global defaults |
| `rule_files` | Recording/alert rule files |
| `scrape_configs` | Target scraping configuration |
| `alerting` | Alertmanager integration |
| `remote_write` | Remote metric write |
| `remote_read` | Remote query integration where supported/configured |

---

## 16. What is `job_name` in Prometheus?

**Priority: 🔴 P1**

`job_name` logically groups scrape targets and is added as a `job` label in normal scraping scenarios.

Example:

```yaml
scrape_configs:
  - job_name: "node"
    static_configs:
      - targets:
          - "10.0.1.10:9100"
```

The resulting metric can contain:

```text
job="node"
```

This lets you query:

```promql
up{job="node"}
```

### L2 point

`job` is useful for logical grouping; `instance` commonly identifies the individual scrape target.

---

## 17. What is service discovery in Prometheus?

**Priority: 🔴 P1**

Service discovery allows Prometheus to dynamically discover scrape targets rather than hardcoding every target.

Examples include discovery from:

- Kubernetes
- AWS EC2
- Consul
- Azure
- GCE
- DNS-based mechanisms
- File-based service discovery

Conceptually:

```text
Cloud / Kubernetes API
          |
          v
   Service Discovery
          |
          v
     Prometheus
          |
          v
       Scrapes
```

Without discovery:

```yaml
static_configs:
  - targets: ["10.0.1.10:9100"]
```

With discovery, targets can appear/disappear dynamically.

---

## 18. Static targets vs service discovery — when would you use each?

**Priority: 🔴 P1**

| Static configuration | Service discovery |
|---|---|
| Small stable environments | Dynamic infrastructure |
| Simple VMs | Kubernetes / cloud autoscaling |
| Lab environments | Production fleets |
| Few target changes | Frequent target changes |

For a static lab:

```yaml
static_configs:
  - targets: ["10.0.1.10:9100"]
```

For AWS or Kubernetes environments, service discovery is usually more scalable.

### Scenario

An ASG scales from 2 to 20 instances every day. Should you manually edit `prometheus.yml` each time?

**No.** Use service discovery and relabeling to discover eligible instances dynamically.

---

## 19. What is PromQL?

**Priority: 🔴 P1 — Very Important**

PromQL is Prometheus's query language.

Example:

```promql
up
```

Filter:

```promql
up{job="node"}
```

Aggregation:

```promql
sum(up)
```

Rate calculation:

```promql
rate(http_requests_total[5m])
```

PromQL is used by Prometheus itself and is commonly used in Grafana panels connected to Prometheus.

---

## 20. What is the difference between instant vector, range vector and scalar?

**Priority: 🔴 P1**

### Instant vector

A set of time series with one sample per series at an evaluation time.

Example:

```promql
up
```

### Range vector

A set of samples over a time range for each series.

Example:

```promql
http_requests_total[5m]
```

Functions such as `rate()` commonly consume a range vector.

```promql
rate(http_requests_total[5m])
```

### Scalar

A single numeric value.

Example:

```promql
scalar(1)
```

### Interview rule

If you write:

```promql
rate(http_requests_total[5m])
```

`http_requests_total[5m]` is a **range vector** input to `rate()`.

---

## 21. What is the difference between `rate()`, `irate()` and `increase()`?

**Priority: 🔴 P1 — Very Important**

### `rate()`

Calculates the per-second average increase of a counter over a range.

```promql
rate(http_requests_total[5m])
```

Usually the preferred choice for dashboards and alerting because it is less sensitive to individual scrape noise.

### `irate()`

Uses the most recent data points to calculate a more immediate rate.

```promql
irate(http_requests_total[5m])
```

Useful for quickly changing counters, but can be noisy.

### `increase()`

Calculates the total increase over the specified range.

```promql
increase(http_requests_total[1h])
```

### Simple rule

```text
Dashboard trend  -> rate()
Very short/reactive view -> irate() when justified
Total count over window -> increase()
```

---

## 22. How do you calculate request rate in PromQL?

**Priority: 🔴 P1**

Suppose your application exports:

```text
http_requests_total
```

Per-second request rate:

```promql
rate(http_requests_total[5m])
```

By service:

```promql
sum by (job) (
  rate(http_requests_total[5m])
)
```

By status code:

```promql
sum by (status) (
  rate(http_requests_total[5m])
)
```

By instance:

```promql
sum by (instance) (
  rate(http_requests_total[5m])
)
```

---

## 23. How do you calculate CPU utilization using Node Exporter?

**Priority: 🔴 P1 — Common Interview Question**

A common approach is to calculate non-idle CPU usage:

```promql
100 * (
  1 - avg by (instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  )
)
```

Interpretation:

```text
100% - idle percentage = approximate busy CPU percentage
```

### Important

Do not simply use a raw cumulative counter such as:

```promql
node_cpu_seconds_total
```

as CPU percentage. It is a cumulative counter and requires a rate calculation.

---

## 24. How do you calculate memory utilization?

**Priority: 🔴 P1**

A common Node Exporter query is:

```promql
100 * (
  1 - node_memory_MemAvailable_bytes
      / node_memory_MemTotal_bytes
)
```

By instance:

```promql
100 * (
  1 - (
    node_memory_MemAvailable_bytes
    / node_memory_MemTotal_bytes
  )
)
```

### Why use `MemAvailable`?

Available memory is generally more useful for estimating how much memory is actually available to applications than simply treating cache as permanently unavailable.

---

## 25. How do you calculate filesystem usage?

**Priority: 🔴 P1**

A common query is:

```promql
100 * (
  1 - node_filesystem_avail_bytes{fstype!="tmpfs"}
      / node_filesystem_size_bytes{fstype!="tmpfs"}
)
```

You may need additional filters for your environment, such as excluding pseudo-filesystems.

For a specific mount:

```promql
100 * (
  1 - node_filesystem_avail_bytes{mountpoint="/"}
      / node_filesystem_size_bytes{mountpoint="/"}
)
```

### Interview trap

Disk usage is not always the same thing as disk I/O saturation. A filesystem can have 20% free space but still experience high latency or I/O wait.

---

## 26. How do you calculate HTTP 5xx error percentage?

**Priority: 🔴 P1 — Common Scenario**

Suppose you have:

```text
http_requests_total{status="500"}
http_requests_total{status="200"}
```

Error percentage:

```promql
100 *
(
  sum(rate(http_requests_total{status=~"5.."}[5m]))
  /
  sum(rate(http_requests_total[5m]))
)
```

This produces an approximate percentage of requests returning 5xx responses.

### Brain teaser

If traffic drops to zero and there are no errors, is the error percentage query automatically proof that the service is healthy?

**Not necessarily.** You should also consider availability, request volume and whether the service is receiving expected traffic.

---

## 27. What is aggregation in PromQL?

**Priority: 🔴 P1**

Aggregation combines series.

Example:

```promql
sum(rate(http_requests_total[5m]))
```

Group by a label:

```promql
sum by (job) (
  rate(http_requests_total[5m])
)
```

Group by multiple labels:

```promql
sum by (job, status) (
  rate(http_requests_total[5m])
)
```

Common aggregators:

```text
sum
avg
min
max
count
stddev
stdvar
```

### Key concept

Aggregation can reduce the number of output series, which is useful for dashboard readability and query efficiency.

---

## 28. What is the difference between `by` and `without` in PromQL?

**Priority: 🔴 P1**

`by` keeps only the listed grouping labels in the aggregation output.

```promql
sum by (job) (rate(http_requests_total[5m]))
```

`without` aggregates while removing the specified labels from grouping.

```promql
sum without (instance) (rate(http_requests_total[5m]))
```

### When is `without` useful?

When you want aggregation behavior to remain stable as new unrelated labels are added.

Example:

```promql
sum without (instance) (
  rate(http_requests_total[5m])
)
```

The query does not require you to enumerate every other label.

---

## 29. What is a histogram and how do you calculate p95 latency?

**Priority: 🔴 P1 — Very Important**

A Prometheus histogram typically exposes bucket counters such as:

```text
http_request_duration_seconds_bucket
http_request_duration_seconds_sum
http_request_duration_seconds_count
```

A common p95 query is:

```promql
histogram_quantile(
  0.95,
  sum by (le) (
    rate(http_request_duration_seconds_bucket[5m])
  )
)
```

### Why `le`?

`le` represents the upper bound of a histogram bucket.

### Important L2 distinction

For classic Prometheus histograms, quantiles are calculated from bucket data. The result is an approximation influenced by bucket boundaries.

---

## 30. Histogram vs Summary — what is the difference?

**Priority: 🔴 P1**

| Histogram | Summary |
|---|---|
| Exposes buckets | Calculates summary statistics client-side |
| Can aggregate across instances using bucket data | Quantiles generally cannot be meaningfully aggregated across instances |
| Good fit for service-level aggregation | Useful for local/client-side quantile observation |
| Quantiles calculated with `histogram_quantile()` | Quantile values may be exposed directly |

For distributed systems where you need a service-wide percentile, histograms are often easier to aggregate correctly.

---

## 31. What is an alerting rule in Prometheus?

**Priority: 🔴 P1**

An alerting rule evaluates a PromQL expression and creates an alert when the expression matches.

Example:

```yaml
groups:
  - name: node-alerts
    rules:
      - alert: HighCpuUsage
        expr: |
          100 * (
            1 - avg by (instance) (
              rate(node_cpu_seconds_total{mode="idle"}[5m])
            )
          ) > 80
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage"
          description: "CPU usage is above 80% for more than 10 minutes."
```

Important fields:

- `alert`
- `expr`
- `for`
- `labels`
- `annotations`

---

## 32. What does the `for` field do in an alert rule?

**Priority: 🔴 P1**

`for` requires the alert condition to remain active for the specified duration before the alert becomes firing.

Example:

```yaml
for: 10m
```

If CPU crosses 80% for 30 seconds and falls back, the alert does not fire.

```text
CPU > 80%
   |------ 30s ------|
                   falls
                  below
=> no firing alert
```

If it remains above 80% for 10 minutes:

```text
CPU > 80%
   |----------- 10m -----------|
                              fires
```

This helps reduce alert flapping caused by short-lived spikes.

---

## 33. What is Alertmanager and why is it needed?

**Priority: 🔴 P1 — Very Important**

Prometheus evaluates alert rules. **Alertmanager handles alert notification workflow.**

Typical flow:

```text
Prometheus
    |
    | firing alerts
    v
Alertmanager
    |
    +--> Grouping
    +--> Inhibition
    +--> Silencing
    +--> Routing
    |
    v
Email / Slack / PagerDuty / Webhook
```

### Why separate it?

You do not want every Prometheus server independently sending duplicate notifications without routing and grouping logic.

Alertmanager is responsible for notification management such as:

- Grouping related alerts
- Routing by labels
- Silencing alerts
- Inhibiting lower-priority alerts
- Sending notifications

---

## 34. What is the difference between alert rule, Alertmanager and Grafana Alerting?

**Priority: 🔴 P1 — Very Important**

These concepts should not be mixed.

| Component | Primary role |
|---|---|
| Prometheus alerting rule | Evaluate PromQL condition |
| Alertmanager | Route/group/silence Prometheus alerts |
| Grafana Alerting | Grafana-managed alert rules and notification workflow across supported data sources |

Prometheus-native alert flow:

```text
PromQL
  |
  v
Prometheus alert rule
  |
  v
Alertmanager
  |
  v
Notification
```

Grafana can also manage alert rules independently. It can use Prometheus as its data source.

### Interview trap

Grafana is **not required** for Prometheus-native alerting.

---

## 35. What is the difference between labels and annotations in an alert?

**Priority: 🔴 P1**

### Labels

Used for classification, routing and grouping.

```yaml
labels:
  severity: critical
  team: platform
```

### Annotations

Used for human-readable information.

```yaml
annotations:
  summary: "Disk usage is high"
  description: "Disk usage is above 90% on {{ $labels.instance }}"
```

### Rule of thumb

```text
Labels      -> machine-readable classification/routing
Annotations -> human-readable context
```

---

## 36. What is alert grouping in Alertmanager?

**Priority: 🔴 P1**

Grouping combines related alerts into a single notification.

Example configuration:

```yaml
route:
  group_by:
    - alertname
    - cluster
```

Suppose 50 instances in one cluster fail due to a network outage.

Instead of 50 independent messages, Alertmanager can group related alerts.

This reduces notification noise.

---

## 37. What is a silence in Alertmanager?

**Priority: 🔴 P1**

A silence temporarily suppresses notifications that match specified label matchers.

Example concept:

```text
alertname = NodeDown
instance = web01
```

You might create a silence during planned maintenance.

### Important

A silence does not delete the underlying alert. It suppresses notification delivery for matching alerts.

---

## 38. What is inhibition in Alertmanager?

**Priority: 🔴 P1**

Inhibition suppresses certain alerts when another alert is already active.

Example:

```text
ClusterDown = firing
       |
       +----> suppress many dependent InstanceDown alerts
```

This avoids alert floods.

### Silence vs inhibition

| Silence | Inhibition |
|---|---|
| Operator-created suppression | Rule-based suppression |
| Often used for maintenance | Often used for dependency relationships |
| Explicit label matching | Source/target alert relationships |

---

## 39. What is Grafana?

**Priority: 🔴 P1**

Grafana is an observability and visualization platform used to query, visualize and correlate data from many data sources.

With Prometheus, Grafana commonly provides:

- Dashboards
- Panels
- PromQL query interface
- Variables
- Annotations
- Explore
- Alerting
- Transformations

Architecture:

```text
Prometheus
     |
   PromQL
     |
     v
  Grafana
     |
 +---+---+---+
 |   |   |   |
Panel Panel Panel Panel
```

Grafana does not normally replace Prometheus as the metric storage engine.

---

## 40. What is a Grafana dashboard and what is a panel?

**Priority: 🔴 P1**

A **dashboard** is a collection of visualizations.

A **panel** is an individual visualization/query unit inside the dashboard.

Example:

```text
Dashboard: Production Overview

+------------------+------------------+
| CPU              | Memory           |
| Time series      | Time series      |
+------------------+------------------+
| Disk Usage       | Request Rate     |
| Gauge            | Time series      |
+------------------+------------------+
| 5xx Rate         | p95 Latency      |
+------------------+------------------+
```

A panel normally contains a query plus visualization settings.

---

## 41. How do you connect Grafana to Prometheus?

**Priority: 🔴 P1**

Grafana includes a Prometheus data source integration.

Typical configuration:

```text
Grafana
   |
   | HTTP/HTTPS
   v
http://prometheus:9090
```

In a simple environment, the Prometheus URL may be:

```text
http://localhost:9090
```

In a server deployment:

```text
http://prometheus-server:9090
```

After configuring the data source, you can test the connection and use PromQL in panels or Explore.

---

## 42. What are Grafana template variables and why are they useful?

**Priority: 🔴 P1**

Variables make dashboards reusable.

Instead of hardcoding:

```promql
rate(http_requests_total{job="api-prod"}[5m])
```

you can create a variable such as:

```text
$job
```

and query:

```promql
rate(http_requests_total{job=~"$job"}[5m])
```

A dashboard can then show a dropdown:

```text
Job: [ api-prod v ]
```

This allows one dashboard to serve multiple environments or services.

### Multi-value variable

When a variable can select multiple values, a regex matcher is commonly needed:

```promql
{job=~"$job"}
```

rather than:

```promql
{job="$job"}
```

---

## 43. What is Grafana Explore?

**Priority: 🔴 P1**

Explore is useful for ad-hoc investigation without first creating a dashboard panel.

Typical troubleshooting flow:

```text
Incident
  |
  v
Grafana Explore
  |
  +--> Query Prometheus
  +--> Inspect labels
  +--> Change time range
  +--> Compare series
  v
Root-cause investigation
```

For example:

```promql
sum by (status) (
  rate(http_requests_total{job="api"}[5m])
)
```

Once a useful query is validated, it can be added to a dashboard.

---

## 44. What are Grafana annotations?

**Priority: 🔴 P1**

Annotations overlay events on dashboard visualizations.

Useful events include:

- Deployments
- Releases
- Incidents
- Scaling events
- Configuration changes

Example concept:

```text
CPU / Error Rate
     |
     |       /
     |      /\
     |_____/  \____________
           |
        deployment
```

This helps correlate:

```text
Deployment -> Error increase
```

instead of inspecting both systems independently.

---

## 45. What is the difference between Prometheus and Grafana?

**Priority: 🔴 P1 — Must Know**

| Prometheus | Grafana |
|---|---|
| Monitoring system | Visualization/observability platform |
| Scrapes metrics | Queries data sources |
| Stores metrics in its TSDB | Normally does not replace Prometheus metric storage |
| Provides PromQL | Uses PromQL against Prometheus |
| Evaluates Prometheus alert rules | Provides Grafana-managed alerting |
| Service discovery support | Dashboard and visualization layer |

Simple answer:

> Prometheus collects, stores and evaluates metrics; Grafana visualizes and explores metrics and other observability data.

---

# 2. P1 — Advanced PromQL, Rules and Collection

## 46. What is a recording rule and why is it useful?

**Priority: 🔴 P1**

A recording rule precomputes a frequently used query and stores its result as a new time series.

Example:

```yaml
groups:
  - name: api-recording-rules
    rules:
      - record: job:http_requests:rate5m
        expr: |
          sum by (job) (
            rate(http_requests_total[5m])
          )
```

Instead of recalculating the expensive query for every dashboard request, Grafana can query:

```promql
job:http_requests:rate5m
```

### Useful when

- Query is expensive
- Query is reused frequently
- Dashboards are large
- Alert rules repeatedly use the same expression

---

## 47. Recording rule vs alerting rule?

**Priority: 🔴 P1**

| Recording rule | Alerting rule |
|---|---|
| Stores a calculated time series | Creates an alert when expression matches |
| Used for query optimization/reuse | Used for notification workflow |
| `record:` | `alert:` |

Example recording rule:

```yaml
- record: instance:cpu_usage:percent
  expr: |
    100 * (
      1 - avg by (instance) (
        rate(node_cpu_seconds_total{mode="idle"}[5m])
      )
    )
```

Example alert:

```yaml
- alert: HighCPU
  expr: instance:cpu_usage:percent > 80
  for: 10m
```

A good design often uses recording rules to simplify repeated expensive expressions.

---

## 48. What is `rate()` extrapolation and why can results differ from simple subtraction?

**Priority: 🔴 P1 — L2 Detail**

Counters may reset when a process restarts. `rate()` handles counter resets and calculates a per-second rate over a range.

It also considers sample timing rather than simply doing:

```text
(last - first) / exact_window_seconds
```

Prometheus rate calculations account for the observed samples and range boundaries, so a result may differ from a simple manual subtraction.

### Interview takeaway

For Prometheus counters:

```promql
rate(counter_total[5m])
```

is preferred over manually calculating differences in most dashboards/alerts.

---

## 49. What happens to a counter when the application restarts?

**Priority: 🔴 P1**

A counter may reset to zero after process restart.

Example:

```text
Before restart: 10000
After restart:      15
```

PromQL functions such as `rate()` and `increase()` are designed to handle counter resets.

Example:

```promql
rate(http_requests_total[5m])
```

### Interview trap

Do not assume that a decrease in a counter means the application deleted requests. A restart/reset may explain it.

---

## 50. What is the difference between `sum`, `avg`, `max` and `count` in PromQL?

**Priority: 🔴 P1**

Example input:

```text
instance=a -> 20
instance=b -> 40
instance=c -> 60
```

| Function | Result |
|---|---:|
| `sum()` | 120 |
| `avg()` | 40 |
| `max()` | 60 |
| `count()` | 3 |

Use aggregators based on the business meaning.

For total request throughput:

```promql
sum(rate(http_requests_total[5m]))
```

For average CPU across instances:

```promql
avg(cpu_usage)
```

For worst instance:

```promql
max(cpu_usage)
```

---

## 51. What are comparison operators in PromQL?

**Priority: 🔴 P1**

Common operators:

```text
==
!=
>
<
>=
<=
```

Example:

```promql
node_filesystem_avail_bytes < 10 * 1024^3
```

You can use label matchers as well:

```promql
up{job="api"} == 0
```

Comparison operators are commonly used in alert rules.

---

## 52. What are logical/set operators in PromQL?

**Priority: 🔴 P1**

Common set operators include:

```text
and
or
unless
```

Example:

```promql
up == 0 and on(instance) node_uname_info
```

These operators can combine or restrict vectors.

### L2 caution

Vector matching can become complex. Always understand the labels on both sides before using `on()` or `group_left()`/`group_right()`.

---

## 53. What is vector matching in PromQL?

**Priority: 🔴 P1 — Important**

PromQL often performs operations between vectors by matching labels.

Suppose:

```promql
A / B
```

The series need compatible label sets for the operation.

You can explicitly control matching:

```promql
on(instance)
```

Example:

```promql
rate(node_network_receive_bytes_total[5m])
on(instance)
node_uname_info
```

### `group_left` / `group_right`

These are used when one side has additional labels and cardinalities are not one-to-one.

Example pattern:

```promql
metric_a
* on(instance) group_left(role)
metric_b
```

### Interview warning

Do not add `group_left` blindly. First understand why the vectors have different cardinalities.

---

## 54. What are label matchers in PromQL?

**Priority: 🔴 P1**

Common matchers:

```text
=    exact match
!=   not equal
=~   regex match
!~   regex not match
```

Examples:

```promql
up{job="api"}
```

```promql
up{job=~"api|worker"}
```

```promql
up{instance!="10.0.1.10:9100"}
```

```promql
up{job!~"test.*"}
```

### Multi-value Grafana variable

A Grafana multi-select variable generally requires `=~` because the selected values are represented as a regex pattern.

---

## 55. What is `absent()` and when is it useful?

**Priority: 🔴 P1**

`absent()` can detect when an expected series is missing.

Example:

```promql
absent(up{job="payment-api"})
```

This can be useful when you need to distinguish:

```text
metric exists and is bad
```

from:

```text
metric does not exist at all
```

### Important distinction

A missing metric is not necessarily the same as a metric with value `0`.

This distinction is important in alerting and troubleshooting.

---

## 56. What is the difference between missing data and zero in Prometheus?

**Priority: 🔴 P1 — Brain Teaser**

These are different states.

```text
metric{job="api"} 0
```

means the time series exists and its value is zero.

A missing series means there may be no current sample for that label set.

### Why it matters

Suppose:

```promql
request_errors == 0
```

You may think the system has no errors, but if the metric disappeared because the target stopped exposing it, your query behavior can differ from an explicit zero.

When metric existence matters, consider tools such as:

```promql
absent(metric_name)
```

and target health checks.

---

## 57. What is staleness in Prometheus?

**Priority: 🔴 P1 — L2 Detail**

Prometheus does not treat an old time series as if it were continuously receiving its previous value forever.

When a target disappears or a series stops being present, Prometheus can mark the series stale so instant queries do not continue returning it indefinitely as current data.

### Interview relevance

This explains why a query can show:

```text
series existed earlier
```

but later:

```text
series no longer appears as an active result
```

Do not assume that “no series returned” means the metric was explicitly set to zero.

---

## 58. How do you debug a PromQL query that returns no data?

**Priority: 🔴 P1 — Common Scenario**

Use this sequence:

```text
Query returns no data
        |
        +--> Confirm metric name
        |
        +--> Remove label filters
        |
        +--> Check label values
        |
        +--> Check time range
        |
        +--> Check target `up`
        |
        +--> Check scrape errors
        |
        +--> Check whether metric is actually emitted
        |
        +--> Check recording rule / rule evaluation
```

Start simple:

```promql
http_requests_total
```

Then:

```promql
http_requests_total{job="api"}
```

Then inspect:

```promql
count(http_requests_total)
```

### Practical tip

Grafana variable values and regex matchers are frequent causes of an apparently valid query returning no data.

---

## 59. What are relabeling and metric relabeling?

**Priority: 🔴 P1 — Very Important**

Prometheus has different relabeling stages.

### Target relabeling

`relabel_configs` operates on discovered target metadata before scraping.

It is commonly used to:

- Keep/drop targets
- Rewrite addresses
- Map discovery metadata into labels
- Construct the final target label set

### Metric relabeling

`metric_relabel_configs` acts on samples after they are scraped, before ingestion.

It can be used to:

- Drop unwanted metrics
- Rewrite metric labels
- Control ingestion/cardinality

Conceptually:

```text
Service Discovery
      |
      v
relabel_configs
      |
      v
 Target / Scrape
      |
      v
metric_relabel_configs
      |
      v
     TSDB
```

### Important

Do not use metric relabeling as your first solution to every cardinality problem. It is better to avoid generating unnecessary high-cardinality labels at the instrumentation/source whenever possible.

---

## 60. How do you drop a metric using `metric_relabel_configs`?

**Priority: 🔴 P1**

Example:

```yaml
scrape_configs:
  - job_name: "api"
    static_configs:
      - targets:
          - "10.0.1.10:8080"

    metric_relabel_configs:
      - source_labels: [__name__]
        regex: "debug_.*"
        action: drop
```

This prevents matching scraped samples from being ingested.

### Interview distinction

`relabel_configs` and `metric_relabel_configs` are not interchangeable. The former acts on target metadata before the scrape; the latter acts on scraped samples.

---

# 3. P1 — Grafana Dashboard Engineering

## 61. What are Grafana transformations?

**Priority: 🔴 P1**

Transformations modify query results inside Grafana before visualization.

Typical use cases include:

- Rename fields
- Join query results
- Reduce values
- Organize fields
- Filter data
- Calculate derived values

Conceptually:

```text
Data source
    |
    v
Query result
    |
    v
Transformation
    |
    v
Visualization
```

### Important

A transformation does not change the data stored in Prometheus. It changes how Grafana processes the returned data.

---

## 62. How would you design a production monitoring dashboard?

**Priority: 🔴 P1 — Scenario**

Do not put 50 random graphs on the first page.

A useful hierarchy is:

```text
Level 1: Service health
    |
    +-- Availability
    +-- Error rate
    +-- Traffic
    +-- Latency

Level 2: Infrastructure
    |
    +-- CPU
    +-- Memory
    +-- Disk
    +-- Network

Level 3: Deep diagnostics
    |
    +-- JVM / Go / Python metrics
    +-- DB metrics
    +-- Queue metrics
```

For an API dashboard, start with:

- Request rate
- 4xx/5xx rate
- p95/p99 latency
- Availability
- Saturation
- Dependency health

### Interview principle

A dashboard should help answer:

> Is the service healthy, what is failing, and where should I investigate next?

---

## 63. How do you create a reusable Grafana dashboard for multiple environments?

**Priority: 🔴 P1**

Use template variables rather than hardcoded labels.

Example variables:

```text
$environment
$cluster
$namespace
$service
$instance
```

Query:

```promql
sum by (service) (
  rate(http_requests_total{
    environment=~"$environment",
    service=~"$service"
  }[5m])
)
```

Dashboard structure:

```text
Environment: [dev | test | prod]
Service:     [api | worker | all]
Instance:    [all | web01 | web02]
```

This avoids maintaining separate dashboards for every environment.

---

## 64. What is the difference between dashboard-level time range and panel query range?

**Priority: 🔴 P1**

Grafana has an overall dashboard time range such as:

```text
Last 6 hours
```

PromQL queries can also use explicit range selectors:

```promql
rate(http_requests_total[5m])
```

The Grafana dashboard time range determines the period being displayed/evaluated by the panel request, while PromQL range selectors define how much historical data a function such as `rate()` uses for each evaluation point.

### Interview trap

`Last 24 hours` in Grafana does not mean `rate(metric[24h])`.

These represent different concepts.

---

## 65. What is a good way to avoid slow Grafana dashboards?

**Priority: 🔴 P1**

Main controls:

1. Avoid extremely high-cardinality queries.
2. Reduce unnecessary panels.
3. Use recording rules for expensive repeated queries.
4. Use appropriate time ranges.
5. Avoid regex-heavy queries over huge datasets when not needed.
6. Aggregate before displaying when detailed series are unnecessary.
7. Use sensible refresh intervals.
8. Avoid loading hundreds of panels on one dashboard.

Example:

Instead of repeatedly calculating:

```promql
sum by (job) (
  rate(http_requests_total[5m])
)
```

create a recording rule and query the recorded metric.

---

# 4. P2 — Service Discovery, Relabeling and Rules

## 66. How does Prometheus discover AWS EC2 instances?

**Priority: 🟠 P2**

Prometheus can use service discovery mechanisms for cloud infrastructure.

For AWS EC2, the configuration can use EC2 discovery and then relabel instances based on metadata/tags.

Conceptual flow:

```text
AWS EC2 API
    |
    v
EC2 Service Discovery
    |
    v
Prometheus discovered targets
    |
    v
relabel_configs
    |
    v
Scrape exporter/application
```

A common production pattern is:

```text
EC2 instances carry tags
        |
        v
Prometheus discovers instances
        |
        v
Keep only required environment/application tags
        |
        v
Scrape :9100
```

This is preferable to manually maintaining IP addresses for a dynamic fleet.

---

## 67. How would you monitor an Auto Scaling Group?

**Priority: 🟠 P2 — Scenario**

Do not build the monitoring around a fixed list of instance IP addresses.

Use:

```text
ASG
 |
 +--> EC2 service discovery
          |
          +--> filter by tags
          |
          +--> scrape Node Exporter
          |
          +--> record/alert
```

Useful dimensions:

```text
environment
application
auto_scaling_group
instance
region
```

When an instance is replaced, service discovery can detect the new target without manual edits.

---

## 68. How would you monitor Kubernetes workloads with Prometheus?

**Priority: 🟠 P2**

A common architecture includes:

```text
Kubernetes API / Pod discovery
          |
          v
     Prometheus
          |
    +-----+------+
    |            |
 kube-state-    node / kubelet
 metrics        metrics
```

Common sources include:

- kube-state-metrics
- kubelet/cAdvisor-related metrics
- application `/metrics` endpoints
- node-level exporters

Useful monitoring layers:

```text
Cluster
  |
  +-- Node
  +-- Namespace
  +-- Deployment
  +-- Pod
  +-- Container
  +-- Application
```

### L2 point

Kubernetes discovery finds targets, but it does not automatically make every application observable. Applications still need suitable instrumentation/exporters.

---

## 69. What is the purpose of `relabel_configs` in Kubernetes monitoring?

**Priority: 🟠 P2**

Kubernetes discovery returns a large set of metadata labels.

Relabeling can transform metadata into useful Prometheus labels such as:

```text
namespace
pod
container
service
node
```

It can also drop targets that should not be scraped.

Conceptual flow:

```text
Kubernetes metadata
       |
       v
relabel_configs
       |
       +--> Keep required targets
       +--> Drop unwanted targets
       +--> Map metadata to labels
       v
Prometheus target
```

---

## 70. What is a black-box monitoring vs white-box monitoring?

**Priority: 🟠 P2**

### Black-box monitoring

Checks what an external observer sees.

Example:

```text
HTTP probe -> https://api.example.com/health
```

Blackbox Exporter is a common tool.

### White-box monitoring

Measures internal application/system behavior.

Examples:

```text
request_count
latency
queue_depth
garbage_collection_time
DB_connections
```

### Best practice

Use both:

```text
External availability
       +
Internal application metrics
```

A service may expose healthy internal metrics while its public endpoint is unreachable due to DNS, load balancer or network issues.

---

## 71. What is the purpose of `external_labels` in Prometheus?

**Priority: 🟠 P2**

`external_labels` adds labels to all metrics sent to external systems and is commonly used to identify a Prometheus instance or cluster in environments with federation/remote systems.

Example:

```yaml
global:
  external_labels:
    cluster: prod-east
    replica: prometheus-01
```

This helps downstream systems distinguish:

```text
prod-east / prometheus-01
prod-west / prometheus-02
```

### L2 use case

External labels are especially important when multiple Prometheus servers send data to a shared remote system or when HA replicas need to be distinguishable.

---

## 72. What is remote write?

**Priority: 🟠 P2 — Important**

Remote write sends Prometheus samples to a compatible remote metrics backend.

Conceptually:

```text
Target
  |
  v
Prometheus
  |
  +----> Local TSDB
  |
  +----> remote_write
             |
             v
       Remote metrics backend
```

Use cases:

- Long-term storage
- Centralized metrics
- Multi-cluster monitoring
- Scaling beyond one local Prometheus

### Important

Remote write is for sending samples outward. It is not the same concept as Grafana querying Prometheus.

---

## 73. What is federation in Prometheus?

**Priority: 🟠 P2**

Federation allows one Prometheus server to scrape selected metrics from another Prometheus server.

Conceptually:

```text
Prometheus A ---->
                  \
                   Prometheus Global
                  /
Prometheus B ---->
```

A global Prometheus may collect a selected subset of metrics from regional Prometheus servers.

### When useful

- Hierarchical monitoring
- Central overview
- Cross-region aggregation
- Collecting a curated subset of series

### Federation vs remote write

| Federation | Remote write |
|---|---|
| Prometheus scrapes another Prometheus | Prometheus sends samples to remote backend |
| Pull-oriented | Push/export-oriented from local Prometheus |
| Can select metrics exposed by source | Designed for remote sample ingestion |

---

## 74. What is a Prometheus rule group and `evaluation_interval`?

**Priority: 🟠 P2**

Rule groups contain recording and/or alerting rules.

Example:

```yaml
groups:
  - name: application-rules
    interval: 30s
    rules:
      - record: job:http_requests:rate5m
        expr: sum by(job) (rate(http_requests_total[5m]))
```

The group interval controls how often rules in that group are evaluated.

Global default:

```yaml
global:
  evaluation_interval: 15s
```

### Why does it matter?

If an alert should react within roughly one evaluation cycle, an unnecessarily large evaluation interval can delay detection.

---

## 75. What is the difference between scrape interval and rule evaluation interval?

**Priority: 🟠 P2**

| Setting | Controls |
|---|---|
| `scrape_interval` | How often metrics are collected |
| `evaluation_interval` | How often rules are evaluated |

Example:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
```

They are independent settings.

### Scenario

A metric updates every 15 seconds but your alert rules evaluate every 1 minute.

The alert may not react immediately to a newly breached condition because rule evaluation is less frequent.

---

# 5. P2 — Troubleshooting and Production Operations

## 76. Prometheus is running but targets show `DOWN`. How do you troubleshoot?

**Priority: 🟠 P2 — Interview Scenario**

Use this order:

### 1. Check the target page

Prometheus:

```text
Status -> Targets
```

Read the exact scrape error.

### 2. Test from the Prometheus host

```bash
curl http://TARGET_IP:9100/metrics
```

### 3. Check DNS

```bash
getent hosts target.example.com
```

### 4. Check connectivity

```bash
nc -vz 10.0.1.10 9100
```

### 5. Check exporter/service

```bash
systemctl status node_exporter
ss -lntp | grep 9100
```

### 6. Check firewall/security groups

Verify both source and destination rules.

### 7. Check TLS/authentication

If `/metrics` is protected, verify the Prometheus configuration.

### Key principle

Start with the **exact scrape error**, not assumptions.

---

## 77. Grafana says `No data`, but Prometheus has data. What do you check?

**Priority: 🟠 P2 — Common Scenario**

Check:

```text
1. Correct Prometheus data source?
2. Data source reachable?
3. Correct time range?
4. Query works in Prometheus UI?
5. Dashboard variable values?
6. Query label matchers?
7. Regex / multi-value matcher?
8. Panel transformation?
9. Query step / resolution?
```

A useful isolation step is to copy the panel query into Grafana Explore or Prometheus directly.

### Common mistake

Dashboard variable:

```text
$job = api-prod|api-stage
```

but query uses:

```promql
{job="$job"}
```

For multi-select variables, this often needs:

```promql
{job=~"$job"}
```

---

## 78. Prometheus disk usage is increasing rapidly. What would you investigate?

**Priority: 🟠 P2 — Scenario**

Possible causes:

- High scrape frequency
- Too many targets
- High-cardinality labels
- Excessive metric ingestion
- Long retention period
- Unexpected exporter/application metrics
- Remote-write buffering/backpressure

Investigate:

```text
Number of series
Samples ingested
Top metric names by series count
Scrape frequency
Retention configuration
Target count
```

A useful principle:

> Do not solve a storage problem only by increasing disk. First identify why the number of samples or series increased.

---

## 79. How do you identify high-cardinality metrics?

**Priority: 🟠 P2**

A useful investigation starts with Prometheus's own series/cardinality information and then drills into metric/label design.

Look for:

```text
metrics with huge label combinations
high-churn labels
request IDs
user IDs
URLs with unique paths/query strings
```

The solution is often:

```text
bad label design
      |
      v
reduce/remove label
      |
      v
lower cardinality
```

not:

```text
bad label design -> buy more disk
```

---

## 80. What is retention in Prometheus?

**Priority: 🟠 P2**

Retention controls how long Prometheus keeps local data.

A common example is:

```bash
--storage.tsdb.retention.time=15d
```

Retention affects local storage requirements.

### Important

Retention is not the same as dashboard time range.

```text
Retention = how long data is stored
Dashboard range = how much data Grafana currently asks to display
```

If data has aged out of local storage, Grafana cannot retrieve it from that Prometheus unless another long-term system contains it.

---

## 81. What is TSDB in Prometheus?

**Priority: 🟠 P2**

TSDB means **time-series database**.

Prometheus stores time-series samples locally in its TSDB.

Conceptually:

```text
Metric + labels
      |
      v
Time-series samples
      |
      v
Prometheus TSDB
```

Important operational concerns include:

- Disk capacity
- Retention
- Compaction
- Write throughput
- Query performance
- Number of active series

---

## 82. How do you safely reload Prometheus configuration?

**Priority: 🟠 P2**

First validate the configuration:

```bash
promtool check config prometheus.yml
```

Then reload the running Prometheus process using your deployment's supported reload mechanism.

Depending on how Prometheus is deployed, this may be done using its reload endpoint or process signal.

### Safe operational sequence

```text
Edit
  |
  v
Validate with promtool
  |
  v
Reload
  |
  v
Check logs / targets / rules
```

### Interview point

Never treat “configuration file edited successfully” as proof that Prometheus accepted it.

---

## 83. What is `promtool` used for?

**Priority: 🟠 P2**

`promtool` is a command-line utility distributed with Prometheus.

Common uses include:

```bash
promtool check config prometheus.yml
```

and validating rule files:

```bash
promtool check rules rules/*.yml
```

It can also support testing workflows such as unit testing Prometheus rules.

### DevOps use case

Integrate validation into CI:

```text
Git commit
   |
   v
CI
   |
   +--> promtool check config
   +--> promtool check rules
   |
   v
Deploy
```

---

## 84. How should Prometheus and Grafana be deployed using configuration as code?

**Priority: 🟠 P2**

Avoid manually creating every production object from the UI.

A common structure is:

```text
monitoring/
├── prometheus/
│   ├── prometheus.yml
│   ├── rules/
│   └── alerts/
│
└── grafana/
    ├── provisioning/
    │   ├── datasources/
    │   └── dashboards/
    └── dashboards/
```

Version-control:

- Prometheus configuration
- Rule files
- Alert rules
- Grafana provisioning
- Dashboard JSON or dashboard-as-code representation where appropriate

### Benefit

```text
Git -> Review -> CI validation -> Deployment
```

rather than undocumented manual UI changes.

---

## 85. How do you provision a Grafana data source?

**Priority: 🟠 P2**

Grafana supports provisioning configuration from files.

Conceptual example:

```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
```

This allows a repeatable environment:

```text
New Grafana instance
       |
       v
Provisioning files
       |
       v
Prometheus data source configured automatically
```

### L2 point

This reduces manual configuration drift across dev, staging and production.

---

## 86. How do you provision Grafana dashboards?

**Priority: 🟠 P2**

Grafana supports dashboard provisioning through configuration files that point to dashboard definitions.

Conceptually:

```text
Git
 |
 +--> dashboard JSON
 |
 +--> dashboard provider config
 |
 v
Grafana
```

A provider configuration tells Grafana where dashboard definitions are stored and which organization/folder they belong to.

### Best practice

Treat dashboard definitions as code when they are production-critical.

---

## 87. What is dashboard drift and how do you prevent it?

**Priority: 🟠 P2**

Dashboard drift occurs when production dashboards differ from their version-controlled definition.

Example:

```text
Git dashboard != Production dashboard
```

Prevent it with:

- Version-controlled dashboards
- Provisioning
- Code review
- Automated deployment
- Avoiding undocumented UI-only changes

### Interview answer

The goal is to make the monitoring configuration reproducible just like application infrastructure.

---

## 88. What is alert fatigue and how do you reduce it?

**Priority: 🟠 P2**

Alert fatigue occurs when engineers receive too many alerts, especially low-value or repetitive alerts.

Reduce it using:

```text
Good thresholds
   +
for duration
   +
Alert grouping
   +
Inhibition
   +
Silences for maintenance
   +
Severity/routing
   +
Meaningful runbooks
```

Bad alert:

```text
CPU > 70% for 30 seconds
```

Better question:

> Is CPU saturation persistent and causing user-visible impact?

Monitoring should generate **actionable** alerts, not every possible anomaly.

---

## 89. How do you design an alert for a service being down?

**Priority: 🟠 P2**

For exporter/target health, a common starting point is:

```promql
up{job="api"} == 0
```

Then add an appropriate `for` duration depending on the operational requirement:

```yaml
for: 5m
```

For application availability, you may need a different SLI-style query, such as successful request rate or black-box probe success.

### Important

`up == 0` tells you that Prometheus could not successfully scrape the target. It does not by itself prove that the user-facing service is unavailable.

---

## 90. How would you alert on high error rate but avoid noise during low traffic?

**Priority: 🟠 P2 — Brain Teaser**

A percentage-only alert can be misleading.

Example:

```text
1 request
1 error
= 100% error rate
```

You may want both:

```promql
error_rate > 5%
and
request_rate > minimum_expected_rate
```

For example:

```promql
(
  sum(rate(http_requests_total{status=~"5.."}[5m]))
  /
  sum(rate(http_requests_total[5m]))
) > 0.05
and
sum(rate(http_requests_total[5m])) > 1
```

The exact minimum depends on your service.

### Interview principle

Always consider **volume and percentage together**.

---

# 6. P2 — Alerting and Incident Scenarios

## 91. A Prometheus alert is firing but no Slack message arrives. What do you check?

**Priority: 🟠 P2 — Scenario**

Trace the flow:

```text
PromQL condition
      |
      v
Alert rule firing?
      |
      v
Prometheus -> Alertmanager?
      |
      v
Alertmanager received alert?
      |
      v
Route matched?
      |
      v
Silence/inhibition?
      |
      v
Receiver configured?
      |
      v
Slack/webhook reachable?
```

Check:

- Prometheus alert state
- Prometheus Alertmanager configuration
- Alertmanager UI/API
- Route matchers
- Grouping timing
- Silences
- Inhibition rules
- Receiver configuration
- Network/TLS/authentication

### Interview answer

Do not jump directly to Slack. Trace the alert end-to-end.

---

## 92. An alert fires and clears repeatedly every few minutes. What is alert flapping?

**Priority: 🟠 P2**

Flapping means the condition repeatedly transitions between firing and resolved states.

Common causes:

- Threshold too close to normal operating range
- Short-lived spikes
- No `for` duration
- Noisy metric
- Scrape instability
- Badly designed query

Example improvement:

```yaml
for: 10m
```

instead of alerting immediately on every small threshold crossing.

### But do not blindly increase `for`

A critical outage alert may need rapid action. Tune the duration to the incident's operational meaning.

---

## 93. What is an SLI, SLO and SLA in a Prometheus/Grafana context?

**Priority: 🟠 P2**

| Term | Meaning |
|---|---|
| SLI | Measured indicator |
| SLO | Target for the indicator |
| SLA | Business/customer agreement, often with consequences |

Example:

```text
SLI = successful requests / total requests
SLO = 99.9% success over a defined period
SLA = contractual commitment based on agreed objectives
```

Prometheus can collect and calculate the SLI; Grafana can visualize it; alerting can monitor SLO-related conditions.

---

## 94. How can Prometheus/Grafana be used for the four golden signals?

**Priority: 🟠 P2**

The four golden signals are:

| Signal | Example metric |
|---|---|
| Latency | Request duration histogram |
| Traffic | Requests/sec |
| Errors | 4xx/5xx or failed requests |
| Saturation | CPU, memory, queue depth, connection pressure |

A service dashboard should make these visible quickly.

Example:

```promql
Traffic:
sum(rate(http_requests_total[5m]))

Errors:
sum(rate(http_requests_total{status=~"5.."}[5m]))

Latency:
histogram_quantile(0.95, ...)

Saturation:
CPU / memory / queue metrics
```

---

## 95. How would you monitor an API end to end?

**Priority: 🟠 P2 — Scenario**

Use multiple layers:

```text
External user
    |
    v
Blackbox / Load Balancer probe
    |
    v
Application
    |
    +--> Request rate
    +--> Error rate
    +--> Latency
    |
    +--> DB health
    +--> Cache health
    +--> Queue depth
    |
    v
Infrastructure
    +--> CPU
    +--> Memory
    +--> Disk
    +--> Network
```

### Why multiple layers?

A host can be healthy while the application is broken.

An application can be healthy while the public endpoint is broken.

A public endpoint can be healthy while the database is approaching saturation.

Good monitoring connects these layers.

---

# 7. P3 — High Availability, Scaling and Architecture

## 96. How do you make Prometheus highly available?

**Priority: 🟢 P3**

A common pattern is to run multiple Prometheus replicas scraping the same targets.

```text
             +--> Prometheus A
Targets -----+
             +--> Prometheus B
```

Then use a system designed to handle duplicate/replica data, such as a compatible remote storage/query layer, depending on architecture.

Important considerations:

- Duplicate samples
- External labels
- Alert deduplication strategy
- Query layer
- Remote storage
- Failure domains

### Important

Running two Prometheus servers alone does not magically make a complete HA monitoring platform. You need to consider storage, querying and alerting semantics.

---

## 97. What are Thanos and Grafana Mimir used for?

**Priority: 🟢 P3**

They are examples of systems used to extend Prometheus-style monitoring for larger-scale or long-term architectures.

Common goals:

- Long-term metrics storage
- Global querying
- Horizontal scaling
- Multi-cluster monitoring
- High availability

Conceptually:

```text
Prometheus A ----\
Prometheus B -----+----> Scalable metrics platform
Prometheus C ----/
                       |
                       v
                    Grafana
```

### Interview point

Do not say “Prometheus cannot scale.” A more accurate answer is that a single local Prometheus instance has specific scale and retention boundaries, and large environments commonly add a scalable metrics architecture around it.

---

## 98. What is a long-term metrics storage architecture?

**Priority: 🟢 P3**

A common architecture is:

```text
Targets
  |
  v
Prometheus
  |
  +--> Local TSDB for local/short-term access
  |
  +--> Remote Write
           |
           v
   Long-term metrics backend
           |
           v
        Grafana
```

Benefits:

- Longer retention
- Centralized querying
- Multi-cluster visibility
- Reduced dependence on one local disk

The exact backend can vary by environment.

---

## 99. What is the role of external labels in a multi-Prometheus architecture?

**Priority: 🟢 P3**

Imagine:

```text
prod-east / Prometheus A
prod-west / Prometheus B
```

Without clear identifying labels, downstream systems may not know which source produced a sample.

Typical labels might include:

```yaml
external_labels:
  cluster: prod-east
  replica: prometheus-a
```

This is useful for federation, remote systems and HA/deduplication architectures depending on the platform.

---

## 100. How would you reduce Prometheus query load in a large environment?

**Priority: 🟢 P3 — Scenario**

Use a combination of:

```text
1. Recording rules
2. Better aggregation
3. Lower-cardinality labels
4. Avoiding expensive regex where unnecessary
5. Reasonable dashboard refresh intervals
6. Query/result caching where the architecture supports it
7. Scalable remote query architecture
```

Example:

Instead of every Grafana panel calculating a complex expression independently:

```promql
complex_expression(...)
```

record:

```text
service:request_rate5m
```

and query the recorded result.

---

## 101. How would you monitor Prometheus itself?

**Priority: 🟢 P3**

Prometheus exposes its own metrics.

Useful areas include:

- Scrape health
- Query performance
- Rule evaluation
- TSDB/storage behavior
- Active series
- Remote-write health
- Target counts

A production monitoring stack should monitor the monitoring system itself.

### Principle

```text
Business systems
      |
      v
Monitoring system
      |
      v
Monitor the monitor
```

Otherwise an outage in the monitoring stack can silently remove visibility exactly when you need it.

---

## 102. What is the difference between monitoring the application and monitoring the user experience?

**Priority: 🟢 P3**

Application monitoring may show:

```text
CPU = 40%
Memory = 50%
Error rate = 0.2%
```

But users may still experience:

```text
DNS failure
Slow load balancer
TLS issue
Broken frontend dependency
```

User-experience monitoring often adds:

- Black-box probing
- Synthetic checks
- Frontend/browser metrics
- End-to-end latency

### Interview takeaway

Infrastructure health is not automatically equal to user experience.

---

# 8. P3 — Frequently Asked Interview Scenarios and Brain Teasers

## 103. Prometheus can scrape a target manually with `curl`, but `up == 0`. What could be wrong?

**Priority: 🟢 P3 — Brain Teaser**

Possibilities include:

- The `curl` test was run from a different host/network location.
- Prometheus is using a different DNS resolution path.
- TLS/authentication settings differ.
- Prometheus target configuration points to another address/port.
- Network ACL/security-group rules differ by source.
- The target is slow and exceeds `scrape_timeout`.
- Relabeling modified the target unexpectedly.

Correct interview answer:

> “A successful curl from my workstation proves the endpoint works from my workstation. It does not prove it is reachable and scrapeable from the Prometheus server.”

---

## 104. Grafana shows stale-looking data. Is Grafana necessarily caching old metrics?

**Priority: 🟢 P3 — Brain Teaser**

No.

Possible causes include:

- Dashboard time range
- Refresh interval
- Query range/step
- Prometheus scrape interval
- Recording rule evaluation interval
- Query result behavior
- Target not updating

Trace the path:

```text
Target metric timestamp
        |
        v
Prometheus scrape
        |
        v
PromQL result
        |
        v
Grafana query
        |
        v
Panel rendering
```

Do not blame Grafana before checking where the data stopped updating.

---

## 105. A CPU alert fires on every server after one bad deployment. Why might this be better handled with routing/grouping than 100 individual pages?

**Priority: 🟢 P3**

If 100 hosts are affected by one common failure, paging 100 times creates noise.

Use labels such as:

```text
cluster
application
environment
severity
```

and configure Alertmanager grouping/routing.

You want:

```text
100 related alerts
      |
      v
1 meaningful notification group
```

while still retaining the individual alert details for investigation.

---

## 106. A service has 99% error rate, but only one request arrived in the last five minutes. Should you immediately page?

**Priority: 🟢 P3 — Brain Teaser**

Not necessarily.

One request failing gives:

```text
100% error rate
```

but provides little statistical evidence compared with:

```text
10,000 requests
8,000 failures
```

Alert design should consider traffic volume and impact.

A more robust condition may combine:

```text
error percentage
+
minimum traffic
```

or use a service-specific SLO/error-budget strategy.

---

## 107. A metric exists in Prometheus, but Grafana cannot display it using a variable. What could be wrong?

**Priority: 🟢 P3**

Check:

```text
1. Variable query
2. Variable data source
3. Label name
4. Label values
5. Multi-value handling
6. Regex matcher
7. Time range
8. Permissions/data-source access
```

For multi-value selections, this is often correct:

```promql
{job=~"$job"}
```

rather than:

```promql
{job="$job"}
```

---

## 108. A dashboard shows 0% CPU because the query uses `node_cpu_seconds_total`. What is wrong?

**Priority: 🟢 P3 — Brain Teaser**

`node_cpu_seconds_total` is a cumulative counter.

It is not CPU percentage by itself.

Use a rate and calculate non-idle time, for example:

```promql
100 * (
  1 - avg by (instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  )
)
```

The key issue is **counter vs derived utilization**.

---

## 109. An exporter exposes 500,000 series. Should you simply increase Prometheus memory?

**Priority: 🟢 P3 — Brain Teaser**

Not as the first response.

First ask:

```text
Why are there 500,000 series?
```

Inspect labels and metric design.

Potential root causes:

- Unbounded labels
- Per-request dimensions
- Dynamic URLs
- User/session IDs
- Too many targets
- Duplicate instrumentation

Correct approach:

```text
Identify cardinality source
        |
        v
Redesign/drop unnecessary dimensions
        |
        v
Scale resources only if still required
```

---

## 110. A production Prometheus restart causes a monitoring gap. What architectural improvement would you consider?

**Priority: 🟢 P3 — Scenario**

Possible improvements depend on business requirements:

- Run multiple Prometheus replicas
- Use remote long-term storage
- Use a scalable query layer
- Separate failure domains
- Provision configuration reproducibly
- Monitor Prometheus itself

The exact architecture should be based on:

```text
Required availability
Required retention
Data volume
Number of clusters
Query load
Operational complexity
```

Do not automatically deploy the most complex architecture. Design for the actual requirement.

---

# 9. Frequently Used Prometheus Queries and Grafana Patterns

## Common PromQL Reference

### Target health

```promql
up
```

```promql
up == 0
```

### Request rate

```promql
sum(rate(http_requests_total[5m]))
```

### Error rate

```promql
sum(rate(http_requests_total{status=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m]))
```

### CPU usage

```promql
100 * (
  1 - avg by (instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  )
)
```

### Memory usage

```promql
100 * (
  1 - node_memory_MemAvailable_bytes
      / node_memory_MemTotal_bytes
)
```

### Root filesystem usage

```promql
100 * (
  1 - node_filesystem_avail_bytes{mountpoint="/"}
      / node_filesystem_size_bytes{mountpoint="/"}
)
```

### Network receive rate

```promql
rate(node_network_receive_bytes_total[5m])
```

### Network transmit rate

```promql
rate(node_network_transmit_bytes_total[5m])
```

### p95 latency

```promql
histogram_quantile(
  0.95,
  sum by (le) (
    rate(http_request_duration_seconds_bucket[5m])
  )
)
```

### Count of active targets

```promql
count(up)
```

### Down targets by job

```promql
sum by (job) (up == 0)
```

---

# 10. Frequently Used Prometheus Configuration Patterns

## Basic static scrape

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "node"
    static_configs:
      - targets:
          - "10.0.1.10:9100"
          - "10.0.1.11:9100"
```

## Job-specific scrape interval

```yaml
scrape_configs:
  - job_name: "fast-api"
    scrape_interval: 5s
    static_configs:
      - targets:
          - "10.0.1.10:8080"
```

## Basic alert rule

```yaml
groups:
  - name: node-alerts
    rules:
      - alert: NodeDown
        expr: up{job="node"} == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Node is down"
          description: "Prometheus cannot scrape {{ $labels.instance }}"
```

## Basic recording rule

```yaml
groups:
  - name: api-recordings
    rules:
      - record: api:http_requests:rate5m
        expr: |
          sum by (job) (
            rate(http_requests_total[5m])
          )
```

## Basic metric drop

```yaml
metric_relabel_configs:
  - source_labels: [__name__]
    regex: "debug_.*"
    action: drop
```

## Basic Grafana dashboard variable query

For a variable representing job names, a common Prometheus label-value query is based on the `job` label.

Use the variable in a panel:

```promql
sum by (status) (
  rate(http_requests_total{job=~"$job"}[5m])
)
```

---

# 11. Frequently Used Monitoring Tools Around Prometheus/Grafana

| Tool | Typical role | Model / behavior |
|---|---|---|
| Prometheus | Metrics collection and alert evaluation | Pull-oriented scraping |
| Grafana | Visualization and observability UI | Queries data sources |
| Alertmanager | Alert routing/grouping/silencing | Receives alerts |
| Node Exporter | Host metrics | Exposes metrics for scraping |
| Blackbox Exporter | External probing | Prometheus scrapes probe results |
| kube-state-metrics | Kubernetes object/state metrics | Prometheus scrapes exporter |
| Pushgateway | Short-lived job metric gateway | Jobs push to gateway; Prometheus scrapes gateway |
| Thanos | Scalable Prometheus architecture | Adds long-term/global querying capabilities |
| Grafana Mimir | Scalable metrics backend | Prometheus-compatible metrics platform |
| Loki | Logs | Optimized for log data |
| OpenTelemetry Collector | Telemetry collection/processing | Receives/processes/exports telemetry |

### Important interview distinction

Prometheus, exporters, Alertmanager and Grafana are different components with different responsibilities. Avoid describing the entire stack as one product.

---

# 12. Quick Revision Table

| Topic | Key Point |
|---|---|
| Prometheus | Pull-based monitoring and time-series database |
| Grafana | Visualization/observability platform |
| Alertmanager | Routes, groups, silences and inhibits alerts |
| Exporter | Converts/exposes system metrics for Prometheus |
| Node Exporter | Linux host metrics |
| Blackbox Exporter | External endpoint probing |
| `up` | Scrape success indicator |
| Counter | Usually increases, except resets |
| Gauge | Can increase/decrease |
| Histogram | Distribution using buckets |
| Summary | Client-side summary/quantiles |
| Label | Dimension of a time series |
| Cardinality | Number of unique series |
| `rate()` | Per-second average counter rate |
| `irate()` | Short-term rate using recent samples |
| `increase()` | Counter increase over a range |
| `sum by()` | Aggregate while retaining grouping labels |
| `without()` | Aggregate while excluding listed labels from grouping |
| `absent()` | Detect missing series |
| `scrape_interval` | How often targets are scraped |
| `scrape_timeout` | Maximum scrape wait time |
| `evaluation_interval` | How often rules are evaluated by default |
| Recording rule | Precomputes and stores a query result |
| Alerting rule | Evaluates a condition and creates an alert |
| `for` | Requires alert condition to persist before firing |
| `relabel_configs` | Modifies target metadata before scrape |
| `metric_relabel_configs` | Modifies/drops scraped samples before ingestion |
| Service discovery | Dynamically finds targets |
| Remote write | Sends samples to remote metrics backend |
| Federation | Scrapes selected metrics from another Prometheus |
| Retention | How long local data is retained |
| TSDB | Prometheus time-series storage engine |
| Grafana variable | Makes dashboards reusable/dynamic |
| Grafana Explore | Ad-hoc query/troubleshooting interface |
| Grafana transformation | Changes returned query data before visualization |
| Annotation | Marks events on dashboards |
| Provisioning | Configuration as code for Grafana resources |
| Alert grouping | Combines related notifications |
| Silence | Temporarily suppresses matching notifications |
| Inhibition | Suppresses alerts based on other firing alerts |
| SLI | Measured service indicator |
| SLO | Target for an SLI |
| SLA | Business/customer agreement |
| Golden signals | Latency, traffic, errors, saturation |
| Cardinality control | Avoid unbounded/high-cardinality labels |
| HA Prometheus | Usually multiple replicas plus suitable downstream architecture |
| Thanos/Mimir | Scalable/long-term Prometheus-compatible architectures |

---

# Interview Checklist

Before an L2 interview, make sure you can explain without memorizing definitions:

```text
Prometheus architecture
        |
        +--> Pull model
        +--> Scrape targets
        +--> Exporters
        +--> Service discovery
        +--> Labels/cardinality
        +--> TSDB/retention
        |
        v
PromQL
        |
        +--> rate / irate / increase
        +--> aggregation
        +--> vector matching
        +--> histograms
        +--> missing data
        +--> recording rules
        |
        v
Alerting
        |
        +--> Alerting rules
        +--> for
        +--> Alertmanager
        +--> grouping
        +--> silences
        +--> inhibition
        |
        v
Grafana
        |
        +--> Data source
        +--> Dashboards
        +--> Panels
        +--> Variables
        +--> Explore
        +--> Transformations
        +--> Annotations
        +--> Alerting
        +--> Provisioning
        |
        v
Production
        |
        +--> High availability
        +--> Remote write
        +--> Federation
        +--> Long-term storage
        +--> Cardinality control
        +--> Monitoring the monitoring stack
```

## Final L2 Interview Rule

When troubleshooting Prometheus/Grafana, always trace the complete path:

```text
Target
  |
  v
Exporter / Application metrics
  |
  v
Network / Discovery
  |
  v
Prometheus scrape
  |
  v
PromQL / Recording Rule
  |
  v
Grafana / Alert Rule
  |
  v
Alertmanager / Notification
```

Do not assume the problem is in Grafana just because the dashboard looks wrong. Find the **first point in the data path where reality diverges from expectation**.
