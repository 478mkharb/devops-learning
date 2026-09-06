# SNS

## Interview Questions & Answers

### Q1. What is Amazon SNS?

**Answer:** Amazon SNS is a managed publish/subscribe messaging service. Publishers send messages to topics, which fan them out to subscribers such as SQS queues, Lambda functions, HTTP endpoints, and notification destinations.

---

### Q2. What is a topic?

**Answer:** An SNS topic is a logical communication channel to which publishers send messages and subscribers attach endpoints.

---

### Q3. What is a subscription?

**Answer:** An SNS subscription connects a subscriber endpoint to a topic and can optionally use filtering rules to receive only matching messages.

---

### Q4. What is a publisher?

**Answer:** A publisher is an application or AWS service that sends a message to an SNS topic. The publisher targets the topic rather than needing direct knowledge of every subscriber.

---

### Q5. What is a subscriber?

**Answer:** A subscriber is an endpoint registered with an SNS topic to receive published messages. Examples include SQS queues, Lambda functions, HTTP/S endpoints, and supported notification destinations.

---

### Q6. What is pub/sub?

**Answer:** Publish/subscribe separates publishers from subscribers. Publishers send messages to a topic, and the messaging service distributes them to subscribed consumers without the publisher needing to know each consumer.

---

### Q7. How does SNS fan out to SQS?

**Answer:** Amazon SNS is a managed publish/subscribe messaging service. Publishers send messages to topics, which fan them out to subscribers such as SQS queues, Lambda functions, HTTP endpoints, and notification destinations.

---

### Q8. How can SNS invoke Lambda?

**Answer:** AWS Lambda runs code without requiring you to provision or manage servers. AWS handles the underlying compute infrastructure and scales execution according to incoming requests/events.

---

### Q9. How can SNS send notifications to HTTP endpoints?

**Answer:** Amazon SNS is a managed publish/subscribe messaging service. Publishers send messages to topics, which fan them out to subscribers such as SQS queues, Lambda functions, HTTP endpoints, and notification destinations.

---

### Q10. When should SNS be used instead of SQS?

**Answer:** Amazon SNS is a managed publish/subscribe messaging service. Publishers send messages to topics, which fan them out to subscribers such as SQS queues, Lambda functions, HTTP endpoints, and notification destinations.

---

### Q11. What is message filtering?

**Answer:** SNS subscription filtering lets subscribers receive only messages whose attributes or payload values match configured filter policies. It reduces unnecessary downstream processing.

---

### Q12. What is a FIFO SNS topic?

**Answer:** Amazon SNS is a managed publish/subscribe messaging service. Publishers send messages to topics, which fan them out to subscribers such as SQS queues, Lambda functions, HTTP endpoints, and notification destinations.

---

### Q13. What is message ordering?

**Answer:** Message ordering means delivering messages in a defined sequence. SNS FIFO topics support ordered delivery for compatible FIFO subscriptions, with message groups defining independent ordered streams; standard SNS is not intended to provide strict ordering.

---

### Q14. What is SNS encryption?

**Answer:** Amazon SNS is a managed publish/subscribe messaging service. Publishers send messages to topics, which fan them out to subscribers such as SQS queues, Lambda functions, HTTP endpoints, and notification destinations.

---

### Q15. What is SNS delivery retry behavior?

**Answer:** Amazon SNS is a managed publish/subscribe messaging service. Publishers send messages to topics, which fan them out to subscribers such as SQS queues, Lambda functions, HTTP endpoints, and notification destinations.

---
