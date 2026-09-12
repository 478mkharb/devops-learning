# Prometheus, Grafana and OpenTelemetry Interview Questions

## L1 — Fundamentals

### Q1. What is Prometheus?
**Answer:** Prometheus is an open-source monitoring and alerting system that collects time-series metrics, stores them with labels, and queries them using PromQL.

### Q2. What is Grafana?
**Answer:** Grafana is a visualization and observability platform used to build dashboards, explore data, configure alerts, and correlate metrics, logs, and traces from different data sources.

### Q3. What is OpenTelemetry?
**Answer:** OpenTelemetry, or OTel, is a vendor-neutral observability framework for generating, collecting, and exporting metrics, logs, and traces.

### Q4. What are the three main observability signals?
**Answer:**

| Signal | Meaning | Example |
|---|---|---|
| Metrics | Numeric measurements over time | CPU usage, request rate |
| Logs | Timestamped event records | Exception stack trace |
| Traces | Request journey across services | Browser → API → DB |

### Q5. What is the difference between monitoring and observability?
**Answer:** Monitoring checks known failure conditions using predefined metrics and alerts. Observability helps investigate unknown problems by correlating metrics, logs, and traces.

### Q6. What is a time-series metric?
**Answer:** A metric is a numeric value recorded with a timestamp and identifying labels. Example:

```text
http_requests_total{method="GET",status="200"} 1542
```

### Q7. What is a label in Prometheus?
**Answer:** A label is a key-value pair that identifies a metric dimension.

```text
http_requests_total{method="GET",status="500",service="salary-api"}
```

### Q8. What is the Prometheus data model?
**Answer:** A time series is uniquely identified by its metric name and complete set of label key-value pairs.

```text
metric_name{label1="value1",label2="value2"}
```

### Q9. What is the difference between Prometheus and Grafana?

| Prometheus | Grafana |
|---|---|
| Collects and stores metrics | Visualizes and explores data |
| Uses PromQL | Uses queries supported by data sources |
| Has a built-in alerting engine | Provides dashboard and alert visualization |
| Primarily a metrics system | Supports metrics, logs, traces, and other sources |

### Q10. What is a Prometheus target?
**Answer:** A target is an endpoint from which Prometheus collects metrics, usually an HTTP endpoint such as `/metrics`.

### Q11. What is scraping?
**Answer:** Scraping is the process in which Prometheus periodically sends an HTTP request to a target’s metrics endpoint and stores the returned metrics.

### Q12. What is the default Prometheus scrape model?
**Answer:** Prometheus normally uses a pull model: Prometheus pulls metrics from targets. Pushgateway is used only for limited short-lived batch-job use cases.

### Q13. What is an exporter?
**Answer:** An exporter converts metrics from a system into Prometheus exposition format.

| Exporter | Monitored system |
|---|---|
| Node Exporter | Linux host |
| Blackbox Exporter | HTTP, TCP, ICMP, DNS endpoints |
| MySQL Exporter | MySQL |
| PostgreSQL Exporter | PostgreSQL |
| Redis Exporter | Redis |

### Q14. What is Node Exporter?
**Answer:** Node Exporter exposes Linux host metrics such as CPU, memory, disk, filesystem, network, load, and uptime.

### Q15. What is Blackbox Exporter?
**Answer:** Blackbox Exporter performs synthetic probes against endpoints and exposes results such as availability, latency, DNS status, HTTP status, and TLS information.

### Q16. What is PromQL?
**Answer:** PromQL is Prometheus Query Language used to select, filter, aggregate, and calculate metrics.

```promql
up
rate(http_requests_total[5m])
node_memory_MemAvailable_bytes
```

### Q17. What does the `up` metric mean?
**Answer:**

```promql
up == 1
```

means the target was successfully scraped. `up == 0` means the scrape failed.

### Q18. What is a Grafana data source?
**Answer:** A data source is a backend from which Grafana queries data, such as Prometheus, Loki, Elasticsearch, Jaeger, Tempo, MySQL, or PostgreSQL.

### Q19. What is a Grafana dashboard?
**Answer:** A dashboard is a collection of panels displaying queried data using graphs, tables, gauges, stat panels, heatmaps, and other visualizations.

### Q20. What is an OpenTelemetry SDK?
**Answer:** The SDK is the implementation that creates, processes, samples, batches, and exports telemetry generated through the OpenTelemetry API.

### Q21. What is the difference between OpenTelemetry API, SDK, and instrumentation?

| Component | Responsibility |
|---|---|
| API | Interfaces used by application or instrumentation code |
| SDK | Actual telemetry implementation and processing pipeline |
| Instrumentation | Code that observes application/framework activity and creates telemetry |
| Exporter | Sends telemetry to a backend or collector |

### Q22. What is automatic instrumentation?
**Answer:** Automatic instrumentation uses agents or instrumentation libraries to create telemetry for supported frameworks, HTTP clients, databases, and messaging systems without manually adding spans around every operation.

### Q23. What is manual instrumentation?
**Answer:** Manual instrumentation is developer-written code that creates custom spans, metrics, and attributes around business operations.

