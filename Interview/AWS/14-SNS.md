# Amazon SNS — Senior DevOps Notes

## 1. What is Amazon SNS?

Amazon Simple Notification Service (SNS) is a managed **publish/subscribe messaging service**. Producers publish messages to topics, and SNS delivers them to subscribed endpoints such as SQS queues, Lambda functions, HTTP/HTTPS endpoints, and notification destinations.

SNS is primarily used for **fan-out, event distribution, and notifications**.

## 2. Core SNS Components

| Component | Responsibility |
|---|---|
| **Topic** | Logical communication channel to which messages are published |
| **Publisher** | Application or AWS service that sends messages to a topic |
| **Subscription** | Connection between a topic and an endpoint |
| **Subscriber/Endpoint** | Destination receiving the message, such as SQS, Lambda, or HTTPS |
| **Message attributes** | Metadata used for filtering and routing |
| **Filter policy** | Rules that determine which messages a subscription receives |
| **Delivery policy** | Controls delivery behavior for supported HTTP/S endpoints |
| **Dead-letter queue** | Captures undeliverable messages for supported subscription scenarios |

## 3. SNS Pub/Sub Model

```text
Publisher
   |
   v
SNS Topic
   |
   +----> SQS Queue A ----> Consumer A
   |
   +----> SQS Queue B ----> Consumer B
   |
   +----> Lambda Function
   |
   +----> HTTPS Endpoint
```

The publisher does not need to know the subscribers. This reduces coupling between services and allows consumers to evolve independently.

## 4. SNS vs SQS

| Feature | SNS | SQS |
|---|---|---|
| Messaging model | Push-based pub/sub | Pull-based queue |
| Main purpose | Fan-out and notification | Durable asynchronous processing |
| Consumers | Multiple subscribers | Consumers read from a queue |
| Message storage | Not a general-purpose consumer queue | Designed for message buffering |
| Replay capability | Limited compared with a queue/stream | Messages remain until consumed or expired |
| Typical use | Broadcast an event | Process a task reliably |
| Best pattern | Distribute an event | Buffer work for a consumer |

### Common production pattern

```text
Application
   |
   v
SNS Topic
   |
   +----> SQS Queue ----> Worker A
   |
   +----> SQS Queue ----> Worker B
   |
   +----> Lambda
```

Use SNS to distribute the event and SQS to provide buffering, retry handling, back-pressure, and independent consumer processing.

## 5. SNS Fan-Out to SQS

SNS fan-out means one published message is delivered to multiple subscribed queues.

### Benefits

- Decouples producers from consumers.
- Allows independent scaling of consumers.
- Prevents one slow consumer from blocking others.
- Supports different retry and retention policies per consumer.
- Enables separate processing pipelines for the same event.

### Important requirement

The SNS topic and SQS queue policies must allow the required service-to-service delivery. A subscription alone is not always sufficient if resource policies deny delivery.

## 6. SNS Integration with AWS Services

| Integration | How it works | Typical use |
|---|---|---|
| **SQS** | SNS pushes a copy of the message to each subscribed queue | Durable fan-out and worker processing |
| **Lambda** | SNS invokes the subscribed function | Lightweight event processing |
| **HTTP/HTTPS** | SNS sends the message to a subscribed endpoint | Webhooks and external integrations |
| **Email/SMS** | SNS delivers supported notification messages | Operational alerts and user notifications |
| **EventBridge** | Usually used alongside SNS for broader event routing | Event-driven integration and rule-based routing |

## 7. SNS Subscription Filtering

Message filtering allows a subscription to receive only messages matching a filter policy.

### Example message attributes

```text
EventType=EmployeeCreated
Environment=prod
Department=finance
```

### Example use case

| Subscription | Filter | Result |
|---|---|---|
| Finance queue | `Department=finance` | Receives finance events only |
| Audit queue | `Environment=prod` | Receives production events only |
| Notification Lambda | `EventType=EmployeeCreated` | Processes employee creation events only |

Filtering reduces unnecessary downstream processing and avoids creating separate topics for every small routing variation.

## 8. Standard vs FIFO SNS Topics

