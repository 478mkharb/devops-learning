# AWS Cost Management

## Interview Questions & Answers

### Q1. What is AWS Cost Explorer?

**Answer:** AWS Cost Explorer provides interactive analysis of AWS costs and usage over time. It helps identify cost drivers, trends, and service-level spend.

---

### Q2. What is AWS Budgets?

**Answer:** AWS Budgets lets you define cost, usage, or reservation/savings-related thresholds and receive alerts when actual or forecasted values exceed configured limits.

---

### Q3. What is the AWS Pricing Calculator?

**Answer:** AWS Pricing Calculator estimates AWS costs before deployment by modeling expected services, usage, and configurations.

---

### Q4. What is Cost and Usage Report?

**Answer:** The AWS Cost and Usage Report provides detailed billing and usage data that can be delivered to S3 for analysis and reporting.

---

### Q5. What is consolidated billing?

**Answer:** Consolidated billing combines usage and billing for accounts in an AWS Organization and can provide centralized cost visibility and applicable volume benefits.

---

### Q6. How can EC2 costs be reduced?

**Answer:** AWS STS issues temporary security credentials for sessions and role assumption. Temporary credentials are time-limited and reduce the risks associated with long-lived keys.

---

### Q7. When should Spot be considered?

**Answer:** Consider Spot when the workload can tolerate interruption and restart, such as batch processing, CI workers, distributed processing, rendering, or stateless services. Avoid relying on Spot as the only capacity for critical stateful workloads.

---

### Q8. How do rightsizing recommendations help?

**Answer:** Rightsizing recommendations identify resources whose provisioned capacity is larger than their observed workload needs. Moving to an appropriate instance size or type can reduce cost while maintaining required performance.

---

### Q9. How can S3 lifecycle policies reduce cost?

**Answer:** An S3 Lifecycle configuration defines automated transitions and expirations for objects. Rules can target prefixes, object tags, or object-size conditions and can move data to lower-cost storage classes or delete it when no longer needed.

---

### Q10. How can Savings Plans reduce compute cost?

**Answer:** Savings Plans provide discounted compute usage in exchange for a committed hourly spend. They generally provide more flexibility than configuration-specific Reserved Instances, depending on the plan type.

---

### Q11. What are cost allocation tags?

**Answer:** Cost allocation tags help categorize AWS costs so teams can analyze spend by application, environment, owner, or other business dimensions.

---

### Q12. What is a budget alert?

**Answer:** A budget alert notifies configured recipients or actions when actual or forecasted cost, usage, or commitment metrics cross a defined threshold. It is a governance control, not a hard spending limit by itself.

---

### Q13. How can Organizations help control spend?

**Answer:** AWS Organizations centrally manages multiple AWS accounts. It provides account grouping, consolidated billing, governance controls, and organization-wide policy mechanisms.

---

### Q14. What is AWS Trusted Advisor cost optimization?

**Answer:** AWS Trusted Advisor provides recommendations across areas such as cost optimization, performance, security, fault tolerance, and service limits, depending on account support/plan capabilities.

---
