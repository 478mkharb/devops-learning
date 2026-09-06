# Secrets Manager & Parameter Store

### Q1. What is AWS Secrets Manager?

**Answer:** AWS Secrets Manager stores sensitive information such as database credentials and API keys, encrypts it, controls access through IAM, and supports automatic rotation for supported secrets.

---

### Q2. What is automatic secret rotation?

**Answer:** Secrets Manager can automatically rotate supported secrets using a rotation workflow. This allows credentials to change without requiring manual updates to applications.

---

### Q3. How can applications retrieve secrets?

**Answer:** An application can call Secrets Manager through the AWS SDK, CLI, or supported integration using permissions granted to its IAM role or identity. The application retrieves the secret at runtime rather than storing long-lived credentials in source code.

---

### Q4. How is Secrets Manager encrypted?

**Answer:** Secrets Manager encrypts secret values at rest using AWS KMS. A customer can use the default AWS managed key or, where supported, a customer managed KMS key for additional control over key policy and lifecycle.

---

### Q5. When should Secrets Manager be preferred?

**Answer:** Prefer Secrets Manager when the application stores sensitive credentials or secrets and needs purpose-built secret lifecycle capabilities such as automatic rotation, versioning, and controlled retrieval.

---

### Q6. What is Systems Manager Parameter Store?

**Answer:** Systems Manager Parameter Store stores configuration values and parameters in a hierarchical namespace. SecureString parameters can be encrypted with KMS.

---

### Q7. What is a String parameter?

**Answer:** A String parameter in Systems Manager Parameter Store stores a plain text configuration value. It is appropriate for non-sensitive configuration such as environment names or feature settings; sensitive values should use SecureString or Secrets Manager.

---

### Q8. What is SecureString?

**Answer:** A SecureString is an encrypted Systems Manager Parameter Store parameter protected with AWS KMS. Access requires appropriate IAM permissions.

---

### Q9. How does Parameter Store differ from Secrets Manager?

**Answer:** Parameter Store is a general configuration and parameter service and supports encrypted SecureString values. Secrets Manager is purpose-built for secrets and provides features such as automatic rotation for supported secret types.

---

### Q10. How can applications retrieve parameters securely?

**Answer:** CloudFormation parameters are input values supplied when creating or updating a stack. They allow a template to be reused with different environment-specific values.

---

### Q11. Why should credentials not be hardcoded?

**Answer:** Credentials should not be hardcoded because source code, Git repositories, logs, machine images, and backups can expose them. Using IAM roles for AWS access and Secrets Manager or Parameter Store for application secrets allows access to be controlled, audited, and rotated without changing source code.

---

### Q12. How can EC2 retrieve secrets using an IAM role?

**Answer:** Attach an IAM role to the EC2 instance through an instance profile and grant the minimum required Secrets Manager permissions, such as `secretsmanager:GetSecretValue` for the required secret. The application then uses the AWS SDK, which obtains temporary credentials from IMDS.

---

### Q13. How can Lambda retrieve secrets?

**Answer:** Give the Lambda execution role least-privilege permission to retrieve the required secret and use the AWS SDK or an appropriate integration to fetch it at runtime. Secrets can also be cached in the execution environment when the application can tolerate the associated rotation/freshness considerations.

---

### Q14. How should secret access be restricted with least privilege?

**Answer:** Least privilege means granting only the permissions required for a task. In AWS this means restricting actions, resources, conditions, principals, and credential lifetime as appropriate.

---
