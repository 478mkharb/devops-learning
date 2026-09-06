# CloudTrail

### Q1. What is AWS CloudTrail?

**Answer:** AWS CloudTrail records AWS API activity and related events for governance, auditing, and security investigation. Trails can deliver events to destinations such as S3 and CloudWatch Logs.

---

### Q2. What is a management event?

**Answer:** CloudTrail management events record control-plane operations such as creating, modifying, or deleting AWS resources. They include activity performed through the console, CLI, SDKs, and APIs.

---

### Q3. What is a data event?

**Answer:** CloudTrail data events record data-plane activity on supported resources, such as object-level S3 operations or Lambda invocations. They provide more granular activity records and can generate higher event volume.

---

### Q4. What is an insight event?

**Answer:** CloudTrail Insights detects unusual patterns in API activity, such as unexpected changes in API call volume, and records insight events for investigation.

---

### Q5. What is CloudTrail Event History?

**Answer:** CloudTrail Event History provides a searchable record of recent management events for an AWS account without requiring a trail for that basic history.

---

### Q6. What is a CloudTrail trail?

**Answer:** A CloudTrail trail is a configuration that records selected AWS events and delivers them to destinations such as an S3 bucket and optionally CloudWatch Logs. A trail can be configured for management events and, where required, data events.

---

### Q7. How can CloudTrail deliver logs to S3?

**Answer:** Configure a CloudTrail trail with an S3 bucket as its destination. CloudTrail writes log files containing recorded events to the bucket, where they can be retained, analyzed, and protected with S3 access controls.

---

### Q8. How can CloudTrail send events to CloudWatch Logs?

**Answer:** Configure a CloudTrail trail with a CloudWatch Logs log group and an IAM role that allows CloudTrail to publish events. CloudWatch Logs can then retain, search, filter, and alert on the recorded activity.

---

### Q9. What is an organization trail?

**Answer:** An organization trail is a CloudTrail trail configured from AWS Organizations to apply logging across member accounts. It centralizes audit collection and reduces per-account configuration work.

---

### Q10. How should CloudTrail logs be protected?

**Answer:** Store CloudTrail logs in a dedicated, tightly controlled S3 bucket with Block Public Access, restrictive bucket policies, encryption, limited read/write permissions, and appropriate retention controls. Consider centralized or cross-account logging and log-file validation for stronger tamper resistance.

---

### Q11. How does CloudTrail support auditing?

**Answer:** CloudTrail records AWS activity such as the principal, API action, time, source, and affected resource. These records provide an audit trail for investigating changes, meeting compliance requirements, and determining who performed an operation.

---

### Q12. How can you detect unexpected API activity?

**Answer:** Use CloudTrail to record AWS API activity and analyze the events for unexpected principals, actions, source IPs, Regions, or resource changes. CloudTrail can integrate with CloudWatch Logs and EventBridge so suspicious activity can trigger automated alerts or investigation workflows.

---

### Q13. What is CloudTrail Lake?

**Answer:** CloudTrail Lake is a managed event data store and query capability for collecting and analyzing CloudTrail activity and related audit data.

---

### Q14. How does CloudTrail differ from CloudWatch?

**Answer:** Amazon CloudWatch provides metrics, logs, alarms, dashboards, and observability capabilities for AWS resources and applications.

---

### Q15. How does CloudTrail differ from AWS Config?

**Answer:** CloudTrail focuses on AWS API activity and records actions performed against services. AWS Config focuses on resource configuration state, configuration history, and compliance evaluation. Together they answer both 'who changed it?' and 'what is its configuration?'

---
