# AWS Lambda — Senior DevOps Notes

## 1. Lambda Fundamentals

| Concept | Explanation |
|---|---|
| **AWS Lambda** | Serverless compute service that runs code in response to events without customers managing servers. AWS manages execution environments and scales them according to demand. |
| **Lambda function** | Deployable unit containing code, runtime configuration, handler, execution role, memory, timeout, environment variables, and related settings. |
| **Runtime** | Language execution environment that loads the function and invokes its handler. |
| **Handler** | Runtime-specific entry point executed for each invocation. |
| **Execution role** | IAM role assumed by the Lambda service. Its permissions control which AWS resources the function can access. |

### Lambda execution flow

```text
Event source
    ↓
Lambda service
    ↓
Execution environment
    ↓
Runtime
    ↓
Handler
    ↓
Application logic
    ↓
AWS services / external systems
```

> The execution role controls what the function can do. The event source permissions control who or which service can invoke the function.

---

## 2. Lambda Configuration

| Setting | DevOps relevance |
|---|---|
| **Memory** | Controls available memory and influences CPU/network resources. More memory can reduce execution time but increases cost per unit of execution. |
| **Timeout** | Maximum duration of one invocation. Set it below the timeout of upstream services where possible and avoid using Lambda for long-running workloads. |
| **Environment variables** | Store non-secret configuration such as environment name, table name, or feature flags. Use Secrets Manager or Parameter Store for sensitive values. |
| **Ephemeral storage** | Temporary filesystem space available during execution, normally under `/tmp`. It is not durable storage and should not be treated as a database. |
| **Architecture** | Selects the CPU architecture, such as x86_64 or arm64. Dependencies and native binaries must support the selected architecture. |
| **Package type** | Lambda can be deployed as a ZIP package or a container image. Container images still run within Lambda's managed execution model. |
| **VPC configuration** | Allows access to private VPC resources but introduces networking considerations such as subnets, security groups, NAT, and VPC endpoints. |

### Important configuration principles

- Keep functions small and focused.
- Externalize configuration from code.
- Do not store persistent data in the execution environment.
- Select memory based on performance testing, not guesswork.
- Set timeout, retry, and concurrency values together.
- Pin dependency versions and build reproducible deployment artifacts.

---

## 3. Lambda Invocation Models

| Invocation model | Behavior | Typical sources | Retry behavior |
|---|---|---|---|
| **Synchronous** | Caller waits for the function response. | API Gateway, direct SDK invocation, ALB | Caller generally handles retries. |
| **Asynchronous** | Lambda queues the event and returns immediately to the caller. | S3, EventBridge, SNS | Lambda retries failed processing according to asynchronous retry settings. |
| **Poll-based** | Lambda polls a source and invokes the function with batches of records. | SQS, Kinesis, DynamoDB Streams | Controlled by event source mapping and source-specific behavior. |

### Invocation comparison

| Requirement | Suitable model |
|---|---|
| HTTP request requiring immediate response | Synchronous |
| Image upload processing | Asynchronous |
| Queue-based background worker | Poll-based with SQS |
| Stream processing | Poll-based with Kinesis or DynamoDB Streams |
| Scheduled task | EventBridge schedule invoking Lambda |

---

## 4. Concurrency and Scaling

**Concurrency** is the number of Lambda invocations running at the same time.

| Type | Purpose | Operational use |
|---|---|---|
| **Account concurrency** | Total concurrent executions available to an account or Region. | Protects the account from unlimited parallel execution. |
| **Reserved concurrency** | Sets the function's concurrency limit and reserves that capacity for the function. | Protects downstream databases and prevents one function from exhausting account concurrency. |
| **Provisioned concurrency** | Keeps a configured number of execution environments initialized. | Reduces cold-start latency for latency-sensitive workloads. |
| **Unreserved concurrency** | Capacity available to functions without reserved concurrency. | Shared by eligible functions. |

### Reserved vs provisioned concurrency

| Feature | Reserved concurrency | Provisioned concurrency |
|---|---|---|
| Main purpose | Limit and reserve concurrency | Reduce cold-start latency |
| Protects downstream systems | Yes | Not by itself |
| Keeps environments warm | No | Yes |
| Controls maximum concurrency | Yes | Used with other concurrency controls |
| Cost impact | No separate provisioned-environment charge | Additional provisioned concurrency cost |

### Senior design rule

For an SQS-triggered function that writes to a database:

```text
SQS batch size
    +
Event source maximum concurrency
    +
Lambda reserved concurrency
    +
Database connection capacity
```

These values must be designed together. Increasing Lambda concurrency without controlling database connections can overload the database.

---

## 5. Cold Starts and Performance

