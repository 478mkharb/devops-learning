# OpenTelemetry L2 Interview Questions

This README is organized for **L2 DevOps / SRE / Observability interviews**. P1 topics are the core interview areas; P2 topics cover operational implementation and troubleshooting; P3 covers advanced architecture, scaling, and ecosystem topics.

| Priority | Questions | Main Focus |
|---|---:|---|
| 🔴 P1 - High | 1–57 | OpenTelemetry fundamentals, signals, SDKs, Collector, OTLP, context propagation, traces, metrics and logs |
| 🟠 P2 - Medium | 58–113 | Production configuration, processors, exporters, sampling, Kubernetes, troubleshooting and Grafana/Prometheus integration |
| 🟢 P3 - Low | 114–142 | Advanced scaling, tail sampling, multi-tenant architecture, observability pipelines and scenario questions |

---

## 1. What is OpenTelemetry and what problem does it solve?

**Priority: 🔴 P1**

OpenTelemetry (OTel) is an open-source observability framework for collecting, processing, and exporting **traces, metrics, and logs** using common APIs, SDKs, and protocols.

The main problem it solves is vendor-specific instrumentation and fragmented telemetry pipelines. Applications can instrument once and send telemetry to different backends.

```text
Application
   |
   +--> Traces
   +--> Metrics
   +--> Logs
          |
          v
   OpenTelemetry Collector
          |
          +--> Prometheus / Grafana
          +--> Jaeger / Tempo
          +--> Loki / Elasticsearch
          +--> Cloud vendors
```

**Interview point:** OpenTelemetry is not the storage backend. It is primarily a standard for instrumentation plus a telemetry pipeline.

---

## 2. What are the three main OpenTelemetry signals?

**Priority: 🔴 P1**

The core signals are:

| Signal | Purpose | Typical data |
|---|---|---|
| Traces | Follow a request across services | spans, parent/child relationships |
| Metrics | Measure system/application behavior over time | counters, gauges, histograms |
| Logs | Record discrete events | timestamped event records |

A production observability design often correlates all three using common resource attributes and trace context.

---

## 3. What is the difference between telemetry data, instrumentation, and a backend?

**Priority: 🔴 P1**

**Instrumentation** creates telemetry. **Telemetry data** is the generated trace, metric, or log data. A **backend** stores and queries it.

```text
Instrumentation -> Telemetry -> Collector/Pipeline -> Backend
     OTel SDK          spans        processors          Tempo
                                   exporters            Prometheus
                                   exporters            Loki
```

Do not describe OpenTelemetry as a replacement for every backend. It commonly sits between applications and backends.

---

## 4. What is the OpenTelemetry Collector?

**Priority: 🔴 P1**

The Collector is a vendor-neutral service that can **receive, process, and export telemetry**. It is commonly deployed as an agent close to workloads, a gateway, or both.

```text
Apps -> OTLP -> Collector -> processors -> exporters -> backends
```

Key benefits are centralized policy, batching, filtering, enrichment, routing, retries, and reduced coupling between applications and backends.

---

## 5. What are receivers, processors, exporters, connectors, and extensions?

**Priority: 🔴 P1**

These are the main Collector building blocks:

| Component | Role | Example |
|---|---|---|
| Receiver | Accepts telemetry | `otlp`, `prometheus`, `filelog` |
| Processor | Modifies/controls telemetry | `batch`, `memory_limiter`, `attributes` |
| Exporter | Sends telemetry out | `otlp`, `prometheusremotewrite`, `debug` |
| Connector | Links pipelines / transforms one pipeline output into another pipeline input | routing or span-to-metrics patterns |
| Extension | Operational support, not normal telemetry flow | health check, pprof |

A component must be enabled in the appropriate pipeline to be active.

---

## 6. What is the difference between OpenTelemetry SDK and Collector?

**Priority: 🔴 P1**

The **SDK** runs inside the application process and creates/records telemetry. The **Collector** runs as a separate service/process and receives, processes, and exports telemetry.

```text
Application
  OTel API/SDK
      |
      | OTLP
      v
 Collector
      |
      v
 Backend
```

You can export directly from an SDK to a backend-compatible endpoint, but the Collector is valuable when you need centralized processing and routing.

---

## 7. What are OTel API, SDK, and instrumentation libraries?

**Priority: 🔴 P1**

**API** defines the interfaces used by application code. **SDK** supplies the implementation and configuration. **Instrumentation libraries** automatically or manually instrument frameworks and libraries.

Typical pattern:

```text
Application code -> OTel API
Framework/library -> OTel instrumentation
SDK -> provider/exporter
```

This separation lets application code depend on the API while deployment chooses SDK behavior.

---

## 8. What is OTLP?

**Priority: 🔴 P1**

OTLP (OpenTelemetry Protocol) is the protocol used to transport OpenTelemetry telemetry between components. It commonly uses **gRPC or HTTP**.

A typical setup is:

```text
Service -> OTLP/gRPC -> OTel Collector -> backend
Service -> OTLP/HTTP  -> OTel Collector -> backend
```

Do not confuse OTLP with a specific storage backend; OTLP is a transport/protocol used in the telemetry pipeline.

---

## 9. What is the difference between OTLP/gRPC and OTLP/HTTP?

**Priority: 🔴 P1**

Both can carry OTLP telemetry. The choice usually depends on network constraints, proxy support, and deployment conventions.

| | OTLP/gRPC | OTLP/HTTP |
|---|---|---|
| Transport | HTTP/2-based gRPC | HTTP |
| Common use | service-to-service telemetry | environments preferring standard HTTP/proxies |
| Typical endpoint style | gRPC endpoint | HTTP path such as `/v1/traces` |

Use the protocol that matches the SDK/exporter and Collector configuration.

---

## 10. What is a span?

**Priority: 🔴 P1**

A span represents a timed unit of work within a trace. It normally has a name, start/end time, attributes, status, events, links, and parent/child relationships.

Example:

```text
Trace: checkout request

HTTP POST /checkout          [Span A]
  |
  +-- call inventory        [Span B]
  +-- charge payment        [Span C]
  +-- write order           [Span D]
```

A trace is composed of related spans.

---

## 11. What is a trace and how is it different from a span?

**Priority: 🔴 P1**

A **trace** represents the end-to-end path of an operation. A **span** is one operation within that trace.

```text
Trace ID: abc123
  Span A: API request
    Span B: DB query
    Span C: downstream HTTP call
```

A trace contains many spans and their relationships.

---

## 12. What is trace context?

**Priority: 🔴 P1**

Trace context carries identifiers needed to connect work across process and service boundaries. The important concepts are the **trace ID**, **span ID**, and propagation metadata.

