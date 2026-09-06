# SQS

## Interview Questions & Answers

### Q1. What is Amazon SQS?

**Answer:** Amazon SQS is a managed message queue used to decouple producers and consumers. It absorbs traffic spikes and lets consumers process work independently of producers.

---

### Q2. What is a queue?

**Answer:** An SQS queue is a durable logical buffer that stores messages until consumers receive and delete them or they expire. It separates message producers from consumers and absorbs temporary traffic spikes.

---

### Q3. What is a producer?

**Answer:** A producer is an application or service that sends messages to an SQS queue. It does not need to wait for the consumer to process the work.

---

### Q4. What is a consumer?

**Answer:** A consumer is an application or service that receives and processes messages from an SQS queue. After successful processing, it should delete the message so it is not delivered again.

---

### Q5. What is visibility timeout?

**Answer:** Lambda timeout is the maximum execution duration for a single invocation. If the function exceeds it, Lambda terminates the invocation.

---

### Q6. What is a Standard queue?

**Answer:** An SQS Standard queue provides very high scalability and at-least-once delivery. Messages can occasionally be delivered more than once and ordering is not guaranteed.

---

### Q7. What is a FIFO queue?

**Answer:** An SQS FIFO queue is designed for ordered processing and deduplication. Ordering is maintained within each message group and duplicate processing can be reduced through deduplication.

---

### Q8. How do Standard and FIFO queues differ?

**Answer:** An SQS FIFO queue is designed for ordered processing and deduplication. Ordering is maintained within each message group and duplicate processing can be reduced through deduplication.

---

### Q9. What is message deduplication?

**Answer:** FIFO SQS deduplication prevents a message with the same deduplication identity from being accepted as a new message during the deduplication interval.

---

### Q10. What is message group ID?

**Answer:** Message Group ID in an SQS FIFO queue defines an ordered stream. Messages within the same group are processed in order while different groups can be processed concurrently.

---

### Q11. What is a dead-letter queue?

**Answer:** A Lambda dead-letter destination or asynchronous failure destination can capture events that could not be processed successfully, depending on the invocation model.

---

### Q12. What is a redrive policy?

**Answer:** An SQS redrive policy defines when messages are moved to a dead-letter queue after repeated receive attempts. A redrive allow policy can control which source queues may use a DLQ.

---

### Q13. What is long polling?

**Answer:** SQS long polling waits for messages to become available before returning, reducing empty responses and unnecessary API calls. It is generally preferred for consumers.

---

### Q14. What is short polling?

**Answer:** SQS short polling returns immediately based on the polling behavior, which can result in empty responses even when messages are available elsewhere. It can generate more API calls.

---

### Q15. What is retention period?

**Answer:** SQS retains messages for a configurable period before deleting them automatically if they have not been successfully removed.

---

### Q16. What is maximum message size?

**Answer:** The maximum SQS message size is 256 KB. For larger payloads, a common pattern is to store the payload in S3 and put an object reference or pointer in the SQS message.

---

### Q17. How can SQS decouple microservices?

**Answer:** Amazon SQS is a managed message queue used to decouple producers and consumers. It absorbs traffic spikes and lets consumers process work independently of producers.

---

### Q18. How can ASG scale based on SQS queue depth?

**Answer:** Amazon SQS is a managed message queue used to decouple producers and consumers. It absorbs traffic spikes and lets consumers process work independently of producers.

---

### Q19. How does Lambda consume SQS?

**Answer:** AWS Lambda runs code without requiring you to provision or manage servers. AWS handles the underlying compute infrastructure and scales execution according to incoming requests/events.

---

### Q20. Why should consumers be idempotent?

**Answer:** An idempotent consumer produces the same intended result even if the same message is processed more than once. This is important because Standard SQS provides at-least-once delivery.

---