A **cold start** occurs when Lambda creates and initializes a new execution environment before running the handler.

```text
Create environment
    ↓
Load runtime
    ↓
Load dependencies
    ↓
Run initialization code
    ↓
Invoke handler
```

### Factors increasing cold-start latency

- Large deployment packages.
- Heavy dependency imports.
- Large container images.
- VPC networking initialization in applicable environments.
- Expensive initialization code.
- Runtime and architecture choice.

### Performance practices

- Keep deployment packages small.
- Initialize reusable clients outside the handler.
- Reuse database connections where safe.
- Avoid unnecessary work during module initialization.
- Use provisioned concurrency only for justified latency requirements.
- Measure `Init Duration`, duration, errors, throttles, and concurrency in CloudWatch.

---

## 6. Event Sources and Integrations

### API Gateway → Lambda

API Gateway exposes HTTP, REST, or WebSocket APIs and invokes Lambda for request processing.

```text
Client
  ↓
API Gateway
  ↓
Lambda
  ↓
Backend AWS service / database
```

Use this pattern for lightweight request/response APIs. Configure authentication, throttling, validation, timeouts, and observability at the API layer.

### S3 → Lambda

S3 can generate events for supported object operations such as object creation.

```text
Object uploaded
    ↓
S3 event
    ↓
Lambda
    ↓
Process object
```

Important controls:

- Restrict notifications to required prefixes and suffixes.
- Prevent recursive triggers when Lambda writes back to the same bucket.
- Validate object keys and event contents.
- Make processing idempotent.

### EventBridge → Lambda

An EventBridge rule invokes Lambda when an event matches an event pattern or schedule.

| EventBridge capability | DevOps use |
|---|---|
| Event pattern | React to deployment, infrastructure, or application events |
| Schedule | Run periodic jobs |
| Filtering | Send only relevant events to the target |
| Multiple targets | Fan out events to different processing paths |

### SQS → Lambda

Lambda uses an **event source mapping** to poll SQS and invoke the function with message batches.

```text
Producer
   ↓
SQS queue
   ↓
Lambda event source mapping
   ↓
Lambda worker
   ↓
Downstream service
```

Operational requirements:

- Set the SQS visibility timeout appropriately.
- Use a dead-letter queue for repeatedly failing messages.
- Make the handler idempotent because duplicate processing is possible.
- Tune batch size and maximum concurrency.
- Monitor queue depth, age of oldest message, errors, and throttles.

### Event source mapping

| Source | Processing model |
|---|---|
| **SQS** | Poll messages and invoke in batches |
| **Kinesis** | Poll stream records and process batches |
| **DynamoDB Streams** | Poll stream records and process changes |

---

## 7. Versions, Aliases, and Deployment Strategies

### Lambda versions

A published version is an immutable snapshot of function code and configuration. It supports repeatable releases and rollback.

### Lambda aliases

An alias is a stable name that points to a published version.

```text
production alias → version 12
staging alias    → version 13
```

Aliases are useful for CI/CD because applications can invoke a stable alias instead of referencing an unqualified `$LATEST` version.

### Deployment strategies

| Strategy | Description | Risk |
|---|---|---|
| **All-at-once** | Shift all traffic to the new version. | Highest release risk |
| **Canary** | Send a small percentage of traffic to the new version first. | Lower blast radius |
| **Linear** | Gradually increase traffic to the new version. | Controlled rollout |
| **Blue/green** | Keep old and new versions available and switch traffic between them. | Fast rollback when designed correctly |

### Recommended CI/CD flow

```text
Commit
  ↓
Build dependencies
  ↓
Run unit and security tests
  ↓
Package Lambda artifact
  ↓
Publish immutable version
  ↓
Update staging alias
  ↓
Run integration tests
  ↓
Shift production alias gradually
  ↓
Monitor metrics
  ↓
Rollback alias if required
```

> Never use `$LATEST` as the production release reference when controlled rollback is required.

---

## 8. Error Handling, Retries, and DLQs

### Asynchronous invocation

For supported asynchronous invocations, Lambda retries failed events according to its retry configuration. Failed events can be sent to a destination.

### Dead-letter destination

A dead-letter destination captures events that could not be processed successfully after the configured retry behavior.

| Mechanism | Purpose |
|---|---|
| **SQS DLQ** | Stores failed messages for later inspection or replay. |
| **SNS destination** | Publishes failure information to subscribers. |
| **Event destination** | Routes invocation success or failure records to supported targets. |

### Error-handling principles

- Log the event identifier, not sensitive payloads.
- Use structured JSON logs.
- Make retries safe through idempotency.
- Separate transient errors from permanent validation errors.
- Alert on DLQ depth and message age.
- Provide a controlled replay process.

---

