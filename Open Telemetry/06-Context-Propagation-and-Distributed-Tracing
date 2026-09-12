# Part 07 — Context Propagation and Distributed Tracing Across Services

> **OT-Micro-Docker Observability Book**  
> A practical, interview-oriented guide to carrying trace context across browsers, proxies, APIs, databases, caches, and asynchronous workers.

---

## Table of Contents

1. Learning Objectives
2. Why Context Propagation Matters
3. The Core Problem: One Request, Many Processes
4. What Is Trace Context?
5. Trace ID, Span ID, Flags, and State
6. The W3C Trace Context Standard
7. `traceparent` Header Anatomy
8. `tracestate`
9. Propagation Is Not Exporting
10. Inject and Extract
11. Context Carriers
12. In-Process Context Propagation
13. HTTP Server and Client Propagation
14. Frontend → NGINX → Backend Flow
15. NGINX and Reverse-Proxy Considerations
16. Python Context Propagation
17. Go Context Propagation
18. Java Context Propagation
19. Browser Context Propagation
20. Database, Redis, and Elasticsearch Spans
21. Asynchronous Processing and Background Workers
22. Message Queues: Parent-Child vs Span Links
23. OT-Micro-Docker End-to-End Examples
24. Broken Trace Patterns
25. Troubleshooting Methodology
26. Sampling and Propagation
27. Security and Privacy
28. Testing Context Propagation
29. Common Mistakes
30. L1 Interview Questions
31. L2 Interview Questions
32. L3 Interview Questions
33. Quick Revision
34. Self-Assessment
35. Final Implementation Checklist

---

# 1. Learning Objectives

After completing this part, you should be able to:

- Explain why distributed tracing requires context propagation.
- Distinguish a trace ID from a span ID.
- Explain the purpose of `traceparent` and `tracestate`.
- Describe the difference between injecting and extracting context.
- Explain how context travels through HTTP headers.
- Trace a request from the React frontend through NGINX and backend services.
- Explain how Python, Go, and Java maintain active trace context.
- Understand why asynchronous workers require special handling.
- Decide when to use parent-child relationships and when to use span links.
- Troubleshoot broken traces across microservices.
- Explain why propagation and exporting are separate concerns.
- Discuss propagation security, privacy, and sampling behavior.

---

# 2. Why Context Propagation Matters

A distributed trace is useful only when spans from different processes can be connected.

Consider this request:

```text
Browser
  |
  v
NGINX
  |
  v
Salary API
  |
  v
ScyllaDB
  |
  v
Notification API
  |
  v
Email provider
```

Each component may create its own span.

Without context propagation:

```text
Trace A: Browser → NGINX

Trace B: Salary API → ScyllaDB

Trace C: Notification API → Email provider
```

The backend systems may all be working, but the tracing backend cannot prove that these operations belong to the same user request.

With propagation:

```text
Trace 123
  ├── Browser request
  ├── NGINX proxy span
  ├── Salary API server span
  ├── ScyllaDB client span
  ├── Notification API span
  └── Email operation span
```

The trace ID connects the distributed operations.

## Key idea

> **Context propagation is the mechanism that carries trace identity from one execution boundary to another.**

Execution boundaries include:

- Browser to reverse proxy.
- Reverse proxy to backend API.
- One API to another API.
- API to database client.
- API to Redis.
- Producer to worker.
- Parent thread to child thread.
- Synchronous code to asynchronous code.
- One process to another process.
- One host to another host.

---

# 3. The Core Problem: One Request, Many Processes

A process has its own memory.

For example:

```text
React browser process
    trace context exists here

NGINX process
    trace context arrives through HTTP headers

Salary API process
    trace context is extracted from headers

ScyllaDB client
    trace context is attached to the database span

Notification worker process
    context must be explicitly propagated through the message
```

A trace context is not automatically shared merely because services communicate.

The receiving service must:

1. Receive the context.
2. Extract it.
3. Make it active.
4. Create a child span or linked span.
5. Propagate it further if another service is called.

## Without extraction

The receiving service starts a new root trace:

```text
Trace A
  └── Frontend request

Trace B
  └── Salary API request
```

## With extraction

The receiving service continues the existing trace:

```text
Trace A
  ├── Frontend request
  └── Salary API request
```

---

# 4. What Is Trace Context?

Trace context is the small amount of information required to associate an operation with an existing distributed trace.

The most important fields are:

| Field | Purpose |
|---|---|
| Trace ID | Identifies the complete distributed trace |
| Span ID | Identifies the current operation/span |
| Trace flags | Carries control flags such as sampling decision |
| Trace state | Carries vendor-specific tracing information |

A simplified model:

```text
Trace Context
├── trace_id
├── span_id
├── trace_flags
└── trace_state
```

The context is not the complete span.

It does not normally contain:

- Span duration.
- Span attributes.
- Span events.
- Exception details.
- Resource attributes.
- Complete request body.
- Complete response body.

Those belong to telemetry data, not propagation context.

---

# 5. Trace ID, Span ID, Flags, and State

## 5.1 Trace ID

The trace ID identifies the entire distributed operation.

Example:

```text
4bf92f3577b34da6a3ce929d0e0e4736
```

All spans belonging to the same trace share this trace ID.

