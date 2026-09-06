# AWS Security Architecture

[⬅️ Back to AWS Topics](../README.md)

## 🔑 Keywords

🔐 Least Privilege | Defense in Depth | IAM | VPC | Encryption | GuardDuty | Inspector | Security Hub | CloudTrail

## 🧠 Core Memory

🧠 **Remember:** **Identity + Network + Data + Detection = defense in depth**.

---

## ❓ Interview Questions

### 📌 Identity

#### Q1. What is least privilege?

**💡 Answer:** Least privilege means granting only the permissions required to perform a task. In AWS this includes restricting actions, resources, conditions, principals, and credential lifetime where practical.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q2. What is defense in depth?

**💡 Answer:** Defense in depth uses multiple independent security controls so that failure of one control does not expose the entire system. AWS examples include IAM, network segmentation, WAF, encryption, logging, and threat detection.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q3. Why use IAM roles instead of long-lived access keys?

**💡 Answer:** AWS IAM controls authentication and authorization to AWS resources. It includes identities such as users, groups, and roles and policies that determine allowed actions.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q4. What is MFA?

**💡 Answer:** Multi-factor authentication requires an additional authentication factor beyond a password. It is especially important for privileged identities and account-root protection.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q5. What is federation?

**💡 Answer:** Identity federation lets users authenticate through an external identity provider and obtain temporary AWS credentials or console access through AWS IAM Identity Center or supported federation mechanisms.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Network

#### Q6. How do private subnets improve isolation?

**💡 Answer:** A subnet is an IP address range inside a VPC and is associated with one Availability Zone. A subnet is considered public when its route table provides a path to an Internet Gateway; otherwise it is commonly private.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q7. What is a bastion host?

**💡 Answer:** A bastion host is a hardened intermediary used to administer private resources. Modern AWS designs often prefer Systems Manager Session Manager to avoid exposing SSH to the network.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q8. What is Systems Manager Session Manager?

**💡 Answer:** Systems Manager Session Manager provides secure shell-like access to managed EC2 instances without requiring inbound SSH ports or bastion hosts. Access is controlled through IAM and Systems Manager prerequisites.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q9. How do SGs and NACLs differ?

**💡 Answer:** Explain the security control, its threat model, where it is enforced, and how it fits into least privilege, defense in depth, encryption, logging, or network isolation.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q10. What is VPC endpoint isolation?

**💡 Answer:** A VPC is a logically isolated virtual network in AWS. It contains subnets, route tables, network interfaces, and security controls and can connect to the Internet, other VPCs, on-premises networks, or AWS services.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Data

#### Q11. What is encryption at rest?

**💡 Answer:** Encryption at rest protects stored data on services such as EBS, S3, RDS, and databases. AWS-managed encryption services such as KMS commonly provide key control.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q12. What is encryption in transit?

**💡 Answer:** Encryption in transit protects data while it moves between clients, services, or networks, typically using TLS or other encrypted protocols.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q13. How does KMS support encryption?

**💡 Answer:** AWS KMS is a managed key service used to create and control cryptographic keys for encrypting data and protecting other secrets. It integrates with many AWS services.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q14. Why should secrets be stored in managed secret stores?

**💡 Answer:** Explain the security control, its threat model, where it is enforced, and how it fits into least privilege, defense in depth, encryption, logging, or network isolation.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Detection

#### Q15. What are CloudTrail, Config, GuardDuty, Inspector, and Security Hub used for?

**💡 Answer:** AWS CloudTrail records AWS API activity and related events for governance, audit, and security investigation. It can deliver events to destinations such as S3 and CloudWatch Logs.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q16. How does centralized logging improve security?

**💡 Answer:** Explain the security control, its threat model, where it is enforced, and how it fits into least privilege, defense in depth, encryption, logging, or network isolation.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q17. What is AWS Organizations security governance?

**💡 Answer:** AWS Organizations centrally manages multiple AWS accounts. It provides account grouping, consolidated billing, governance controls, and organization-wide policy mechanisms.

**🔑 Keywords:** `AWS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

## 🚀 Last-Minute Revision

> 🧠 **Remember:** **Identity + Network + Data + Detection = defense in depth**.

[⬆️ Back to top](#aws-security-architecture)

[⬅️ Back to AWS Topics](../README.md)