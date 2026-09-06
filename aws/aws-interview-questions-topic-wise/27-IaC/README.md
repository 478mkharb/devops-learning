# Infrastructure as Code

### Q1. What is AWS CloudFormation?

**Answer:** AWS CloudFormation is an infrastructure-as-code service that provisions AWS resources from declarative templates. A stack represents a deployed collection of resources managed together.

---

### Q2. What is a stack?

**Answer:** A CloudFormation stack is a collection of AWS resources created and managed as a single unit from a template.

---

### Q3. What is a template?

**Answer:** A CloudFormation template is a declarative document, usually YAML or JSON, that defines AWS resources and their configuration.

---

### Q4. What are parameters?

**Answer:** CloudFormation parameters are input values supplied when creating or updating a stack. They allow a template to be reused with different environment-specific values.

---

### Q5. What are outputs?

**Answer:** CloudFormation outputs expose values from a stack, such as resource IDs or endpoints, so they can be viewed, exported, or referenced by other stacks.

---

### Q6. What is drift detection?

**Answer:** CloudFormation drift detection identifies differences between the expected template configuration and the actual configuration of supported resources.

---

### Q7. What is Terraform?

**Answer:** Terraform is an infrastructure-as-code tool that uses declarative configuration to describe desired infrastructure. It maintains state mapping configuration to real resources and creates an execution plan before applying changes.

---

### Q8. What is a provider?

**Answer:** A Terraform provider is a plugin that implements resources and data sources for a platform or service, such as AWS.

---

### Q9. What is state?

**Answer:** Terraform state records the relationship between configuration and real infrastructure. It is required for planning changes and should be stored securely because it can contain sensitive information.

---

### Q10. What is a remote backend?

**Answer:** A Terraform remote backend stores state outside the local workstation, supporting team collaboration, durability, and locking where the selected backend provides it.

---

### Q11. What is a data source?

**Answer:** A Terraform data source reads information about existing infrastructure or external data without creating the referenced resource. It is useful for discovering AMIs, VPCs, subnets, and other existing objects.

---

### Q12. What is a resource?

**Answer:** In Terraform, a resource is a block that represents infrastructure Terraform manages. Examples include an EC2 instance, VPC, security group, S3 bucket, or load balancer. Terraform creates, updates, and destroys resources according to configuration and state.

---

### Q13. What is plan vs apply?

**Answer:** terraform apply executes the approved execution plan and creates, updates, or deletes managed infrastructure.

---

### Q14. Why should state be protected?

**Answer:** Terraform state can contain resource identifiers, configuration details, and potentially sensitive values. Protect it with a secure remote backend, encryption, strict access controls, state locking where supported, and carefully scoped CI/CD credentials.

---

### Q15. Terraform vs CloudFormation: when would you choose each?

**Answer:** CloudFormation is AWS-native and tightly integrated with AWS resource lifecycle management. Terraform supports multiple cloud and infrastructure platforms through providers and maintains its own state model. Choose CloudFormation for AWS-native IaC when its capabilities fit; choose Terraform when you need multi-platform IaC or already standardize on Terraform.

---

### Q16. What is immutable infrastructure?

**Answer:** Immutable infrastructure replaces instances or servers with newly built versions instead of modifying existing servers in place. It improves consistency, rollback, and repeatability.

---

### Q17. Why is IaC useful for repeatability and auditability?

**Answer:** Infrastructure as Code makes infrastructure repeatable and reviewable because the desired configuration is stored as version-controlled code. It improves consistency, supports peer review and automation, provides an auditable change history, and reduces manual configuration drift.

---
