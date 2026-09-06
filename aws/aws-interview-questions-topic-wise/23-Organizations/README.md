# AWS Organizations

### Q1. What is AWS Organizations?

**Answer:** AWS Organizations centrally manages multiple AWS accounts. It supports account grouping, consolidated billing, governance policies, and centralized security controls.

---

### Q2. What is a management account?

**Answer:** The AWS Organizations management account creates and manages the organization and has special administrative capabilities. It should be tightly protected and used sparingly for workloads.

---

### Q3. What is a member account?

**Answer:** A member account belongs to an AWS Organization but is governed by organization-level controls such as SCPs. Workloads are normally deployed into member accounts rather than the management account.

---

### Q4. What is an organizational unit?

**Answer:** An Organizational Unit (OU) groups AWS accounts so governance policies and controls can be applied to a logical set of accounts.

---

### Q5. What is consolidated billing?

**Answer:** Consolidated billing combines billing information for accounts in an AWS Organization and can provide centralized cost visibility and applicable volume pricing benefits.

---

### Q6. What is an SCP?

**Answer:** An SCP defines the maximum permissions available to principals in member accounts. It acts as a guardrail and does not grant permissions by itself.

---

### Q7. How do SCPs constrain member accounts?

**Answer:** An SCP defines the maximum permissions available to principals in member accounts. It does not grant access; an IAM or resource policy must still allow the action. If an SCP denies an action, a lower-level Allow cannot override that restriction.

---

### Q8. What is an organization trail?

**Answer:** An organization trail is a CloudTrail trail configured from AWS Organizations to apply logging across member accounts. It centralizes audit collection and reduces per-account configuration work.

---

### Q9. What is delegated administration?

**Answer:** Delegated administration allows a designated member account to administer supported AWS services on behalf of the organization without requiring every administrative task to use the management account.

---

### Q10. What are service quotas and account governance concerns?

**Answer:** AWS service quotas limit how much of a service an account can use, such as the number of VPCs, elastic IP addresses, or API requests. Account governance should monitor quotas, request increases where supported, and consider quota limits when designing account, Region, and resource boundaries.

---
