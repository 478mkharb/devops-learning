# AWS DevOps Services

### Q1. What is CodeCommit?

**Answer:** AWS CodeCommit is a managed private Git repository service. It provides source control repositories that can integrate with AWS CI/CD workflows.

---

### Q2. What is CodeBuild?

**Answer:** AWS CodeBuild is a managed build service that compiles source code, runs tests, and produces build artifacts without requiring you to manage build servers.

---

### Q3. What is CodeDeploy?

**Answer:** AWS CodeDeploy automates application deployments to supported targets such as EC2, Lambda, and ECS. It supports deployment strategies including in-place and blue/green.

---

### Q4. What is CodePipeline?

**Answer:** AWS CodePipeline is a CI/CD orchestration service that automates stages such as source, build, test, approval, and deployment.

---

### Q5. What is CodeArtifact?

**Answer:** AWS CodeArtifact is a managed artifact repository for storing and sharing software packages and dependencies used by build systems.

---

### Q6. What is in-place deployment?

**Answer:** An in-place deployment updates the existing fleet of servers by installing the new application version on those instances. It is simpler but can temporarily reduce capacity or availability during deployment.

---

### Q7. What is blue/green deployment?

**Answer:** A blue/green deployment uses two environments: the current blue environment and the new green environment. Traffic is shifted to green after validation, enabling quick rollback by switching traffic back.

---

### Q8. What is canary deployment?

**Answer:** A canary deployment sends a small portion of traffic or instances to the new version first. If the new version behaves correctly, the rollout is expanded gradually.

---

### Q9. How can CodeDeploy deploy to EC2?

**Answer:** CodeDeploy uses an agent installed on target EC2 instances. A deployment group identifies the instances, CodeDeploy delivers the application revision, and the agent executes the deployment lifecycle hooks defined by the application's deployment specification.

---

### Q10. How can pipelines integrate with CloudFormation or Terraform?

**Answer:** AWS CloudFormation is an infrastructure-as-code service that provisions AWS resources from declarative templates. A stack represents a deployed collection of resources managed together.

---

### Q11. What is Systems Manager?

**Answer:** AWS Systems Manager provides operational tools for managing AWS and hybrid resources, including Session Manager, Run Command, Patch Manager, Parameter Store, and automation capabilities.

---

### Q12. What is Session Manager?

**Answer:** Systems Manager Session Manager provides secure interactive access to managed instances without requiring inbound SSH or RDP ports. Access is controlled through IAM and Systems Manager prerequisites.

---

### Q13. What is Parameter Store?

**Answer:** Systems Manager Parameter Store stores configuration values and parameters in a hierarchical namespace. SecureString parameters can be encrypted with KMS.

---

### Q14. How can AWS DevOps services integrate with CloudWatch?

**Answer:** Amazon CloudWatch provides metrics, logs, alarms, dashboards, and observability capabilities for AWS resources and applications.

---
