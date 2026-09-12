# Part 04 — OpenTelemetry Metrics and Logs in Depth

> **Series:** OpenTelemetry for OT-Micro-Docker  
> **Level:** Beginner → Intermediate → Advanced  
> **Focus:** Metrics, logs, correlation, dashboards, alerting, and production troubleshooting

---

## Table of Contents

1. Learning Objectives
2. Metrics vs Traces vs Logs
3. What Is a Metric?
4. Metric Data Model
5. OpenTelemetry Metric Instruments
6. Counter
7. UpDownCounter
8. Gauge-Like Measurements
9. Histogram
10. Observable Instruments
11. Metric Temporality
12. Aggregation
13. Attributes, Labels, and Cardinality
14. Histograms and Percentiles
15. RED and USE Methods
16. Prometheus Functions: `rate`, `irate`, and `increase`
17. What Is a Log?
18. Structured vs Unstructured Logs
19. Log Severity and Log Fields
20. Exceptions and Stack Traces
21. Correlating Logs with Traces
22. Metrics vs Events vs Logs
23. OT-Micro-Docker Business Metrics
24. OT-Micro-Docker Technical Metrics
25. Database and Dependency Metrics
26. Queue and Worker Metrics
27. SLI, SLO, and SLA
28. Dashboard Design
29. Alerting Principles
30. Common Mistakes
31. L1 Interview Questions
32. L2 Interview Questions
33. L3 Interview Questions
34. Quick Revision
35. Self-Assessment
36. Final Checklist

---

# 1. Learning Objectives

After completing this part, you should be able to:

- Explain the difference between metrics, logs, and traces.
- Understand the OpenTelemetry metric data model.
- Select the correct instrument for a measurement.
- Explain counters, up-down counters, gauges, histograms, and observable instruments.
- Understand cumulative and delta temporality.
- Explain aggregation and why it matters.
- Understand histogram buckets and percentiles such as p50, p90, p95, and p99.
- Write useful Prometheus queries.
- Explain structured logging.
- Correlate logs with traces using `trace_id` and `span_id`.
- Design useful metrics for OT-Micro-Docker.
- Avoid high-cardinality labels.
- Design dashboards and alerts based on user impact.
- Answer practical DevOps and observability interview questions.

---

# 2. Metrics vs Traces vs Logs

OpenTelemetry supports three major observability signals:

| Signal | Main Question | Example |
|---|---|---|
| **Metrics** | How much? How often? How many? | Request rate is 120 requests/second |
| **Traces** | Where did the request spend time? | Salary API spent 800 ms querying ScyllaDB |
| **Logs** | What happened? | `Salary generation failed: employee not found` |

A useful mental model:

```text
Metrics tell you that something is wrong.
Traces tell you where it is wrong.
Logs tell you what happened.
```

## Example

Suppose the salary page is slow.

### Metrics

```text
salary_api_http_server_duration_p95 = 2.8 seconds
```

This tells you that the service is slow.

### Trace

```text
Browser
  └── Frontend request
       └── Salary API
            ├── ScyllaDB query: 2.4 seconds
            └── Elasticsearch indexing: 100 ms
```

This tells you where the time was spent.

### Log

```json
{
  "severity": "ERROR",
  "message": "ScyllaDB query timed out",
  "employee_id": "E1024",
  "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",
  "span_id": "00f067aa0ba902b7"
}
```

This gives the detailed event.

## Important

Metrics, logs, and traces are not competing alternatives. They answer different questions and become much more powerful when correlated.

---

# 3. What Is a Metric?

A **metric** is a numerical measurement recorded over time.

Examples:

- Number of HTTP requests.
- Number of failed requests.
- Request duration.
- CPU utilization.
- Memory usage.
- Active connections.
- Queue depth.
- Database query duration.
- Number of emails sent.
- Number of salary slips generated.

A metric normally contains:

```text
Metric name
+ Value
+ Time
+ Attributes
+ Resource information
```

Example:

```text
http.server.request.duration
value = 0.842 seconds
timestamp = 2026-09-12T07:20:00Z
attributes:
  http.request.method = "GET"
  http.response.status_code = 200
  server.address = "salary-api"
resource:
  service.name = "salary-api"
  deployment.environment.name = "dev"
```

A single measurement is not automatically useful. It becomes useful when collected consistently, aggregated correctly, visualized, and compared with a threshold or objective.

---

# 4. Metric Data Model

A metric has several important dimensions.

## 4.1 Metric Name

The name describes what is measured.

Good examples:

```text
http.server.request.duration
http.server.request.count
db.client.operation.duration
otms.salary.slips.generated
otms.notification.emails.sent
```

Poor examples:

```text
value1
metric123
slowthing
count
```

A good metric name should answer:

> What is being measured, and in what unit?

## 4.2 Value

The value can be:

- Integer.
- Floating-point number.
- Distribution represented by a histogram.
- Boolean-like state represented numerically.

Examples:

```text
requests = 150
active_connections = 12
duration = 0.423
queue_depth = 27
cpu_utilization = 0.72
```