```text
Trace ID: 4bf92f3577b34da6a3ce929d0e0e4736

Span 1 → same trace ID
Span 2 → same trace ID
Span 3 → same trace ID
```

A trace ID is not:

- A user ID.
- An employee ID.
- A session ID.
- An order ID.
- A database primary key.

Do not use business identifiers as trace IDs.

## 5.2 Span ID

A span ID identifies one operation inside the trace.

Example:

```text
Span A: 00f067aa0ba902b7
Span B: 00f067aa0ba902b8
Span C: 00f067aa0ba902b9
```

Every span has its own span ID.

## 5.3 Parent Span ID

A child span records the span ID of its logical parent.

Example:

```text
Parent span:
  span_id = 1111111111111111

Child span:
  span_id = 2222222222222222
  parent_span_id = 1111111111111111
```

The parent-child relationship is normally used for nested synchronous work.

## 5.4 Trace Flags

Trace flags are bit flags associated with the trace context.

The best-known flag is the sampling flag.

A sampled trace context commonly uses:

```text
01
```

An unsampled context commonly uses:

```text
00
```

Important:

> A sampled flag is a decision signal, not a guarantee that every backend will store every span.

The Collector and backend may still apply additional policies.

## 5.5 Trace State

`tracestate` carries vendor-specific tracing information.

For example, a tracing vendor may need to carry:

- Sampling metadata.
- Routing information.
- Vendor-specific trace state.
- Processing hints.

Application developers generally should not manually invent or modify `tracestate` values unless they understand the relevant propagation specification.

---

# 6. The W3C Trace Context Standard

OpenTelemetry commonly uses the W3C Trace Context propagation format.

The two important HTTP headers are:

```http
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
tracestate: vendor=value
```

The W3C format allows independent systems and vendors to exchange trace context in a standard way.

This prevents every vendor from inventing a different header format.

## Why standardization matters

Without a standard:

```text
Vendor A → x-vendor-a-trace
Vendor B → x-vendor-b-context
Vendor C → x-vendor-c-id
```

With a standard:

```text
Any compatible service → traceparent/tracestate → any compatible service
```

---

# 7. `traceparent` Header Anatomy

A typical `traceparent` header looks like:

```http
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
```

Conceptually:

```text
00
│
├── Version
│
4bf92f3577b34da6a3ce929d0e0e4736
│
├── Trace ID
│
00f067aa0ba902b7
│
├── Parent Span ID
│
01
└── Trace Flags
```

The format is:

```text
version-trace-id-parent-id-trace-flags
```

## Important interpretation

When a service receives a request:

```text
Incoming traceparent contains:
  trace_id = T1
  parent_span_id = S1
```

The receiving service should normally create a new server span:

```text
New server span:
  trace_id = T1
  span_id = S2
  parent_span_id = S1
```

The receiver does not reuse the incoming span ID as its own span ID.

That would incorrectly identify two operations as one span.

## Example

```text
Frontend span:
  trace_id = T1
  span_id  = F1

HTTP request carries:
  traceparent = ...-T1-F1-01

Salary API creates:
  trace_id = T1
  span_id  = S1
  parent    = F1
```

---

# 8. `tracestate`

`tracestate` is optional and is used for vendor-specific information.

Example:

```http
tracestate: vendor1=value1,vendor2=value2
```

The exact contents depend on the tracing ecosystem.

## Rules of thumb

- Forward it when propagating compatible context.
- Do not treat it as application business data.
- Do not log it blindly if it may contain sensitive vendor metadata.
- Do not overwrite it casually.
- Let the OpenTelemetry propagator manage it whenever possible.

---

# 9. Propagation Is Not Exporting

This distinction is essential for interviews.

## Propagation

Propagation carries context between operations.

```text
Service A
  |
  | traceparent header
  v
Service B
```

## Exporting

Exporting sends telemetry data to a Collector or backend.

```text
Service A
  |
  | OTLP
  v
OpenTelemetry Collector
  |
  v
Tracing backend
```

## Complete flow

```text
1. Service A creates span.
2. Service A injects context into outgoing request.
3. Service B extracts context.
4. Service B creates child span.
5. Service B exports its span through OTLP.
```

Propagation answers:

> Which trace does this operation belong to?

Exporting answers:

> Where should the telemetry data be sent?

A service can propagate context even when exporting is temporarily disabled.

A service can export spans even when propagation is broken.

---

# 10. Inject and Extract

## 10.1 Inject

Inject means writing trace context into a carrier.

Example carrier:

```text
HTTP request headers
```

Conceptually:

```text
Current active span
      |
      v
Inject context
      |
      v
HTTP headers
```

Example:

```http
traceparent: 00-TRACE_ID-SPAN_ID-01
```

## 10.2 Extract

Extract means reading trace context from a carrier.

```text
HTTP headers
      |
      v
Extract context
      |
      v
Create child span
```

## 10.3 The complete pattern

```text
Outgoing service:
  current context → inject → HTTP headers

Incoming service:
  HTTP headers → extract → current context → new span
```

## 10.4 Automatic instrumentation

Automatic instrumentation often performs injection and extraction for supported frameworks.

Examples:

- Flask server instrumentation.
- Spring Boot HTTP instrumentation.
- Go HTTP client instrumentation.
- Browser fetch instrumentation.
- gRPC instrumentation.

However, automatic instrumentation is not universal.

Custom protocols, custom queues, and unusual background workers may require manual propagation.

---

# 11. Context Carriers

A carrier is the medium used to transport context.

| Boundary | Typical carrier |
|---|---|
| HTTP | Request headers |
| gRPC | Metadata |
| Kafka-like messaging | Message headers |
| RabbitMQ-like messaging | Message properties/headers |
| In-process async | Runtime context object |
| Task queue | Task metadata |
| Custom TCP protocol | Application-defined metadata |
| Batch job | Explicit serialized context |

## HTTP carrier

```http
traceparent: ...
tracestate: ...
```

## Message carrier

Conceptually:

```json
{
  "headers": {
    "traceparent": "00-...",
    "tracestate": "..."
  },
  "payload": {
    "employee_id": 101
  }
}
```

Do not place trace context inside the business payload unless the protocol requires it.

Prefer message headers or metadata.

---

# 12. In-Process Context Propagation

Not all propagation crosses a network.

Context may be lost inside one application.

Example:

```text
HTTP request handler
    |
    ├── database operation
    ├── Redis operation
    └── background task
```

If the application starts a new thread or asynchronous task incorrectly, the active context may not be available.

## Common context mechanisms

| Language | Common mechanism |
|---|---|
| Python | `contextvars` and framework instrumentation |
| Go | `context.Context` |
| Java | Thread-local/context mechanisms used by instrumentation |
| JavaScript | Async context mechanisms such as AsyncLocalStorage in Node.js |
| Browser | Instrumentation-managed context |

## Important principle

> Context is normally scoped to an execution flow, not globally shared across every operation.

Avoid storing the current trace context in a global variable.

A global variable can cause:

- Cross-request contamination.
- Incorrect parent spans.
- Race conditions.
- Mixed trace IDs.
- Broken concurrency behavior.

---

# 13. HTTP Server and Client Propagation

## 13.1 Incoming HTTP request

A backend receives:

```http
GET /salary/101 HTTP/1.1
Host: salary-api
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
```

The server instrumentation:

1. Reads the header.
2. Validates the context.
3. Extracts trace ID and parent span ID.
4. Creates a server span.
5. Makes that span active during request handling.

## 13.2 Outgoing HTTP request

The Salary API calls Notification API.

The client instrumentation:

1. Reads the current active context.
2. Creates a client span.
3. Injects context into the outgoing request.
4. Sends the request.

The Notification API:

1. Extracts the header.
2. Creates a server span.
3. Continues the trace.

## 13.3 Conceptual tree

```text
GET /salary/101
└── Salary API server
    ├── ScyllaDB query
    └── POST /notifications
        └── Notification API server
            └── SMTP operation
```

---

# 14. Frontend → NGINX → Backend Flow

For OT-Micro-Docker, a realistic request path is:

```text
React Browser
    |
    | HTTP request
    v
NGINX
    |
    | proxied HTTP request
    v
Salary API
    |
    v
ScyllaDB
```

## 14.1 Browser creates or continues context

The browser instrumentation may create a client span for:

```text
GET /api/salary/101
```

It injects:

```http
traceparent: 00-TRACE_ID-BROWSER_SPAN-01
```

## 14.2 NGINX forwards headers

NGINX acts as a reverse proxy.

It must not accidentally remove or replace tracing headers.

Conceptually:

```text
Browser request
  ├── traceparent
  └── tracestate

NGINX proxy request
  ├── traceparent
  └── tracestate
```

Whether NGINX itself creates a span depends on the instrumentation architecture.

## 14.3 Salary API extracts context

The Salary API receives the forwarded headers and creates its server span.

```text
Browser span
    |
    v
Salary API server span
```

## 14.4 Browser CORS considerations

If the browser sends tracing headers across origins, the server/proxy configuration must permit the required headers.

Relevant concepts include:

- Allowed request headers.
- Preflight requests.
- Cross-origin policy.
- Reverse proxy header forwarding.

Do not assume that a browser can send arbitrary headers across origins without server configuration.

---

# 15. NGINX and Reverse-Proxy Considerations

A reverse proxy can break trace continuity if it:

- Drops `traceparent`.
- Drops `tracestate`.
- Rewrites headers incorrectly.
- Creates a new trace ID without a clear reason.
- Sends requests to services that do not support the selected propagation format.
- Routes through multiple proxies that handle context inconsistently.

## Diagnostic checks

Inspect the incoming request:

```bash
curl -v http://frontend.example/api/salary/101
```

Inspect backend logs or temporary request-header logging.

Check whether the backend receives:

```text
traceparent
tracestate
```

## Do not log sensitive headers permanently

Tracing headers are generally identifiers and metadata, but they should still be handled carefully.

Use temporary diagnostic logging and remove verbose logging after troubleshooting.

---

# 16. Python Context Propagation

Python applications may use:

- Flask.
- FastAPI.
- Requests.
- HTTPX.
- AsyncIO.
- Celery-like workers.
- Custom background threads.

## 16.1 Flask server

With supported OpenTelemetry Flask instrumentation, the incoming request context is commonly extracted automatically.

Conceptual flow:

```text
HTTP request
  |
  v
Flask instrumentation
  |
  v
Extract context
  |
  v
Create server span
  |
  v
Flask route
```

## 16.2 Python HTTP client

When supported client instrumentation is enabled:

```text
Current active span
  |
  v
Requests/HTTPX instrumentation
  |
  v
Inject traceparent
  |
  v
Outgoing request
```

## 16.3 Manual context usage

Conceptually:

```python
from opentelemetry import context, trace

tracer = trace.get_tracer(__name__)

with tracer.start_as_current_span("business-operation"):
    # Work performed here inherits the active context.
    perform_work()
```

The exact APIs may vary by instrumentation and SDK version, but the principle is stable:

> Start the span as current when child operations should inherit it.

## 16.4 Python async work

Async code can lose context when tasks are created or scheduled incorrectly.

Be careful with:

- Custom thread pools.
- Detached threads.
- Callback-based libraries.
- Long-lived worker loops.
- Manually stored global context.

Use framework-supported context handling whenever possible.

## 16.5 Python background worker

If a worker receives a message containing trace context:

```text
Message headers
  |
  v
Extract context
  |
  v
Start worker span
```

Do not assume the worker automatically inherits the producer's context merely because both use Python.

---

# 17. Go Context Propagation

Go uses `context.Context` as a central mechanism for request-scoped values, deadlines, cancellation, and tracing context.

## 17.1 Pass context explicitly

Conceptually:

```go
func HandleRequest(ctx context.Context) error {
    return performDatabaseCall(ctx)
}
```

Avoid silently replacing the context with:

```go
context.Background()
```

inside a request path.

That can discard:

- Trace context.
- Cancellation.
- Deadlines.
- Request-scoped values.

## 17.2 HTTP server

A Go HTTP handler commonly receives a request context:

```go
func handler(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()
    process(ctx)
}
```

The instrumentation can place the extracted tracing context into `r.Context()`.

## 17.3 HTTP client

The outgoing request should use the active context:

```go
req = req.WithContext(ctx)
```

If the context is not attached to the request, the client instrumentation may not know which span should be the parent.

## 17.4 Gin applications

For a Gin service:

```text
Incoming request
    |
    v
OpenTelemetry Gin middleware
    |
    v
Request context
    |
    v
Handler
    |
    v
HTTP client / database calls
```

The exact middleware and exporter setup depends on the selected Go libraries.

## 17.5 Common Go mistake

Bad conceptual pattern:

```go
func handler(...) {
    ctx := context.Background()
    callAnotherService(ctx)
}
```

Better:

```go
func handler(...) {
    ctx := request.Context()
    callAnotherService(ctx)
}
```

---

# 18. Java Context Propagation

Java instrumentation often relies on framework and runtime context mechanisms.

Relevant environments include:

- Spring Boot.
- Servlet containers.
- WebFlux.
- Executor services.
- CompletableFuture.
- Scheduled tasks.
- Messaging clients.

## 18.1 Spring Boot HTTP flow

```text
Incoming HTTP request
  |
  v
Servlet/Web framework instrumentation
  |
  v
Active server span
  |
  v
Controller
  |
  v
Service layer
  |
  v
Database/HTTP client span
```

## 18.2 Thread boundaries

A common problem is moving work to another thread:

```text
HTTP thread
    |
    v
Executor thread
```

The active context may not automatically transfer unless the framework or executor is instrumented correctly.

## 18.3 CompletableFuture

Asynchronous chains can break trace parenting if context is not preserved across stages.

Check:

- Whether the executor is instrumented.
- Whether the async library is supported.
- Whether custom thread pools bypass instrumentation.
- Whether context is manually detached too early.

## 18.4 Java agent advantage

The Java agent can instrument many common frameworks without requiring extensive application-code changes.

However:

> Automatic instrumentation coverage is not the same as universal instrumentation coverage.

Custom executors, proprietary clients, and unusual async frameworks may still need manual work.

---

# 19. Browser Context Propagation

Browser tracing introduces additional concerns.

## 19.1 Browser spans

A browser instrumentation library may create spans for:

- Page loads.
- Fetch requests.
- XMLHttpRequest calls.
- User interactions, depending on configuration.
- Document loading resources.

## 19.2 Cross-origin requests

For a browser request to another origin:

```text
https://frontend.example
        |
        v
https://api.example
```

The API must allow the required tracing headers when browser policy requires it.

Potential concerns:

- CORS allowed headers.
- Preflight requests.
- Credentials.
- Proxy header forwarding.
- Whether the API accepts W3C Trace Context.

## 19.3 Do not expose sensitive data

Do not put the following into trace context:

- Passwords.
- Access tokens.
- Session cookies.
- Employee personal data.
- Salary values.
- Database credentials.

Trace context is transport metadata, not a secure business-data channel.

---

# 20. Database, Redis, and Elasticsearch Spans

Propagation inside a service is usually more important than sending trace headers directly to a database.

## 20.1 Database calls

A database client span should normally be a child of the active application span:

```text
Salary API server span
    |
    └── ScyllaDB query span
```

The database protocol may not carry W3C HTTP headers.

The instrumentation associates the database span with the current in-process context.

## 20.2 PostgreSQL