Without context propagation, each service could create an unrelated trace and distributed tracing would be broken into disconnected pieces.

---

## 13. What is context propagation and why is it important?

**Priority: 🔴 P1**

Context propagation transfers trace context from one component to another, commonly through HTTP headers or messaging metadata.

```text
Frontend -> Service A -> Service B -> Database
   Trace Context  -------------------->
```

If Service A receives a request but fails to propagate the context to Service B, the downstream span may not appear under the same trace.

---

## 14. What is W3C Trace Context?

**Priority: 🔴 P1**

W3C Trace Context is the widely used standard for propagating distributed tracing context. The main HTTP headers are `traceparent` and optionally `tracestate`.

Example:

```text
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
```

The application framework or OTel instrumentation normally manages this rather than application developers manually constructing headers.

---

## 15. What are resource attributes in OpenTelemetry?

**Priority: 🔴 P1**

Resource attributes describe the entity producing telemetry, such as service name, version, deployment environment, host, or Kubernetes workload identity.

Common examples:

```text
service.name=employee-api
service.version=1.4.2
deployment.environment=prod
k8s.namespace.name=payroll
```

Resource attributes are different from span-specific attributes: they identify the producer/resource rather than one individual operation.

---

## 16. Why is service.name important?

**Priority: 🔴 P1**

`service.name` provides a stable logical identity for a service. It becomes a key dimension for grouping and querying telemetry across traces, metrics, and logs.

A weak naming strategy creates dashboards and traces that are difficult to correlate. Use a consistent service naming convention across environments.

---

## 17. What are span attributes, events, and links?

**Priority: 🔴 P1**

**Attributes** are key/value metadata on a span. **Events** are timestamped annotations inside a span. **Links** connect a span to another trace/span context without making it a parent/child.

Example:

```text
Span: process payment
  attributes: payment.provider=stripe
  event: retry_attempt=1
  link: original_batch_span=...
```

---

## 18. What is automatic instrumentation?

**Priority: 🔴 P1**

Automatic instrumentation uses existing framework/library hooks to create spans or metrics with minimal application code changes.

Examples include HTTP servers, client libraries, databases, messaging systems, and common frameworks.

**Benefit:** fast coverage.

**Limitation:** business-specific operations often still need manual spans or attributes.

---

## 19. What is manual instrumentation?

**Priority: 🔴 P1**

Manual instrumentation is explicit application code that creates spans, metrics, or log correlation where automatic instrumentation does not provide enough detail.

Conceptually:

```python
with tracer.start_as_current_span("calculate_salary"):
    result = calculate_salary(employee)
```

Use manual instrumentation for important business operations, custom workflows, and domain-specific attributes.

---

## 20. What are semantic conventions?

**Priority: 🔴 P1**

Semantic conventions define standardized attribute names, event names, metric names, and related conventions for common technologies.

They reduce the problem of one team calling an attribute `http.status` while another uses `status_code`.

**Interview point:** semantic conventions improve portability and consistent querying across services.

---

## 21. What are OTel metric types?

**Priority: 🔴 P1**

OpenTelemetry metrics support common instruments such as:

| Instrument | Typical use |
|---|---|
| Counter | monotonically increasing count |
| UpDownCounter | value can increase/decrease |
| Gauge | current measurement |
| Histogram | distribution of values |
| Observable variants | values observed asynchronously |

Metric instrument semantics matter because they affect aggregation and interpretation.

---

## 22. What is a histogram and why is it useful?

**Priority: 🔴 P1**

A histogram records a distribution of measurements, such as request latency or response size. It lets you reason about p50, p95, p99-style behavior using buckets/aggregation rather than only averages.

For latency, a histogram can answer:

```text
How many requests were <= 100 ms?
How many were <= 500 ms?
```

This is more useful than an average when tail latency matters.

---

## 23. What is a log record in OpenTelemetry?

**Priority: 🔴 P1**

A log record is a structured log event that can carry timestamp, severity, body, attributes, resource context, and trace/span correlation fields.

The key observability benefit is correlation: a log can be connected to the trace/span that generated it.

---

## 24. How do you correlate logs with traces?

**Priority: 🔴 P1**

The application/logger should include trace context such as trace ID and span ID in emitted logs, or a log pipeline should enrich records with that context.

```text
Trace ID = abc123
  -> log 1: database timeout
  -> log 2: retry started
  -> span: DB query
```

In a Grafana-based stack, this enables moving from a trace to related logs quickly.

---

## 25. What is an OpenTelemetry instrumentation scope?

**Priority: 🔴 P1**

An instrumentation scope identifies the library/module that produced telemetry, including instrumentation library metadata. It helps distinguish telemetry generated by different instrumentation libraries and versions. It is not the same as the service resource identity.

---

## 26. What is a Collector pipeline?

**Priority: 🔴 P1**

A pipeline defines the flow of one signal through configured components.

Example:

```yaml
service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [otlp]
```

Each pipeline explicitly lists its receivers, processors, and exporters.

---

## 27. What is the difference between agent and gateway Collector patterns?

**Priority: 🔴 P1**

An **agent** Collector runs close to the workload, commonly as a DaemonSet or sidecar-style deployment. A **gateway** Collector is a centralized tier that receives telemetry from agents/services and forwards it to backends.

```text
Pods/VMs -> Agent Collectors -> Gateway Collectors -> Backends
```

Agents provide local collection; gateways centralize processing and export policy.

---

## 28. Why use the Collector instead of sending telemetry directly to the backend?

**Priority: 🔴 P1**

A Collector can provide:

- batching
- retries
- filtering/redaction
- attribute enrichment
- sampling for traces
- routing to multiple backends
- centralized configuration
- protocol translation where supported

The trade-off is additional components and operational overhead.

---

## 29. What is the basic Collector YAML structure?

**Priority: 🔴 P1**

A common structure is:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:

processors:
  memory_limiter:
  batch:

exporters:
  debug:

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [debug]
```

Defining a component alone is not enough; it must be referenced by a pipeline.

---

## 30. Why is the batch processor commonly used?

**Priority: 🔴 P1**

The batch processor groups telemetry before export. This can reduce network overhead and improve exporter efficiency.

Typical placement:

```text
receiver -> processors -> batch -> exporter
```

Batch settings should be tuned with throughput, latency, memory, and backend limits in mind.

---

## 31. What does the memory_limiter processor do?

**Priority: 🔴 P1**

It helps protect the Collector from memory pressure by applying configured limits and refusing/delaying telemetry under pressure. It is commonly placed early in pipelines.

**Interview point:** it is not a replacement for correct capacity planning; it is a safety mechanism.

---

## 32. What is the attributes processor?

**Priority: 🔴 P1**

The attributes processor can insert, update, delete, or hash telemetry attributes. It is useful for normalization, enrichment, and removing sensitive values.

Example concept:

```yaml
processors:
  attributes:
    actions:
      - key: environment
        value: prod
        action: upsert