## 4.3 Timestamp

The timestamp tells when the measurement occurred.

For example:

```text
2026-09-12T07:20:00Z
```

## 4.4 Attributes

Attributes provide dimensions for filtering and grouping.

Example:

```text
http.request.method = "GET"
http.response.status_code = 500
service.name = "attendance-api"
deployment.environment.name = "dev"
```

Attributes are powerful, but they must be bounded. This is discussed in detail later.

## 4.5 Resource

Resource attributes describe the entity producing telemetry.

Example:

```text
service.name = "salary-api"
service.version = "1.4.2"
deployment.environment.name = "dev"
host.name = "ip-10-0-2-15"
cloud.provider = "aws"
cloud.region = "us-east-1"
```

A resource is different from a metric attribute:

| Resource Attribute | Metric Attribute |
|---|---|
| Describes the telemetry source | Describes a measurement dimension |
| Usually stable | Can vary per measurement |
| Service name, host, version | HTTP method, status code, route |
| Helps identify the producer | Helps filter or group data |

---

# 5. OpenTelemetry Metric Instruments

An instrument is the API object used by application code to record measurements.

| Instrument | Behavior | Example |
|---|---|---|
| Counter | Only increases | Total requests |
| UpDownCounter | Increases and decreases | Active requests |
| Histogram | Records a distribution | Request duration |
| ObservableCounter | Reports a monotonic value through callback | Total bytes sent from a library |
| ObservableUpDownCounter | Reports a changing value through callback | Current queue size |
| ObservableGauge | Reports the current value through callback | CPU temperature or memory usage |

## Choosing an instrument

| Question | Instrument |
|---|---|
| Does the value only increase? | Counter |
| Can the value increase and decrease? | UpDownCounter |
| Do I need duration or size distribution? | Histogram |
| Is the value obtained by periodically observing something? | Observable instrument |
| Do I need p95 or p99 latency? | Histogram |

---

# 6. Counter

A **Counter** represents a monotonically increasing value.

It should not decrease during the lifetime of a metric stream.

## Examples

```text
Total HTTP requests
Total HTTP errors
Total employees created
Total salary slips generated
Total emails sent
Total database queries
```

Example:

```text
10 → 11 → 12 → 13 → 14
```

Invalid counter behavior:

```text
10 → 11 → 7 → 8
```

If the application restarts, the counter may start again from zero. Backends generally account for process restarts using timestamps, resource identity, and aggregation rules.

## Example

```python
request_counter.add(
    1,
    {
        "http.request.method": "GET",
        "http.response.status_code": "200",
        "service.name": "salary-api"
    }
)
```

## Counter vs current value

Use a counter for:

```text
Total requests received
```

Do not use a counter for:

```text
Currently active requests
```

Active requests can go up and down, so an UpDownCounter is more appropriate.

---

# 7. UpDownCounter

An **UpDownCounter** represents a value that can increase or decrease.

## Examples

- Active HTTP requests.
- Active database connections.
- Number of jobs currently being processed.
- Number of workers currently busy.
- Number of open file descriptors.

Example:

```text
Active requests:

0 → 1 → 2 → 3 → 2 → 1 → 0
```

The value changes using positive and negative additions:

```python
active_requests.add(1)
# Request starts

active_requests.add(-1)
# Request finishes
```

## Important distinction

```text
Total requests received       → Counter
Requests currently in flight  → UpDownCounter
```

---

# 8. Gauge-Like Measurements

A gauge represents a current state rather than a cumulative total.

Examples:

- Current CPU usage.
- Current memory usage.
- Current queue depth.
- Current temperature.
- Current number of connected clients.

In OpenTelemetry, current values are commonly reported through observable instruments such as `ObservableGauge`.

Example concept:

```text
queue_depth = 10
queue_depth = 14
queue_depth = 8
queue_depth = 0
```

A gauge-like value can move in either direction and does not represent a total accumulated count.

## Gauge vs UpDownCounter

| Gauge-like value | UpDownCounter |
|---|---|
| Reports current state | Records changes to a value |
| Often observed periodically | Usually updated when events happen |
| Example: current CPU percentage | Example: active requests |
| Example: current queue depth | Example: workers entering/leaving |

The right choice depends on how the application obtains the value.

---

# 9. Histogram

A **Histogram** records a collection of observations and describes their distribution.

It is commonly used for:

- Request duration.
- Database query duration.
- Response size.
- Message processing time.
- File upload size.
- Queue wait time.

Example observations:

```text
0.10 s
0.12 s
0.15 s
0.20 s
0.30 s
1.20 s
2.80 s
```

A histogram helps answer:

- What is the average duration?
- How many requests were below 100 ms?
- How many requests were below 1 second?
- What is approximately the p95 latency?
- Is there a long tail of slow requests?

## Why not just use average?

Suppose 99 requests take 100 ms and one request takes 10 seconds.

The average may look acceptable, while one user experiences a very slow request.