| Feature | Standard topic | FIFO topic |
|---|---|---|
| Throughput | Very high | Ordered and controlled throughput model |
| Ordering | Not guaranteed | Preserved within a message group |
| Deduplication | Not provided as FIFO semantics | Supported through FIFO deduplication behavior |
| Typical use | Notifications and event fan-out | Ordered workflows and duplicate-sensitive processing |
| Compatible consumers | Standard-compatible endpoints | FIFO-compatible subscriptions such as SQS FIFO |

### Message ordering

Ordering is generally guaranteed **within the same message group**, not globally across all messages. Different message groups may be processed independently.

## 9. SNS Message Delivery and Retries

SNS delivery behavior depends on the subscription protocol.

| Endpoint | Delivery model | Operational concern |
|---|---|---|
| SQS | SNS sends the message to the queue | Queue policy and consumer retry behavior |
| Lambda | SNS invokes the function | Function errors, retry behavior, and async failure handling |
| HTTP/HTTPS | SNS sends an HTTP request | Endpoint availability, response codes, retry policy |
| Email/SMS | Notification delivery | Provider and destination delivery limitations |

For HTTP/S endpoints, configure an appropriate delivery policy and monitor failed deliveries. For durable processing, prefer **SNS → SQS** rather than directly invoking an unreliable external endpoint.

## 10. Dead-Letter Queues

A dead-letter queue captures messages that cannot be delivered successfully after the supported retry behavior.

### Why use a DLQ?

- Prevents repeatedly failing messages from being lost silently.
- Supports troubleshooting and replay workflows.
- Separates poison messages from healthy traffic.
- Provides operational visibility into delivery failures.

A DLQ is not a replacement for application-level idempotency or proper retry handling.

## 11. SNS Security

### Security controls

| Control | Purpose |
|---|---|
| IAM identity policies | Control who can publish, subscribe, or manage topics |
| SNS topic policy | Control access to the topic as a resource |
| KMS encryption | Encrypt message data at rest |
| TLS/HTTPS | Protect data in transit to HTTPS endpoints |
| Condition keys | Restrict access by source, account, VPC endpoint, or other conditions |
| CloudTrail | Audit SNS API activity |
| Least privilege | Limit publishing and subscription permissions |

### Typical publisher permissions

```json
{
  "Effect": "Allow",
  "Action": "sns:Publish",
  "Resource": "arn:aws:sns:us-east-1:123456789012:dev-otms-events"
}
```

The publisher normally needs `sns:Publish`; it does not need permissions to manage subscriptions or the topic itself.

## 12. SNS Encryption with KMS

SNS supports server-side encryption using AWS KMS.

| Area | Responsibility |
|---|---|
| SNS | Encrypts and decrypts message data at rest |
| IAM policy | Allows the principal to use SNS and, where required, KMS actions |
| KMS key policy | Allows the required AWS service and principals to use the key |
| CloudTrail | Records relevant KMS key usage and API activity |

Use a customer-managed KMS key when you need stronger control over key administration, access, rotation, or auditing.

## 13. SNS and Lambda: Operational Considerations

When SNS invokes Lambda:

1. SNS publishes the event to the function.
2. Lambda receives the event envelope containing SNS records.
3. The function processes each record.
4. Failures must be handled through retry, logging, alerting, and suitable failure destinations.

### Production recommendations

- Make the function idempotent.
- Validate the message schema.
- Use structured logging.
- Set alarms for invocation errors and throttles.
- Avoid placing long-running work directly in the function.
- Use SQS when buffering and controlled retry are required.

## 14. SNS and HTTP/HTTPS Endpoints

An HTTP/HTTPS subscription is useful for webhooks and external integrations.

### Important considerations

- HTTPS should be preferred over HTTP.
- Subscription confirmation may be required.
- The endpoint must return an appropriate success response.
- Configure retry behavior for transient failures.
- Validate SNS message authenticity where required.
- Protect the endpoint against replay and duplicate delivery.
- Use a DLQ or durable intermediary when the endpoint cannot guarantee availability.

## 15. SNS Delivery Is Not Exactly-Once Processing

SNS-based consumers should generally assume that messages may be delivered more than once depending on the integration and failure scenario.