```

---

## 33. What is the resource processor?

**Priority: 🔴 P1**

The resource processor modifies resource attributes. Use it when you need to enrich or normalize information describing the telemetry-producing resource. This is different from modifying a span/event attribute directly.

---

## 34. What is filtering in an OTel pipeline?

**Priority: 🔴 P1**

Filtering prevents selected telemetry from continuing through the pipeline. It can be used to drop noisy health checks, low-value spans, sensitive records, or unwanted metrics.

A production filter should be explicit and tested because an incorrect condition can silently remove important telemetry.

---

## 35. What is sampling in distributed tracing?

**Priority: 🔴 P1**

Sampling decides which traces/spans are retained or processed. The goal is to reduce cost and storage while keeping useful traces.

Two broad approaches are:

- **Head sampling:** decision is made early.
- **Tail sampling:** decision can use information observed later in the trace.

Tail sampling can preserve error or high-latency traces more intelligently, but it requires more state and architecture.

---

## 36. What is head sampling?

**Priority: 🔴 P1**

Head sampling makes the sampling decision near the start of a trace. It is simple and low overhead, but it may discard a trace before learning that the request later failed.

Example policy idea: sample 10% of normal traffic.

For high-value error traces, head sampling may be insufficient by itself.

---

## 37. What is tail sampling?

**Priority: 🔴 P1**

Tail sampling waits long enough to inspect trace data before deciding whether to keep it. Policies can retain traces based on error status, latency, attributes, or probabilistic rules.

```text
Many spans -> Collector holds trace context -> policy decision -> keep/drop
```

This generally requires traces from the same trace to reach the appropriate sampling decision point.

---

## 38. Why is tail sampling harder to scale?

**Priority: 🔴 P1**

The Collector must retain state for in-flight traces and coordinate enough telemetry to make a decision. This increases memory, routing, and capacity requirements. In a distributed deployment, consistent routing of a trace to the same sampling decision point becomes important.

---

## 39. What is the `debug` exporter used for?

**Priority: 🔴 P1**

The `debug` exporter is useful for inspecting telemetry flowing through a Collector, especially during troubleshooting. It should not be treated as a production long-term backend.

---

## 40. How do you test whether an OTel Collector is receiving telemetry?

**Priority: 🔴 P1**

Use several layers of verification:

1. Check Collector startup/configuration logs.
2. Enable an inspection exporter such as `debug`.
3. Verify receiver endpoints and network connectivity.
4. Check Collector self-metrics.
5. Confirm the downstream exporter/backend.

This separates an instrumentation problem from a transport or backend problem.

---

## 41. What is OTEL_EXPORTER_OTLP_ENDPOINT?

**Priority: 🔴 P1**

It is commonly used by OTel SDKs to specify the OTLP destination endpoint. Exact environment variable behavior depends on the SDK/exporter and protocol.

Example concept:

```bash
OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
```

Always verify whether the selected exporter expects gRPC or HTTP and whether TLS is enabled.

---

## 42. What is the difference between 4317 and 4318?

**Priority: 🔴 P1**

In common OpenTelemetry Collector configurations:

| Port | Typical protocol |
|---|---|
| 4317 | OTLP over gRPC |
| 4318 | OTLP over HTTP |

Treat these as conventional defaults, not hard requirements. A deployment can configure different ports.

---

## 43. What is an OTel exporter?

**Priority: 🔴 P1**

An exporter sends telemetry from the SDK or Collector to another destination. In the Collector, exporter examples include OTLP and backend-specific exporters.

The exporter is the last step of a normal pipeline after receivers and processors.

---

## 44. What happens if the Collector exporter cannot reach the backend?

**Priority: 🔴 P1**

The outcome depends on exporter/queue/retry configuration. A resilient production setup commonly uses retry and queue mechanisms where supported, while monitoring Collector self-metrics for failures and queue growth.

Troubleshooting flow:

```text
Receiver OK -> Processor OK -> Exporter error -> network/auth/backend check
```

---

## 45. How do retries and queues help the Collector?

**Priority: 🔴 P1**

Exporter retry mechanisms can re-attempt transient failures. Persistent queues can buffer telemetry across some exporter/backend disruptions, depending on component support and deployment configuration.

They reduce immediate telemetry loss but introduce memory/disk consumption and recovery considerations.

---

## 46. How do you secure OTLP traffic?

**Priority: 🔴 P1**

Use TLS where traffic crosses trust boundaries and protect endpoints with authentication when supported/required by the deployment. Avoid sending credentials in logs or plaintext configuration.

Typical controls include:

```text
TLS + authentication + network policy/firewall + least privilege
```

---

## 47. How do you prevent sensitive data from entering telemetry?

**Priority: 🔴 P1**

Control it at multiple layers:

- do not instrument secrets directly
- avoid high-risk attributes
- redact/filter at the Collector
- hash identifiers when correlation is needed but raw values are not
- restrict access to backends

A Collector processor can be a final safety layer, but source-level prevention is preferable.

---

## 48. How does OpenTelemetry integrate with Prometheus?

**Priority: 🔴 P1**

OpenTelemetry can expose/export metrics in ways that Prometheus-compatible systems can consume. A Collector can also receive metrics and route them to a compatible metrics backend/export path.

The important distinction is that **OpenTelemetry is the telemetry framework/pipeline**, while **Prometheus is a metrics monitoring system and ecosystem**.

---

## 49. How does OpenTelemetry integrate with Grafana?

**Priority: 🔴 P1**

Grafana can visualize telemetry stored in compatible backends. OpenTelemetry commonly feeds backends such as Prometheus-compatible metrics systems, Tempo-compatible traces, and Loki-compatible logs.

```text
Application -> OTel -> metrics/traces/logs backends -> Grafana
```

Grafana is the visualization/query layer, not the Collector.

---

## 50. What is the difference between OpenTelemetry and Prometheus?

**Priority: 🔴 P1**

| OpenTelemetry | Prometheus |
|---|---|
| Observability framework | Monitoring/metrics system |
| Traces, metrics, logs | Primarily metrics |
| APIs, SDKs, Collector, protocol | TSDB, PromQL, scraping, alerting ecosystem |
| Vendor-neutral telemetry pipeline | Strong metrics ecosystem |

They are complementary and are often used together.

---

## 51. What is the difference between OpenTelemetry and Jaeger/Tempo?

**Priority: 🔴 P1**

OpenTelemetry generates/collects/transports telemetry; Jaeger and Tempo are trace backends/query systems.

```text
Service -> OTel SDK/Collector -> Trace backend -> UI
```

OTel is therefore not simply another trace UI.

---

## 52. What is the difference between OpenTelemetry and Fluent Bit/Fluentd?

**Priority: 🔴 P1**

Fluent Bit/Fluentd are commonly used for log collection and routing. OpenTelemetry is broader and standardizes traces, metrics, and logs plus instrumentation and a common transport model.

The right choice depends on whether you need a general observability pipeline, a specialized log pipeline, or both.

---

## 53. How would you instrument a microservices application for distributed tracing?

**Priority: 🔴 P1**

A practical flow is:

1. Install the language SDK and relevant instrumentation.
2. Set `service.name` and environment/resource attributes.
3. Configure OTLP export to a Collector.
4. Ensure context propagation across HTTP/messaging boundaries.
5. Add manual spans for business-critical operations.
6. Validate traces end-to-end in the backend.

```text
Frontend -> API -> employee-api -> DB
           Trace Context propagates through every hop