A histogram preserves distribution information, allowing the backend to estimate percentiles and identify tail latency.

---

# 10. Observable Instruments

Observable instruments obtain values through callbacks or observation functions.

They are useful when the application or operating system owns the current value.

Examples:

```text
CPU utilization
Memory usage
Disk usage
Current queue size
Current number of open connections
Total bytes received from a system API
```

Conceptual flow:

```text
Metric reader
    |
    | periodically invokes callback
    v
Application / OS / library
    |
    v
Current observed value
```

Observable instruments are especially useful for infrastructure and runtime measurements.

---

# 11. Metric Temporality

Temporality describes how a metric is accumulated over time.

The two important types are:

- Cumulative.
- Delta.

## 11.1 Cumulative Temporality

Cumulative data represents the total since the beginning of the metric stream.

Example:

```text
10:00 → 100 requests
10:01 → 160 requests
10:02 → 220 requests
```

The values are cumulative totals.

The increase between 10:01 and 10:02 is:

```text
220 - 160 = 60 requests
```

## 11.2 Delta Temporality

Delta data represents the change during a specific interval.

Example:

```text
10:00–10:01 → 100 requests
10:01–10:02 → 60 requests
10:02–10:03 → 80 requests
```

Each value describes only that interval.

## Comparison

| Cumulative | Delta |
|---|---|
| Total since start | Change during interval |
| Values generally increase | Values can vary each interval |
| Backend calculates differences | Backend receives interval changes |
| Common for counters | Common in some export pipelines |

## Why this matters

If a backend interprets cumulative data as delta, or delta data as cumulative, dashboards and alerts may be wrong.

---

# 12. Aggregation

Aggregation converts many raw measurements into a compact representation.

Suppose request durations are:

```text
100 ms
120 ms
140 ms
200 ms
250 ms
800 ms
1200 ms
```

Possible aggregations include:

- Sum.
- Count.
- Min.
- Max.
- Average.
- Histogram buckets.
- Exponential histogram data.

## Example

For request count:

```text
Count = 7
```

For total duration:

```text
Sum = 2810 ms
```

For average:

```text
Average = 2810 / 7 = 401.4 ms
```

The average is useful, but it does not describe the complete distribution.

## Histogram aggregation

A histogram may produce:

```text
Bucket <= 100 ms: 1
Bucket <= 250 ms: 5
Bucket <= 1 s:    6
Bucket <= 5 s:    7
Count:            7
Sum:              2810 ms
```

A backend can use this distribution to estimate percentiles.

---

# 13. Attributes, Labels, and Cardinality

Attributes are dimensions attached to metrics.

Example:

```text
http.request.method = "GET"
http.route = "/employees/{id}"
http.response.status_code = 200
```

In Prometheus terminology, these dimensions are commonly called **labels**.

## Good attributes

```text
http.request.method
http.route
http.response.status_code
service.name
deployment.environment.name
db.system
db.operation.name
```

## Dangerous attributes

```text
user_id
employee_id
request_id
trace_id
session_id
email
full_url_with_query_parameters
```

These values may be nearly unique for every request.

## What is cardinality?

Cardinality is the number of unique combinations of attribute values.

Suppose a metric has:

```text
method = GET, POST
status = 200, 400, 500
route = 5 routes
```

Maximum combinations:

```text
2 × 3 × 5 = 30
```

If you add 1 million possible user IDs:

```text
2 × 3 × 5 × 1,000,000 = 30,000,000
```

That can create a serious metrics storage and query problem.

## Rule

> Use bounded dimensions for metrics. Put high-cardinality identifiers in logs or traces instead.

## Route template vs raw URL

Bad:

```text
http.url = "/employees/1001"
http.url = "/employees/1002"
http.url = "/employees/1003"
```

Better:

```text
http.route = "/employees/{id}"
```

The route template groups requests into a bounded series.

---

# 14. Histograms and Percentiles

Percentiles describe the position below which a percentage of observations fall.

## p50

p50 is the median.

Approximately 50% of observations are at or below this value.

## p90

Approximately 90% of observations are at or below this value.

## p95

Approximately 95% of observations are at or below this value.

## p99

Approximately 99% of observations are at or below this value.

Example:

```text
p50 = 100 ms
p90 = 250 ms
p95 = 600 ms
p99 = 2.5 s
```

Interpretation:

- Typical requests are around 100 ms.
- 10% take longer than 250 ms.
- 5% take longer than 600 ms.
- 1% take longer than 2.5 seconds.

## Why p95 and p99 matter

Average latency can hide slow users.

Percentiles expose tail latency.

| Metric | Meaning |
|---|---|
| Average | Overall arithmetic mean |
| p50 | Typical middle observation |
| p90 | Slow tail affecting 10% |
| p95 | Slow tail affecting 5% |
| p99 | Extreme tail affecting 1% |

## Important limitation

A percentile is not normally obtained by simply averaging individual request durations.

It is calculated from a distribution, often using histogram buckets or another backend-supported aggregation.

## Histogram buckets