## 9. Lambda Layers and Dependencies

A **Lambda layer** packages reusable libraries, dependencies, or files separately from function code.

| Use case | Recommendation |
|---|---|
| Shared internal library | Layer may be appropriate if release coupling is acceptable. |
| Large native dependency | Layer can simplify reuse across functions. |
| Frequently changing application dependency | Package with the function to avoid hidden coupling. |
| Strictly versioned runtime artifact | Use a pinned layer version or immutable deployment package. |

Avoid excessive layers. They can make dependency ownership, debugging, and release management harder.

---

## 10. Security and IAM

### Execution role vs invocation permission

| Permission type | Controls |
|---|---|
| **Execution-role policy** | What the Lambda function can access after it starts. |
| **Resource-based policy** | Which principals or services can invoke the function. |
| **KMS permissions** | Whether the function can use required encryption keys. |
| **VPC security groups/NACLs** | Network connectivity to private resources. |

### Security baseline

- Use a dedicated execution role per function or workload boundary.
- Apply least privilege.
- Avoid wildcard permissions where practical.
- Store secrets in Secrets Manager or Parameter Store.
- Encrypt environment variables with KMS when required.
- Restrict function URLs and public invocation.
- Use VPC endpoints where suitable to avoid unnecessary NAT dependency.
- Enable CloudTrail and CloudWatch logging.
- Scan dependencies and deployment packages for vulnerabilities.

---

## 11. Lambda in a VPC

A VPC-connected Lambda function can access private resources such as RDS, ElastiCache, or internal services.

```text
Lambda subnets
    ↓
Security group rules
    ↓
Private database / service
```

### Common requirements

| Requirement | Reason |
|---|---|
| Private subnets | Place Lambda network interfaces in appropriate subnets. |
| Security groups | Permit only required destination traffic. |
| NAT Gateway or VPC endpoints | Required for access to external services depending on the architecture. |
| DNS resolution | Required when using service hostnames. |
| Sufficient IP capacity | Lambda scaling can consume subnet IP addresses. |

### Common failure

A Lambda function in private subnets may reach an RDS instance but fail to call public AWS APIs if there is no NAT Gateway or suitable VPC endpoint.

---

## 12. Lambda@Edge

Lambda@Edge runs supported Lambda functions in association with CloudFront events at AWS edge locations.

Typical uses include:

- Viewer request customization.
- Origin request customization.
- Viewer response modification.
- Origin response modification.

Use Lambda@Edge only when edge execution is required. For simpler request manipulation, evaluate CloudFront Functions because they have a lighter execution model and different capabilities.

---

## 13. Observability and Operations

### Important CloudWatch metrics

| Metric | Meaning |
|---|---|
| **Invocations** | Number of function invocations. |
| **Errors** | Invocations that failed. |
| **Duration** | Execution time, including percentile analysis such as p95 or p99. |
| **Throttles** | Invocations rejected because concurrency limits were reached. |
| **ConcurrentExecutions** | Current concurrent execution count. |
| **IteratorAge** | Age of the oldest record for supported stream-based sources. |
| **DeadLetterErrors** | Errors while sending failed events to a dead-letter destination. |

### Operational checklist

- Monitor error rate and latency percentiles.
- Alert on throttles and sustained concurrency growth.
- Monitor SQS queue depth and oldest message age.
- Track cold-start and initialization duration.
- Correlate Lambda logs with request IDs and distributed tracing IDs.
- Set alarms for DLQ messages.
- Review cost by invocation count, duration, memory, and provisioned concurrency.

---

## 14. Common Failure Scenarios

| Symptom | Likely cause | Investigation |
|---|---|---|
| `AccessDeniedException` | Execution role lacks permission or KMS key policy denies access. | Check execution role, resource policy, SCP, permissions boundary, and KMS key policy. |
| Function times out | Slow dependency, blocked network, database connection issue, or insufficient memory. | Check logs, duration, VPC routes, security groups, and downstream latency. |
| High throttles | Reserved/account concurrency exhausted. | Inspect concurrency metrics and configured limits. |
| SQS messages repeatedly return | Handler failure, visibility timeout too short, or processing exceeds timeout. | Check errors, visibility timeout, batch size, and DLQ. |
| Lambda cannot access internet | Private subnet has no NAT or required VPC endpoint. | Check route tables, NAT, endpoints, and security groups. |
| High cold-start latency | Large package or expensive initialization. | Review initialization code, package size, runtime, and provisioned concurrency. |
| Duplicate processing | Retry or at-least-once delivery behavior. | Add idempotency keys and durable processing state. |
| Deployment works in staging but not production | Configuration, IAM, architecture, or dependency mismatch. | Compare immutable artifacts, environment variables, roles, and deployment settings. |