```python
with tracer.start_as_current_span("salary.generate"):
    generate_salary_slip()
```

### Q24. What is a trace?
**Answer:** A trace represents the complete journey of one request or transaction across one or more services.

### Q25. What is a span?
**Answer:** A span represents one timed operation within a trace, such as an HTTP request, database query, cache lookup, or business operation.

### Q26. What is a trace ID and span ID?
**Answer:** A trace ID identifies the complete trace. A span ID identifies one operation within that trace.

### Q27. What is context propagation?
**Answer:** Context propagation transfers trace context between services, usually through HTTP headers such as W3C `traceparent`, allowing distributed spans to belong to the same trace.

### Q28. What is OTLP?
**Answer:** OTLP is the OpenTelemetry Protocol used to transmit telemetry between SDKs, collectors, and backends. It supports gRPC and HTTP transports.

---

## L2 — Practical and Intermediate

### Q29. Explain the Prometheus architecture.
**Answer:**

```text
Application/Exporter
        ↓
Prometheus Scraper
        ↓
Prometheus TSDB
        ↓
PromQL / Rules / Alerts
        ↓
Grafana / Alertmanager / Remote Storage
```

### Q30. What is Alertmanager?
**Answer:** Alertmanager receives alerts from Prometheus and handles grouping, deduplication, silencing, inhibition, routing, and notifications.

### Q31. Difference between Prometheus alerting rules and recording rules?

| Alerting rule | Recording rule |
|---|---|
| Detects a condition | Precomputes and stores a query result |
| Produces an alert | Produces a new time series |
| Example: service is down | Example: request rate per service |

### Q32. What is the difference between `rate()`, `irate()`, and `increase()`?

| Function | Use |
|---|---|
| `rate()` | Average per-second increase over a range; preferred for alerting |
| `irate()` | Instant rate based on latest samples; useful for volatile dashboards |
| `increase()` | Total increase over a range |

```promql
rate(http_requests_total[5m])
irate(http_requests_total[5m])
increase(http_requests_total[1h])
```

### Q33. Why should `rate()` be applied before aggregation?
**Answer:** Apply `rate()` before `sum()` so Prometheus can correctly handle counter resets per individual time series.

```promql
sum(rate(http_requests_total[5m]))
```

Preferred over aggregating the raw counters first.

### Q34. What is a counter?
**Answer:** A counter is a metric that only increases or resets to zero when the process restarts. Examples are total requests, errors, and processed messages.

### Q35. What is a gauge?
**Answer:** A gauge can increase or decrease. Examples are memory usage, active connections, queue length, and temperature.

### Q36. What is a histogram?
**Answer:** A histogram records observations into configurable buckets and exposes `_bucket`, `_sum`, and `_count` series. It is commonly used for latency distributions and percentile estimation.

```text
http_request_duration_seconds_bucket{le="0.5"}
http_request_duration_seconds_bucket{le="1"}
http_request_duration_seconds_sum
http_request_duration_seconds_count
```

### Q37. What is a summary?
**Answer:** A summary calculates quantiles on the client side and exposes quantile, sum, and count values. Quantiles from different instances generally cannot be accurately aggregated.

### Q38. Histogram vs summary?

| Histogram | Summary |
|---|---|
| Uses buckets | Calculates quantiles in client |
| Percentiles calculated by PromQL | Quantiles exposed directly |
| Aggregatable across instances | Quantiles usually not aggregatable |
| Bucket boundaries must be selected | Quantile objectives must be selected |
| Preferred for service-wide latency percentiles | Useful for local/client-side quantiles |

### Q39. What are P50, P90, P95, and P99?
**Answer:** They are latency percentiles.

| Percentile | Meaning |
|---|---|
| P50 | 50% of requests are at or below this latency |
| P90 | 90% are at or below this latency |
| P95 | 95% are at or below this latency |
| P99 | 99% are at or below this latency |

If P95 is 800 ms, 95% of requests completed within 800 ms and 5% took longer.

### Q40. How do you calculate P95 latency from a histogram?

```promql
histogram_quantile(
  0.95,
  sum by (le) (
    rate(http_request_duration_seconds_bucket[5m])
  )
)
```

### Q41. What is cardinality?
**Answer:** Cardinality is the number of unique label combinations for a metric. High-cardinality labels such as user ID, request ID, email, or full URL can cause high memory usage and expensive queries.

### Q42. Why should user ID not be used as a Prometheus label?
**Answer:** Every unique user ID creates a separate time series. With millions of users, this can create a cardinality explosion. Use logs or traces for user-level investigation instead.

### Q43. What is the difference between `sum`, `avg`, `min`, `max`, and `count` in PromQL?

| Aggregator | Meaning |
|---|---|
| `sum` | Adds values |
| `avg` | Calculates average |
| `min` | Smallest value |
| `max` | Largest value |
| `count` | Counts input series |

### Q44. What is the difference between `by` and `without`?
**Answer:**

```promql
sum by (service) (rate(http_requests_total[5m]))
```

keeps only the grouping label `service`.