```text
Attendance API request
    |
    └── PostgreSQL query
```

The query span should contain safe attributes such as:

- Database system.
- Server address.
- Database name where appropriate.
- Operation name.

Avoid recording:

- Passwords.
- Full sensitive query parameters.
- Personal data.
- Full query text if it contains secrets.

## 20.3 Redis

```text
API request
    |
    └── Redis GET/SET
```

Redis spans should inherit the active trace context.

A Redis command is not necessarily a separate distributed trace merely because Redis is another process.

## 20.4 Elasticsearch

```text
Notification worker
    |
    └── Elasticsearch search
```

If the worker's active span is correctly established, the Elasticsearch span becomes part of the same trace.

---

# 21. Asynchronous Processing and Background Workers

Asynchronous systems are more difficult because the producer and consumer do not execute in one continuous call stack.

Example:

```text
Salary API
    |
    | publish message
    v
Queue
    |
    | later
    v
Notification worker
```

The worker may execute seconds or minutes later.

## 21.1 Synchronous propagation

```text
Salary API
    |
    | HTTP request
    v
Notification API
```

The child operation starts immediately.

Parent-child relationship is usually natural.

## 21.2 Asynchronous propagation

```text
Salary API
    |
    | publish event
    v
Queue
    |
    | delayed delivery
    v
Notification worker
```

The worker must receive context from message metadata.

## 21.3 Producer span

```text
Salary API server span
    |
    └── Publish salary-notification message
```

The producer injects context into the message.

## 21.4 Consumer span

The worker extracts the context and starts a consumer span.

```text
Notification worker
    └── Consume salary-notification message
```

Depending on the system's semantics, this may be modeled as:

- A child span.
- A span with a link to the producer span.
- A combination of consumer and processing spans.

---

# 22. Message Queues: Parent-Child vs Span Links

## 22.1 Parent-child relationship

Use parent-child when the consumer operation is logically a direct continuation of the producer operation.

```text
Producer span
    |
    └── Consumer span
```

## 22.2 Span link

A span link associates a span with another span without making it a strict parent-child descendant.

This is often useful when:

- Messages are processed later.
- One message is consumed by multiple workers.
- A batch combines multiple messages.
- A worker processes work from multiple producers.
- The consumer is decoupled from the producer lifecycle.

Conceptually:

```text
Producer span A ─────┐
Producer span B ─────┼──> Batch consumer span
Producer span C ─────┘
```

## 22.3 Why links matter

A single consumer span may depend on many messages.

A single parent cannot accurately represent all upstream causes.

Span links preserve causal relationships without forcing an artificial tree.

## 22.4 OT-Micro-Docker notification example

```text
Salary API
  └── Publish salary slip event
          |
          v
Elasticsearch / queue-like pending state
          |
          v
Notification worker
  ├── Search pending records
  ├── Generate PDF
  └── Send email
```

If the worker polls Elasticsearch rather than consuming a message containing context, direct trace continuity may not exist automatically.

In that case, consider:

- Persisting a safe correlation identifier.
- Creating a new worker trace.
- Adding a link if the originating trace context is available.
- Avoiding fake parent-child relationships when the original context is no longer available.

---

# 23. OT-Micro-Docker End-to-End Examples

## 23.1 Salary request

```text
Browser
  |
  | GET /salary/101
  | traceparent = T1/B1
  v
NGINX
  |
  | forwards traceparent
  v
Salary API
  |
  | server span S1, parent B1
  |
  ├── ScyllaDB query span D1
  |
  └── Notification API HTTP client span N1
          |
          v
      Notification API server span N2
          |
          └── SMTP/email span E1
```

Simplified relationships:

```text
B1
└── S1
    ├── D1
    └── N1
        └── N2
            └── E1
```

All spans share the same trace ID if propagation is correct.

## 23.2 Attendance request

```text
Browser
  |
  v
NGINX
  |
  v
Attendance API
  |
  ├── PostgreSQL query
  └── Redis cache lookup
```

Possible trace:

```text
Browser span
└── Attendance API server span
    ├── Redis GET span
    └── PostgreSQL SELECT span
```

## 23.3 Employee request

```text
Browser
  |
  v
Employee API
  |
  └── ScyllaDB operation
```

Possible trace:

```text
Browser span
└── Employee API server span
    └── ScyllaDB query span
```

## 23.4 Notification worker

```text
Worker scheduler
    |
    └── Worker iteration span
        ├── Elasticsearch search
        ├── PDF generation
        └── SMTP send
```

If the worker is not directly triggered by an incoming request, it may correctly create a new root trace for each processing iteration.

The important point is not to force every operation into an unrelated trace.

---

# 24. Broken Trace Patterns

## 24.1 Every service has a different trace ID

```text
Browser:      T1
Salary API:   T2
ScyllaDB:     T3
Notification: T4
```

Likely causes:

- Header not injected.
- Header not forwarded.
- Header not extracted.
- Unsupported propagation format.
- Instrumentation disabled.
- New root span created manually.
- Context lost across an async boundary.

## 24.2 Backend starts a new root span

A service may create a span without using the extracted context.

Incorrect conceptual result:

```text
Incoming context exists
        |
        X ignored
        |
New root span created
```

Correct:

