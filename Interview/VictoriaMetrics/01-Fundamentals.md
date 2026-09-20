# 01 — VictoriaMetrics Fundamentals

## What is VictoriaMetrics?

VictoriaMetrics (VM) is a time-series database and monitoring backend designed to store, ingest, and query metrics efficiently.

A metric sample can be understood as:

```text
metric + labels + timestamp + value
```

Example:

```text
http_requests_total{
  service="employee-api",
  method="GET",
  status="200"
} 1532
```

## Why use it?

Large monitoring environments can produce millions of time series, high ingestion rates, and large retention requirements. VictoriaMetrics focuses on efficient metrics storage and querying.

A common setup is:

```text
Application
    |
    | /metrics
    v
Prometheus / vmagent
    |
    | Remote Write
    v
VictoriaMetrics
    |
    v
Grafana
```

## Time-Series Basics

### Metric
```text
http_requests_total
```

### Labels
```text
http_requests_total{
  service="employee-api",
  method="GET",
  status="200"
}
```

### Sample

A sample contains a timestamp and value:

```text
timestamp        value
1726800000000    1532
```

### Time series

A unique combination of metric name and label values defines a time series.

```text
requests_total{status="200"}
requests_total{status="500"}
```

These are two different series.

## Cardinality

Cardinality is the number of unique time series.

If:

```text
service = 5
method  = 4
status  = 5
```

the theoretical maximum is:

```text
5 × 4 × 5 = 100 series
```

## Churn

Churn describes how quickly new series are created and disappear.

Example:

```text
request_id="abc" → new series
request_id="def" → new series
request_id="ghi" → new series
```

## Single-Node VM

```text
Prometheus / vmagent
        |
        v
 VictoriaMetrics
        |
        v
     Grafana
```

## VM vs Prometheus

| Area | Prometheus | VictoriaMetrics |
|---|---|---|
| Scraping | Excellent | vmagent can scrape |
| Local storage | Yes | Yes |
| Long-term metrics storage | Often paired with remote storage | Core capability |
| Query | PromQL | MetricsQL + PromQL compatibility |
| Distributed architecture | Different model | VM Cluster |

## Mental Model

```text
Metric
  ↓
Labels
  ↓
Unique label combination
  ↓
Time series
  ↓
Samples over time

Prometheus/vmagent
        ↓
     ingest
        ↓
VictoriaMetrics
        ↓
      query
        ↓
     Grafana
```

## Interview Answer

> VictoriaMetrics is a high-performance time-series database and monitoring backend. It can receive metrics from Prometheus-compatible systems through Remote Write, store them efficiently for long retention, and expose a Prometheus-compatible query interface for tools such as Grafana.