Example buckets:

```text
<= 5 ms
<= 10 ms
<= 25 ms
<= 50 ms
<= 100 ms
<= 250 ms
<= 500 ms
<= 1 s
<= 2.5 s
<= 5 s
```

Bucket selection matters:

- Too few buckets reduce accuracy.
- Too many buckets increase cost.
- Buckets should reflect expected service latency.

---

# 15. RED and USE Methods

## 15.1 RED Method

RED is designed primarily for request-driven services.

| Letter | Meaning | Example |
|---|---|---|
| R | Rate | Requests per second |
| E | Errors | Failed requests per second or error percentage |
| D | Duration | Request latency |

For `salary-api`:

```text
Rate:
  80 requests/second

Errors:
  4 errors/second

Duration:
  p95 = 1.8 seconds
```

RED is excellent for API dashboards.

## 15.2 USE Method

USE is commonly used for resources and infrastructure.

| Letter | Meaning | Example |
|---|---|---|
| U | Utilization | CPU at 75% |
| S | Saturation | Run queue or backlog |
| E | Errors | Disk errors or network errors |

For an EC2 instance:

```text
CPU utilization = 78%
Memory utilization = 84%
Disk saturation = high
Network errors = 0
```

## RED vs USE

| RED | USE |
|---|---|
| Focuses on requests/services | Focuses on resources |
| Rate, Errors, Duration | Utilization, Saturation, Errors |
| API and microservice dashboards | CPU, disk, network, database resources |

---

# 16. Prometheus Functions: `rate`, `irate`, and `increase`

These functions are commonly used when OpenTelemetry metrics are exported to Prometheus or a Prometheus-compatible backend.

## 16.1 `rate()`

`rate()` calculates the average per-second increase of a counter over a time range.

Example:

```promql
rate(http_server_request_count_total[5m])
```

Meaning:

> Average requests per second during the last five minutes.

Use `rate()` for:

- Dashboards.
- Stable alerting.
- Request throughput.
- Error rate.

## 16.2 `irate()`

`irate()` uses the most recent samples to estimate the instantaneous per-second rate.

Example:

```promql
irate(http_server_request_count_total[5m])
```

It is more sensitive to short spikes and noise.

Use it carefully for:

- Fast-moving graphs.
- Detecting sudden bursts.
- Troubleshooting short-lived behavior.

It is usually less suitable for stable alerting.

## 16.3 `increase()`

`increase()` estimates the total increase of a counter during a range.

Example:

```promql
increase(http_server_request_count_total[1h])
```

Meaning:

> Approximately how many requests occurred during the last hour?

## Comparison

| Function | Result | Typical Use |
|---|---|---|
| `rate()` | Average per-second increase | Dashboards and alerts |
| `irate()` | Recent per-second rate | Spikes and short-term behavior |
| `increase()` | Total increase over range | Requests in an hour |

## Error percentage

```promql
sum(rate(http_server_request_count_total{
  http_response_status_code=~"5.."
}[5m]))
/
sum(rate(http_server_request_count_total[5m]))
* 100
```

This calculates the approximate percentage of HTTP 5xx requests.

---

# 17. What Is a Log?

A log is a record of an event that occurred in a system.

Examples:

```text
Application started
Database connection established
Employee created
Salary generation failed
Email sent
Redis connection refused
```

A log answers:

> What happened, and what details were associated with it?

A log may contain:

- Timestamp.
- Severity.
- Message.
- Service name.
- Host.
- Trace ID.
- Span ID.
- Exception details.
- Business identifiers.
- Operational context.

---

# 18. Structured vs Unstructured Logs

## 18.1 Unstructured Log

```text
2026-09-12 07:30:12 ERROR Salary generation failed for employee E1024
```

Humans can read it, but machines must parse the message.

## 18.2 Structured Log

```json
{
  "timestamp": "2026-09-12T07:30:12Z",
  "severity": "ERROR",
  "service.name": "salary-api",
  "event.name": "salary_generation_failed",
  "employee_id": "E1024",
  "error.type": "DatabaseTimeout",
  "message": "Salary generation failed"
}
```

Structured logs are easier to:

- Search.
- Filter.
- Aggregate.
- Correlate.
- Alert on.
- Process automatically.

## Recommended practice

Use a stable event name and structured fields.

Prefer:

```json
{
  "event.name": "notification_email_failed",
  "smtp.provider": "internal",
  "error.type": "TimeoutError"
}
```

Over:

```text
Something bad happened while sending email maybe timeout
```

---

# 19. Log Severity and Log Fields

Common severity levels include:

| Severity | Meaning |
|---|---|
| TRACE | Very detailed diagnostic information |
| DEBUG | Developer troubleshooting information |
| INFO | Normal application lifecycle or business event |
| WARN | Unexpected condition that does not necessarily stop work |
| ERROR | Operation failed |
| FATAL | Severe failure that may terminate the process |

## Example