```text
Incoming context
        |
        v
Extract
        |
        v
Start child span
```

## 24.3 Trace breaks at NGINX

Symptoms:

- Browser trace exists.
- Backend trace exists.
- They do not connect.

Check:

- Browser actually injected the header.
- NGINX received it.
- NGINX forwarded it.
- Backend received it.
- Backend instrumentation extracted it.

## 24.4 Trace breaks at HTTP client

Symptoms:

```text
Salary API span
    └── no Notification API child span
```

Possible causes:

- HTTP client not instrumented.
- Wrong context passed.
- Client request created outside active span.
- Manual HTTP call bypassed instrumentation.
- Context replaced with a background context.

## 24.5 Trace breaks in worker

Symptoms:

```text
Producer trace ends at publish/update
Worker creates unrelated trace
```

Possible causes:

- No context stored in message metadata.
- Polling architecture has no original trace context.
- Worker starts a new root span intentionally.
- Context was stored in unsafe global state.
- Worker extracts the wrong header.

---

# 25. Troubleshooting Methodology

Use a boundary-by-boundary approach.

## Step 1: Identify the first span

Confirm that the initial service creates a span.

Questions:

- Is instrumentation enabled?
- Is the service name correct?
- Is the span exported?
- Is the trace visible in the backend?

## Step 2: Inspect outgoing propagation

Check the outgoing request or message.

For HTTP:

```text
traceparent present?
tracestate present?
```

For messaging:

```text
traceparent in message headers?
```

## Step 3: Inspect the receiver

Confirm that the receiving service gets the same context.

Compare:

```text
Outgoing trace ID
Incoming trace ID
```

## Step 4: Confirm extraction

Check whether the receiving instrumentation supports the carrier.

Examples:

- HTTP headers.
- gRPC metadata.
- Message headers.
- Custom task metadata.

## Step 5: Confirm child-span creation

The receiving service should create a new span with:

```text
same trace ID
new span ID
incoming span as parent or linked context
```

## Step 6: Repeat for every boundary

```text
Browser → NGINX
NGINX → Salary API
Salary API → ScyllaDB
Salary API → Notification API
Notification API → SMTP
```

## Useful temporary diagnostic fields

Log only safe values such as:

```text
service.name
trace_id
span_id
parent_span_id
route
operation
```

Avoid logging:

- Authorization headers.
- Cookies.
- Passwords.
- Salary data.
- Personal information.

---

# 26. Sampling and Propagation

Sampling and propagation are related but not identical.

## 26.1 Sampling decision travels with context

The trace flags can carry a sampling decision.

```text
Service A
  |
  | sampled context
  v
Service B
```

Service B should normally respect the incoming trace decision according to the configured sampling model.

## 26.2 Sampling does not necessarily stop propagation

An unsampled trace may still carry context.

Why?

Because downstream services may need consistent trace identity and sampling behavior.

## 26.3 Head sampling

A decision is made near the beginning of the trace.

```text
Request begins
    |
    v
Sampling decision
    |
    v
Context propagated downstream
```

## 26.4 Tail sampling

The Collector may decide after observing more of the trace.

```text
Services emit spans
    |
    v
Collector receives trace data
    |
    v
Tail sampling decision
```

Propagation still matters because the Collector needs spans with consistent trace IDs to assemble the complete trace.

---

# 27. Security and Privacy

Trace context is metadata, but it crosses trust boundaries.

## 27.1 Do not trust incoming trace IDs as identity

A trace ID does not authenticate a user.

Never use it as proof that a request came from:

- An administrator.
- A trusted service.
- A specific employee.
- A known browser session.

Authentication and authorization must use proper mechanisms.

## 27.2 Avoid sensitive data in headers

Do not place these in `traceparent`, `tracestate`, or custom tracing headers:

- Passwords.
- Tokens.
- API keys.
- Salary values.
- Employee names.
- Personal addresses.
- Medical data.
- Full business payloads.

## 27.3 Header spoofing

External clients may send arbitrary tracing headers.

Your system should:

- Validate propagation format.
- Avoid trusting trace context for authorization.
- Apply gateway trust policies.
- Consider whether external trace context should be accepted.
- Avoid allowing arbitrary vendor state to control security decisions.

## 27.4 Trace ID cardinality

Trace IDs are high-cardinality values.

Do not use them as unbounded metric labels:

```text
http_requests_total{trace_id="..."}
```

That can create a severe metrics-cardinality problem.

Trace IDs belong primarily in traces and correlated logs.

---

# 28. Testing Context Propagation

## 28.1 Unit-test injection and extraction

Test that:

1. A context is created.
2. It is injected into a carrier.
3. The carrier is passed to a simulated receiver.
4. The receiver extracts the same trace ID.
5. The receiver creates a new span with the expected parent.

## 28.2 Integration-test HTTP propagation

Test:

```text
Client → reverse proxy → service
```

Verify:

- Header exists at client output.
- Proxy forwards header.
- Backend extracts header.
- Trace IDs match.
- Span IDs differ.

## 28.3 Test async propagation

Test:

```text
Producer → message metadata → consumer
```

Verify:

- Context is injected into message headers.
- Consumer extracts it.
- Consumer span is correctly parented or linked.
- Multiple messages are handled without context contamination.

## 28.4 Concurrency test

