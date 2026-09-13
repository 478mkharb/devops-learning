# SNS

### Q1. What is Amazon SNS?

**Answer:** Amazon SNS is a managed publish/subscribe messaging service. Publishers send messages to topics, and SNS delivers them to subscribed endpoints such as SQS, Lambda, HTTP endpoints, and notification destinations.

---

### Q2. What is a topic?

**Answer:** An SNS topic is a logical communication channel. Publishers send messages to the topic and SNS delivers them to its subscriptions.

---

### Q3. What is a subscription?

**Answer:** An SNS subscription connects an endpoint to a topic. A subscription can also include filtering rules so the endpoint receives only matching messages.

---

### Q4. What is a publisher?

**Answer:** An SNS publisher is a producer that sends messages to an SNS topic. The publisher does not need to know the individual subscribers.

---

### Q5. What is a subscriber?

**Answer:** An SNS subscriber is an endpoint that receives messages from a topic, such as an SQS queue, Lambda function, HTTP endpoint, or supported notification destination.

---

### Q6. What is pub/sub?

**Answer:** Publish/subscribe decouples publishers from subscribers. The publisher sends to a topic, and the messaging service distributes the message to interested subscribers.

---

### Q7. How does SNS fan out to SQS?

**Answer:** Create an SNS topic and subscribe multiple SQS queues to it. When a publisher sends one message to the topic, SNS delivers a copy to each subscribed queue, allowing different consumers to process the event independently.

---

### Q8. How can SNS invoke Lambda?

**Answer:** AWS Lambda is a serverless compute service that runs code in response to events without requiring the customer to manage servers. AWS provisions and scales the execution environment.

---

### Q9. How can SNS send notifications to HTTP endpoints?

**Answer:** An SNS topic can have an HTTP or HTTPS endpoint subscription. SNS sends published messages to the subscribed endpoint according to the protocol's delivery and retry behavior, and the endpoint must confirm the subscription where required.

---

### Q10. When should SNS be used instead of SQS?

**Answer:** Use SNS when you need one-to-many pub/sub notification or fan-out to multiple subscribers. Use SQS when you need a durable queue that consumers pull from and process independently. SNS and SQS are often combined: SNS distributes an event, and each SQS queue buffers it for a consumer.

---

### Q11. What is message filtering?

**Answer:** SNS message filtering lets a subscription receive only messages whose attributes or payload values match its filter policy. It reduces unnecessary downstream processing.

---

### Q12. What is a FIFO SNS topic?

**Answer:** An SNS FIFO topic supports ordered message delivery and deduplication for compatible FIFO subscriptions. Message groups preserve ordering within each group.

---

### Q13. What is message ordering?

**Answer:** Message ordering means consumers receive messages in a defined sequence. SNS FIFO and SQS FIFO use message groups to preserve order within a group.

---

### Q14. What is SNS encryption?

**Answer:** SNS supports server-side encryption using AWS KMS keys. Encryption protects message data at rest while IAM and KMS policies control key use.

---

### Q15. What is SNS delivery retry behavior?

**Answer:** SNS retries delivery to supported endpoints according to the delivery protocol and configured retry behavior. If delivery continues to fail, the endpoint can use a dead-letter queue where supported.

---