---

## 15. Practical AWS CLI Commands

### Inspect a function

```bash
aws lambda get-function \
  --function-name my-function
```

### List functions

```bash
aws lambda list-functions
```

### Invoke synchronously

```bash
aws lambda invoke \
  --function-name my-function \
  --payload '{"action":"health-check"}' \
  response.json
```

### Publish a version

```bash
aws lambda publish-version \
  --function-name my-function
```

### Update an alias

```bash
aws lambda update-alias \
  --function-name my-function \
  --name production \
  --function-version 12
```

### Configure reserved concurrency

```bash
aws lambda put-function-concurrency \
  --function-name my-function \
  --reserved-concurrent-executions 10
```

### Remove reserved concurrency

```bash
aws lambda delete-function-concurrency \
  --function-name my-function
```

### View recent logs

```bash
aws logs tail \
  /aws/lambda/my-function \
  --since 1h \
  --follow
```

---

## 16. Senior DevOps Interview Questions

### Q1. Why should Lambda functions be idempotent?

Because events can be retried or delivered more than once. Idempotent processing ensures repeated execution does not create duplicate side effects.

### Q2. How do you protect RDS from Lambda concurrency?

Use reserved concurrency, event-source maximum concurrency, controlled SQS batch size, connection reuse, and—where necessary—a connection pooler such as RDS Proxy.

### Q3. What is the difference between reserved and provisioned concurrency?

Reserved concurrency limits and reserves execution capacity. Provisioned concurrency keeps execution environments initialized to reduce cold-start latency.

### Q4. Why can a VPC Lambda lose internet access?

Because placing Lambda in private subnets does not automatically provide internet access. The function needs a valid NAT route or appropriate VPC endpoints.

### Q5. Why should production use published versions and aliases?

Published versions are immutable. Aliases provide stable release references, controlled traffic shifting, and fast rollback.

### Q6. How would you troubleshoot Lambda throttling?

Check account concurrency, reserved concurrency, provisioned concurrency, burst behavior, invocation rate, downstream limits, and CloudWatch `Throttles` and concurrency metrics.

### Q7. How would you process failed SQS messages safely?

Use retries, an appropriate visibility timeout, idempotent processing, a DLQ, structured logging, alerting, and a controlled replay mechanism.

### Q8. What is the difference between Lambda execution permissions and invocation permissions?

Execution permissions are granted through the function's IAM execution role. Invocation permissions are controlled through the caller's permissions and, where applicable, the Lambda resource-based policy.

### Q9. How would you reduce Lambda deployment risk?

Use immutable artifacts, automated tests, security scanning, published versions, aliases, canary or linear deployments, CloudWatch alarms, and automated rollback.

### Q10. When is Lambda not a good choice?

When the workload requires long-running processes, sustained high compute utilization, specialized operating-system control, persistent local state, or predictable workloads where another compute model is more economical.

---

## 17. One-Line Interview Answers

| Question | Short answer |
|---|---|
| What is Lambda? | Serverless, event-driven compute managed by AWS. |
| What is a handler? | The runtime-specific function entry point. |
| What is an execution role? | IAM role defining what the function can access. |
| What is concurrency? | Number of simultaneous running invocations. |
| What is reserved concurrency? | A function-level concurrency limit and reservation. |
| What is provisioned concurrency? | Pre-initialized execution environments for lower cold-start latency. |
| What is a cold start? | Initialization latency when a new execution environment is created. |
| What is an event source mapping? | Lambda-managed polling integration for sources such as SQS and streams. |
| What is a version? | Immutable published code and configuration snapshot. |
| What is an alias? | A stable pointer to a published version. |
| What is a DLQ? | A destination for events that repeatedly fail processing. |
| What is a Lambda layer? | Reusable packaged dependencies shared across functions. |
| What is Lambda@Edge? | Lambda execution associated with CloudFront events at edge locations. |

---

## Canonical Ownership

| Concept | Primary README |
|---|---|
| IAM users, roles, policies, trust policies, `iam:PassRole` | [08-IAM](../08-IAM/README.md) |
| S3 buckets, artifacts, lifecycle, event integration | [09-S3](../09-S3/README.md) |
| CloudWatch metrics, logs, alarms | [13-CloudWatch](../13-CloudWatch/README.md) |
| KMS keys and encryption policy | [16-KMS](../16-KMS/README.md) |
| API Gateway architecture | [18-API-Gateway](../18-API-Gateway/README.md) |
| CloudFront and edge delivery | [19-CloudFront](../19-CloudFront/README.md) |
| Terraform deployment and state | [22-Terraform-AWS-IaC](../22-Terraform-AWS-IaC/README.md) |
