# CloudWatch

### Q1. What is Amazon CloudWatch?

**Answer:** Amazon CloudWatch provides metrics, logs, alarms, dashboards, and observability capabilities for AWS resources and applications.

---

### Q2. What is a metric?

**Answer:** A CloudWatch metric is a time series of numerical measurements identified by a namespace and dimensions. AWS services publish metrics and applications can publish custom metrics.

---

### Q3. What is a namespace?

**Answer:** A CloudWatch namespace groups related metrics, commonly by AWS service or application.

---

### Q4. What is a dimension?

**Answer:** A CloudWatch dimension is a name/value pair that identifies and filters a metric time series.

---

### Q5. What is a datapoint?

**Answer:** A CloudWatch datapoint is a single measurement of a metric at a particular timestamp and period.

---

### Q6. What is a CloudWatch alarm?

**Answer:** A CloudWatch alarm evaluates a metric or metric expression against configured conditions over evaluation periods. It can enter OK, ALARM, or INSUFFICIENT_DATA and can trigger actions such as SNS notifications or Auto Scaling.

---

### Q7. What are OK, ALARM, and INSUFFICIENT_DATA states?

**Answer:** OK means the alarm condition is not breached. ALARM means the configured condition is breached. INSUFFICIENT_DATA means CloudWatch cannot determine the state because there is not enough valid data.

---

### Q8. What is an alarm threshold?

**Answer:** An alarm threshold is the comparison boundary used to determine whether a metric or expression meets the alarm condition, such as CPU utilization greater than 80 percent.

---

### Q9. What is evaluation period?

**Answer:** An alarm evaluation period is the duration of each metric period considered when determining the alarm state. CloudWatch evaluates the configured number of recent periods and datapoints.

---

### Q10. What is a composite alarm?

**Answer:** A composite alarm combines the states of multiple CloudWatch alarms using a rule. It can reduce alert noise by requiring a specific combination of conditions before raising an alert.

---

### Q11. What is a log group?

**Answer:** A CloudWatch Logs log group is a logical container for log streams and their retention, encryption, and access configuration.

---

### Q12. What is a log stream?

**Answer:** A CloudWatch Logs log stream is a sequence of log events from a particular source within a log group.

---

### Q13. How can EC2 send logs to CloudWatch Logs?

**Answer:** Install and configure the Amazon CloudWatch agent on the EC2 instance, specify the log files and destination log groups in its configuration, and grant the instance IAM role permission to publish logs. The agent then sends log events to CloudWatch Logs.

---

### Q14. What is Logs Insights?

**Answer:** CloudWatch Logs Insights provides a query language and interactive analysis for searching, filtering, and aggregating log data stored in CloudWatch Logs.

---

### Q15. What is a metric filter?

**Answer:** A CloudWatch Logs metric filter scans log events for configured patterns and publishes matching counts or values as CloudWatch metrics.

---

### Q16. What are dashboards?

**Answer:** A CloudWatch dashboard is a customizable collection of widgets that displays metrics, alarms, logs, and other monitoring information. It provides a single operational view for application and infrastructure health.

---

### Q17. What are custom metrics?

**Answer:** Custom metrics are application- or business-specific measurements published to CloudWatch, such as request latency, transaction count, or queue-processing time. They can be graphed and used in alarms and automation just like AWS service metrics.

---

### Q18. What is detailed monitoring for EC2?

**Answer:** EC2 detailed monitoring provides metric data at one-minute intervals for supported monitoring metrics, compared with the default five-minute basic monitoring interval.

---

### Q19. How can CloudWatch trigger Auto Scaling?

**Answer:** A CloudWatch alarm can monitor a metric such as CPU utilization, request count per target, or queue backlog and invoke an ASG scaling policy when the configured condition is met. The ASG then changes capacity within its minimum and maximum limits.

---

### Q20. How can CloudWatch trigger SNS notifications?

**Answer:** Configure a CloudWatch alarm with an SNS topic as an alarm action. When the alarm enters the configured ALARM state, CloudWatch publishes a notification to the SNS topic, which can then deliver it to subscribed endpoints.

---