```text
INFO:
  Salary API started

DEBUG:
  Executing ScyllaDB query for employee E1024

WARN:
  Redis cache unavailable; continuing without cache

ERROR:
  Salary slip generation failed

FATAL:
  Application cannot start because required configuration is missing
```

Severity should reflect operational importance, not developer emotion.

Avoid logging every normal event as `ERROR`.

---

# 20. Exceptions and Stack Traces

An exception log should contain:

- Exception type.
- Human-readable message.
- Stack trace.
- Operation being performed.
- Relevant safe context.
- Trace ID and span ID.
- Service and version.

Example:

```json
{
  "severity": "ERROR",
  "event.name": "scylla_query_failed",
  "error.type": "TimeoutError",
  "error.message": "Query timed out after 2 seconds",
  "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",
  "span_id": "00f067aa0ba902b7",
  "db.system": "scylladb"
}
```

## Do not log

- Passwords.
- Access tokens.
- API keys.
- Session cookies.
- Full payment card numbers.
- Sensitive personal data.
- Entire request bodies without a clear need.

A stack trace is valuable, but it must be handled as potentially sensitive operational data.

---

# 21. Correlating Logs with Traces

The most useful observability setup allows an engineer to move from:

```text
Metric → Trace → Log
```

A log should include:

```text
trace_id
span_id
```

Example:

```json
{
  "severity": "ERROR",
  "message": "Elasticsearch indexing failed",
  "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",
  "span_id": "00f067aa0ba902b7"
}
```

The trace can then show:

```text
Trace:
  Salary API request
    └── Elasticsearch index operation
```

The log explains:

```text
Connection refused by Elasticsearch
```

## Why correlation matters

Without correlation:

```text
Thousands of logs
Unknown request
Unknown service operation
```

With correlation:

```text
Specific trace
  └── Specific span
       └── Exact log event
```

This dramatically reduces troubleshooting time.

---

# 22. Metrics vs Events vs Logs

These concepts are related but different.

| Concept | Meaning | Example |
|---|---|---|
| Metric | Numerical measurement over time | 120 requests/sec |
| Log | Detailed record of an event | Email failed |
| Event | Something that happened at a point in time | Employee created |
| Trace span event | Annotation inside a span | Cache miss |
| Alert | Notification generated from a condition | Error rate above 5% |

An event may be represented as:

- A log.
- A span event.
- A metric increment.
- A message in a queue.

The representation depends on how the event will be used.

---

# 23. OT-Micro-Docker Business Metrics

Business metrics describe whether the application is performing its actual business function.

These are often more valuable than infrastructure metrics alone.

## 23.1 Employee Metrics

```text
otms.employee.created
otms.employee.updated
otms.employee.deleted
otms.employee.lookup
```

Useful dimensions:

```text
operation
result
```

Avoid:

```text
employee_id
```

as a metric label.

## 23.2 Attendance Metrics

```text
otms.attendance.records.processed
otms.attendance.records.failed
otms.attendance.calculation.duration
```

Useful dimensions:

```text
operation
result
source
```

## 23.3 Salary Metrics

```text
otms.salary.slips.generated
otms.salary.slips.failed
otms.salary.calculation.duration
otms.salary.records.indexed
```

Useful dimensions:

```text
result
payment_period
```

Only use `payment_period` if it remains bounded and useful.

## 23.4 Notification Metrics

```text
otms.notification.emails.sent
otms.notification.emails.failed
otms.notification.pdf.generated
otms.notification.jobs.processed
otms.notification.jobs.failed
```

Useful dimensions:

```text
result
notification_type
```

## 23.5 Business Success Rate

Example:

```promql
sum(rate(otms_salary_slips_generated_total{
  result="success"
}[5m]))
/
sum(rate(otms_salary_slips_generated_total[5m]))
* 100
```

This is more meaningful than CPU utilization when determining whether salary processing is working.

---

# 24. OT-Micro-Docker Technical Metrics

## Frontend and NGINX

Useful metrics:

```text
HTTP request count
HTTP request duration
HTTP response status
Active connections
Requests by route
5xx error rate
```

## Employee API

```text
Request rate
Request duration
ScyllaDB query duration
ScyllaDB errors
Cache hit count
Cache miss count
```

## Attendance API

```text
Request rate
PostgreSQL query duration
PostgreSQL connection errors
Redis operation duration
Attendance processing failures
```

## Salary API

```text
Request rate
Salary calculation duration
ScyllaDB query duration
Elasticsearch indexing duration
Indexing failures
```

## Notification Worker

```text
Jobs received
Jobs processed
Jobs failed
Job processing duration
PDF generation duration
SMTP request duration
Email success count
Email failure count
Pending queue depth
Oldest pending job age
```

## Example dashboard groups

```text
Service Health
  - Request rate
  - Error rate
  - p95 latency

Dependencies
  - ScyllaDB latency
  - PostgreSQL latency
  - Redis latency
  - Elasticsearch errors
  - SMTP failures

Business
  - Salary slips generated
  - Emails sent
  - Failed notifications
  - Pending jobs
```

