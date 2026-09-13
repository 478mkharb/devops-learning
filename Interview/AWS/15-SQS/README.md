# SQS

### Q1. What is Amazon SQS?

**Answer:** Amazon SQS is a managed message queue used to decouple producers and consumers. It absorbs traffic spikes and allows consumers to process work independently.

---

### Q2. What is a queue?

**Answer:** An SQS queue stores messages until consumers retrieve and successfully delete them or the retention period expires.

---

### Q3. What is a producer?

**Answer:** An SQS producer sends messages to a queue. Producers are decoupled from consumers and do not need to know when the work will be processed.

---

### Q4. What is a consumer?

**Answer:** An SQS consumer receives messages from a queue, processes them, and deletes them after successful processing.

---

### Q5. What is visibility timeout?

**Answer:** Visibility timeout temporarily hides a message after a consumer receives it so another consumer does not process it at the same time. The consumer should delete the message after successful processing.

---

### Q6. What is a Standard queue?

**Answer:** An SQS Standard queue provides very high scalability and at-least-once delivery. Messages can occasionally be delivered more than once and ordering is not guaranteed.

---

### Q7. What is a FIFO queue?

**Answer:** An SQS FIFO queue provides ordered processing and deduplication features. Ordering is maintained within each message group and duplicate processing can be reduced.

---

### Q8. How do Standard and FIFO queues differ?

**Answer:** Standard queues provide very high throughput and at-least-once delivery without guaranteed ordering. FIFO queues provide ordered processing within message groups and deduplication capabilities. Choose FIFO when ordering and duplicate-control requirements are part of the business logic.

---

### Q9. What is message deduplication?

**Answer:** FIFO SQS message deduplication prevents a message with the same deduplication identity from being treated as a new message during the deduplication interval.

---

### Q10. What is message group ID?

**Answer:** An SQS FIFO Message Group ID identifies an ordered stream. Messages in the same group are processed in order, while different groups can be processed concurrently.

---

### Q11. What is a dead-letter queue?

**Answer:** An SQS dead-letter queue stores messages that repeatedly fail processing after the configured receive-attempt threshold. It helps isolate poison messages for troubleshooting.

---

### Q12. What is a redrive policy?

**Answer:** An SQS redrive policy defines when messages are moved from a source queue to a dead-letter queue after repeated receives. A redrive allow policy can restrict which source queues may use a DLQ.

---

### Q13. What is long polling?

**Answer:** SQS long polling waits for messages to become available before returning a response, reducing empty responses and unnecessary API calls. It is generally preferred for consumers.

---

### Q14. What is short polling?

**Answer:** SQS short polling returns without waiting for messages to become available, which can produce empty responses and more API calls.

---

### Q15. What is retention period?

**Answer:** SQS message retention is the period for which messages remain in a queue if they are not deleted. After the retention period, SQS removes them automatically.

---

### Q16. What is maximum message size?

**Answer:** SQS supports messages up to 256 KB. Larger application payloads can be stored externally, such as in S3, with a reference placed in the queue message.

---

### Q17. How can SQS decouple microservices?

**Answer:** SQS decouples microservices by placing a durable queue between the producer and consumer. The producer can continue accepting work even when the consumer is busy or temporarily unavailable, while consumers process messages asynchronously and independently.

---

### Q18. How can ASG scale based on SQS queue depth?

**Answer:** Publish a CloudWatch metric representing queue depth or use an appropriate queue-based scaling metric, then configure an ASG scaling policy against it. As backlog increases, the policy can add instances; as backlog falls, it can reduce capacity within the ASG limits.

---

### Q19. How does Lambda consume SQS?

**Answer:** AWS Lambda is a serverless compute service that runs code in response to events without requiring the customer to manage servers. AWS provisions and scales the execution environment.

---

### Q20. Why should consumers be idempotent?

**Answer:** SQS processing can result in the same message being delivered more than once, especially with Standard queues or when processing succeeds but deletion does not. An idempotent consumer detects or safely tolerates duplicates so the business result is not applied twice.

---
