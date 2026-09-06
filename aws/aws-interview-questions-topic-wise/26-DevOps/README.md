# AWS DevOps Services

## Interview Questions & Answers

### Q1. What is CodeCommit?

**Answer:** AWS CodeCommit is a managed Git-based source-control service for private repositories. It provides repository hosting and integrates with AWS development and CI/CD services, although teams may also use external Git providers.

---

### Q2. What is CodeBuild?

**Answer:** AWS CodeBuild is a managed build service that compiles source code, runs tests, and produces deployable artifacts without requiring you to manage build servers.

---

### Q3. What is CodeDeploy?

**Answer:** AWS CodeDeploy automates application deployments to supported compute targets such as EC2, Lambda, and ECS, with deployment strategies such as in-place and blue/green.

---

### Q4. What is CodePipeline?

**Answer:** AWS CodePipeline automates CI/CD workflows by orchestrating source, build, test, approval, and deployment stages.

---

### Q5. What is CodeArtifact?

**Answer:** AWS CodeArtifact is a managed artifact repository for package dependencies and software components used by build systems.

---

### Q6. What is in-place deployment?

**Answer:** An in-place deployment updates the existing compute instances with the new application version. It is simple and cost-effective but can temporarily reduce capacity and has more rollback risk than maintaining a separate environment.

---

### Q7. What is blue/green deployment?

**Answer:** Blue/green deployment maintains the current blue environment and a separate green environment containing the new version. After validation, traffic is shifted to green; rollback can be performed by shifting traffic back to blue.

---

### Q8. What is canary deployment?

**Answer:** Canary deployment sends a small percentage of traffic to the new version first. The new version is monitored before increasing traffic, reducing blast radius if the release has a defect.

---

### Q9. How can CodeDeploy deploy to EC2?

**Answer:** AWS CodeDeploy automates application deployments to supported compute targets such as EC2, Lambda, and ECS, with deployment strategies such as in-place and blue/green.

---

### Q10. How can pipelines integrate with CloudFormation or Terraform?

**Answer:** AWS CloudFormation is an infrastructure-as-code service that provisions AWS resources from declarative templates. A stack represents a deployed collection of resources managed together.

---

### Q11. What is Systems Manager?

**Answer:** AWS Systems Manager is a suite of operational capabilities for managing AWS and hybrid resources. It includes features such as Session Manager, Parameter Store, Patch Manager, Automation, and inventory capabilities.

---

### Q12. What is Session Manager?

**Answer:** Systems Manager Session Manager provides secure shell-like access to managed EC2 instances without requiring inbound SSH ports or bastion hosts. Access is controlled through IAM and Systems Manager prerequisites.

---

### Q13. What is Parameter Store?

**Answer:** Systems Manager Parameter Store provides hierarchical configuration and parameter storage. SecureString parameters can be encrypted with KMS. It is useful for application configuration and simpler secret/configuration use cases.

---

### Q14. How can AWS DevOps services integrate with CloudWatch?

**Answer:** Amazon CloudWatch provides metrics, logs, alarms, dashboards, and observability features for AWS resources and applications.

---