---

# 25. Database and Dependency Metrics

A service can be healthy from an HTTP perspective while its dependency is failing.

## ScyllaDB

Useful measurements:

```text
Query count
Query duration
Query errors
Timeout count
Connection failures
Read/write latency
```

## PostgreSQL

Useful measurements:

```text
Query duration
Connection pool usage
Connection errors
Transaction failures
Lock wait time
Slow query count
```

## Redis

Useful measurements:

```text
Cache hits
Cache misses
Operation duration
Connection failures
Evictions
Memory usage
```

## Elasticsearch

Useful measurements:

```text
Indexing requests
Indexing failures
Indexing duration
Search duration
Rejected requests
Cluster health
```

## SMTP

Useful measurements:

```text
Email attempts
Email success
Email failures
SMTP connection duration
Timeouts
Retries
```

## Dependency error rate

A useful dependency dashboard should distinguish:

```text
Application errors
Dependency errors
Timeouts
Retries
Fallbacks
```

For example:

```text
Salary API 5xx errors = 5%
ScyllaDB timeout rate = 4.8%
```

This strongly suggests that the database may be contributing to the application failure.

---

# 26. Queue and Worker Metrics

Notification processing is often asynchronous.

The worker may read pending salary records, generate PDFs, and send emails.

Important queue metrics include:

## Queue Depth

```text
Number of pending jobs
```

High queue depth means work is accumulating.

## Processing Rate

```text
Jobs processed per second
```

## Failure Rate

```text
Failed jobs / total jobs
```

## Job Age

```text
Age of the oldest pending job
```

Job age is often more meaningful than queue depth.

Example:

```text
Queue depth = 10
Oldest job age = 4 hours
```

Even though the queue is small, users may be waiting too long.

## Worker Utilization

```text
Busy workers / total workers
```

## Retry Count

```text
Number of retries per job
```

A retry storm can overload dependencies and make the original incident worse.

---

# 27. SLI, SLO, and SLA

## SLI — Service Level Indicator

An SLI is a measured indicator of service performance.

Examples:

```text
Successful request percentage
p95 latency
Email delivery success rate
Salary processing completion rate
```

## SLO — Service Level Objective

An SLO is the target for an SLI.

Example:

```text
99.5% of salary API requests should succeed over 30 days.
```

Another example:

```text
95% of salary API requests should complete within 500 ms.
```

## SLA — Service Level Agreement

An SLA is a formal agreement, often involving commitments and consequences.

Example:

```text
The service provider guarantees 99.9% availability.
```

## Comparison

| Term | Meaning |
|---|---|
| SLI | What you measure |
| SLO | The target |
| SLA | The contractual commitment |

## Error budget

If the SLO is 99.9% availability, the allowed unavailability is approximately:

```text
0.1%
```

The error budget helps teams balance reliability and release velocity.

---

# 28. Dashboard Design

A dashboard should help answer a decision-oriented question.

Bad dashboard:

```text
200 random graphs
```

Good dashboard:

```text
Is the application healthy?
If not, which service is affected?
What dependency is causing the issue?
Are users affected?
```

## Recommended dashboard layout

### Row 1: User Impact

```text
Request rate
Error rate
p50 latency
p95 latency
p99 latency
```

### Row 2: Service Health

```text
Per-service request rate
Per-service error rate
Per-service latency
```

### Row 3: Dependencies

```text
ScyllaDB latency
PostgreSQL errors
Redis hit ratio
Elasticsearch indexing failures
SMTP failures
```

### Row 4: Business

```text
Employees created
Salary slips generated
Emails sent
Failed notifications
Pending jobs
```

### Row 5: Infrastructure

```text
CPU
Memory
Disk
Network
Container or process restarts
```

## Dashboard principles

- Put user impact first.
- Use consistent units.
- Show time ranges clearly.
- Use rates for throughput.
- Use percentages for ratios.
- Avoid excessive colors.
- Keep labels readable.
- Link panels to traces or logs where possible.
- Do not hide failures behind averages.

---

# 29. Alerting Principles

An alert should represent an actionable condition.

## Bad alert

```text
CPU > 70%
```

CPU may be high during normal traffic.

## Better alert

```text
HTTP 5xx error rate > 5% for 10 minutes
```

This is closer to user impact.

## Good alert examples

### High error rate

```promql
(
  sum(rate(http_server_request_count_total{
    http_response_status_code=~"5.."
  }[5m]))
/
  sum(rate(http_server_request_count_total[5m]))
) > 0.05
```

### High latency

```text
p95 request duration > 1 second for 10 minutes
```

### Queue backlog

```text
Oldest pending notification job > 15 minutes
```

### Dependency failure

```text
ScyllaDB timeout rate > 2% for 5 minutes
```

## Alert quality checklist

An alert should answer:

- What is wrong?
- Which service is affected?
- How severe is it?
- Who owns it?
- What should the engineer check first?
- Is the alert actionable?
- Is the condition sustained or just a brief spike?

## Avoid alert fatigue