```promql
sum without (instance) (rate(http_requests_total[5m]))
```

aggregates while removing `instance` from the result label set.

### Q45. What is the difference between `on()` and `ignoring()` in PromQL joins?
**Answer:** They control which labels are used when matching two vectors.

```promql
metric_a / on(service) metric_b
```

matches only on `service`.

```promql
metric_a / ignoring(instance) metric_b
```

ignores `instance` during matching.

### Q46. What are `group_left` and `group_right`?
**Answer:** They allow many-to-one or one-to-many vector matching when joining metrics with different label cardinalities.

```promql
rate(container_cpu_usage_seconds_total[5m])
  * on(namespace,pod) group_left(node)
  kube_pod_info
```

### Q47. What is a stale series?
**Answer:** A time series becomes stale when Prometheus no longer receives samples for it. Staleness affects queries and can cause graphs to disappear or alerts to change state.

### Q48. What is a scrape interval?
**Answer:** It defines how frequently Prometheus scrapes a target.

```yaml
scrape_interval: 15s
```

### Q49. What is an evaluation interval?
**Answer:** It defines how frequently Prometheus evaluates recording and alerting rules.

```yaml
evaluation_interval: 15s
```

### Q50. What is the difference between scrape timeout and scrape interval?
**Answer:** The scrape interval is how often scraping starts. The scrape timeout is the maximum time allowed for one scrape. Timeout must be lower than the interval.

### Q51. How do you monitor an HTTP endpoint using Blackbox Exporter?
**Answer:** Prometheus scrapes Blackbox Exporter with probe parameters. Blackbox Exporter probes the configured target and exposes results.

```yaml
- job_name: blackbox-http
  metrics_path: /probe
  params:
    module: [http_2xx]
  static_configs:
    - targets:
        - https://example.com
  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: instance
    - target_label: __address__
      replacement: blackbox-exporter:9115
```

### Q52. What is Grafana provisioning?
**Answer:** Provisioning creates data sources, dashboards, alert rules, and folders from configuration files instead of manual UI configuration.

### Q53. What is a Grafana variable?
**Answer:** A variable makes dashboards reusable by allowing users to select values such as environment, service, namespace, instance, or job.

```promql
label_values(up, job)
```

### Q54. What is the difference between Grafana dashboard time range and PromQL range vector?
**Answer:** The dashboard time range controls the visible time window. A PromQL range vector such as `[5m]` controls the samples used by a function for each evaluation point.

### Q55. What is a Grafana alert rule?
**Answer:** A Grafana alert rule evaluates a query or expression, applies a condition, and sends notifications through configured contact points and notification policies.

### Q56. What is the difference between Grafana alerting and Prometheus Alertmanager?

| Grafana Alerting | Prometheus + Alertmanager |
|---|---|
| Can alert on multiple data sources | Primarily receives Prometheus alerts |
| Evaluates rules in Grafana | Prometheus evaluates alerting rules |
| Uses Grafana contact points and policies | Uses Alertmanager routing and grouping |
| Useful for multi-source dashboards | Strong Prometheus-native alert workflow |

### Q57. What is an OpenTelemetry Collector?
**Answer:** The Collector is a vendor-neutral service that receives, processes, and exports telemetry. It reduces the need for every application to communicate directly with every backend.

### Q58. What are the main OpenTelemetry Collector components?

| Component | Responsibility |
|---|---|
| Receiver | Accepts telemetry |
| Processor | Modifies, filters, batches, samples, or enriches telemetry |
| Exporter | Sends telemetry to a backend |
| Connector | Connects pipelines and can generate derived signals |
| Extension | Provides auxiliary capabilities such as health checks |
| Pipeline | Defines signal flow through receivers, processors, and exporters |

### Q59. What is the difference between agent and gateway Collector deployment?

| Agent | Gateway |
|---|---|
| Runs close to application or node | Centralized service |
| Collects local telemetry | Receives telemetry from many agents |
| Performs local enrichment or filtering | Performs centralized processing and export |
| Easier local fault isolation | Easier centralized management |

### Q60. What is the difference between manual and automatic OpenTelemetry instrumentation?

| Manual | Automatic |
|---|---|
| Developer creates custom spans/metrics | Agent/library creates standard telemetry |
| Captures business operations | Captures supported frameworks and libraries |
| More control | Less code |
| Requires code changes | Usually configuration or startup changes |

### Q61. Does automatic instrumentation remove the need for the OpenTelemetry SDK?
**Answer:** No. Automatic instrumentation creates telemetry through the OpenTelemetry API or agent mechanism. The SDK still manages providers, sampling, processors, batching, and exporting.

### Q62. Explain the OpenTelemetry application flow.

```text
Application
   ↓
Instrumentation
   ↓
OpenTelemetry API
   ↓
OpenTelemetry SDK
   ↓
Sampler / Processor / Batch
   ↓
OTLP Exporter
   ↓
OpenTelemetry Collector
   ↓
Backend such as Jaeger, Tempo, Prometheus, or a vendor platform
```

### Q63. What is a Resource in OpenTelemetry?
**Answer:** A Resource describes the entity producing telemetry, such as service name, service version, deployment environment, host, container, or Kubernetes pod.