### Idempotency pattern

```text
Receive event
   |
   v
Extract event ID
   |
   v
Check processed-event store
   |
   +---- Already processed ----> Return success
   |
   +---- New event ------------> Process and record event ID
```

Use a unique event ID, business key, or deduplication record to prevent duplicate side effects.

## 16. SNS Naming and Environment Strategy

Use environment-aware names and tags.

```text
dev-otms-events
stage-otms-events
prod-otms-events
```

Recommended tags:

| Tag | Example |
|---|---|
| `Environment` | `dev` |
| `Application` | `otms` |
| `Owner` | `Infra-Titans` |
| `CostCenter` | `Snaatak` |
| `ManagedBy` | `Terraform` |

Avoid sharing one production topic with development workloads unless the routing and security model explicitly requires it.

## 17. SNS in CI/CD and DevOps Architectures

SNS is useful for publishing deployment and operational events.

```text
Jenkins / CodePipeline
          |
          v
      SNS Topic
          |
          +----> Slack/Webhook notification
          |
          +----> SQS audit queue
          |
          +----> Lambda deployment reporter
```

Example events:

- Deployment started.
- Deployment succeeded.
- Deployment failed.
- AMI build completed.
- Security scan failed.
- Production alarm triggered.

Do not use SNS alone as the system of record for deployment history. Store authoritative deployment state in the CI/CD platform, database, or durable artifact store.

## 18. Monitoring and Alerting

Monitor SNS using CloudWatch metrics and application logs.

| Signal | What it may indicate |
|---|---|
| Number of messages published | Producer traffic volume |
| Number of notifications delivered | Delivery success volume |
| Number of notifications failed | Endpoint or permission problems |
| Number of messages filtered | Filter policy behavior |
| Lambda invocation errors | Consumer processing failures |
| SQS queue depth | Downstream backlog after fan-out |
| DLQ message count | Persistent delivery or processing failures |

### Recommended alarms

- SNS delivery failures above threshold.
- DLQ message count greater than zero.
- SQS queue age increasing.
- Lambda errors or throttles.
- Sudden drop in published messages.
- Unexpected increase in message volume.

## 19. Common Failure Scenarios

| Symptom | Likely causes | Checks |
|---|---|---|
| Message is not published | Missing `sns:Publish`, wrong ARN, explicit deny | `sts get-caller-identity`, IAM policy, topic ARN |
| SQS receives nothing | Missing queue policy, subscription not confirmed, filter mismatch | SNS subscription, SQS policy, filter policy |
| Lambda is not invoked | Missing invoke permission, function error, throttling | Lambda resource policy, CloudWatch logs, metrics |
| HTTP endpoint receives retries | Non-2xx response, timeout, endpoint unavailable | Endpoint logs, SNS delivery status, retry policy |
| Messages are unexpectedly missing | Filter policy excludes them | Message attributes and subscription filter |
| Duplicate processing | Retries or consumer failure after side effects | Idempotency implementation |
| KMS access denied | Missing key policy or KMS permissions | KMS key policy, IAM policy, CloudTrail |
| DLQ is growing | Persistent delivery or processing failures | DLQ messages, endpoint health, permissions |

## 20. Useful AWS CLI Commands

### Create a topic

```bash
aws sns create-topic \
  --name dev-otms-events \
  --region us-east-1
```

### List topics

```bash
aws sns list-topics \
  --region us-east-1
```

### Publish a message

```bash
aws sns publish \
  --topic-arn arn:aws:sns:us-east-1:123456789012:dev-otms-events \
  --message '{"event":"EmployeeCreated","employee_id":"E1001"}' \
  --region us-east-1
```

### Publish with message attributes

```bash
aws sns publish \
  --topic-arn arn:aws:sns:us-east-1:123456789012:dev-otms-events \
  --message 'Employee created' \
  --message-attributes '{
    "EventType":{"DataType":"String","StringValue":"EmployeeCreated"},
    "Environment":{"DataType":"String","StringValue":"prod"}
  }' \
  --region us-east-1
```

### Subscribe an SQS queue