Too many alerts cause engineers to ignore alerts.

Prefer fewer, high-value alerts over hundreds of noisy ones.

---

# 30. Common Mistakes

## Mistake 1: Using a counter for current active requests

Wrong:

```text
Counter = active requests
```

Better:

```text
UpDownCounter = active requests
```

## Mistake 2: Using raw user IDs as metric labels

Wrong:

```text
employee_id = E1024
```

Better:

```text
employee_id in trace/log context
```

## Mistake 3: Using average latency only

Average hides tail latency.

Use:

```text
p50, p95, p99
```

alongside average.

## Mistake 4: Logging secrets

Never log:

```text
password
access token
API key
session cookie
```

## Mistake 5: Every log is ERROR

Normal events should be INFO or DEBUG.

## Mistake 6: No trace correlation

Logs without `trace_id` and `span_id` are harder to connect to requests.

## Mistake 7: Dashboard full of infrastructure metrics

CPU can be normal while users are unable to generate salary slips.

Always include business and user-impact metrics.

## Mistake 8: Alerting on every spike

Use duration windows and appropriate thresholds.

## Mistake 9: Too many dimensions

Every additional label combination creates more time series.

## Mistake 10: Treating metrics as logs

Metrics should be aggregated numerical signals. Do not encode arbitrary text into metric labels.

---

# 31. L1 Interview Questions

## Q1. What is a metric?

A metric is a numerical measurement collected over time, such as request count, latency, CPU usage, or queue depth.

## Q2. What is the difference between a counter and a gauge?

A counter represents a monotonically increasing total. A gauge-like measurement represents a current value that can move up or down.

## Q3. Which instrument is suitable for request duration?

A Histogram.

## Q4. What does p95 mean?

Approximately 95% of observations are at or below the p95 value, and approximately 5% are above it.

## Q5. What is structured logging?

Logging data in machine-readable fields, commonly JSON, instead of only writing free-form text.

## Q6. Why are logs and traces correlated?

To connect a detailed log event to the exact request and span that produced it.

## Q7. What are RED metrics?

Rate, Errors, and Duration.

## Q8. What are USE metrics?

Utilization, Saturation, and Errors.

## Q9. Why should user IDs not be metric labels?

They create high cardinality and can increase storage and query costs dramatically.

## Q10. What does `rate()` do?

It calculates the average per-second increase of a counter over a selected time range.

---

# 32. L2 Interview Questions

## Q1. Why is a histogram better than an average for latency?

An average hides the distribution and tail latency. A histogram supports bucket-based analysis and percentile estimation.

## Q2. Explain cumulative vs delta temporality.

Cumulative values represent the total since the start of the metric stream. Delta values represent the change during a specific interval.

## Q3. What is cardinality?

Cardinality is the number of unique combinations of metric attribute values.

## Q4. Why is `/employees/{id}` better than `/employees/1024` as a metric route?

The route template is bounded. Individual IDs create a new time series for every employee.

## Q5. What metrics would you create for a notification worker?

- Jobs received.
- Jobs processed.
- Jobs failed.
- Processing duration.
- PDF generation duration.
- SMTP duration.
- Email success and failure counts.
- Pending queue depth.
- Oldest pending job age.
- Retry count.

## Q6. Why can CPU be normal while the service is failing?

The application may be blocked on a database, network, external API, queue, or configuration issue. CPU alone does not represent user experience.

## Q7. What is the difference between a log and a metric?

A log records detailed events. A metric provides numerical measurements that are aggregated over time.

## Q8. What is an actionable alert?

An alert that identifies a meaningful problem and gives the responsible engineer a clear next step.

## Q9. Why should alerts use a time window?

To avoid alerting on brief, harmless spikes.

## Q10. How do you calculate error percentage?

```text
Error rate =
failed requests / total requests × 100
```

In Prometheus, use rates over the same time range for numerator and denominator.

---

# 33. L3 Interview Questions

## Q1. How would you design metrics for a microservice platform?

Start with:

1. Resource attributes.
2. Standard HTTP/RPC/database instrumentation.
3. RED metrics per service.
4. Dependency metrics.
5. Business metrics.
6. Histograms for latency and sizes.
7. Bounded attributes.
8. Trace/log correlation.
9. Dashboards.
10. Actionable alerts.

## Q2. How can high cardinality affect a metrics backend?

High cardinality creates many unique time series. This increases:

- Memory usage.
- Storage requirements.
- Ingestion cost.
- Query latency.
- Compaction workload.
- Dashboard rendering time.

## Q3. Why should a metric backend calculate percentiles from histograms?

Because raw individual observations are expensive to store and query at scale. Histograms preserve an approximate distribution in an aggregatable form.

## Q4. What is the difference between p99 latency and the maximum latency?

p99 describes the value below which approximately 99% of observations fall. Maximum latency is the single highest observation and may be an outlier.

## Q5. How would you troubleshoot high notification queue age?

Check:

