# Part 02 — OpenTelemetry Core Concepts

> **OpenTelemetry Book | OT-Micro-Docker**
>
> Understand how application activity becomes traces, metrics, and logs.

## Table of Contents

1. [What Is OpenTelemetry?](#1-what-is-opentelemetry)
2. [Why OpenTelemetry Exists](#2-why-opentelemetry-exists)
3. [OpenTelemetry Is Not a Backend](#3-opentelemetry-is-not-a-backend)
4. [Complete Telemetry Lifecycle](#4-complete-telemetry-lifecycle)
5. [Main Components](#5-main-components)
6. [API vs SDK](#6-api-vs-sdk)
7. [Instrumentation](#7-instrumentation)
8. [Automatic vs Manual Instrumentation](#8-automatic-vs-manual-instrumentation)
9. [Providers, Processors, Readers, and Samplers](#9-providers-processors-readers-and-samplers)
10. [Resources and Attributes](#10-resources-and-attributes)
11. [Semantic Conventions](#11-semantic-conventions)
12. [Exporters and OTLP](#12-exporters-and-otlp)
13. [Collector](#13-collector)
14. [Context Propagation](#14-context-propagation)
15. [OT-Micro-Docker Mapping](#15-ot-micro-docker-mapping)
16. [Common Misunderstandings](#16-common-misunderstandings)
17. [Interview Questions](#17-interview-questions)
18. [Quick Revision](#18-quick-revision)
19. [Self-Assessment](#19-self-assessment)

---

## 1. What Is OpenTelemetry?

**OpenTelemetry (OTel)** is an open-source, vendor-neutral framework for **generating, collecting, and exporting telemetry**.

It supports three major signals:

- **Traces:** the path and timing of an operation.
- **Metrics:** numerical measurements over time.
- **Logs:** timestamped events or messages.

OpenTelemetry provides APIs, SDKs, instrumentation libraries, exporters, the Collector, and semantic conventions.

> OpenTelemetry is the instrumentation and telemetry-pipeline layer between an application and an observability backend.

---

## 2. Why OpenTelemetry Exists

Without a standard, an application may become coupled to one vendor:

```text
Application → Vendor SDK → Vendor Backend
```

With OpenTelemetry:

```text
Application → OTel API/SDK → Exporter/OTLP → Collector or Backend
```

This improves:

| Benefit | Meaning |
|---|---|
| Vendor neutrality | Less dependence on one backend vendor |
| Consistency | Common APIs, protocols, and conventions |
| Multi-signal support | Traces, metrics, and logs |
| Central processing | Batch, filter, enrich, route, and retry telemetry |
| Migration flexibility | Backend changes require fewer application changes |

---

## 3. OpenTelemetry Is Not a Backend

OpenTelemetry primarily creates, collects, processes, and exports telemetry.

A backend stores, queries, visualizes, and alerts on telemetry.

| Component | Main responsibility |
|---|---|
| OTel API | Defines instrumentation interfaces |
| OTel SDK | Implements telemetry behavior |
| Instrumentation | Creates telemetry from application activity |
| OTel Collector | Receives, processes, and exports telemetry |
| Prometheus | Metrics storage/querying ecosystem |
| Grafana | Dashboards and visualization |
| Jaeger/Tempo | Trace backends |
| Loki/Elasticsearch | Log/search backends |

Installing OpenTelemetry does not automatically provide dashboards, trace search, retention, or alerts.

---

## 4. Complete Telemetry Lifecycle

```text
Application operation
        ↓
Instrumentation
        ↓
OTel API
        ↓
Provider / SDK
        ↓
Sampler + Processor / Reader
        ↓
Exporter
        ↓
OTLP or another protocol
        ↓
Collector Receiver
        ↓
Collector Processor
        ↓
Collector Exporter
        ↓
Backend
        ↓
Dashboards / Queries / Alerts
```

### Important clarification

The API does **not** independently send telemetry to a backend.

```python
from opentelemetry import trace

tracer = trace.get_tracer("salary-api")
```

This is a library API call that obtains a `Tracer`. The configured SDK, processors, samplers, and exporters perform the actual telemetry work.

---

## 5. Main Components

| Component | Responsibility |
|---|---|
| API | Contract used by instrumentation |
| SDK | Implementation behind the API |
| Provider | Supplies tracer, meter, or logger implementations |
| Instrumentation | Observes application activity and creates telemetry |
| Processor | Handles telemetry after creation/collection |
| Reader | Collects and exports metric data |
| Sampler | Controls which traces/spans are recorded or sampled |
| Exporter | Sends telemetry to another destination |
| Collector | Central telemetry pipeline |
| Backend | Stores and analyzes telemetry |

---

## 6. API vs SDK

### API

The API defines what instrumentation can call:

- Trace API
- Metrics API
- Logs API
- Context and propagation APIs

`trace.get_tracer("employee-api")` is a programming-library API call, not a REST request.

### SDK

The SDK implements what happens after the API is used:

```text
Instrumentation → API → Provider → SDK → Processor/Reader → Exporter
```

| Aspect | API | SDK |
|---|---|---|
| Meaning | Interface/contract | Implementation |
| Creates implementation? | No | Yes |
| Configures exporters? | No | Yes |
| Handles sampling? | No implementation | Yes |
| Main question | What can be called? | How is telemetry handled? |

---

## 7. Instrumentation

**Instrumentation** is the process of adding or enabling telemetry collection for application activity. It is not the process of writing the OTel API itself.

### Technical instrumentation

- HTTP requests
- Database queries
- Cache calls
- Runtime CPU and memory
- External API calls
- Error status codes

### Business instrumentation

- `employee.create`
- `attendance.mark`
- `salary.generate`
- `salary.persist`
- `salary_pdf.generate`
- `email.send`

Do not create a span for every small helper function. Prefer meaningful operations and boundaries.

---

## 8. Automatic vs Manual Instrumentation

### Automatic instrumentation

Creates telemetry with minimal business-code changes using agents, framework integrations, runtime hooks, or instrumentation packages.

**Good for:** HTTP servers, HTTP clients, supported databases, runtimes, and common frameworks.

**Limitations:** It may miss business workflows, unsupported libraries, custom retry logic, and domain-specific failures.

### Manual instrumentation

Developers explicitly create telemetry:

```python
with tracer.start_as_current_span("salary_pdf.generate"):
    generate_pdf()
```

**Good for:** business operations, custom workflows, queue processing, retry logic, and important internal operations.

| Aspect | Automatic | Manual |
|---|---|---|
| Setup | Lower | Higher |
| Business context | Limited | Strong |
| Code changes | Minimal/none in business code | Required |
| Control | Lower | High |
| Best use | Standard libraries/frameworks | Business-critical operations |

Recommended approach:

```text
Automatic instrumentation + Manual business spans = Useful observability
```

---

## 9. Providers, Processors, Readers, and Samplers

### Provider

Supplies the implementation used by the API:

- `TracerProvider`
- `MeterProvider`
- Logger provider

### Processor

Handles telemetry after creation. A span processor may receive ended spans, batch them, and pass them to an exporter.

### Reader

Controls how metric data is collected and exported, such as periodic metric export or pull-based collection.

### Sampler

Controls whether traces/spans are recorded or sampled. Sampling reduces overhead, network traffic, and storage cost, but aggressive sampling can remove useful evidence.

---

## 10. Resources and Attributes

A **resource** describes the telemetry producer:

```text
service.name = salary-api
service.version = 1.2.0
deployment.environment.name = dev
service.instance.id = salary-api-7f8d9
```

A span or metric attribute describes the operation/event:

```text
http.request.method = POST
http.response.status_code = 200
db.system = cassandra
business.operation = salary.generate
```

| Question | Resource | Span/metric/log attributes |
|---|---|---|
| Describes | Who produced telemetry? | What happened? |
| Examples | `service.name`, version | HTTP method, DB operation |
| Identity | Service/instance | Request/operation details |

`service.name` must be stable and logical. Do not use a changing container ID as the service name.

Good:

```text
service.name = employee-api
service.instance.id = employee-api-7f8d9
```

---

## 11. Semantic Conventions

Semantic conventions standardize names and meanings for telemetry attributes, events, and operations.

They improve:

- Consistency
- Query portability
- Dashboard reuse
- Cross-team troubleshooting
- Backend interoperability

Custom business attributes are allowed when standard conventions do not cover the domain:

```text
business.operation = salary.generate
notification.channel = email
employee.type = full_time
```

Avoid recording:

```text
employee.salary = 125000
employee.email = user@example.com
full_request_body = ...
```

Protect salary data, personal data, tokens, and other sensitive information.

---

## 12. Exporters and OTLP

An exporter sends telemetry to another destination.

### Direct export

```text
Application → Exporter → Backend
```

### Collector-based export

```text
Application → OTLP Exporter → Collector → Backend
```

Collector-based export provides centralized processing, routing, batching, retry policy, and backend migration flexibility, at the cost of another component to operate.

**OTLP** means **OpenTelemetry Protocol**. It is a standard protocol for transmitting OpenTelemetry telemetry.

---

## 13. Collector

The Collector is a separate service or process that receives, processes, and exports telemetry.

```text
Receivers → Processors → Exporters
```

### Receivers

Accept telemetry, such as through OTLP or Prometheus-compatible inputs.

### Processors

Batch, filter, enrich, modify, limit, or route telemetry.

### Exporters

Send telemetry to trace, metric, log, or vendor backends.

The Collector is optional. Direct export is possible, but a Collector is valuable in multi-service environments.

---

## 14. Context Propagation

Context propagation carries trace context across process and service boundaries. A trace context commonly includes a trace ID, span ID, flags, and trace state.

### HTTP

```text
Upstream service
    ↓ inject context
HTTP request
    ↓ extract context
Downstream service
    ↓
Child span
```

### Messaging

```text
Producer → inject context → Queue → extract context → Consumer span
```

Without propagation, one user request may appear as multiple unrelated traces.

| Concept | Purpose |
|---|---|
| Context propagation | Connects operations into one distributed trace |
| Exporting | Sends telemetry to a Collector or backend |

---

## 15. OT-Micro-Docker Mapping

### Conceptual flow

```text
Frontend / NGINX
      ↓
Employee API ──→ ScyllaDB
      ↓
Attendance API ──→ PostgreSQL + Redis
      ↓
Salary API ──→ ScyllaDB ──→ Elasticsearch mirror
                                      ↓
                              Notification worker
                                      ↓
                                     SMTP
```

### Suggested service names

| Component | `service.name` |
|---|---|
| Employee API | `employee-api` |
| Attendance API | `attendance-api` |
| Salary API | `salary-api` |
| Notification API | `notification-api` |
| Notification worker | `notification-worker` |
| Frontend | `frontend` |
| NGINX | `nginx` |
| Collector | `otel-collector` |

### Instrumentation priorities

1. HTTP server and client instrumentation.
2. Database and cache instrumentation.
3. Manual business spans.
4. Context propagation validation.
5. Trace/log correlation.
6. Metrics, dashboards, and alerts.

### Salary workflow spans

```text
salary.generate
salary.persist
salary.index
notification.process
salary_pdf.generate
email.send
```

Use stable, low-cardinality span names. Do not embed employee IDs in span names.

---

## 16. Common Misunderstandings

1. **OTel is only tracing.** No; it supports traces, metrics, and logs.
2. **Instrumentation means writing the OTel API.** No; it means adding/enabling telemetry collection.
3. **The API sends telemetry.** The SDK and exporters perform the implementation work.
4. **API and SDK are identical.** API is the contract; SDK is the implementation.
5. **Automatic instrumentation needs no configuration.** Dependencies, startup settings, agents, and exporters may still be required.
6. **Automatic instrumentation covers every function.** It mainly covers supported libraries and frameworks.
7. **The Collector is mandatory.** No; direct export is possible.
8. **`service.name` should be the container ID.** No; use a stable logical service name.
9. **More attributes are always better.** High cardinality and sensitive data create risk.
10. **Propagation equals exporting.** Propagation connects spans; exporting sends telemetry.
11. **OTel automatically provides dashboards.** A backend and visualization layer are still required.

---

## 17. Interview Questions

### L1

1. What is OpenTelemetry?
2. Which signals does it support?
3. Is OTel a backend?
4. What is instrumentation?
5. What is an exporter?
6. What is the Collector?
7. What is `service.name`?
8. What are semantic conventions?
9. What is automatic instrumentation?
10. What is context propagation?

### L2

1. Explain API versus SDK.
2. Why is OTel vendor-neutral?
3. Why use a Collector?
4. Compare direct export and Collector export.
5. Why is automatic instrumentation not enough?
6. Explain resource versus span attributes.
7. Why should every function not become a span?
8. What happens if propagation fails?
9. What is OTLP?
10. Can automatic and manual instrumentation be combined?

### L3

1. Design an OTel architecture for OT-Micro-Docker.
2. Explain the complete path of a span.
3. Troubleshoot a missing trace.
4. Explain how sampling affects observability.
5. Prevent salary or personal data from entering telemetry.
6. Explain what happens when the Collector is unavailable.
7. Instrument the salary-slip workflow end to end.
8. Explain how to monitor the Collector itself.

---

## 18. Quick Revision

```text
OpenTelemetry = Generate + collect + export telemetry
Instrumentation = Add/enable telemetry collection
API = Defines interfaces
SDK = Implements behavior
Provider = Supplies tracer/meter/logger implementation
Processor/Reader = Handles or collects telemetry
Sampler = Controls recording/sampling
Exporter = Sends telemetry
OTLP = OpenTelemetry Protocol
Collector = Receives + processes + exports
Resource = Describes the telemetry producer
service.name = Stable logical service identity
Semantic conventions = Standardized names and meanings
Propagation = Connects telemetry across boundaries
Backend = Stores, queries, visualizes, and alerts
```

### One-minute answer

> OpenTelemetry is a vendor-neutral framework for generating, collecting, and exporting traces, metrics, and logs. Instrumentation uses the OTel APIs, while the SDK implements recording, processing, sampling, and exporting. Telemetry can be sent directly to a backend or through OTLP to a Collector. The Collector receives, processes, batches, filters, routes, and exports telemetry. Resources identify the producer, semantic conventions standardize telemetry, and context propagation connects distributed operations.

---

## 19. Self-Assessment

1. Define OpenTelemetry.
2. Explain API versus SDK.
3. Is `trace.get_tracer()` a REST call?
4. Explain instrumentation.
5. Compare automatic and manual instrumentation.
6. What does a provider do?
7. What does a processor do?
8. What is a metric reader?
9. What is sampling?
10. What is OTLP?
11. What does the Collector do?
12. Explain resource versus span attributes.
13. Why is `service.name` important?
14. Explain semantic conventions.
15. Explain context propagation.
16. Why should every function not become a span?
17. How would you instrument salary generation and email delivery?
18. What telemetry data should not be recorded?
19. How would you troubleshoot a missing trace?
20. What is the difference between propagation and exporting?

### Mastery standard

You should be able to explain:

> Instrumentation uses the API to create telemetry. Providers and SDKs implement the behavior. Processors, readers, and samplers control handling. Exporters send telemetry to a Collector or backend. Resources identify the producer, semantic conventions standardize the data, and propagation connects distributed operations.

---

## References

- https://opentelemetry.io/docs/
- https://opentelemetry.io/docs/concepts/
- https://opentelemetry.io/docs/concepts/components/
- https://opentelemetry.io/docs/concepts/signals/
- https://opentelemetry.io/docs/concepts/context-propagation/
- https://opentelemetry.io/docs/specs/otlp/
- https://opentelemetry.io/docs/specs/semconv/

---

**Next part:** Part 03 — Distributed Tracing in Depth