Send multiple requests simultaneously.

Verify:

```text
Request A → Trace A only
Request B → Trace B only
Request C → Trace C only
```

Look for:

- Mixed trace IDs.
- Wrong parents.
- Global context contamination.
- Incorrect thread-local reuse.

## 28.5 Failure tests

Test behavior when:

- `traceparent` is missing.
- `traceparent` is malformed.
- `tracestate` is invalid.
- A proxy removes headers.
- A message has no context.
- A worker receives multiple messages.
- A downstream service times out.

A missing context should generally result in a new root trace or an appropriately handled context—not application failure merely because tracing metadata is absent.

---

# 29. Common Mistakes

## Mistake 1: Confusing trace ID and span ID

Incorrect:

> Every service should use the same span ID.

Correct:

> Services in the same trace share the trace ID, but each operation gets its own span ID.

## Mistake 2: Reusing the incoming span ID

The receiver should create a new span ID.

The incoming span ID identifies the upstream operation.

## Mistake 3: Treating propagation as exporting

Headers connect operations.

OTLP sends telemetry to a Collector or backend.

## Mistake 4: Using a global trace variable

This breaks under concurrency.

Use framework-supported context mechanisms.

## Mistake 5: Replacing request context with background context

Especially dangerous in Go and asynchronous code.

It can discard tracing and cancellation.

## Mistake 6: Assuming NGINX automatically creates a span

A proxy may forward context without generating its own OpenTelemetry span.

The architecture must clearly define which components are instrumented.

## Mistake 7: Assuming database protocols use HTTP headers

Database spans usually inherit application context in-process. They do not necessarily receive W3C headers over the database wire protocol.

## Mistake 8: Treating polling as direct causality

A worker polling Elasticsearch later may not have the original request context.

Do not invent a parent-child relationship without a valid causal context.

## Mistake 9: Putting trace IDs into metrics labels

This creates high cardinality.

Use traces and logs for per-request identity.

## Mistake 10: Trusting trace context for authorization

Trace context is not authentication.

---

# 30. L1 Interview Questions

## Q1. What is context propagation?

Context propagation is the process of carrying trace context from one operation or service to another so that distributed spans can be connected into one trace.

## Q2. What is the difference between trace ID and span ID?

- Trace ID identifies the complete distributed trace.
- Span ID identifies one operation inside that trace.

## Q3. What is `traceparent`?

`traceparent` is a W3C Trace Context header carrying trace identity, parent span identity, version, and trace flags.

## Q4. What is `tracestate`?

It carries vendor-specific tracing information associated with the trace context.

## Q5. What is injection?

Writing context into a carrier, such as HTTP headers.

## Q6. What is extraction?

Reading context from a carrier and making it available to the receiving operation.

## Q7. Is propagation the same as exporting?

No.

- Propagation connects operations.
- Exporting sends telemetry to a Collector or backend.

## Q8. Why does every service need a new span ID?

Because each service performs a distinct operation. Reusing the upstream span ID would merge separate operations into one span.

## Q9. Can a trace continue without a `traceparent` header?

Not from that header alone. If no valid context is available, the service may start a new root trace.

## Q10. Is a trace ID an authentication token?

No. It is only tracing metadata.

---

# 31. L2 Interview Questions

## Q1. How does trace context travel through NGINX?

The client injects context into HTTP headers. NGINX forwards the headers to the backend. The backend extracts the context and creates a child server span.

## Q2. Why might browser and backend traces be disconnected?

Possible reasons:

- Browser instrumentation did not inject context.
- CORS prevented the header.
- NGINX dropped the header.
- Backend instrumentation did not extract it.
- Different propagation formats were used.

## Q3. How does a database span become part of the HTTP trace?

The database instrumentation reads the active in-process context and creates a child span. It does not necessarily need to receive an HTTP `traceparent` header.

## Q4. Why is Go's `context.Context` important?

It carries request-scoped cancellation, deadlines, values, and tracing context. Replacing it with `context.Background()` can break tracing and cancellation.

## Q5. How does a worker continue a producer trace?

The producer injects context into message metadata. The worker extracts it and creates a consumer or processing span, using parent-child or span-link modeling as appropriate.

## Q6. What causes context loss across threads?

The runtime may not automatically transfer the active context to a new thread or executor. Instrumented executors or explicit context propagation may be required.

## Q7. Should every worker span be a child of the original API span?

No. If the work is asynchronous, delayed, batched, or caused by multiple messages, span links or a new root trace may be more accurate.

## Q8. Why should trace IDs not be metric labels?

Trace IDs have extremely high cardinality. Adding them to metrics can create excessive time series and overload the metrics backend.

---

# 32. L3 Interview Questions

## Q1. Design propagation for React → NGINX → APIs → worker.

A strong design would:

1. Instrument browser HTTP calls.
2. Inject W3C context into requests.
3. Configure CORS for required headers.
4. Ensure NGINX forwards `traceparent` and `tracestate`.
5. Instrument backend HTTP servers.
6. Pass active context to database and client libraries.
7. Inject context into asynchronous message metadata.
8. Extract context in the worker.
9. Use span links for batch or multi-cause processing.
10. Validate the complete trace with integration tests.

