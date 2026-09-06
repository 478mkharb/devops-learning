# Secrets Manager & Parameter Store

## Interview Questions & Answers

### Q1. What is AWS Secrets Manager?

**Answer:** AWS Secrets Manager stores secrets such as database credentials and API keys and provides controlled retrieval, encryption, and optional automatic rotation. Applications can retrieve secrets using IAM permissions.

---

### Q2. What is automatic secret rotation?

**Answer:** Secrets Manager can rotate supported secrets automatically by invoking a rotation workflow, allowing applications to use managed credentials without manually changing them.

---

### Q3. How can applications retrieve secrets?

**Answer:** Applications can call Secrets Manager using an SDK/API and an IAM role or other authorized identity. The application retrieves the current secret at runtime instead of storing a permanent credential in source code.

---

### Q4. How is Secrets Manager encrypted?

**Answer:** AWS Secrets Manager stores secrets such as database credentials and API keys and provides controlled retrieval, encryption, and optional automatic rotation. Applications can retrieve secrets using IAM permissions.

---

### Q5. When should Secrets Manager be preferred?

**Answer:** AWS Secrets Manager stores secrets such as database credentials and API keys and provides controlled retrieval, encryption, and optional automatic rotation. Applications can retrieve secrets using IAM permissions.

---

### Q6. What is Systems Manager Parameter Store?

**Answer:** Systems Manager Parameter Store provides hierarchical configuration and parameter storage. SecureString parameters can be encrypted with KMS. It is useful for application configuration and simpler secret/configuration use cases.

---

### Q7. What is a String parameter?

**Answer:** A String parameter in Systems Manager Parameter Store stores a plain-text configuration value. It is appropriate for non-sensitive settings such as environment names, feature flags, or application configuration.

---

### Q8. What is SecureString?

**Answer:** A SecureString parameter is an encrypted Systems Manager Parameter Store value protected with KMS. Access requires appropriate IAM permissions.

---

### Q9. How does Parameter Store differ from Secrets Manager?

**Answer:** AWS Secrets Manager stores secrets such as database credentials and API keys and provides controlled retrieval, encryption, and optional automatic rotation. Applications can retrieve secrets using IAM permissions.

---

### Q10. How can applications retrieve parameters securely?

**Answer:** Give the workload an IAM role with least-privilege permission to read the required parameters. For SecureString parameters, the workload also needs the required KMS decrypt permission. Retrieve the value through the AWS SDK or Systems Manager integration.

---

### Q11. Why should credentials not be hardcoded?

**Answer:** Hardcoded credentials can leak through source control, container images, AMIs, logs, backups, or configuration files and are difficult to rotate. Managed secret stores provide controlled access, encryption, auditing, and rotation mechanisms.

---

### Q12. How can EC2 retrieve secrets using an IAM role?

**Answer:** AWS IAM controls authentication and authorization to AWS resources. It includes identities such as users, groups, and roles and policies that determine allowed actions.

---

### Q13. How can Lambda retrieve secrets?

**Answer:** AWS Lambda runs code without requiring you to provision or manage servers. AWS handles the underlying compute infrastructure and scales execution according to incoming requests/events.

---

### Q14. How should secret access be restricted with least privilege?

**Answer:** Least privilege means granting only the permissions required to perform a task. In AWS this includes restricting actions, resources, conditions, principals, and credential lifetime where practical.

---
