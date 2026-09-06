# SQS

[⬅️ Back to AWS Topics](../README.md)

## 🔑 Keywords

📬 SQS | Queue | Producer | Consumer | Visibility Timeout | DLQ | Long Polling | FIFO | Deduplication

## 🧠 Core Memory

🧠 **Remember:** **SQS = queue / decouple**. Producer sends → Consumer receives → Consumer deletes.

---

## ❓ Interview Questions

### 📌 Core

#### Q1. What is Amazon SQS?

**💡 Answer:** Amazon SQS is a managed message queue used to decouple producers and consumers. It absorbs traffic spikes and lets consumers process work independently of producers.

**🔑 Keywords:** `SQS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q2. What is a queue?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `SQS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q3. What is a producer?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `SQS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q4. What is a consumer?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `SQS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q5. What is visibility timeout?

**💡 Answer:** Lambda timeout is the maximum execution duration for a single invocation. If the function exceeds it, Lambda terminates the invocation.

**🔑 Keywords:** `SQS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Queue Types

#### Q6. What is a Standard queue?

**💡 Answer:** An SQS Standard queue provides very high scalability and at-least-once delivery. Messages can occasionally be delivered more than once and ordering is not guaranteed.

**🔑 Keywords:** `SQS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q7. What is a FIFO queue?

**💡 Answer:** An SQS FIFO queue is designed for ordered processing and deduplication. Ordering is maintained within each message group and duplicate processing can be reduced through deduplication.

**🔑 Keywords:** `SQS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q8. How do Standard and FIFO queues differ?

**💡 Answer:** An SQS FIFO queue is designed for ordered processing and deduplication. Ordering is maintained within each message group and duplicate processing can be reduced through deduplication.

**🔑 Keywords:** `SQS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q9. What is message deduplication?

**💡 Answer:** FIFO SQS deduplication prevents a message with the same deduplication identity from being accepted as a new message during the deduplication interval.

**🔑 Keywords:** `SQS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q10. What is message group ID?

**💡 Answer:** Message Group ID in an SQS FIFO queue defines an ordered stream. Messages within the same group are processed in order while different groups can be processed concurrently.

**🔑 Keywords:** `SQS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Reliability

#### Q11. What is a dead-letter queue?

**💡 Answer:** A Lambda dead-letter destination or asynchronous failure destination can capture events that could not be processed successfully, depending on the invocation model.

**🔑 Keywords:** `SQS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q12. What is a redrive policy?

**💡 Answer:** An SQS redrive policy defines when messages are moved to a dead-letter queue after repeated receive attempts. A redrive allow policy can control which source queues may use a DLQ.

**🔑 Keywords:** `SQS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q13. What is long polling?

**💡 Answer:** SQS long polling waits for messages to become available before returning, reducing empty responses and unnecessary API calls. It is generally preferred for consumers.

**🔑 Keywords:** `SQS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q14. What is short polling?

**💡 Answer:** SQS short polling returns immediately based on the polling behavior, which can result in empty responses even when messages are available elsewhere. It can generate more API calls.

**🔑 Keywords:** `SQS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q15. What is retention period?

**💡 Answer:** SQS retains messages for a configurable period before deleting them automatically if they have not been successfully removed.

**🔑 Keywords:** `SQS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q16. What is maximum message size?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `SQS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Scaling

#### Q17. How can SQS decouple microservices?

**💡 Answer:** Amazon SQS is a managed message queue used to decouple producers and consumers. It absorbs traffic spikes and lets consumers process work independently of producers.

**🔑 Keywords:** `SQS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q18. How can ASG scale based on SQS queue depth?

**💡 Answer:** Amazon SQS is a managed message queue used to decouple producers and consumers. It absorbs traffic spikes and lets consumers process work independently of producers.

**🔑 Keywords:** `SQS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q19. How does Lambda consume SQS?

**💡 Answer:** AWS Lambda runs code without requiring you to provision or manage servers. AWS handles the underlying compute infrastructure and scales execution according to incoming requests/events.

**🔑 Keywords:** `SQS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q20. Why should consumers be idempotent?

**💡 Answer:** An idempotent consumer produces the same intended result even if the same message is processed more than once. This is important because Standard SQS provides at-least-once delivery.

**🔑 Keywords:** `SQS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

## 🚀 Last-Minute Revision

> 🧠 **Remember:** **SQS = queue / decouple**. Producer sends → Consumer receives → Consumer deletes.

[⬆️ Back to top](#sqs)

[⬅️ Back to AWS Topics](../README.md)