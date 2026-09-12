# Part 03 — Distributed Tracing in Depth

> **OpenTelemetry Book | OT-Micro-Docker**
>
> Learn how a request travels across services, how spans form a trace, how context is propagated, and how to investigate latency and failures.

---

## Table of Contents

- [1. Learning Objectives](#1-learning-objectives)
- [2. What Is Distributed Tracing?](#2-what-is-distributed-tracing)
- [3. Trace vs Span](#3-trace-vs-span)
- [4. Anatomy of a Span](#4-anatomy-of-a-span)
- [5. Trace and Span IDs](#5-trace-and-span-ids)
- [6. Parent-Child Relationships](#6-parent-child-relationships)
- [7. Span Kinds](#7-span-kinds)
- [8. Span Lifecycle](#8-span-lifecycle)
- [9. Span Attributes, Events, Status, and Links](#9-span-attributes-events-status-and-links)
- [10. Context Propagation](#10-context-propagation)
- [11. W3C Trace Context](#11-w3c-trace-context)
- [12. Synchronous vs Asynchronous Tracing](#12-synchronous-vs-asynchronous-tracing)
- [13. Sampling](#13-sampling)
- [14. Trace Structure in OT-Micro-Docker](#14-trace-structure-in-ot-micro-docker)
- [15. Investigating a Slow Request](#15-investigating-a-slow-request)
- [16. Broken Traces and Troubleshooting](#16-broken-traces-and-troubleshooting)
- [17. Good and Bad Span Design](#17-good-and-bad-span-design)
- [18. Security and Privacy](#18-security-and-privacy)
- [19. Interview Questions — L1](#19-interview-questions--l1)
- [20. Interview Questions — L2](#20-interview-questions--l2)
- [21. Interview Questions — L3](#21-interview-questions--l3)
- [22. Quick Revision](#22-quick-revision)
- [23. Self-Assessment](#23-self-assessment)

---

## 1. Learning Objectives

After completing this part, you should be able to:

- Define distributed tracing.
- Explain the relationship between traces and spans.
- Describe the anatomy of a span.
- Explain trace IDs, span IDs, and parent-child relationships.
- Differentiate server, client, producer, consumer, and internal spans.
- Explain span lifecycle and status.
- Use attributes, events, links, and exceptions correctly.
- Explain trace-context propagation.
- Describe the W3C Trace Context model.
- Explain synchronous and asynchronous tracing.
- Understand sampling and its trade-offs.
- Design useful traces for OT-Micro-Docker.
- Troubleshoot incomplete or disconnected traces.

---

## 2. What Is Distributed Tracing?

**Distributed tracing** records the path of a request as it moves through multiple services, processes, databases, queues, and external dependencies.

A trace helps answer:

- Which services handled the request?
- Where did the request spend time?
- Which dependency caused the delay?
- Where did the error occur?
- Which operations were executed in parallel?
- Which downstream service was called?
- Did the request cross an asynchronous boundary?

### Example

```text
User
  |
  v
Frontend
  |
  v
Employee API
  |
  v
ScyllaDB
```

A distributed trace may look like:

```text
Trace: Create employee

Frontend request       0 ms ───────────────────── 320 ms
  Employee API         15 ms ──────────────── 300 ms
    ScyllaDB insert    45 ms ─────── 110 ms
```

The trace shows the complete request journey instead of isolated logs from each component.

---

## 3. Trace vs Span

### 3.1 Trace

A **trace** represents the complete journey of one operation through a distributed system.

Example:

```text
Trace ID: abc123

Frontend
Employee API
ScyllaDB
```

A trace can contain many spans.

### 3.2 Span

A **span** represents one timed operation within a trace.

Examples:

- HTTP server request.
- HTTP client request.
- Database query.
- Cache lookup.
- PDF generation.
- Email send.
- Message publish.
- Message consume.

### 3.3 Relationship

```text
One trace
   |
   +-- Span: frontend request
   |
   +-- Span: employee API request
   |      |
   |      +-- Span: ScyllaDB insert
   |
   +-- Span: response processing
```

### Comparison

| Concept | Meaning |
|---|---|
| Trace | Complete request journey |
| Span | One operation in that journey |
| Trace ID | Identifies the complete trace |
| Span ID | Identifies one span |
| Parent span | Operation that initiated the current span |
| Child span | Operation initiated by another span |

### Important Rule

> A trace is a tree or graph of spans, not one large span.

---

## 4. Anatomy of a Span

A span commonly contains:

```text
Span
├── Trace ID
├── Span ID
├── Parent Span ID
├── Name
├── Start timestamp
├── End timestamp
├── Duration
├── Span kind
├── Attributes
├── Events
├── Status
├── Links
└── Resource information
```

### 4.1 Span Name

The name describes the operation.

Good:

```text
HTTP GET
employee.create
salary.generate
db.query
email.send
```

Bad:

```text
request-for-E102-at-06-30-15
```

Span names should be stable and low-cardinality.

### 4.2 Start and End Time

A span records when an operation started and ended.

```text
Start: 06:30:10.000
End:   06:30:10.250

Duration: 250 ms
```

### 4.3 Duration

Duration is one of the most useful trace properties.

It helps identify:

- slow database calls,
- slow external APIs,
- long queue waits,
- expensive PDF generation,
- blocked worker operations.

### 4.4 Resource

Resource data describes the service or process that produced the span.

Example:

```text
service.name = salary-api
deployment.environment.name = dev
```

---

## 5. Trace and Span IDs

### 5.1 Trace ID

The trace ID identifies the entire distributed operation.

Example:

```text
trace_id = 4bf92f3577b34da6a3ce929d0e0e4736
```

All spans belonging to the same trace share the same trace ID.

### 5.2 Span ID

The span ID identifies one operation within the trace.

Example:

```text
span_id = 00f067aa0ba902b7
```

Every span has its own span ID.

### 5.3 Parent Span ID

The parent span ID identifies the span that initiated the current operation.

Example:

```text
Parent span:
  employee.create

Child span:
  scylladb.insert
```

### 5.4 Example

```text
Trace ID: T1

Span A:
  Span ID: A
  Parent: none

Span B:
  Span ID: B
  Parent: A

Span C:
  Span ID: C
  Parent: B
```

This creates:

```text
A
└── B
    └── C
```

---

## 6. Parent-Child Relationships

### 6.1 Normal Request Flow

```text
Frontend request
    |
    v
Employee API request
    |
    v
ScyllaDB operation
```

Span relationship:

```text
Frontend span
└── Employee API span
    └── ScyllaDB span
```

### 6.2 Parallel Operations

Some services perform operations concurrently.

```text
Salary API
   |
   +-- ScyllaDB query
   |
   +-- Redis lookup
   |
   +-- External service
```

Trace:

```text
Salary API span
├── ScyllaDB span
├── Redis span
└── External service span
```

The child spans may overlap in time.

### 6.3 Why Parent-Child Relationships Matter

They show:

- execution hierarchy,
- service dependencies,
- operation ownership,
- parallel work,
- timing relationships.

### 6.4 Parent-Child Is Not Always a Strict Call Stack

Asynchronous systems may not behave like a simple nested function call.

For queues and background workers, links and propagated context may be more appropriate than a direct synchronous parent-child relationship.

---

## 7. Span Kinds

OpenTelemetry defines span kinds to describe the role of an operation.

### 7.1 Internal

An internal span represents an operation inside an application.

Examples:

```text
salary.generate
pdf.render
cache.calculate
```

### 7.2 Server

A server span represents handling an incoming request.

Example:

```text
Employee API receives HTTP request
```

```text
Client -> Employee API
          [SERVER span]
```

### 7.3 Client

A client span represents an outgoing request to another service or dependency.

Example:

```text
Employee API -> ScyllaDB
```

```text
Employee API
    [CLIENT span]
          |
          v
       ScyllaDB
```

### 7.4 Producer

A producer span represents sending a message to a messaging system.

Example:

```text
Notification producer -> Queue
```

### 7.5 Consumer

A consumer span represents receiving or processing a message.

Example:

```text
Queue -> Notification worker
```

### 7.6 Comparison

| Span kind | Represents |
|---|---|
| Internal | Work inside a process |
| Server | Incoming request handling |
| Client | Outgoing request/dependency call |
| Producer | Message publication |
| Consumer | Message consumption |

### Important Note

Span kind describes the role of the span. It does not mean that every database or internal operation must use the same kind in every implementation. Follow the instrumentation conventions of the relevant library and protocol.

---

## 8. Span Lifecycle

A span generally follows this lifecycle:

```text
Not started
    |
    v
Started
    |
    +-- Set attributes
    +-- Add events
    +-- Record exception
    +-- Set status
    |
    v
Ended
    |
    v
Processed/exported
```

### 8.1 Start

The span begins when the operation starts.

```python
span = tracer.start_span("salary.generate")
```

### 8.2 Active Context

A span may become the current active span.

This allows child operations to automatically inherit context.

### 8.3 Record Work

During the operation, record:

- attributes,
- events,
- exceptions,
- status changes.

### 8.4 End

The span ends when the operation is complete.

```python
span.end()
```

### 8.5 Always End Spans

A common implementation mistake is failing to end spans when exceptions occur.

Use structured context management where supported:

```python
with tracer.start_as_current_span("salary.generate"):
    generate_salary()
```

This helps ensure the span is ended when the block exits.

---

## 9. Span Attributes, Events, Status, and Links

### 9.1 Attributes

Attributes are key-value data describing the operation.

Example:

```text
http.request.method = GET
http.response.status_code = 200
db.system = postgresql
business.operation = salary.generate
```

Good attributes are:

- useful for filtering,
- stable,
- meaningful,
- low-cardinality,
- non-sensitive.

Bad attribute:

```text
employee_id = a_unique_value_for_every_request
```

A business identifier may be useful, but high-cardinality identifiers should be used carefully.

### 9.2 Events

An event is a timestamped occurrence inside a span.

Example:

```text
Span: email.send

Events:
- smtp.connection.started
- smtp.authentication.completed
- smtp.response.received
```

Events are useful for important moments that do not deserve separate spans.

### 9.3 Exceptions

Record exceptions when an operation fails.

Useful exception data includes:

- exception type,
- exception message,
- stack trace,
- timestamp,
- related span status.

Do not automatically expose secrets or sensitive request content in exception messages.

### 9.4 Status

A span status commonly indicates:

- unset,
- OK,
- ERROR.

Use error status when the operation failed according to its contract.

Do not mark every expected business outcome as an error.

Example:

```text
HTTP 404 for a valid search with no matching employee
```

Whether this is an error depends on the API design.

### 9.5 Links

A span link associates a span with another span or trace context without requiring a direct parent-child relationship.

Links are useful for:

- batch processing,
- asynchronous work,
- fan-in,
- fan-out,
- message processing,
- workflows involving multiple source contexts.

### Attributes vs Events vs Links

| Feature | Best use |
|---|---|
| Attributes | Describe the operation |
| Events | Record important moments |
| Exceptions | Record failures |
| Status | Indicate operation outcome |
| Links | Associate related contexts |

---

## 10. Context Propagation

**Context propagation** transfers trace context between operations and service boundaries.

Without propagation:

```text
Frontend trace A
Employee API trace B
ScyllaDB trace C
```

With propagation:

```text
One trace
├── Frontend
├── Employee API
└── ScyllaDB
```

### 10.1 Inject

The caller places context into an outgoing carrier.

For HTTP, the carrier is usually request headers.

```text
Application A
    |
    +-- inject trace context
    |
    v
HTTP request
```

### 10.2 Extract

The receiving service reads the context from the carrier.

```text
HTTP request
    |
    +-- extract trace context
    |
    v
Application B
```

### 10.3 Propagation Carriers

Examples:

- HTTP headers.
- Message metadata.
- RPC metadata.
- Custom transport metadata.

### 10.4 Common Propagation Problems

- Middleware not installed.
- Headers removed by a proxy.
- Context not injected into messages.
- Context extracted too late.
- Async task loses current context.
- Multiple incompatible propagation formats.
- Sampling or exporter configuration hides spans.

---

## 11. W3C Trace Context

The W3C Trace Context specification defines a standard way to carry trace context across services.

The most common header is:

```http
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
```

Conceptually:

```text
version
trace-id
parent-id
trace-flags
```

### 11.1 Why Standardization Matters

If every vendor uses a different header format, interoperability becomes difficult.

A standard format allows:

```text
Service A -> Service B -> Service C
```

to preserve trace context even when the services use different frameworks or vendors.

### 11.2 Trace State

Additional vendor-specific information may be carried through a trace-state mechanism.

Do not assume that every trace-state value is present or required.

### 11.3 Security Considerations

Trace context is metadata, not proof of identity.

Do not use a trace ID as:

- an authentication token,
- an authorization credential,
- a secret,
- a replacement for request validation.

---

## 12. Synchronous vs Asynchronous Tracing

### 12.1 Synchronous Flow

A synchronous request waits for the downstream operation.

```text
Frontend
   |
   v
Salary API
   |
   v
ScyllaDB
   |
   v
Response
```

The parent span remains active while the child operation runs.

### 12.2 Asynchronous Flow

An asynchronous workflow may return before the downstream operation finishes.

```text
Salary API
   |
   v
Publish notification message
   |
   v
Return response

Later:

Queue
   |
   v
Notification worker
   |
   v
SMTP
```

The trace may cross a time boundary.

### 12.3 Producer and Consumer Context

```text
Producer span
    |
    +-- inject context into message
    |
    v
Queue
    |
    v
Consumer span
    |
    +-- extract context
    |
    v
Email send span
```

### 12.4 Why This Is Difficult

The consumer may run:

- on another process,
- on another host,
- much later,
- in parallel with other consumers,
- after the producer request has ended.

Use propagated context and, where appropriate, span links.

---

## 13. Sampling

**Sampling** determines which telemetry is recorded or exported.

Sampling is useful because exporting every span may be expensive.

### 13.1 Why Sample?

Reasons include:

- storage cost,
- network usage,
- backend capacity,
- application overhead,
- high traffic volume.

### 13.2 Head Sampling

Head sampling decides early, often when a trace starts.

Example:

```text
Keep 10% of traces
Drop 90% of traces
```

Advantages:

- simple,
- low overhead,
- predictable volume.

Limitations:

- a trace may be dropped before its later error is known,
- rare failures may be missed.

### 13.3 Tail Sampling

Tail sampling decides after more of the trace is available.

It can retain traces based on conditions such as:

- error status,
- high latency,
- specific service,
- important route,
- business priority.

Example policy:

```text
Keep:
- all errors,
- all traces above 2 seconds,
- 10% of normal traces.
```

Tail sampling generally requires a Collector architecture capable of making decisions after collecting the relevant trace data.

### 13.4 Sampling Trade-Off

| Strategy | Advantage | Limitation |
|---|---|---|
| Head sampling | Simple and inexpensive | May discard important traces early |
| Tail sampling | Can preserve errors and slow traces | More complex and resource intensive |

### 13.5 Sampling Warning

A missing trace does not always mean the request never happened.

It may have been:

- sampled out,
- dropped by an exporter,
- rejected by the Collector,
- lost due to configuration,
- unavailable due to backend retention.

---

## 14. Trace Structure in OT-Micro-Docker

Consider a salary-slip workflow.

```text
Frontend
   |
   v
Salary API
   |
   +-- ScyllaDB
   |
   +-- Elasticsearch
   |
   v
Notification processing
   |
   +-- PDF generation
   |
   +-- SMTP
```

### 14.1 Possible Trace

```text
Trace: salary-slip workflow

frontend.request
└── salary.request
    ├── scylla.salary.read
    ├── elasticsearch.salary.index
    ├── notification.publish
    └── notification.process
        ├── pdf.generate
        └── email.send
```

### 14.2 Important Qualification

If notification processing is asynchronous, it may not appear as a normal nested child span of the original HTTP request.

The actual relationship depends on:

- message context propagation,
- producer/consumer instrumentation,
- whether the worker starts a new trace,
- whether links are used,
- backend rendering behavior.

### 14.3 Suggested Span Names

```text
employee.create
attendance.mark
salary.generate
salary.persist
salary.index
notification.process
pdf.generate
email.send
```

Avoid embedding dynamic values in span names.

Bad:

```text
salary.generate.employee.E102
```

Good:

```text
salary.generate
```

### 14.4 Useful Attributes

```text
service.name = salary-api
business.operation = salary.generate
db.system = scylladb
messaging.operation.type = publish
notification.channel = email
```

Do not record:

- salary amount,
- full employee profile,
- email body,
- SMTP password,
- access tokens,
- full request bodies,

unless there is a clear, protected, and reviewed requirement.

---

## 15. Investigating a Slow Request

### Scenario

The user reports:

```text
Salary request takes 8 seconds.
```

### Step 1: Find the Trace

Search using:

- trace ID,
- request ID,
- route,
- time range,
- service name,
- user-visible error.

### Step 2: Inspect the Root Span

```text
salary.request = 8.1 seconds
```

### Step 3: Inspect Child Spans

```text
ScyllaDB read       100 ms
PDF generation       90 ms
Notification call   7.8 s
```

The slow operation is likely notification-related.

### Step 4: Inspect Downstream Trace

```text
notification.process = 7.7 seconds
email.send           = 7.5 seconds
```

### Step 5: Inspect Logs

```text
ERROR SMTP connection timeout
retry_count=3
```

### Step 6: Inspect Metrics

```text
notification queue depth = 98%
SMTP latency p95         = 6.8 seconds
```

### Step 7: Form a Hypothesis

```text
SMTP latency increased
    |
    v
Notification workers remained busy
    |
    v
Queue backlog increased
    |
    v
Salary workflow became slow
```

### Step 8: Verify

Check whether:

- SMTP latency returned to normal.
- Queue depth decreased.
- Error rate reduced.
- New traces completed within the expected SLO.

---

## 16. Broken Traces and Troubleshooting

### Symptom 1: Every service has a separate trace

Possible causes:

- context propagation disabled,
- middleware missing,
- headers removed,
- downstream client not instrumented.

Check:

- incoming trace headers,
- outgoing trace headers,
- instrumentation setup,
- proxy behavior.

### Symptom 2: Trace starts at the backend, not the frontend

Possible causes:

- frontend is not instrumented,
- browser telemetry is disabled,
- frontend cannot reach the Collector,
- CORS or network configuration blocks export.

### Symptom 3: Database spans are missing

Possible causes:

- unsupported database driver,
- no database instrumentation,
- instrumentation initialized too late,
- queries executed through an uninstrumented wrapper.

### Symptom 4: Child span has no parent

Possible causes:

- active context lost,
- asynchronous task not attached to context,
- manual span created without a parent,
- context extraction failed.

### Symptom 5: Traces appear only sometimes

Possible causes:

- sampling,
- intermittent exporter failures,
- Collector overload,
- backend retention,
- batch processor delay,
- application shutdown before export.

### Symptom 6: Trace duration is shorter than user-perceived duration

Possible causes:

- frontend time is not included,
- queue wait is not instrumented,
- proxy time is missing,
- client-side rendering is excluded,
- spans end before the actual operation completes.

### Troubleshooting Checklist

```text
[ ] Is instrumentation initialized before requests arrive?
[ ] Is service.name configured?
[ ] Is context propagation enabled?
[ ] Are outgoing headers present?
[ ] Are database/client libraries instrumented?
[ ] Are spans ended correctly?
[ ] Is sampling configured?
[ ] Is the exporter working?
[ ] Is the Collector receiving spans?
[ ] Is the backend storing and displaying spans?
[ ] Are clocks reasonably synchronized?
```

---

## 17. Good and Bad Span Design

### Good Span Design

Use spans for:

- meaningful external calls,
- important database operations,
- business workflows,
- expensive internal operations,
- queue publish/consume,
- retryable operations.

Example:

```text
salary.generate
    |
    +-- scylla.read
    +-- pdf.generate
    +-- email.send
```

### Bad Span Design

Avoid:

- a span for every trivial function,
- dynamic IDs in span names,
- full request bodies,
- passwords and tokens,
- duplicate spans for the same operation,
- spans that remain open for unrelated work,
- vague names such as `process`.

### Span Naming Guidelines

| Poor name | Better name |
|---|---|
| `process` | `salary.generate` |
| `do_work` | `pdf.generate` |
| `employee-E102-create` | `employee.create` |
| `call_database` | `db.query` |
| `send_something` | `email.send` |

### Span Boundaries

A span should represent a coherent operation.

Bad:

```text
One span starts before the request
and ends after unrelated background work.
```

Good:

```text
One span for the request.
One span for the database call.
One span for the background job.
```

---

## 18. Security and Privacy

Tracing can expose sensitive information if implemented carelessly.

### Never Record by Default

- passwords,
- access tokens,
- API keys,
- session cookies,
- complete email bodies,
- full salary data,
- full employee profiles,
- authorization headers,
- private database credentials.

### Safer Alternatives

Instead of:

```text
employee.email = mukesh@example.com
```

Prefer:

```text
employee.type = employee
employee.operation = salary.generate
```

If an identifier is required:

- use a controlled internal identifier,
- restrict access,
- document its purpose,
- apply retention rules,
- avoid placing it in span names.

### Security Principle

> Telemetry is production data and must be protected like production data.

---

## 19. Interview Questions — L1

### Q1. What is distributed tracing?

It records the path of a request across multiple services and dependencies.

### Q2. What is a trace?

A complete distributed request journey.

### Q3. What is a span?

A timed operation within a trace.

### Q4. What is a trace ID?

An identifier shared by all spans in one trace.

### Q5. What is a span ID?

An identifier for one span.

### Q6. What is a parent span?

The span that initiated the current operation.

### Q7. Name the five span kinds.

- Internal
- Server
- Client
- Producer
- Consumer

### Q8. What is context propagation?

The transfer of trace context across process or service boundaries.

### Q9. What is sampling?

The process of deciding which telemetry is recorded or exported.

### Q10. Why are traces useful?

They show request flow, timing, dependencies, and failure location.

---

## 20. Interview Questions — L2

### Q1. Explain trace vs span.

A trace represents the full request journey. A span represents one operation within that journey.

### Q2. Why is context propagation important?

Without it, each service may create disconnected traces, making distributed investigation difficult.

### Q3. What is the difference between server and client spans?

A server span represents incoming request handling. A client span represents an outgoing call to another service or dependency.

### Q4. What is the difference between attributes and events?

Attributes describe an operation. Events record timestamped moments during the operation.

### Q5. When are span links useful?

They are useful for asynchronous workflows, batch processing, fan-in, and fan-out where a direct parent-child relationship is insufficient.

### Q6. Why can sampling hide errors?

Head sampling may decide to drop a trace before a later error occurs.

### Q7. Why should span names have low cardinality?

High-cardinality span names make grouping, querying, storage, and visualization more expensive and noisy.

### Q8. Why might a trace be missing even though the request succeeded?

It may have been sampled out, dropped during export, rejected by the Collector, or expired from backend retention.

### Q9. How do you trace a queue-based workflow?

Inject context into the message at publish time, extract it at consumption time, and create producer/consumer spans as appropriate.

### Q10. Why should sensitive data not be placed in spans?

Telemetry is stored and accessed by operational systems. Sensitive data increases privacy, security, and compliance risk.

---

## 21. Interview Questions — L3

### Q1. Design a trace for salary-slip generation.

A strong design may include:

```text
salary.request
├── salary.read
├── salary.persist
├── salary.index
└── notification.publish
```

The worker may create:

```text
notification.process
├── pdf.generate
└── email.send
```

Use context propagation across the message boundary and avoid sensitive attributes.

### Q2. A trace shows an 8-second request, but all database spans are fast. What do you investigate?

Investigate:

- external services,
- queue wait time,
- SMTP calls,
- retries,
- connection pools,
- lock contention,
- uninstrumented internal work,
- frontend or proxy delay,
- missing child spans.

### Q3. Why can a trace be represented as a graph instead of a simple tree?

Because asynchronous operations, links, batching, and fan-in/fan-out workflows may associate operations without a strict single parent-child hierarchy.

### Q4. How would you preserve trace context across a queue?

1. Extract the current context in the producer.
2. Inject it into message metadata.
3. Publish the message.
4. Extract context in the consumer.
5. Create the consumer or processing span.
6. Use links where multiple source contexts are involved.

### Q5. What is the risk of creating a span for every function?

It creates excessive telemetry, increases overhead, produces noisy traces, and makes meaningful operations harder to identify.

### Q6. How do you investigate disconnected traces?

Check:

- propagation headers,
- middleware,
- client instrumentation,
- proxy behavior,
- async context,
- message metadata,
- sampler decisions,
- Collector ingestion.

### Q7. Why is tail sampling useful in production?

It can retain important traces based on completed behavior, such as errors or high latency, while reducing normal traffic volume.

### Q8. What is the difference between a trace ID and a request ID?

A trace ID identifies distributed tracing context. A request ID is an application-level correlation identifier. They may be used together but are not automatically identical.

### Q9. Why should telemetry clocks be reasonably synchronized?

Trace timelines depend on timestamps. Large clock differences can make spans appear out of order or create confusing duration relationships.

### Q10. How would you prevent trace instrumentation from affecting application availability?

Use bounded queues, batching, timeouts, appropriate sampling, non-blocking export where supported, monitored Collector capacity, and safe failure behavior when telemetry export is unavailable.

---

## 22. Quick Revision

```text
Trace
    = Complete distributed operation

Span
    = One timed operation

Trace ID
    = Identifies the trace

Span ID
    = Identifies one span

Parent
    = Operation that initiated the current span

Attributes
    = Describe the operation

Events
    = Timestamped moments

Status
    = Operation outcome

Links
    = Associate related contexts

Propagation
    = Move context across boundaries

Sampling
    = Decide which telemetry is retained
```

### Span Kinds

```text
INTERNAL  -> Work inside a process
SERVER    -> Incoming request
CLIENT    -> Outgoing request
PRODUCER  -> Message publication
CONSUMER  -> Message consumption
```

### Investigation Pattern

```text
Find trace
    |
    v
Inspect root span
    |
    v
Find slow/error child span
    |
    v
Inspect correlated logs
    |
    v
Check metrics and saturation
    |
    v
Verify the hypothesis
```

### One-Minute Interview Answer

> Distributed tracing records the journey of a request across services and dependencies. A trace contains spans, and each span represents a timed operation with a name, IDs, timestamps, attributes, events, status, and resource information. Parent-child relationships describe normal execution flow, while links help with asynchronous and batch workflows. Context propagation transfers trace context through HTTP headers, RPC metadata, or message metadata. Sampling controls telemetry volume, and good span design uses meaningful low-cardinality names while avoiding sensitive data.

---

## 23. Self-Assessment

Answer these without looking back:

1. Define distributed tracing.
2. Differentiate a trace and a span.
3. What is a trace ID?
4. What is a span ID?
5. Explain parent-child relationships.
6. Name and explain the five span kinds.
7. What is the difference between attributes and events?
8. What are span links?
9. Explain context injection and extraction.
10. What is the W3C `traceparent` header?
11. Explain synchronous and asynchronous tracing.
12. What is head sampling?
13. What is tail sampling?
14. Why might traces become disconnected?
15. Design a trace for the OT-Micro-Docker salary workflow.
16. What data should never be recorded in a salary-related span?
17. How would you investigate an 8-second request?
18. Why should span names have low cardinality?

### Mastery Standard

You are ready for Part 4 when you can explain:

> “A distributed trace is made of spans. Each span represents one operation and contains IDs, timing, attributes, events, and status. Context propagation connects spans across services, span kinds describe operation roles, sampling controls volume, and meaningful span design helps locate latency and failures without exposing sensitive data.”

---

## References

- [OpenTelemetry Traces](https://opentelemetry.io/docs/concepts/signals/traces/)
- [OpenTelemetry Context Propagation](https://opentelemetry.io/docs/concepts/context-propagation/)
- [OpenTelemetry Trace API](https://opentelemetry.io/docs/specs/otel/trace/api/)
- [OpenTelemetry Trace SDK](https://opentelemetry.io/docs/specs/otel/trace/sdk/)
- [OpenTelemetry Semantic Conventions](https://opentelemetry.io/docs/specs/semconv/)
- [W3C Trace Context](https://www.w3.org/TR/trace-context/)

---

**Next part:** Part 04 — Metrics and Logs