```text
service.name = salary-api
service.version = 1.4.0
deployment.environment = production
```

### Q64. What is the difference between `service.name` and instrumentation scope?
**Answer:** `service.name` identifies the application or service. Instrumentation scope identifies the library or instrumentation component that produced telemetry, such as `io.opentelemetry.http` or `salary-business-instrumentation`.

### Q65. What is a Span Processor?
**Answer:** A Span Processor receives finished spans and performs actions such as immediate export, batching, filtering, or forwarding to an exporter.

### Q66. Simple Span Processor vs Batch Span Processor?

| Simple processor | Batch processor |
|---|---|
| Exports each span immediately | Buffers spans and exports in batches |
| Easy for debugging | Better production performance |
| More network calls | Fewer network calls |
| Can increase latency | Uses memory and requires flush handling |

### Q67. What is sampling in OpenTelemetry?
**Answer:** Sampling decides which traces are recorded and/or exported to control storage, CPU, network, and backend costs.

### Q68. Common OpenTelemetry sampling strategies?

| Strategy | Behavior |
|---|---|
| AlwaysOn | Samples every trace |
| AlwaysOff | Samples no trace |
| TraceIdRatioBased | Samples a configured percentage |
| ParentBased | Follows parent sampling decision where possible |
| Tail sampling | Collector decides after seeing trace data |

### Q69. Head sampling vs tail sampling?

| Head sampling | Tail sampling |
|---|---|
| Decision made near trace start | Decision made after trace data is available |
| Low resource cost | Can retain slow/error traces intelligently |
| May discard a trace before seeing outcome | Requires buffering and complete trace visibility |
| Usually SDK-side | Usually Collector-side |

### Q70. What is baggage in OpenTelemetry?
**Answer:** Baggage carries key-value context across service boundaries. It is not automatically a span attribute and must be handled carefully because it can increase request size and expose sensitive data.

### Q71. What is the difference between span attributes, events, and links?

| Feature | Purpose |
|---|---|
| Attribute | Key-value metadata about a span |
| Event | Timestamped event within a span |
| Link | Relationship to another span context, often used for batch or asynchronous work |

### Q72. What is a span status?
**Answer:** Span status indicates whether an operation succeeded, failed, or is unset. Typical values are `UNSET`, `OK`, and `ERROR`.

### Q73. How should exceptions be recorded in OpenTelemetry?
**Answer:** Record the exception and set the span status to error when appropriate.

```python
span.record_exception(exc)
span.set_status(Status(StatusCode.ERROR))
```

### Q74. How does OpenTelemetry instrumentation know which metrics to create?
**Answer:** Instrumentation libraries are written for specific frameworks or libraries. They know the framework lifecycle and expose predefined metrics such as request count, duration, status code, connection count, or database operation count. Custom business metrics require manual instrumentation.

### Q75. Can Prometheus collect OpenTelemetry metrics?
**Answer:** Yes. Common approaches include exposing Prometheus-format metrics from an OTel SDK or Collector, using an OTel Collector Prometheus exporter, or sending OTLP metrics to a backend that supports them.

---

## L3 — Advanced Architecture and Troubleshooting

### Q76. Why is a counter reset important?
**Answer:** A process restart resets counters to zero. `rate()`, `irate()`, and `increase()` detect counter resets, while manually subtracting two raw counter values may produce incorrect negative or unusually large results.

### Q77. Why is this query often wrong for request rate?

```promql
http_requests_total / 60
```

**Answer:** `http_requests_total` is cumulative, not a current rate. Use:

```promql
rate(http_requests_total[5m])
```

### Q78. Why can average latency hide problems?
**Answer:** Averages hide tail latency. A service may have a 100 ms average while 5% of requests take several seconds. Use P95/P99 histograms and error rate together with averages.

### Q79. Why might a target show `up == 0`?
**Answer:** Possible causes include:

1. Target process is down.
2. Wrong target address or port.
3. `/metrics` endpoint is missing.
4. Network or security group restriction.
5. DNS failure.
6. TLS or authentication failure.
7. Scrape timeout.
8. Prometheus configuration or relabeling error.

### Q80. A metric exists in the application but not in Prometheus. What do you check?
**Answer:**

1. Confirm the application exposes `/metrics`.
2. Curl the endpoint from the Prometheus host.
3. Check Prometheus target status.
4. Verify job labels and scrape configuration.
5. Check metric name and label filters.
6. Check whether the metric has been updated or initialized.
7. Check exporter and application logs.
8. Verify authentication, TLS, DNS, and network rules.

### Q81. A Grafana panel shows “No data.” What is your troubleshooting sequence?
**Answer:**

1. Verify the selected data source.
2. Run the query directly in Prometheus or Explore.
3. Check dashboard time range and timezone.
4. Check template variable values.
5. Check metric name and label matchers.
6. Check whether the metric is stale.
7. Check query step, resolution, and transformations.
8. Check permissions and data-source connectivity.

### Q82. Prometheus memory usage is continuously increasing. What are likely causes?
**Answer:**