```

---

## 54. How would you monitor an OTel Collector itself?

**Priority: 🔴 P1**

Treat the Collector as production infrastructure. Monitor:

- process health
- CPU/memory
- receiver accepted/refused telemetry
- processor drops
- exporter failures
- queue size
- retry count
- telemetry throughput

Use Collector self-observability and infrastructure monitoring together.

---

## 55. What are Collector self-metrics?

**Priority: 🔴 P1**

The Collector can expose metrics about its own operation. These metrics help answer questions such as whether a receiver is accepting data, whether processors are dropping data, or whether an exporter is failing. They are essential for diagnosing telemetry-pipeline issues.

---

## 56. What is a common OTel troubleshooting decision tree?

**Priority: 🔴 P1**

Use this sequence:

```text
No telemetry?
   |
   +-- Is instrumentation enabled?
   |
   +-- Is endpoint/port correct?
   |
   +-- Is network/DNS/TLS working?
   |
   +-- Is Collector receiver accepting data?
   |
   +-- Are processors dropping it?
   |
   +-- Is exporter succeeding?
   |
   +-- Is backend querying the correct resource/signal?
```

This avoids randomly changing application settings.

---

## 57. Scenario: traces arrive for one service but not downstream services. What do you check?

**Priority: 🔴 P1**

Most likely areas:

1. context propagation between services
2. unsupported or missing HTTP/messaging instrumentation
3. async/background boundaries
4. proxy or gateway header handling
5. different sampling decisions
6. missing downstream service instrumentation

Start with the trace context and instrumentation path, not the backend.

---

## 58. Scenario: Collector CPU and memory keep increasing. What would you investigate?

**Priority: 🔴 P1**

Check:

- telemetry rate
- batch sizes
- processor memory behavior
- tail-sampling state
- exporter backpressure
- retry queues
- slow/unavailable backends
- cardinality/attribute explosion

A rising Collector resource footprint is often a symptom of downstream backpressure or excessive telemetry volume.

---

## 59. Scenario: traces are visible but logs cannot be correlated. Why?

**Priority: 🔴 P1**

Likely causes:

- logs do not contain trace/span context
- logger integration is missing
- context is lost before log emission
- Collector/backend mapping is inconsistent
- querying uses the wrong trace identifiers/fields

Check one request end-to-end and verify the same trace ID appears in the trace and log record.

---

## 60. Scenario: one service sends telemetry directly to a backend while others use the Collector. Is that necessarily wrong?

**Priority: 🔴 P1**

Not necessarily. Direct export can be valid. The design becomes problematic when configuration, security, routing, retries, and telemetry policy become inconsistent across many services. A Collector can centralize those concerns.

The interview answer should be based on operational requirements, not “Collector is always mandatory.”

---

## 61. What is the difference between application-level and Collector-level sampling?

**Priority: 🟠 P2**

Application-level sampling happens inside or near the instrumentation SDK. Collector-level sampling occurs in the telemetry pipeline.

Collector-level sampling is useful when you want centralized policies, especially tail sampling. Application-level sampling can reduce traffic before it ever reaches the Collector.

---

## 62. What is the difference between span attributes and resource attributes?

**Priority: 🟠 P2**

**Span attributes** describe an individual operation. **Resource attributes** describe the entity producing telemetry.

```text
Resource: service.name=employee-api
Span: http.request.method=GET
```

Do not use per-request IDs as resource attributes because that can destroy resource-level grouping.

---

## 63. What is attribute cardinality and why does it matter?

**Priority: 🟠 P2**

Cardinality is the number of distinct values for an attribute/dimension. High-cardinality attributes can increase memory, storage, query complexity, and backend costs.

Bad examples for metric dimensions include raw request IDs, user IDs, or arbitrary URLs with unique IDs. Normalize or remove such dimensions when appropriate.

---

## 64. What is baggage in OpenTelemetry?

**Priority: 🟠 P2**

Baggage is key/value context that can be propagated across service boundaries. It is separate from trace context. It can be useful for carrying selected business or routing context, but because baggage travels across services it should not contain secrets or large payloads.

---

## 65. How does baggage differ from span attributes?

**Priority: 🟠 P2**

A span attribute belongs to a specific telemetry record. Baggage is context propagated to downstream operations.

```text
Incoming request
   baggage -> Service A -> Service B
