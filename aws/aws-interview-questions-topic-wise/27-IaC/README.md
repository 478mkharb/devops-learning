# Infrastructure as Code

## Interview Questions & Answers

### Q1. What is AWS CloudFormation?

**Answer:** AWS CloudFormation is an infrastructure-as-code service that provisions AWS resources from declarative templates. A stack represents a deployed collection of resources managed together.

---

### Q2. What is a stack?

**Answer:** A CloudFormation stack is a deployed collection of AWS resources managed together from a CloudFormation template. Stack updates and deletion are performed as coordinated infrastructure operations.

---

### Q3. What is a template?

**Answer:** A CloudFormation template is a YAML or JSON document that declaratively describes AWS resources and their configuration. It can include parameters, mappings, conditions, resources, and outputs.

---

### Q4. What are parameters?

**Answer:** CloudFormation parameters are input values supplied when a stack is created or updated. They allow the same template to be reused across environments without hardcoding environment-specific values.

---

### Q5. What are outputs?

**Answer:** CloudFormation outputs expose useful values from a stack, such as resource IDs or endpoints. Outputs can be viewed after deployment and, where supported, exported for use by other stacks.

---

### Q6. What is drift detection?

**Answer:** Infrastructure drift occurs when real infrastructure differs from the declared configuration. CloudFormation provides drift detection, while Terraform can detect differences during refresh/plan operations.

---

### Q7. What is Terraform?

**Answer:** Terraform is an infrastructure-as-code tool that uses configuration files to describe desired infrastructure. It maintains state to map configuration to real resources and creates an execution plan before changes.

---

### Q8. What is a provider?

**Answer:** A Terraform provider is a plugin that implements resources and data sources for a platform or API. The AWS provider translates Terraform configuration into AWS API operations.

---

### Q9. What is state?

**Answer:** Terraform state records the relationship between Terraform configuration and real infrastructure and stores attributes needed to calculate future changes. It is a critical part of Terraform's operation and must be protected.

---

### Q10. What is a remote backend?

**Answer:** A Terraform remote backend stores state outside the local workstation, improving collaboration, durability, and locking support where the selected backend provides it.

---

### Q11. What is a data source?

**Answer:** A Terraform data source reads information about existing infrastructure or external data without creating the referenced resource. It is useful for discovering VPCs, AMIs, subnets, and other existing objects.

---

### Q12. What is a resource?

**Answer:** A Terraform resource represents infrastructure that Terraform manages, such as an EC2 instance, security group, S3 bucket, or load balancer.

---

### Q13. What is plan vs apply?

**Answer:** terraform plan calculates and displays the proposed infrastructure changes without applying them. terraform apply executes the approved changes against the provider.

---

### Q14. Why should state be protected?

**Answer:** Terraform state can contain resource identifiers, configuration details, and sometimes sensitive values. Store it in a secured remote backend where possible, enable encryption and access control, and use locking/coordination features supported by the backend.

---

### Q15. Terraform vs CloudFormation: when would you choose each?

**Answer:** AWS CloudFormation is an infrastructure-as-code service that provisions AWS resources from declarative templates. A stack represents a deployed collection of resources managed together.

---

### Q16. What is immutable infrastructure?

**Answer:** Immutable infrastructure means replacing infrastructure with a new version rather than modifying running instances in place. AMIs, Launch Templates, ASGs, and blue/green deployment patterns are common AWS implementations.

---

### Q17. Why is IaC useful for repeatability and auditability?

**Answer:** Infrastructure as code stores infrastructure definitions in version-controlled files. Changes can be reviewed, reproduced across environments, automated in pipelines, and traced through commits and plans, reducing manual configuration drift.

---
