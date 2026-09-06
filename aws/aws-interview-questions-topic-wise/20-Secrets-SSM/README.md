# Secrets Manager & Parameter Store

[⬅️ Back to AWS Topics](../README.md)

## 🔑 Keywords

🔒 Secrets Manager | Rotation | Parameter Store | SecureString | KMS | IAM

## 🧠 Core Memory

🧠 **Remember:** Secrets Manager = **secrets + rotation**; Parameter Store = **configuration/parameters + SecureString**.

---

## ❓ Interview Questions

### 📌 Secrets Manager

#### Q1. What is AWS Secrets Manager?

**💡 Answer:** AWS Secrets Manager stores secrets such as database credentials and API keys and provides controlled retrieval, encryption, and optional automatic rotation. Applications can retrieve secrets using IAM permissions.

**🔑 Keywords:** `Secrets` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q2. What is automatic secret rotation?

**💡 Answer:** Secrets Manager can rotate supported secrets automatically by invoking a rotation workflow, allowing applications to use managed credentials without manually changing them.

**🔑 Keywords:** `Secrets` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q3. How can applications retrieve secrets?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `Secrets` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q4. How is Secrets Manager encrypted?

**💡 Answer:** AWS Secrets Manager stores secrets such as database credentials and API keys and provides controlled retrieval, encryption, and optional automatic rotation. Applications can retrieve secrets using IAM permissions.

**🔑 Keywords:** `Secrets` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q5. When should Secrets Manager be preferred?

**💡 Answer:** AWS Secrets Manager stores secrets such as database credentials and API keys and provides controlled retrieval, encryption, and optional automatic rotation. Applications can retrieve secrets using IAM permissions.

**🔑 Keywords:** `Secrets` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Parameter Store

#### Q6. What is Systems Manager Parameter Store?

**💡 Answer:** Systems Manager Parameter Store provides hierarchical configuration and parameter storage. SecureString parameters can be encrypted with KMS. It is useful for application configuration and simpler secret/configuration use cases.

**🔑 Keywords:** `Secrets` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q7. What is a String parameter?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `Secrets` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q8. What is SecureString?

**💡 Answer:** A SecureString parameter is an encrypted Systems Manager Parameter Store value protected with KMS. Access requires appropriate IAM permissions.

**🔑 Keywords:** `Secrets` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q9. How does Parameter Store differ from Secrets Manager?

**💡 Answer:** AWS Secrets Manager stores secrets such as database credentials and API keys and provides controlled retrieval, encryption, and optional automatic rotation. Applications can retrieve secrets using IAM permissions.

**🔑 Keywords:** `Secrets` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q10. How can applications retrieve parameters securely?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `Secrets` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Design

#### Q11. Why should credentials not be hardcoded?

**💡 Answer:** Define the AWS service or component, explain its key behavior and internal relationship with adjacent services, then state the practical use case and the main trade-off an interviewer should know.

**🔑 Keywords:** `Secrets` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q12. How can EC2 retrieve secrets using an IAM role?

**💡 Answer:** AWS IAM controls authentication and authorization to AWS resources. It includes identities such as users, groups, and roles and policies that determine allowed actions.

**🔑 Keywords:** `Secrets` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q13. How can Lambda retrieve secrets?

**💡 Answer:** AWS Lambda runs code without requiring you to provision or manage servers. AWS handles the underlying compute infrastructure and scales execution according to incoming requests/events.

**🔑 Keywords:** `Secrets` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q14. How should secret access be restricted with least privilege?

**💡 Answer:** Least privilege means granting only the permissions required to perform a task. In AWS this includes restricting actions, resources, conditions, principals, and credential lifetime where practical.

**🔑 Keywords:** `Secrets` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

## 🚀 Last-Minute Revision

> 🧠 **Remember:** Secrets Manager = **secrets + rotation**; Parameter Store = **configuration/parameters + SecureString**.

[⬆️ Back to top](#secrets-manager-parameter-store)

[⬅️ Back to AWS Topics](../README.md)