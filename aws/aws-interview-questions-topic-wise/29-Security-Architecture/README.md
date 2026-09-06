# AWS Security Architecture

### Q1. What is least privilege?

**Answer:** Least privilege means granting only the permissions required for a task. In AWS this means restricting actions, resources, conditions, principals, and credential lifetime as appropriate.

---

### Q2. What is defense in depth?

**Answer:** Defense in depth uses multiple independent security controls so that failure of one control does not expose the entire workload. Examples include IAM, network segmentation, encryption, WAF, logging, and threat detection.

---

### Q3. Why use IAM roles instead of long-lived access keys?

**Answer:** AWS Identity and Access Management (IAM) controls authentication and authorization for AWS resources. It includes identities such as users, groups, and roles and policies that determine what actions are allowed or denied.

---

### Q4. What is MFA?

**Answer:** Multi-factor authentication requires an additional authentication factor beyond a password. It is especially important for privileged identities and protecting the AWS account root user.

---

### Q5. What is federation?

**Answer:** Identity federation allows users to authenticate through an external identity provider and obtain temporary AWS access instead of maintaining separate long-lived IAM user credentials.

---

### Q6. How do private subnets improve isolation?

**Answer:** Private subnets do not provide a direct route to an Internet Gateway, so resources placed there cannot receive unsolicited Internet traffic through a normal public route. Required outbound access can be provided through controlled paths such as NAT Gateways, while VPC endpoints can provide private access to supported AWS services.

---

### Q7. What is a bastion host?

**Answer:** A bastion host is a hardened intermediary used to administer private resources. Modern AWS architectures often prefer Systems Manager Session Manager because it avoids exposing inbound SSH to private servers.

---

### Q8. What is Systems Manager Session Manager?

**Answer:** Systems Manager Session Manager provides secure interactive access to managed instances without requiring inbound SSH or RDP ports. Access is controlled through IAM and Systems Manager prerequisites.

---

### Q9. How do SGs and NACLs differ?

**Answer:** Security groups are stateful firewalls attached to ENIs and support allow rules. NACLs are stateless filters applied at the subnet boundary and support ordered allow and deny rules. Security groups automatically allow response traffic for permitted connections; NACLs require both directions to be allowed explicitly.

---

### Q10. What is VPC endpoint isolation?

**Answer:** A VPC endpoint provides private connectivity from a VPC to supported AWS services or endpoint services without requiring the service path to traverse the public Internet.

---

### Q11. What is encryption at rest?

**Answer:** Encryption at rest protects stored data using encryption mechanisms provided by the storage or database service. AWS KMS commonly provides customer-controlled key management for supported services.

---

### Q12. What is encryption in transit?

**Answer:** Encryption in transit protects data while it moves between clients, services, or networks, typically using TLS or another encrypted protocol.

---

### Q13. How does KMS support encryption?

**Answer:** AWS KMS is a managed service for creating and controlling cryptographic keys used to protect data. It integrates with many AWS services and provides authorization and audit controls.

---

### Q14. Why should secrets be stored in managed secret stores?

**Answer:** Managed secret stores such as AWS Secrets Manager and Systems Manager Parameter Store reduce the need to place credentials in source code or configuration files. They provide centralized access control, encryption, auditing, and, for Secrets Manager, automated rotation for supported secrets.

---

### Q15. What are CloudTrail, Config, GuardDuty, Inspector, and Security Hub used for?

**Answer:** AWS Security Hub centralizes security findings from AWS services and supported third-party products and provides security posture and compliance-oriented views.

---

### Q16. How does centralized logging improve security?

**Answer:** The security pillar focuses on protecting information and systems through strong identity controls, detection, infrastructure protection, data protection, and incident response.

---

### Q17. What is AWS Organizations security governance?

**Answer:** AWS Organizations centrally manages multiple AWS accounts. It supports account grouping, consolidated billing, governance policies, and centralized security controls.

---