## Q2. How would you troubleshoot a trace that breaks only at NGINX?

Check the boundary in order:

```text
Browser outgoing headers
        ↓
NGINX incoming headers
        ↓
NGINX upstream headers
        ↓
Backend incoming headers
        ↓
Backend extracted context
```

Compare trace IDs at each stage.

## Q3. How would you handle a worker that polls Elasticsearch?

Do not assume the polling worker is a direct child of the original API request.

Create a worker iteration span. If a safe causal context is persisted and available, use a link or appropriate parent relationship. Otherwise, start a new root trace and correlate using safe business metadata in logs or attributes.

## Q4. How can propagation be secure?

- Validate incoming context.
- Never use it for authorization.
- Avoid sensitive data in headers.
- Apply trust boundaries at gateways.
- Avoid blindly accepting vendor-specific state.
- Prevent trace IDs from becoming security identifiers.

## Q5. Why can a trace be broken even when every service exports spans successfully?

Exporting only proves that spans reached a telemetry destination. If context was not propagated or extracted, each service may export valid but unrelated traces.

## Q6. How would you test context propagation under concurrency?

Send parallel requests with unique test markers, inspect spans by trace ID, and verify that each request's spans remain inside its own trace. Also test thread pools, async tasks, retries, and worker concurrency.

## Q7. What is the difference between a span link and a parent-child relationship?

Parent-child expresses a hierarchical execution relationship. A span link expresses a causal association without claiming strict hierarchy. Links are useful for batches, asynchronous work, and operations influenced by multiple upstream spans.

## Q8. What happens if an incoming `traceparent` is malformed?

The receiver should reject or ignore the invalid propagation context according to the propagator's behavior and normally create a new root trace rather than allowing malformed metadata to break the business request.

---

# 33. Quick Revision

```text
Trace ID
  = complete distributed trace

Span ID
  = one operation

traceparent
  = standard propagation header

tracestate
  = vendor-specific propagation state

Inject
  = write context into carrier

Extract
  = read context from carrier

Propagation
  = connect operations

Exporting
  = send telemetry to Collector/backend

HTTP
  = headers

Messaging
  = message metadata/headers

Database
  = usually inherits active in-process context

Async worker
  = extract context explicitly; use links when appropriate

Security
  = trace context is not authentication
```

---

# 34. Self-Assessment

Answer these without looking at the previous sections.

1. Why is context propagation required in distributed tracing?
2. What information does `traceparent` carry?
3. Why does the receiving service create a new span ID?
4. What is the difference between injecting and extracting?
5. How does NGINX affect trace continuity?
6. Why can a Go service lose tracing when it uses `context.Background()`?
7. How does a Python worker receive context from a message?
8. Why might a batch consumer use span links?
9. Why should trace IDs not be used as metric labels?
10. How would you troubleshoot a broken Browser → Salary API trace?
11. Why does exporting not guarantee trace continuity?
12. Why should trace context never be used for authorization?

---

# 35. Final Implementation Checklist

## Application instrumentation

- [ ] HTTP server instrumentation enabled.
- [ ] HTTP client instrumentation enabled.
- [ ] Database instrumentation enabled where supported.
- [ ] Redis instrumentation enabled where supported.
- [ ] Elasticsearch instrumentation enabled where supported.
- [ ] Service resource attributes configured.
- [ ] Active context preserved through business logic.

## HTTP propagation

- [ ] Browser injects W3C context.
- [ ] CORS allows required tracing headers.
- [ ] NGINX forwards `traceparent`.
- [ ] NGINX forwards `tracestate` when applicable.
- [ ] Backend extracts incoming context.
- [ ] Backend creates a new span ID.
- [ ] Downstream clients inject context.

## Async propagation

- [ ] Producer injects context into message metadata.
- [ ] Consumer extracts context.
- [ ] Worker creates a processing span.
- [ ] Parent-child vs span-link modeling is intentional.
- [ ] Context is not stored in global mutable state.
- [ ] Multiple concurrent messages are tested.

## Security

- [ ] No secrets in trace headers.
- [ ] No credentials in span attributes.
- [ ] Trace context is not used for authorization.
- [ ] Incoming context is validated.
- [ ] Trace IDs are not metric labels.
- [ ] Sensitive payloads are not recorded by default.

## Validation

- [ ] Browser → NGINX trace verified.
- [ ] NGINX → API trace verified.
- [ ] API → database spans verified.
- [ ] API → API propagation verified.
- [ ] Worker propagation verified.
- [ ] Concurrent requests tested.
- [ ] Missing and malformed context tested.
- [ ] Trace IDs and parent IDs reviewed in the backend.

---

## Final Takeaway

> **Distributed tracing is not achieved merely by creating spans. It is achieved when the correct context travels across every execution boundary and each receiving operation creates the correct relationship to the upstream operation.**

For OT-Micro-Docker, the critical path is:

```text
React Browser
    ↓ traceparent
NGINX
    ↓ forwarded traceparent
Backend API
    ↓ active context
Database / Redis / Elasticsearch
    ↓ message metadata when asynchronous
Notification Worker
    ↓
PDF generation and email delivery
```

If context propagation is correct, the tracing backend can show how one user request moved through the system. If propagation is broken, every service may appear healthy while the trace remains fragmented.
