# 05 — Cardinality & Churn

## 1. Cardinality

Cardinality is the number of unique time series.

A time series is:

```text
metric name + unique label values
```

Example:

```text
http_requests_total{method="GET",status="200"}
```

is one series.

Changing `status` creates another series.

## 2. Cardinality Calculation

Suppose:

```text
service = 5
method  = 4
status  = 5
```

Potential combinations:

```text
5 × 4 × 5 = 100
```

Add:

```text
endpoint = 100
```

Now:

```text
5 × 4 × 5 × 100 = 10,000
```

Adding a high-cardinality `user_id` dimension can multiply this dramatically.

## 3. Good Labels

```text
http_requests_total{
  service="employee-api",
  method="GET",
  route="/employees/:id",
  status="200"
}
```

These labels generally have bounded, meaningful values.

## 4. Dangerous Labels

Common examples:

```text
user_id
request_id
session_id
trace_id
full URL
random UUID
```

Example:

```text
http_requests_total{
  request_id="9f7a..."
}
```

Every request could create another series.

## 5. Cardinality Explosion

```text
More unique label values
          ↓
More label combinations
          ↓
More time series
          ↓
Higher cardinality
          ↓
More memory/index/disk/ingestion work
```

## 6. What is Churn?

Churn describes how quickly new series are created.

```text
10:00  request_id=AAA → new series
10:01  request_id=BBB → new series
10:02  request_id=CCC → new series
10:03  request_id=DDD → new series
```

You can have moderate active cardinality but very high churn.

## 7. Cardinality vs Churn

| Concept | Meaning |
|---|---|
| Cardinality | Number of unique series |
| Active series | Series currently receiving samples |
| Churn | Rate of new series creation |

## 8. Kubernetes Example

Potential dimensions:

```text
namespace
pod
service
container
endpoint
method
status
```

Adding:

```text
request_id
```

can create a much larger number of series.

## 9. Reduce Cardinality Before Storage

Example:

```yaml
metric_relabel_configs:
  - action: labeldrop
    regex: "request_id|session_id"
```

Drop entire metrics when appropriate:

```yaml
metric_relabel_configs:
  - source_labels: [__name__]
    regex: "debug_.*"
    action: drop
```

## 10. vmagent Series Limits

Useful controls include:

```text
series_limit
-promscrape.seriesLimitPerTarget
-remoteWrite.maxHourlySeries
-remoteWrite.maxDailySeries
```

Hourly limits help with active-series growth.

Daily limits help with churn.

## 11. VM Storage Limits

VictoriaMetrics storage nodes can also use:

```text
-storage.maxHourlySeries
-storage.maxDailySeries
```

These are safety mechanisms, not replacements for good instrumentation.

## 12. Cardinality Explorer

VMUI's Cardinality Explorer can show:

- Metrics with the highest series count
- Labels associated with many series
- Values with many series
- Label/value pairs
- Labels with many unique values

## 13. vmestimator

`vmestimator` can provide continuous cardinality estimates as metrics.

Conceptually:

```text
Metrics
   ↓
vmagent
   ↓
vmestimator
   ↓
cardinality_estimate
   ↓
monitor / alert
```

## 14. Best Strategy

```text
Find high-cardinality metrics
          ↓
Identify problematic labels
          ↓
Remove unnecessary labels
          ↓
Aggregate where appropriate
          ↓
Apply limits as protection
          ↓
Monitor cardinality
```

## Interview Answer

> Cardinality is the number of unique time series, while churn describes how quickly new series are created. High-cardinality labels such as user IDs, request IDs, session IDs, and dynamic URLs can increase memory, index, storage, and ingestion costs. VictoriaMetrics provides Cardinality Explorer and vmestimator for analysis, while vmagent and storage-level limits can provide protection. The preferred solution is good metric design with bounded, meaningful labels.
