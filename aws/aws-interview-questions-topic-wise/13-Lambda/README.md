# Lambda

[⬅️ Back to AWS Topics](../README.md)

## 🔑 Keywords

λ Lambda | Runtime | Handler | Execution Role | Concurrency | Cold Start | Timeout | Version | Alias | Layer

## 🧠 Core Memory

🧠 **Remember:** **Event → Lambda → Execution Role → Code → Response**. Concurrency controls parallel executions.

---

## ❓ Interview Questions

### 📌 Core

#### Q1. What is AWS Lambda?

**💡 Answer:** AWS Lambda runs code without requiring you to provision or manage servers. AWS handles the underlying compute infrastructure and scales execution according to incoming requests/events.

**🔑 Keywords:** `Lambda` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q2. What is a Lambda function?

**💡 Answer:** AWS Lambda runs code without requiring you to provision or manage servers. AWS handles the underlying compute infrastructure and scales execution according to incoming requests/events.

**🔑 Keywords:** `Lambda` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q3. What is a Lambda execution role?

**💡 Answer:** AWS Lambda runs code without requiring you to provision or manage servers. AWS handles the underlying compute infrastructure and scales execution according to incoming requests/events.

**🔑 Keywords:** `Lambda` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q4. What is a Lambda runtime?

**💡 Answer:** AWS Lambda runs code without requiring you to provision or manage servers. AWS handles the underlying compute infrastructure and scales execution according to incoming requests/events.

**🔑 Keywords:** `Lambda` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q5. What is the handler?

**💡 Answer:** The Lambda handler is the function entry point that Lambda invokes for an event. Its exact form depends on the runtime and programming language.

**🔑 Keywords:** `Lambda` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Execution

#### Q6. What is Lambda concurrency?

**💡 Answer:** AWS Lambda runs code without requiring you to provision or manage servers. AWS handles the underlying compute infrastructure and scales execution according to incoming requests/events.

**🔑 Keywords:** `Lambda` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q7. What is reserved concurrency?

**💡 Answer:** Lambda concurrency is the number of function invocations executing at the same time. It directly affects how much parallel processing a function can perform.

**🔑 Keywords:** `Lambda` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q8. What is provisioned concurrency?

**💡 Answer:** Lambda concurrency is the number of function invocations executing at the same time. It directly affects how much parallel processing a function can perform.

**🔑 Keywords:** `Lambda` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q9. What is a cold start?

**💡 Answer:** A Lambda cold start occurs when AWS must initialize a new execution environment before running the handler. It can add latency due to runtime and application initialization.

**🔑 Keywords:** `Lambda` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q10. What is Lambda timeout?

**💡 Answer:** AWS Lambda runs code without requiring you to provision or manage servers. AWS handles the underlying compute infrastructure and scales execution according to incoming requests/events.

**🔑 Keywords:** `Lambda` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q11. What is memory allocation?

**💡 Answer:** Lambda memory allocation controls the memory available to the function and also scales associated CPU/network resources. Increasing memory can therefore improve execution speed as well as memory capacity.

**🔑 Keywords:** `Lambda` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Integration

#### Q12. How can API Gateway invoke Lambda?

**💡 Answer:** AWS Lambda runs code without requiring you to provision or manage servers. AWS handles the underlying compute infrastructure and scales execution according to incoming requests/events.

**🔑 Keywords:** `Lambda` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q13. How can S3 invoke Lambda?

**💡 Answer:** AWS Lambda runs code without requiring you to provision or manage servers. AWS handles the underlying compute infrastructure and scales execution according to incoming requests/events.

**🔑 Keywords:** `Lambda` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q14. How can EventBridge invoke Lambda?

**💡 Answer:** AWS Lambda runs code without requiring you to provision or manage servers. AWS handles the underlying compute infrastructure and scales execution according to incoming requests/events.

**🔑 Keywords:** `Lambda` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q15. How can SQS invoke Lambda?

**💡 Answer:** AWS Lambda runs code without requiring you to provision or manage servers. AWS handles the underlying compute infrastructure and scales execution according to incoming requests/events.

**🔑 Keywords:** `Lambda` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q16. What is an event source mapping?

**💡 Answer:** An event source mapping connects Lambda to poll-based event sources such as SQS, Kinesis, and DynamoDB Streams. Lambda polls the source and invokes the function with batches of records.

**🔑 Keywords:** `Lambda` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Operations

#### Q17. What is Lambda versioning?

**💡 Answer:** S3 Versioning keeps multiple versions of an object under the same key. It helps recover from accidental deletion or overwrite and is commonly combined with lifecycle rules to manage noncurrent versions.

**🔑 Keywords:** `Lambda` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q18. What is an alias?

**💡 Answer:** A Route 53 Alias record maps a name to supported AWS resources or another supported Route 53 target without requiring a CNAME at the zone apex. It is AWS-specific and does not incur a Route 53 query charge for alias queries to AWS resources.

**🔑 Keywords:** `Lambda` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q19. What is a dead-letter destination?

**💡 Answer:** A Lambda dead-letter destination or asynchronous failure destination can capture events that could not be processed successfully, depending on the invocation model.

**🔑 Keywords:** `Lambda` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q20. What is Lambda layers?

**💡 Answer:** AWS Lambda runs code without requiring you to provision or manage servers. AWS handles the underlying compute infrastructure and scales execution according to incoming requests/events.

**🔑 Keywords:** `Lambda` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q21. How do environment variables work?

**💡 Answer:** Lambda environment variables provide configuration values to the function without hardcoding them in source code. Sensitive values should normally be protected using a managed secret mechanism rather than plain environment variables.

**🔑 Keywords:** `Lambda` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q22. What is Lambda@Edge?

**💡 Answer:** AWS Lambda runs code without requiring you to provision or manage servers. AWS handles the underlying compute infrastructure and scales execution according to incoming requests/events.

**🔑 Keywords:** `Lambda` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

## 🚀 Last-Minute Revision

> 🧠 **Remember:** **Event → Lambda → Execution Role → Code → Response**. Concurrency controls parallel executions.

[⬆️ Back to top](#lambda)

[⬅️ Back to AWS Topics](../README.md)