- High-cardinality labels.
- Too many targets.
- Long retention period.
- Expensive recording rules.
- Excessive scrape frequency.
- Unbounded dynamic labels.
- Large WAL or pending remote-write queues.

### Q83. How do you reduce Prometheus cardinality?
**Answer:**

1. Remove user IDs, request IDs, email addresses, and full URLs from labels.
2. Normalize URL paths.
3. Use bounded values such as HTTP method and status class.
4. Drop unnecessary labels with relabeling.
5. Reduce unnecessary metrics.
6. Use logs or traces for high-dimensional investigation.

### Q84. Why is `http://host/order/123` a bad metric label value?
**Answer:** Every order ID creates a new time series. Prefer a route template:

```text
route="/order/{id}"
```

or use a bounded route name.

### Q85. What is the difference between target relabeling and metric relabeling?

| Type | Applied when | Purpose |
|---|---|---|
| `relabel_configs` | Before scraping | Modify target labels or decide which target to scrape |
| `metric_relabel_configs` | After scraping | Modify or drop individual scraped metrics |

### Q86. How do you drop high-cardinality metrics?

```yaml
metric_relabel_configs:
  - source_labels: [__name__]
    regex: "expensive_metric.*"
    action: drop
```

### Q87. What is remote write?
**Answer:** Remote write sends Prometheus samples to an external long-term or horizontally scalable storage system.

### Q88. What is remote read?
**Answer:** Remote read allows Prometheus queries to retrieve data from compatible external storage systems.

### Q89. What is Thanos or Cortex/Mimir used for?
**Answer:** They provide capabilities such as long-term storage, high availability, horizontal scalability, and querying metrics from multiple Prometheus instances.

### Q90. Why should two Prometheus servers scrape the same targets?
**Answer:** Independent Prometheus servers provide high availability. If one fails, the other can continue collecting metrics. Deduplication may be required in the query layer.

### Q91. What is the difference between federation and remote write?

| Federation | Remote write |
|---|---|
| One Prometheus scrapes selected metrics from another | Prometheus pushes samples to remote storage |
| Useful for hierarchical aggregation | Useful for long-term/scalable storage |
| Pull model | Push from Prometheus |
| Selective metric collection | Usually broader sample forwarding |

### Q92. Why can a Prometheus alert be delayed?
**Answer:** Delay can come from scrape interval, rule evaluation interval, `for` duration, query range, target response time, Prometheus load, Alertmanager routing, or notification provider delay.

### Q93. What is the purpose of the `for` field in an alerting rule?
**Answer:** It requires the alert condition to remain true for a specified duration before the alert becomes firing.

```yaml
- alert: HighErrorRate
  expr: rate(http_requests_total{status=~"5.."}[5m]) > 1
  for: 10m
```

### Q94. Why should alert rules avoid very short windows?
**Answer:** Very short windows are sensitive to scrape gaps, traffic fluctuations, and transient spikes, causing alert flapping. Use a suitable range, threshold, and `for` duration.

### Q95. What is alert flapping?
**Answer:** Alert flapping occurs when an alert repeatedly changes between firing and resolved states because the condition is near the threshold or data is unstable.

### Q96. How do you design a useful alert?
**Answer:** An alert should be actionable, symptom-oriented, bounded, and include service, environment, severity, summary, description, dashboard, and runbook information.

### Q97. What is the difference between symptom and cause alerts?
**Answer:** A symptom alert detects user impact, such as high error rate or unavailable service. A cause alert detects a possible underlying reason, such as high CPU or disk pressure. Symptom alerts should generally receive higher priority.

### Q98. How do you monitor Kubernetes using Prometheus?
**Answer:** Common components include:

| Component | Purpose |
|---|---|
| kube-state-metrics | Kubernetes object state |
| Node Exporter | Node OS metrics |
| cAdvisor/kubelet metrics | Container and kubelet metrics |
| Prometheus Operator | Manages Prometheus resources |
| Alertmanager | Alert routing |
| Grafana | Visualization |

### Q99. What is the difference between kube-state-metrics and cAdvisor?
**Answer:** kube-state-metrics exposes Kubernetes object state such as deployment replicas and pod phases. cAdvisor exposes resource usage of containers such as CPU and memory.

### Q100. How do you detect a Kubernetes deployment with unavailable replicas?

```promql
kube_deployment_status_replicas_unavailable > 0
```

### Q101. How do you detect pods restarting frequently?

```promql
increase(kube_pod_container_status_restarts_total[15m]) > 3
```

### Q102. How do you monitor application SLOs?
**Answer:** Define service-level indicators such as availability and latency, then calculate the percentage meeting the objective.

```promql
sum(rate(http_requests_total{status=~"2.."}[5m]))
/
sum(rate(http_requests_total[5m]))
```

### Q103. What is an error budget?
**Answer:** Error budget is the allowed amount of unreliability within an SLO period. For a 99.9% availability SLO, the error budget is 0.1% of eligible request time or requests, depending on the SLI model.

### Q104. What is the OpenTelemetry Collector pipeline configuration structure?

```yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:

processors:
  batch:
  memory_limiter:

exporters:
  debug:
  otlp:
    endpoint: tempo:4317
    tls:
      insecure: true

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [otlp]
```

### Q105. Why is `memory_limiter` useful in the Collector?
**Answer:** It prevents uncontrolled memory growth by applying memory pressure handling, such as refusing or delaying data and triggering garbage collection where supported.

### Q106. Why is the batch processor useful?
**Answer:** It groups telemetry before export, reducing network calls and backend overhead. It also improves throughput, but must be balanced against export delay and memory usage.

### Q107. What happens if the OpenTelemetry Collector is down?
**Answer:** Applications may queue telemetry temporarily depending on exporter configuration and SDK behavior. Queues can fill, telemetry can be dropped, and application overhead may increase. Applications should not become unavailable merely because telemetry export fails.

### Q108. How do you prevent telemetry from affecting application performance?
**Answer:**

- Use batch processors.
- Configure bounded queues.
- Use sampling.
- Set export timeouts.
- Avoid expensive attributes.
- Avoid synchronous exporters in production.
- Use memory limits.
- Monitor dropped telemetry.
- Prevent recursive instrumentation.

### Q109. What is the difference between OTLP gRPC and OTLP HTTP?

| OTLP gRPC | OTLP HTTP |
|---|---|
| Uses gRPC transport | Uses HTTP transport |
| Common port 4317 | Common port 4318 |
| Efficient streaming/binary transport | Easier through HTTP infrastructure |
| Requires gRPC support | Works well with HTTP proxies and gateways |

### Q110. What is the difference between OTLP, Prometheus exposition, and Jaeger protocols?
**Answer:** OTLP is OpenTelemetry’s general telemetry protocol. Prometheus exposition is a text or compatible format mainly for metrics scraping. Jaeger protocols are trace-specific protocols used by Jaeger-compatible systems.

### Q111. Why might traces be broken across microservices?
**Answer:**

1. Context propagation is missing.
2. HTTP headers are removed by a proxy.
3. Different propagation formats are configured.
4. Async tasks lose context.
5. Instrumentation is missing on one service.
6. Sampling decisions are inconsistent.
7. Trace context is not injected into messaging headers.

### Q112. How do you troubleshoot missing traces?
**Answer:**

1. Confirm SDK initialization.
2. Confirm `service.name`.
3. Confirm spans are created.
4. Confirm exporter endpoint and protocol.
5. Check Collector receiver and pipeline.
6. Check Collector logs and internal metrics.
7. Verify backend ingestion.
8. Verify sampling.
9. Verify context propagation.
10. Use a debug exporter temporarily.

### Q113. Why are traces visible but metrics are missing?
**Answer:** Traces and metrics have separate providers, instruments, pipelines, exporters, and backend configuration. Trace export working does not prove that the metrics pipeline is configured.

### Q114. Why are metrics visible but traces are missing?
**Answer:** Possible causes include missing tracer provider, no spans, trace sampling set to off, missing trace exporter, incorrect OTLP trace pipeline, or backend trace ingestion failure.

### Q115. What is the difference between an OTel metric counter and a Prometheus counter?
**Answer:** Both represent monotonically increasing values conceptually. The OTel SDK maps instruments to the target backend’s metric model. Exporters may translate names, temporality, attributes, and aggregation details.

### Q116. What is delta vs cumulative temporality?

| Temporality | Meaning |
|---|---|
| Cumulative | Value represents total since start/reset |
| Delta | Value represents change during the reporting interval |

### Q117. Why can duplicate metrics appear after adding OTel and Prometheus instrumentation?
**Answer:** The same application may expose identical measurements through both a Prometheus client library and OTel metrics exporter. Avoid registering duplicate instruments or scraping duplicate endpoints.

### Q118. What is semantic convention?
**Answer:** Semantic conventions define standardized names and attributes for telemetry, such as HTTP method, route, status code, database system, messaging system, and service identity. They improve cross-service query consistency.

### Q119. Why should sensitive data not be added to spans or baggage?
**Answer:** Telemetry is often sent to shared systems and retained for long periods. Passwords, tokens, personal data, and financial information can create security and compliance risks.

### Q120. What is tail-based sampling useful for?
**Answer:** Tail sampling can retain traces based on final characteristics such as errors, high latency, specific services, or rare status codes, even when most normal traces are discarded.

---

## Brain Teasers

### Q121. A service receives 1,000 requests in five minutes. What is the average request rate?
**Answer:**

```text
1000 / 300 = 3.33 requests per second
```

### Q122. A counter was 9,000 and is now 500 after a restart. What is the request increase?
**Answer:** Do not calculate `500 - 9000`. The counter reset. Use `increase()` or `rate()` so Prometheus can handle the reset.

### Q123. P50 latency is 100 ms and P99 latency is 8 seconds. Is the service healthy?
**Answer:** Not necessarily. Most requests are fast, but the tail is extremely slow. Investigate P95/P99, traffic volume, endpoint distribution, dependencies, and user impact.

