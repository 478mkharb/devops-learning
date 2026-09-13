# Lambda

### Q1. What is AWS Lambda?

**Answer:** AWS Lambda is a serverless compute service that runs code in response to events without requiring the customer to manage servers. AWS provisions and scales the execution environment.

---

### Q2. What is a Lambda function?

**Answer:** A Lambda function is the unit of code and configuration that Lambda invokes. It includes the function code, runtime, handler, execution role, memory, timeout, and related settings.

---

### Q3. What is a Lambda execution role?

**Answer:** A Lambda execution role is an IAM role assumed by the Lambda service while running a function. Its permissions determine which AWS resources the function can access.

---

### Q4. What is a Lambda runtime?

**Answer:** A Lambda runtime provides the language execution environment and invokes the function handler. AWS provides managed runtimes for supported languages and versions.

---

### Q5. What is the handler?

**Answer:** The Lambda handler is the function entry point that Lambda invokes for an event. Its format depends on the selected runtime and programming language.

---

### Q6. What is Lambda concurrency?

**Answer:** Lambda concurrency is the number of function invocations executing simultaneously. It determines how much parallel work a function can process.

---

### Q7. What is reserved concurrency?

**Answer:** Reserved concurrency sets a maximum concurrency for a function and reserves that amount of account concurrency for it. It can protect downstream systems and prevent one function from consuming all available concurrency.

---

### Q8. What is provisioned concurrency?

**Answer:** Provisioned concurrency keeps a configured number of execution environments initialized so requests can avoid most cold-start initialization latency.

---

### Q9. What is a cold start?

**Answer:** A Lambda cold start occurs when Lambda creates and initializes a new execution environment before invoking the function. Runtime and application initialization add latency to that invocation.

---

### Q10. What is Lambda timeout?

**Answer:** Lambda timeout is the maximum execution time allowed for one invocation. If the function exceeds the configured timeout, Lambda stops the invocation.

---

### Q11. What is memory allocation?

**Answer:** Lambda memory allocation determines the memory available to the function and also influences the CPU and other resources allocated to the execution environment. Increasing memory can therefore improve execution speed.

---

### Q12. How can API Gateway invoke Lambda?

**Answer:** Amazon API Gateway is a managed service for creating, publishing, securing, throttling, monitoring, and operating APIs. It supports REST, HTTP, and WebSocket APIs.

---

### Q13. How can S3 invoke Lambda?

**Answer:** S3 can invoke Lambda in response to supported object events such as object creation. The S3 event identifies the bucket and object, and the Lambda execution role provides permissions for any AWS resources the function needs to access.

---

### Q14. How can EventBridge invoke Lambda?

**Answer:** An EventBridge rule can use a Lambda function as its target. When an event matches the rule's event pattern or schedule, EventBridge invokes the function and passes the event payload to it.

---

### Q15. How can SQS invoke Lambda?

**Answer:** Lambda uses an event source mapping to poll an SQS queue and invoke the function with batches of messages. Successful processing causes messages to be deleted according to the integration behavior; failed processing can lead to retries and eventually a DLQ when configured.

---

### Q16. What is an event source mapping?

**Answer:** An event source mapping connects Lambda to poll-based sources such as SQS, Kinesis, and DynamoDB Streams. Lambda polls the source and invokes the function with batches of records.

---

### Q17. What is Lambda versioning?

**Answer:** A published Lambda version is an immutable snapshot of function code and configuration. Versions allow controlled releases and rollback.

---

### Q18. What is an alias?

**Answer:** A Lambda alias is a named pointer to a published Lambda function version. It lets applications use a stable name such as production while the underlying version changes, and it can support controlled traffic shifting between versions.

---

### Q19. What is a dead-letter destination?

**Answer:** For supported asynchronous Lambda invocations, a dead-letter destination can capture events that could not be successfully processed after the configured retry behavior.

---

### Q20. What is Lambda layers?

**Answer:** A Lambda layer packages reusable libraries, dependencies, or other files separately from function code. Multiple functions can use the same layer.

---

### Q21. How do environment variables work?

**Answer:** Lambda environment variables provide configuration values to the function without hardcoding them in source code. Sensitive values should generally be stored in Secrets Manager or Parameter Store rather than exposed as plain configuration.

---

### Q22. What is Lambda@Edge?

**Answer:** Lambda@Edge runs supported Lambda functions in association with CloudFront events at AWS edge locations. It is used to customize viewer or origin requests and responses close to users.

---