```bash
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:dev-otms-events \
  --protocol sqs \
  --notification-endpoint arn:aws:sqs:us-east-1:123456789012:dev-otms-worker \
  --region us-east-1
```

### List subscriptions

```bash
aws sns list-subscriptions-by-topic \
  --topic-arn arn:aws:sns:us-east-1:123456789012:dev-otms-events \
  --region us-east-1
```

### Set a subscription filter policy

```bash
aws sns set-subscription-attributes \
  --subscription-arn <subscription-arn> \
  --attribute-name FilterPolicy \
  --attribute-value '{"EventType":["EmployeeCreated"]}' \
  --region us-east-1
```

### Get topic attributes

```bash
aws sns get-topic-attributes \
  --topic-arn arn:aws:sns:us-east-1:123456789012:dev-otms-events \
  --region us-east-1
```

## 21. Senior DevOps Interview Questions

### Q1. Why use SNS and SQS together?

SNS provides fan-out; SQS provides durable buffering, controlled consumption, retries, and back-pressure. This combination decouples producers from independent consumers.

### Q2. Why should a production consumer be idempotent?

Because messaging systems and integrations may retry delivery. Idempotency prevents duplicate side effects when the same event is processed more than once.

### Q3. What is the difference between SNS standard and FIFO topics?

Standard topics prioritize high throughput without ordering guarantees. FIFO topics support ordering and deduplication semantics, generally within message groups and compatible FIFO subscriptions.

### Q4. Why is an SNS subscription receiving no messages?

Check subscription confirmation, topic and endpoint policies, filter policies, endpoint permissions, region/account values, and CloudWatch delivery metrics.

### Q5. What is the role of a subscription filter policy?

It performs server-side filtering so a subscriber receives only matching messages, reducing unnecessary processing and traffic.

### Q6. How do you troubleshoot SNS to SQS delivery?

Verify the subscription, confirm the SQS queue policy allows SNS, check filter policies, publish a test message, inspect queue metrics, and review CloudTrail or service metrics where necessary.

### Q7. What is the difference between an SNS topic policy and an IAM identity policy?

An IAM identity policy grants permissions to a principal. An SNS topic policy is a resource-based policy attached to the topic and can control which principals or AWS accounts may access it.

### Q8. How would you design reliable external webhook delivery?

Publish to SNS, subscribe an SQS queue, process through a worker, implement retries and idempotency, monitor failures, and use a DLQ for messages requiring investigation.

### Q9. How can SNS be secured?

Use least-privilege IAM, restrictive topic policies, HTTPS endpoints, KMS encryption where required, condition keys, CloudTrail auditing, and environment-specific topics.

### Q10. What should be monitored in an SNS-based architecture?

Published messages, delivered messages, failed deliveries, filtered messages, consumer errors, queue depth, message age, and DLQ count.

## 22. One-Line Interview Answers

| Question | One-line answer |
|---|---|
| What is SNS? | A managed pub/sub service used for fan-out and notifications. |
| What is a topic? | A logical channel to which publishers send messages. |
| What is a subscription? | A connection between an SNS topic and an endpoint. |
| What is fan-out? | Delivering one published message to multiple subscribers. |
| SNS vs SQS? | SNS distributes messages; SQS buffers messages for consumers. |
| What is filtering? | Delivering messages only to subscriptions whose policies match. |
| What is SNS FIFO? | A topic supporting ordering and deduplication semantics. |
| Why use a DLQ? | To isolate messages that repeatedly fail delivery or processing. |
| Why use SNS with SQS? | To combine fan-out with durable asynchronous processing. |
| Why is idempotency important? | It prevents duplicate side effects during retries or duplicate delivery. |

## 23. Ownership Boundaries

| Concept | Canonical README |
|---|---|
| IAM users, roles, policies, trust policies | `08-IAM/README.md` |
| SQS queues, visibility timeout, DLQ processing | `12-SQS/README.md` or messaging architecture section |
| Lambda execution and retries | `10-Lambda/README.md` |
| CloudWatch alarms and logs | `13-CloudWatch/README.md` |
| KMS keys and key policies | `16-KMS/README.md` |
| Terraform topic and subscription resources | `22-Terraform-AWS-IaC/README.md` |