### Q124. CPU is 95%, but request latency and error rate are normal. Should you page immediately?
**Answer:** Usually not solely on CPU. CPU may be intentionally utilized. Check saturation, throttling, latency, errors, queueing, and capacity headroom. Use a lower-priority warning unless user impact exists.

### Q125. A target is `up == 1`, but the application is unavailable to users. How is that possible?
**Answer:** Prometheus may only be scraping the metrics endpoint successfully. The application endpoint, load balancer, DNS, authentication path, or user-facing route may still be failing. Monitor both internal health and blackbox user journeys.

### Q126. A dashboard shows zero requests, but the application is receiving traffic. What could be wrong?
**Answer:** Possible causes are wrong metric name, wrong job label, wrong environment variable, dashboard variable mismatch, counter reset, incorrect time range, or the application exposing metrics on a different endpoint.

### Q127. Why can `sum(rate(metric[5m]))` and `rate(sum(metric)[5m])` produce different results?
**Answer:** Aggregating before rate hides individual counter resets. Apply `rate()` to each series first, then aggregate.

### Q128. A metric has labels `service`, `instance`, `method`, `status`, and `user_id`. Which label should be questioned first?
**Answer:** `user_id`, because it is usually unbounded and creates high cardinality.

### Q129. A P95 alert fires even though average latency is low. Is the alert wrong?
**Answer:** No. Average latency and tail latency measure different behavior. A small percentage of very slow requests can trigger a P95 alert while keeping the average low.

### Q130. A trace contains an API span but no database span. What is the likely issue?
**Answer:** Database auto-instrumentation may not be installed, the database client may not be supported, the connection may be created before instrumentation starts, or the database operation may occur in a separate process or unsupported driver.

### Q131. Automatic instrumentation is enabled, but a custom business operation is missing. Why?
**Answer:** Automatic instrumentation usually knows framework and library operations, not business meaning. Add a manual span such as `salary.generate` or `notification.send`.

### Q132. An application creates spans, but the Collector receives nothing. What should be checked first?
**Answer:** Check the exporter endpoint, protocol, port, TLS setting, DNS, network access, SDK initialization, and Collector receiver configuration.

### Q133. A Collector receives traces but exports none. What is likely wrong?
**Answer:** The exporter may be missing, the exporter endpoint may be incorrect, the pipeline may not reference the exporter, authentication may fail, or the backend may reject the data.

### Q134. Why can a trace be present in one service but not another?
**Answer:** Sampling, missing propagation, missing instrumentation, asynchronous context loss, proxy header removal, or separate trace IDs can cause partial traces.

### Q135. A Grafana dashboard is slow. What should you optimize?
**Answer:** Reduce high-cardinality queries, use recording rules, reduce dashboard panel count, avoid large range windows, use appropriate step size, filter labels early, and avoid expensive regex queries.

### Q136. A Prometheus server is overloaded after reducing scrape interval from 30 seconds to 5 seconds. Why?
**Answer:** Scrape frequency increased six times, increasing target requests, sample ingestion, CPU, memory, WAL activity, and query load.

### Q137. Why can a `for: 5m` alert take more than five minutes to fire?
**Answer:** The condition must first be observed during a rule evaluation. Add scrape interval, evaluation interval, query delay, and scheduling delay to the five-minute pending period.

### Q138. A service has no traffic. Its error rate is zero. Is it healthy?
**Answer:** Error rate alone is inconclusive because there may be no requests. Combine availability probes, request volume, latency, and health checks.

### Q139. A histogram has buckets up to 10 seconds, but some requests take 30 seconds. Can you calculate an accurate P99?
**Answer:** Not accurately if the configured buckets do not provide enough resolution beyond 10 seconds. Add suitable higher buckets or use a different measurement design.

### Q140. Why is a 99.99% availability SLO harder than 99%?
**Answer:** The allowed error budget is much smaller. A 99% SLO allows 1% unavailability, while 99.99% allows only 0.01%.

---

## Scenario-Based Questions

### Q141. Scenario: The production API returns HTTP 500 errors. How do you investigate using Prometheus, Grafana, and OTel?
**Answer:**

1. Check request error rate:

```promql
sum(rate(http_requests_total{status=~"5.."}[5m]))
```

2. Check affected service, route, instance, and deployment.
3. Open Grafana panels for request rate, latency, errors, CPU, memory, and dependency health.
4. Find a failed trace using the error status or trace ID.
5. Inspect child spans for database, Redis, external API, or messaging failures.
6. Correlate the trace with application logs.
7. Check recent deployment or configuration changes.
8. Roll back or fix the failing dependency.

### Q142. Scenario: CPU is high on one EC2 instance only. What do you check?
**Answer:**

- `node_cpu_seconds_total` by instance and mode.
- Process-level CPU metrics if available.
- Request distribution by instance.
- Traffic imbalance from the load balancer.
- Runaway process or thread.
- CPU throttling or instance limits.
- Recent deployment.
- Memory pressure and I/O wait.

### Q143. Scenario: Disk usage reaches 90%. What is the correct response?
**Answer:**