```

Use baggage carefully because it increases propagation and privacy considerations.

---

## 66. How would you propagate context over asynchronous messaging?

**Priority: 🟠 P2**

Inject trace context into message metadata/headers when publishing and extract it when consuming. Preserve the parent relationship or use span links when the messaging topology does not fit a simple parent/child relationship.

---

## 67. Why are span links useful in batch processing?

**Priority: 🟠 P2**

A batch consumer may process messages from several independent traces. Making one trace the parent of everything can misrepresent causality. Span links can associate the processing span with multiple source contexts instead.

---

## 68. What are exemplars and how do they help observability?

**Priority: 🟠 P2**

Exemplars associate a metric observation with trace context or another useful reference. They help connect a metrics signal such as high request latency to an example trace.

Conceptually:

```text
Metric spike -> exemplar -> trace -> downstream spans/logs
```

Support depends on the metric/backend/tooling path being used.

---

## 69. How would you deploy the Collector on Kubernetes?

**Priority: 🟠 P2**

Common patterns include:

| Pattern | Use |
|---|---|
| DaemonSet | node-local collection |
| Deployment | centralized gateway |
| Sidecar | tightly coupled workload-specific pipeline |

A common production design is DaemonSet agents plus a gateway tier for centralized processing/export.

---

## 70. What is the Kubernetes Operator approach for OpenTelemetry?

**Priority: 🟠 P2**

The OpenTelemetry Operator can manage Collector deployments and automate some instrumentation/deployment workflows in Kubernetes. It is useful when you want Kubernetes-native lifecycle management rather than managing every manifest independently.

---

## 71. How do you inject OpenTelemetry configuration into Kubernetes workloads?

**Priority: 🟠 P2**

Typical mechanisms include environment variables, ConfigMaps/Secrets, admission/instrumentation tooling, Helm values, and workload manifests. Keep endpoint and resource conventions consistent across namespaces and environments.

---

## 72. How would you configure environment-specific Collector settings?

**Priority: 🟠 P2**

Use configuration management rather than editing production files manually. Keep common pipeline structure stable and vary environment-specific values such as endpoints, sampling policies, resource attributes, and credentials through environment-specific manifests/values.

---

## 73. What should be stored in Kubernetes Secret rather than ConfigMap?

**Priority: 🟠 P2**

Credentials, tokens, certificates/private keys, and other secret material should use an appropriate secret-management mechanism. Non-sensitive Collector configuration can use ConfigMaps. Never put backend passwords directly into a publicly readable ConfigMap.

---

## 74. How do you validate an OTel Collector configuration before deployment?

**Priority: 🟠 P2**

Validate syntax and component configuration using the Collector's supported validation/startup mechanisms and test the exact image/version you deploy. A good CI workflow is:

```text
render config -> validate -> start test Collector -> smoke test telemetry -> deploy
```

Configuration compatibility is version-sensitive, so test with the same Collector distribution/version used in production.

---

## 75. How would you troubleshoot “unknown receiver/exporter/processor” errors?

**Priority: 🟠 P2**

The component may not exist in that Collector distribution, may have been renamed/deprecated, or the config may not match the installed version. Check the Collector distribution and version first. Do not assume every component exists in every build.

---

## 76. What is the difference between OpenTelemetry Collector Core and distribution builds?

**Priority: 🟠 P2**

Collector distributions can package different sets of components. A distribution may include components beyond the minimal/core set.

Therefore a configuration copied from another environment can fail if the target image does not contain the referenced receiver, processor, exporter, or connector.

---

## 77. Why should you pin the Collector image version?

**Priority: 🟠 P2**

Pinning reduces surprise changes in components, defaults, and compatibility. It makes incident reproduction and rollback easier. In production, update deliberately and test configuration compatibility before rollout.

---

## 78. How do you roll out Collector configuration safely?

**Priority: 🟠 P2**

Use version-controlled configuration, CI validation, staged deployment, health checks, and rollback. For gateways, use multiple replicas and progressive rollout so one configuration mistake does not stop all telemetry collection.

---

## 79. How do you design a resilient Collector gateway?

**Priority: 🟠 P2**

Use multiple gateway replicas, appropriate load balancing, bounded queues, retry policy, capacity headroom, and clear ownership of sampling/routing state. For stateful decisions such as tail sampling, architecture must preserve the required trace locality.

---

## 80. What does “same trace to the same Collector” mean for tail sampling?

**Priority: 🟠 P2**

Tail sampling needs the decision-maker to see the relevant spans for a trace. Therefore routing must ensure spans belonging to a trace arrive at the same sampling context/Collector instance or architecture designed to coordinate the state. Random load balancing can break this assumption.

---

## 81. What is remote sampling?

**Priority: 🟠 P2**

Remote sampling allows sampling policy to be centrally controlled rather than hard-coded independently in every application. This can simplify policy changes across many services, but support and implementation vary by language SDK and deployment.

---

## 82. How do you decide where to sample?

**Priority: 🟠 P2**

Sample as early as needed to control volume, but keep enough information to satisfy operational objectives. A common policy is to retain all or most errors and selected slow traces while sampling normal successful traffic. Measure storage and network savings before and after changes.

---

## 83. How would you reduce noisy health-check telemetry?

**Priority: 🟠 P2**

Prefer targeted filtering or instrumentation configuration rather than globally dropping large classes of telemetry. For example, low-value liveness/readiness calls may be filtered if they overwhelm traces without adding diagnostic value. Keep enough health information in metrics for service availability monitoring.

---

## 84. How do you add environment metadata to all telemetry?

**Priority: 🟠 P2**

Set resource attributes consistently at the application or Collector layer. For example:

```text
deployment.environment=prod
cloud.region=us-east-1
cluster.name=prod-eks
```

Use standardized attribute names where available.

---

## 85. How do you route telemetry to different backends by environment or tenant?

**Priority: 🟠 P2**

Use routing logic based on resource/attribute values and separate exporters/pipelines where appropriate. Ensure tenant identifiers are trustworthy and cannot be spoofed by untrusted clients without authentication/authorization controls.

---

## 86. How would you send metrics to Prometheus-compatible storage and traces to Tempo?

**Priority: 🟠 P2**

Conceptually configure separate pipelines:

```yaml
service:
  pipelines:
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [prometheusremotewrite]

    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [otlp/tempo]
