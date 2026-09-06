# AWS Organizations

## Interview Questions & Answers

### Q1. What is AWS Organizations?

**Answer:** AWS Organizations centrally manages multiple AWS accounts. It provides account grouping, consolidated billing, governance controls, and organization-wide policy mechanisms.

---

### Q2. What is a management account?

**Answer:** The management account is the AWS account that creates and administers the organization. It has powerful organization-level privileges and should be tightly protected and avoided for routine application workloads.

---

### Q3. What is a member account?

**Answer:** A member account belongs to an AWS Organization and is governed by organization-level controls such as SCPs. Workloads are normally separated into member accounts to improve security and operational boundaries.

---

### Q4. What is an organizational unit?

**Answer:** An Organizational Unit groups member accounts within AWS Organizations so policies and governance controls can be applied to a logical set of accounts.

---

### Q5. What is consolidated billing?

**Answer:** Consolidated billing combines usage and billing for accounts in an AWS Organization and can provide centralized cost visibility and applicable volume benefits.

---

### Q6. What is an SCP?

**Answer:** A Service Control Policy is an AWS Organizations guardrail that limits the maximum permissions available in member accounts. It does not grant permissions by itself.

---

### Q7. How do SCPs constrain member accounts?

**Answer:** A Service Control Policy is an AWS Organizations guardrail that limits the maximum permissions available in member accounts. It does not grant permissions by itself.

---

### Q8. What is an organization trail?

**Answer:** A CloudTrail trail is a configuration that records selected events and delivers them to destinations such as an S3 bucket and optionally CloudWatch Logs.

---

### Q9. What is delegated administration?

**Answer:** Delegated administration lets designated member accounts administer specific AWS services on behalf of the organization without using the management account for every task.

---

### Q10. What are service quotas and account governance concerns?

**Answer:** AWS service quotas limit resource consumption for services and Regions. Account governance should also cover account creation, security baselines, logging, allowed Regions, IAM controls, networking standards, and cost controls so new accounts remain compliant and operationally safe.

---