1. Queue depth.
2. Worker count.
3. Worker utilization.
4. Job processing duration.
5. Retry count.
6. PDF generation duration.
7. SMTP latency.
8. SMTP failures.
9. Elasticsearch read/index latency.
10. Database latency.
11. Worker logs and traces.
12. Oldest job age.

## Q6. How would you distinguish application errors from dependency errors?

Use separate metrics and span attributes for:

- Application validation failures.
- Database errors.
- Cache errors.
- Search/indexing errors.
- SMTP errors.
- Timeout errors.
- Retry and fallback behavior.

## Q7. How do you avoid exposing sensitive data in telemetry?

- Do not log secrets.
- Avoid raw request bodies.
- Redact authorization headers.
- Avoid personal data in metric labels.
- Use safe identifiers.
- Apply collector processors or backend filtering.
- Restrict telemetry access.
- Define retention policies.

## Q8. What is the relationship between SLI, SLO, and alerting?

The SLI is the measured signal. The SLO is the target. Alerts should detect meaningful SLO risk or error-budget consumption, not arbitrary infrastructure values.

## Q9. Why is queue age often better than queue depth?

Queue depth tells how many jobs are waiting. Queue age tells how long users may have been waiting. A small queue can still contain very old jobs.

## Q10. What would you investigate if p50 is normal but p99 is very high?

Investigate tail-specific causes:

- Slow database queries.
- Lock contention.
- Network retries.
- Garbage collection pauses.
- Cold starts.
- External service delays.
- Resource saturation.
- Large or unusual requests.
- Uneven load distribution.

---

# 34. Quick Revision

## Metrics

```text
Metric = numerical measurement over time
Counter = monotonically increasing total
UpDownCounter = value can increase/decrease
ObservableGauge = current observed value
Histogram = distribution of observations
```

## Percentiles

```text
p50 = median
p90 = 90% at or below
p95 = 95% at or below
p99 = 99% at or below
```

## Methods

```text
RED = Rate, Errors, Duration
USE = Utilization, Saturation, Errors
```

## Prometheus

```text
rate()     = average per-second increase
irate()    = recent per-second rate
increase() = total increase over range
```

## Logs

```text
Use structured fields.
Include severity.
Include trace_id and span_id.
Do not log secrets.
```

## Cardinality

```text
Use bounded labels.
Avoid user_id, employee_id, request_id, trace_id as metric labels.
```

## OT-Micro-Docker

```text
Measure both technical health and business success.
```

---

# 35. Self-Assessment

Try answering these without looking back.

1. Why is a Histogram appropriate for HTTP duration?
2. Why is a Counter inappropriate for active requests?
3. What is the difference between cumulative and delta temporality?
4. Explain p95 in your own words.
5. Why can average latency be misleading?
6. What is high cardinality?
7. Why is `http.route` better than the raw URL?
8. What is the difference between a log and a metric?
9. Why should logs contain `trace_id`?
10. What are the RED metrics?
11. What are the USE metrics?
12. What does `rate()` calculate?
13. What metrics would you create for an email worker?
14. Why is queue age useful?
15. What is the difference between SLI and SLO?
16. Why is CPU-only alerting insufficient?
17. How would you detect a ScyllaDB problem?
18. How would you detect an SMTP problem?
19. What sensitive data must never be logged?
20. How would you investigate high p99 but normal p50?

---

# 36. Final Checklist

Before considering your metrics and logs design complete, verify:

- [ ] Every service has a stable `service.name`.
- [ ] Metrics use meaningful names and units.
- [ ] Counters are used for cumulative totals.
- [ ] UpDownCounters are used for changing active values.
- [ ] Histograms are used for duration and size distributions.
- [ ] Latency dashboards show p50, p95, and p99.
- [ ] Metric attributes are bounded.
- [ ] Raw IDs are not used as metric labels.
- [ ] Logs are structured.
- [ ] Logs include severity.
- [ ] Logs include trace and span correlation where possible.
- [ ] Secrets and sensitive data are redacted.
- [ ] RED dashboards exist for APIs.
- [ ] USE dashboards exist for infrastructure.
- [ ] Business metrics exist for important workflows.
- [ ] Queue depth and queue age are measured.
- [ ] Dependency failures are visible separately.
- [ ] Alerts are actionable.
- [ ] Alerts use appropriate time windows.
- [ ] Dashboards prioritize user impact.
- [ ] SLI and SLO targets are documented.

---

## Final Takeaway

Metrics tell you **how much**, logs tell you **what happened**, and traces tell you **where the request went**.

For OT-Micro-Docker, the strongest observability design combines:

```text
RED metrics
+ Dependency metrics
+ Business metrics
+ Structured logs
+ Trace/log correlation
+ Histograms and percentiles
+ Actionable alerts
```

The goal is not to collect every possible measurement.

The goal is to collect the right signals so that when an employee cannot generate a salary slip, attendance processing is delayed, or notification emails are stuck, you can quickly answer:

```text
What is failing?
Who is affected?
Where is the failure?
Why did it happen?
What should we do next?
```