```

Exact exporter names and configuration depend on the Collector distribution/version.

---

## 87. What is OpenTelemetry Collector feature gating/deprecation and why should interviewers care?

**Priority: 🟠 P2**

Collector components can be introduced, changed, deprecated, or removed over time. Production engineers should read release notes, validate configuration against the deployed version, and avoid treating old blog examples as authoritative.

---

## 88. What are common anti-patterns in OpenTelemetry deployments?

**Priority: 🟠 P2**

Common examples:

- exporting everything at 100% without capacity planning
- using high-cardinality attributes everywhere
- no Collector self-monitoring
- unbounded queues/retries
- putting secrets into telemetry attributes
- mixing incompatible Collector components/versions
- making every service own a complex backend-specific configuration
- losing context at async boundaries

---

## 89. How would you troubleshoot missing metrics but working traces?

**Priority: 🟠 P2**

Treat signals independently. Check:

1. metric instrument/export enabled
2. metrics pipeline exists in the Collector
3. metric receiver/exporter configuration
4. processor filtering
5. backend ingestion
6. query/label/resource mapping

Working traces prove connectivity only for the trace pipeline; they do not prove the metric pipeline is correct.

---

## 90. How would you troubleshoot missing logs but working traces?

**Priority: 🟠 P2**

Check whether logs are actually emitted through an OTel log pipeline or are still handled by a separate logging stack. Then verify log collection, receiver/file permissions, processing/filtering, exporter delivery, and backend indexing/query. Trace success does not prove log pipeline success.

---

## 91. What should you monitor on the Collector gateway?

**Priority: 🟠 P2**

At minimum monitor telemetry throughput, accepted/refused data, exporter errors, queue size, retries, memory, CPU, and instance health. Alert on sustained exporter failures or queue saturation rather than only process-down events.

---

## 92. What is backpressure in an observability pipeline?

**Priority: 🟠 P2**

Backpressure occurs when the downstream stage cannot consume telemetry as quickly as the upstream stage produces it. Symptoms include queues growing, retries increasing, memory pressure, dropped telemetry, or increasing end-to-end latency.

---

## 93. What happens when telemetry volume spikes suddenly?

**Priority: 🟠 P2**

The pipeline absorbs the spike only up to its configured capacity. Beyond that, queues fill, memory pressure rises, exporters slow, and data may be dropped. A production design uses batching, bounded queues, scaling, selective sampling, and backend capacity planning.

---

## 94. How would you estimate Collector capacity?

**Priority: 🟠 P2**

Measure telemetry volume, payload sizes, span/metric/log rates, processor CPU cost, exporter throughput, and memory per in-flight item. Then load test with realistic traffic and leave headroom for bursts. Avoid sizing from CPU alone because exporter backpressure and memory can become the bottleneck.

---

## 95. What is a canary Collector deployment?

**Priority: 🟠 P2**

A canary deployment sends a small portion of telemetry through a new Collector version/configuration before broad rollout. Compare error rates, throughput, memory, queue behavior, and backend ingestion before increasing traffic.

---

## 96. How do you handle Collector upgrades safely?

**Priority: 🟠 P2**

Pin versions, review release notes, validate configuration, run integration tests, canary the new version, monitor self-metrics, and retain rollback capability. Component availability can change between versions/distributions.

---

## 97. How would you secure the Collector in a multi-tenant environment?

**Priority: 🟠 P2**

Use network segmentation, TLS, authentication/authorization at supported boundaries, tenant-aware routing, quotas/capacity isolation, and careful resource attributes. Do not trust client-supplied tenant metadata without controls.

---

## 98. How would you design observability for an AWS EC2 microservice platform?

**Priority: 🟠 P2**

A practical model is:

```text
EC2 services -> OTel SDKs/agents -> Collector ->
   metrics -> Prometheus-compatible backend
   traces  -> Tempo/Jaeger backend
   logs    -> Loki/other log backend
                       |
                    Grafana