1. Alert before 100%.
2. Identify filesystem and mount point.
3. Check large directories and log growth.
4. Check deleted files still held by processes.
5. Rotate or ship logs.
6. Clean safe temporary files.
7. Expand the volume if required.
8. Check inode usage separately.

### Q144. Scenario: Grafana displays data for one environment but not another.
**Answer:** Check the selected Prometheus data source, `environment` label values, dashboard variables, scrape jobs, target status, and whether the second environment uses different metric names or labels.

### Q145. Scenario: Prometheus target status is healthy, but Blackbox probe fails.
**Answer:** Prometheus is successfully scraping Blackbox Exporter, but Blackbox Exporter cannot reach the actual target. Check DNS, routing, security groups, proxy settings, TLS, target URL, and Blackbox module configuration.

### Q146. Scenario: You need to alert when an API is down for five minutes.
**Answer:**

```yaml
groups:
  - name: availability
    rules:
      - alert: APIDown
        expr: up{job="salary-api"} == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: Salary API is down
          description: Salary API target has failed scraping for five minutes.
```

### Q147. Scenario: You need a P95 latency alert for an API.
**Answer:**

```yaml
- alert: HighP95Latency
  expr: |
    histogram_quantile(
      0.95,
      sum by (le, service) (
        rate(http_request_duration_seconds_bucket[5m])
      )
    ) > 1
  for: 10m
  labels:
    severity: warning
```

### Q148. Scenario: You need to monitor a private API endpoint from outside the application host.
**Answer:** Use Blackbox Exporter from a network location that represents the required user path. Ensure routing, DNS, security groups, TLS, and authentication are configured. Do not rely only on an internal `/metrics` scrape.

### Q149. Scenario: An application uses Flask, PostgreSQL, Redis, and an external SMTP service. What should be automatic and manual instrumentation?
**Answer:**

| Area | Recommended instrumentation |
|---|---|
| Flask HTTP requests | Automatic |
| PostgreSQL queries | Automatic if supported driver is instrumented |
| Redis calls | Automatic if supported client is instrumented |
| SMTP client | Automatic if supported; otherwise manual |
| `salary.generate` | Manual |
| `salary.persist` | Manual |
| `notification.process` | Manual |
| Business success/failure count | Manual metric |

### Q150. Scenario: You have traces but cannot identify which service generated them.
**Answer:** Configure a unique and stable `service.name` resource attribute for every service. Also add service version, environment, instance, namespace, and deployment metadata.

### Q151. Scenario: A trace is sampled out, but you need all failed requests.
**Answer:** Use tail-based sampling in the Collector or a sampling strategy that retains error traces. Ensure the Collector receives all relevant spans and has sufficient memory and queue capacity.

### Q152. Scenario: A notification worker processes messages asynchronously. How should tracing be designed?
**Answer:** Create a producer span when publishing the message, propagate trace context through message headers, and create a consumer span when processing the message. Use span links when one consumer operation relates to multiple producer contexts.

### Q153. Scenario: Prometheus is scraping the same application twice.
**Answer:** Check service discovery, duplicate scrape jobs, static configurations, Kubernetes ServiceMonitors, relabeling, and target labels. Duplicate scraping creates duplicate series or duplicate samples and can distort dashboards.

### Q154. Scenario: You need one dashboard for dev, test, and production.
**Answer:** Use dashboard variables for environment, service, namespace, job, and instance. Ensure all environments expose consistent metric names and label keys.

### Q155. Scenario: You need to reduce observability cost without losing important information.
**Answer:**

- Use head or tail sampling for traces.
- Always retain errors and slow traces.
- Remove high-cardinality labels.
- Use recording rules.
- Reduce unnecessary scrape frequency.
- Apply metric filtering.
- Use short retention locally and long-term storage selectively.
- Keep business-critical metrics and SLO indicators.

### Q156. Scenario: An application becomes slow after enabling tracing.
**Answer:** Check synchronous exporters, excessive span creation, large attributes, stack-trace recording, high sampling rate, unbounded queues, exporter timeouts, and Collector availability. Use batch exporting and controlled sampling.

### Q157. Scenario: The Collector is receiving telemetry faster than the backend accepts it.
**Answer:** Configure bounded queues, retries, backoff, batch processing, memory limits, and appropriate scaling. Monitor exporter failures, queue size, dropped telemetry, and backend throttling.

### Q158. Scenario: A database is slow, but application CPU is normal.
**Answer:** Use traces to inspect database span duration, Prometheus database metrics for connections and query latency, database logs for slow queries, and Grafana correlation with application latency and traffic.

### Q159. Scenario: An alert fires every night during a scheduled batch job.
**Answer:** Determine whether the behavior is expected. Use maintenance windows, alert inhibition, schedule-aware thresholds, separate batch-job alerts, or an appropriate `for` duration. Do not simply silence a real production failure.

### Q160. Scenario: The service is healthy, but no metrics are available after deployment.
**Answer:** Check whether the new version exposes the metrics endpoint, whether the port changed, whether the service discovery labels changed, whether the instrumentation provider initializes before application startup, and whether the new image contains the required exporter packages.