```

Node-level infrastructure metrics can be collected separately or through appropriate OTel/Prometheus components.

---

## 99. How would you combine OTel with Prometheus and Grafana in a DevOps project?

**Priority: 🟠 P2**

Use OTel for standardized application telemetry and a Collector for routing/processing. Use Prometheus-compatible monitoring for infrastructure/application metrics and Grafana as the shared visualization layer. Traces can go to Tempo/Jaeger and logs to Loki/another backend, with cross-links in Grafana.

---

## 100. What is the role of OpenTelemetry in an observability reference architecture?

**Priority: 🟠 P2**

OTel is the standardization and collection layer between workloads and observability backends. It helps decouple instrumentation from storage/visualization choices and centralizes policy such as enrichment, batching, filtering, and sampling.

---

## 101. Scenario: a new deployment suddenly doubles telemetry volume. What would you inspect first?

**Priority: 🟠 P2**

Compare before/after instrumentation changes. Look for:

- new instrumentation libraries
- new span attributes/events
- changed sampling
- duplicated telemetry
- high-frequency metrics
- noisy endpoints

Use Collector and backend metrics to determine which signal and service caused the increase.

---

## 102. Scenario: only one Kubernetes namespace loses traces after a Collector upgrade. What is your approach?

**Priority: 🟠 P2**

Compare that namespace's endpoint, DNS, NetworkPolicy, service account, resource attributes, injection configuration, and Collector routing against a working namespace. Also inspect whether the new Collector distribution still contains the configured components.

---

## 103. Scenario: backend is healthy but Collector exporter queue is constantly full. What does that suggest?

**Priority: 🟠 P2**

It suggests the effective export path is not keeping up with ingestion. Investigate exporter throughput, network latency, concurrency, batch configuration, authentication delays, load balancer behavior, and whether traffic is unevenly distributed across gateway replicas.

---

## 104. Scenario: one trace contains spans from two unrelated traces. What could cause it?

**Priority: 🟠 P2**

Possible causes include incorrect context propagation, reused context/span objects, instrumentation bugs, or incorrect manual parent assignment. Distributed tracing relies on correct context boundaries; backend UI alone is not the right place to fix causality.

---

## 105. Scenario: trace latency looks high, but the service average is normal. How can OTel help?

**Priority: 🟠 P2**

Inspect the distributed trace and break total latency into downstream spans. You may find a small percentage of requests have slow database, network, queue, or external API spans even when the service average looks normal. Histograms and exemplars can complement traces for this use case.

---

## 106. Scenario: you need all error traces but only 5% of successful traces. What architecture would you choose?

**Priority: 🟠 P2**

Use a trace sampling policy that can distinguish errors from normal traffic. Tail sampling is a common fit because the decision can be based on information observed later in the trace. Ensure trace-locality and capacity requirements are satisfied.

---

## 107. What is the difference between logs collected by OTel and application logs written to files?

**Priority: 🟠 P2**

OTel provides a standardized telemetry model and pipeline. A file is merely one possible source of logs. A Collector can read file logs with an appropriate receiver, parse/enrich them, correlate them with resource/context information, and export them to a backend.

---

## 108. How do you avoid duplicate telemetry when both an agent and application export directly?

**Priority: 🟠 P2**

Map the pipeline explicitly. If the app exports directly and also writes to an agent that exports the same signal, the backend may receive duplicates. Define a single intended path per signal or use clear routing rules.

---

## 109. What is the difference between Collector processor orderings?

**Priority: 🟠 P2**

Processors execute in the order configured in the pipeline. Order matters. For example, enriching attributes before filtering may allow the filter to use the added attribute; batching generally belongs toward the end of a processing chain before export.

---

## 110. Why might a filter processor accidentally drop everything?

**Priority: 🟠 P2**

A condition can be broader than expected, the attribute may be absent or typed differently, or the expression may match more records than intended. Test filters with representative telemetry and use temporary inspection/exporting to confirm behavior.

---

## 111. What is the difference between dropping telemetry at the SDK and at the Collector?

**Priority: 🟠 P2**

SDK-level dropping reduces traffic before network transmission, while Collector-level dropping centralizes policy but consumes some resources to receive/process the telemetry first. Use the earliest safe control point when volume reduction is important.

---

## 112. How would you version-control OpenTelemetry configuration?

**Priority: 🟠 P2**

Keep Collector configuration, Helm values/manifests, environment overlays, and dashboards in source control. Use pull requests, CI validation, image/version pinning, and release tags. Treat observability configuration like production application infrastructure.

---

## 113. What should a CI pipeline test for an OTel Collector change?

**Priority: 🟠 P2**

At minimum:

```text
YAML/config validation
component availability
startup smoke test
representative telemetry ingestion
export success
negative cases for filtering/sampling
```

Production-critical pipelines should also load-test significant topology changes.

---

## 114. How do you debug TLS certificate problems between an app and Collector?

**Priority: 🟠 P2**

Check hostname/SAN, CA trust, certificate expiry, client/server TLS mode, protocol/port mismatch, and whether a proxy terminates TLS. Validate from the same runtime environment as the application/Collector, not only from your laptop.

---

## 115. Why can DNS issues look like an OpenTelemetry problem?

**Priority: 🟠 P2**

The application may be healthy while the telemetry endpoint hostname does not resolve from its network namespace. Always separate application failures from telemetry transport failures. Check DNS, routing, firewall/NetworkPolicy, and endpoint connectivity explicitly.

---

## 116. What is traceparent sampling flag used for?

**Priority: 🟠 P2**

The trace context contains flags that communicate trace state, including whether the trace is sampled. Instrumentation and downstream components can use this context when deciding how to handle a trace. Do not manually mutate trace context unless you understand the propagation semantics.

---

## 117. What is tracestate?

**Priority: 🟠 P2**

`tracestate` carries vendor-specific trace context alongside W3C `traceparent`. It allows propagation of additional tracing information without changing the standard trace ID/span ID relationship.

---

## 118. What is a Collector extension used for?

**Priority: 🟠 P2**

Extensions provide operational features around the Collector rather than being normal telemetry transformation stages. Common examples include health checking or profiling support.

---

## 119. How do you know whether a Collector receiver is listening?

**Priority: 🟠 P2**

Use the Collector startup configuration/logs and the operating system/network layer to confirm the expected bind address and port. Then send a known test payload or check the receiver's operational metrics. Listening on the wrong interface is a common deployment error.

---

## 120. What is a common reason OTLP works locally but fails in Kubernetes?

**Priority: 🟠 P2**

Common causes are wrong service DNS name, wrong port/protocol, NetworkPolicy, TLS mismatch, service exposure, container listening only on localhost, or environment variable differences. Compare the actual runtime configuration rather than the local development configuration.

---

## 121. What is the difference between push and pull for OpenTelemetry?

**Priority: 🟠 P2**

OTel application SDKs commonly **push** telemetry to an OTLP endpoint. Prometheus commonly **pulls** metrics from scrape targets. These models can coexist in one architecture.

```text
Apps --push--> OTel Collector
Prometheus --pull--> /metrics
```

The correct model depends on the signal and components used.

---

## 122. How does the Prometheus receiver fit into an OpenTelemetry Collector?

**Priority: 🟠 P2**

The Prometheus receiver can scrape Prometheus-style endpoints and bring those metrics into an OTel pipeline. This is useful when you want the Collector to act as a collection point for existing Prometheus-format metrics.

---

## 123. When would you use an OTel Collector instead of a Prometheus server to collect node metrics?

**Priority: 🟢 P3**

Choose based on requirements. Prometheus is purpose-built for metrics scraping, time-series storage, and PromQL. OTel Collector can provide a broader vendor-neutral telemetry pipeline. Many platforms use both rather than replacing one with the other.

---

## 124. What is federation in an OpenTelemetry architecture?

**Priority: 🟢 P3**

Federation is generally associated with metrics systems such as Prometheus, while OTel commonly uses Collector-to-Collector pipelines and backend-specific mechanisms. Do not force Prometheus terminology onto OTel architecture. Use the actual data path and component roles.

---

## 125. What is an observability gateway pattern?

**Priority: 🟢 P3**

A gateway is a centralized Collector tier that receives telemetry from many workloads/agents, applies shared policies, and exports to one or more backends. It simplifies backend credentials and centralizes routing, but becomes a capacity and availability boundary.

---

## 126. How would you make a gateway tier highly available?

**Priority: 🟢 P3**

Run multiple replicas, load balance telemetry, size for node failure and traffic spikes, monitor exporter queues, and avoid singleton state unless the sampling/routing architecture specifically requires it. Tail sampling requires special consideration for trace locality.

---

## 127. How would you design multi-region OpenTelemetry collection?

**Priority: 🟢 P3**

A common pattern is regional agent collectors sending to regional gateways/backends. Keep telemetry traffic local where practical and forward centrally only when required. Consider latency, data residency, failure domains, and cross-region bandwidth.

---

## 128. How would you design observability for 10,000+ services?

**Priority: 🟢 P3**

Standardize resource attributes and instrumentation, centralize configuration, control cardinality, sample intelligently, use Collector tiers, scale gateways horizontally, monitor telemetry pipelines, and establish ownership for schemas/semantic conventions. The key problem is governance and cost as much as raw throughput.

---

## 129. How do you prevent an observability system from becoming its own outage source?

**Priority: 🟢 P3**

Use bounded resources, sampling, retries with limits, queues, rate control, HA collectors, backend capacity planning, and monitoring of the telemetry pipeline itself. Observability should degrade gracefully rather than taking application traffic down.

---

## 130. What is collector-to-collector communication?

**Priority: 🟢 P3**

One Collector can export telemetry to another Collector using supported protocols, allowing multi-tier pipelines. This is useful for separating edge collection from centralized processing/export.

---

## 131. When would you use sidecar collectors?

**Priority: 🟢 P3**

Sidecars can provide workload-specific processing or network locality, but they increase resource overhead and operational complexity. They are useful when isolation or per-workload policy is more important than shared agent efficiency.

---

## 132. What are the trade-offs of a DaemonSet versus sidecar Collector?

**Priority: 🟢 P3**

| DaemonSet | Sidecar |
|---|---|
| shared per-node collector | collector per workload/pod |
| lower duplicated overhead | stronger isolation |
| easier node-level collection | easier workload-specific routing |
| failure domain is node-local | more pods/components to operate |

---

## 133. How would you route high-value traces differently from low-value traces?

**Priority: 🟢 P3**

Use attributes, resource metadata, or sampling policies to classify telemetry. High-value traffic can be retained at a higher rate or routed to a dedicated backend/retention tier. Keep policies explicit and measurable.

---

## 134. What is the observability cost-control strategy for OTel?

**Priority: 🟢 P3**

Control cost through instrumentation hygiene, attribute/cardinality limits, sensible sampling, batching, filtering, retention policies, backend tiering, and capacity measurement. “Collect everything” is usually not a sustainable production strategy.

---

## 135. How would you migrate from vendor-specific tracing to OpenTelemetry?

**Priority: 🟢 P3**

Inventory current instrumentation and exporters, define standard resource/semantic conventions, migrate one service or domain at a time, send to the existing backend where possible, validate trace continuity, and only then change backend dependencies. Keep rollback paths during migration.

---

## 136. How would you migrate a fleet from direct backend export to a Collector?

**Priority: 🟢 P3**

Introduce the Collector alongside the existing path, validate parity, canary selected services, compare telemetry volume and quality, then cut over endpoint configuration gradually. Remove the direct path only after duplicate-data and loss risks are understood.

---

## 137. Scenario brain teaser: A Collector is healthy, backend is healthy, but users report no traces. Where can the failure be?

**Priority: 🟢 P3**

There are many possible points:

```text
Instrumentation -> SDK provider -> exporter -> DNS/network/TLS
-> Collector receiver -> processors -> exporter -> backend ingestion
-> backend query/index -> UI/filter
```

“Collector healthy” only proves the process itself is healthy; it does not prove every pipeline stage is receiving and exporting telemetry.

---

## 138. Scenario brain teaser: Trace IDs are present in logs, but clicking from a trace to logs returns nothing. What would you check?

**Priority: 🟢 P3**

Check exact field names/types, backend indexing, Grafana data-source configuration, time ranges, tenant/org context, and whether the stored log record contains the same trace ID value without transformation. Correlation requires both telemetry data and correct UI/backend linking configuration.

---

## 139. Scenario brain teaser: 100% sampling is enabled and traces are still missing. Is sampling the first thing to blame?

**Priority: 🟢 P3**

No. First verify instrumentation, context propagation, exporter endpoint, network/TLS, Collector receiver acceptance, processor drops, exporter errors, backend ingestion, and query filters. Sampling is only one stage in the path.

---

## 140. Scenario brain teaser: Adding one user ID attribute causes backend cost to explode. Why?

**Priority: 🟢 P3**

User IDs can create extreme cardinality. If they are used on metrics or high-volume telemetry dimensions, storage/index/query costs can grow dramatically. Keep such identifiers out of metric labels/dimensions unless there is a clear bounded-cardinality use case.

---

## 141. Scenario brain teaser: Tail sampling keeps error traces but some traces are fragmented. What is a likely architectural issue?

**Priority: 🟢 P3**

Trace locality may be broken: spans from one trace are reaching different sampling decision-makers. Check load balancing/routing between Collector tiers and ensure the architecture preserves the state required for tail sampling.

---

## 142. What are the first 10 OpenTelemetry concepts an L2 engineer should memorize?

**Priority: 🟢 P3**

Memorize these relationships:

```text
Signal: traces / metrics / logs
Trace -> spans
Context propagation -> distributed correlation
Resource attributes -> producer identity
SDK -> creates/exports telemetry
Collector -> receives/processes/exports
OTLP -> transport protocol
Receivers -> processors -> exporters
Head vs tail sampling
Backend + Grafana -> storage/query/visualization
```

---

# 4. Frequently Used OpenTelemetry Collector Components

| Component | Typical purpose | Example use |
|---|---|---|
| `otlp` receiver | receive OTLP | app -> Collector |
| `prometheus` receiver | scrape Prometheus targets | existing `/metrics` workloads |
| `filelog` receiver | collect files | application logs |
| `batch` processor | batch telemetry | efficient export |
| `memory_limiter` | protect Collector memory | production safety |
| `attributes` | modify attributes | redact/enrich |
| `resource` | modify resource metadata | environment/cluster identity |
| filtering processor | drop selected telemetry | remove noise |
| tail sampling processor | policy-based trace retention | keep errors/slow traces |
| `otlp` exporter | send OTLP | Collector -> gateway/backend |
| Prometheus-compatible exporter | metrics export | metrics backend integration |
| `debug` exporter | inspect telemetry | troubleshooting |

### Basic Collector pipeline

```yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:

processors:
  memory_limiter:
  batch:

exporters:
  debug:

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [debug]
```

**Rule:** defining a receiver/processor/exporter does not activate it. The component must be referenced by a `service.pipelines` entry.

---

# 5. Common OpenTelemetry Environment Variables

| Variable | Typical purpose |
|---|---|
| `OTEL_SERVICE_NAME` | logical service name |
| `OTEL_RESOURCE_ATTRIBUTES` | resource metadata |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | OTLP destination |
| `OTEL_EXPORTER_OTLP_PROTOCOL` | OTLP transport choice |
| `OTEL_TRACES_SAMPLER` | trace sampling strategy |
| `OTEL_TRACES_SAMPLER_ARG` | sampler argument |
| `OTEL_PROPAGATORS` | propagation formats |
| `OTEL_METRICS_EXPORTER` | metrics exporter selection |
| `OTEL_LOGS_EXPORTER` | logs exporter selection |

Exact supported values depend on the language SDK and version.

---

# 6. Quick Revision Table

| Concept | Remember |
|---|---|
| OpenTelemetry | framework + instrumentation + telemetry pipeline |
| Signals | traces, metrics, logs |
| Span | one unit of work |
| Trace | collection of related spans |
| Context propagation | connects distributed work |
| Resource | identifies telemetry producer |
| OTLP | telemetry transport protocol |
| SDK | application-side telemetry implementation |
| Collector | receive/process/export telemetry |
| Receiver | telemetry input |
| Processor | modify/control telemetry |
| Exporter | telemetry output |
| Connector | connect Collector pipelines |
| Extension | operational Collector capability |
| Batch | group telemetry for efficient export |
| Memory limiter | protect Collector memory |
| Head sampling | early sampling decision |
| Tail sampling | decision after observing trace data |
| DaemonSet | node-local Collector pattern |
| Gateway | centralized Collector tier |
| Cardinality | number of unique attribute values |
| Semantic conventions | standardized telemetry naming |
| Baggage | propagated key/value context |
| Prometheus | metrics system; often used with OTel |
| Grafana | visualization/query layer |

---

# 7. L2 Interview Checklist

Before an interview, be able to explain without notes:

- OTel vs Prometheus vs Grafana vs Jaeger/Tempo
- traces, spans, context propagation and W3C Trace Context
- SDK vs Collector
- OTLP/gRPC vs OTLP/HTTP
- Collector receivers/processors/exporters/connectors/extensions
- agent vs gateway architecture
- head vs tail sampling
- resource vs span attributes
- cardinality and telemetry cost
- Prometheus receiver and metrics integration
- Kubernetes DaemonSet/Deployment patterns
- Collector troubleshooting and self-monitoring
- HA, backpressure, retries and queues
- secure telemetry pipelines
- practical production scenarios
