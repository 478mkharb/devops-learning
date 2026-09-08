# Terraform — Interview Questions & Answers

This README contains practical Terraform interview questions covering Infrastructure as Code, Terraform workflow, providers, resources, data sources, variables, locals, expressions, dependencies, meta-arguments, lifecycle rules, modules, state, backends, drift, import, provisioning, `terraform_data`, `null_resource`, dynamic blocks, workspaces, locking, Sentinel, Terragrunt, and Terraform vs Ansible.

---

## 1. What is Terraform and why is it used in DevOps?

Terraform is a declarative Infrastructure as Code (IaC) tool that lets teams define and manage infrastructure using configuration files.

Instead of manually creating resources through a cloud console, infrastructure is described as code and Terraform uses providers to communicate with APIs.

### Why Terraform is used

- Infrastructure can be version-controlled.
- Environments can be reproduced consistently.
- Changes can be reviewed through Git pull requests.
- Terraform creates an execution plan before changes are applied.
- Dependencies between resources are handled through a dependency graph.
- Infrastructure can be managed across multiple providers.

Example:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-xxxxxxxx"
  instance_type = "t3.micro"
}
```

---

## 2. What is Infrastructure as Code (IaC)?

Infrastructure as Code means defining infrastructure through machine-readable configuration instead of creating and maintaining it manually.

| Manual Infrastructure | IaC |
|---|---|
| Console-driven | Code-driven |
| Harder to reproduce | Repeatable |
| Limited auditability | Git history provides change history |
| More manual work | Automated |
| Configuration can drift easily | Desired state is defined in code |

Terraform is a declarative IaC tool.

---

## 3. How is Terraform different from Ansible and CloudFormation?

| Feature | Terraform | Ansible | CloudFormation |
|---|---|---|---|
| Primary purpose | Infrastructure provisioning/management | Configuration management and automation | AWS infrastructure management |
| Model | Declarative | Task/playbook oriented | Declarative |
| State | Terraform state | No Terraform-style state model | AWS-managed stack state |
| Scope | Multi-provider | Multi-platform | AWS |
| Dependency graph | Yes | Not the same Terraform resource graph | Yes |
| Typical use | VPC, EC2, RDS, IAM | Packages, files, services, application deployment | AWS resources |

Terraform and Ansible are often complementary:

```text
Terraform
    ↓
Create VPC / EC2 / Load Balancer
    ↓
Ansible
    ↓
Configure OS / Install packages / Deploy application
```

---

## 4. What is a Terraform provider?

A provider is a plugin that allows Terraform to communicate with an external API.

Examples:

- AWS
- Azure
- Google Cloud
- Kubernetes
- GitHub

Example:

```hcl
provider "aws" {
  region = "ap-south-1"
}
```

The provider exposes resource types and data sources that Terraform can use.

---

## 5. What is a Terraform resource?

A resource represents an infrastructure object managed by Terraform.

Example:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-xxxxxxxx"
  instance_type = "t3.micro"
}
```

Here:

```text
aws_instance → resource type
web          → local resource name
```

Terraform can create, update, replace, and destroy resources according to configuration and state.

---

## 6. What does `terraform init` do?

`terraform init` initializes a Terraform working directory.

It can:

- Initialize the backend.
- Download required providers.
- Download modules.
- Install dependencies needed by the configuration.
- Create/update `.terraform.lock.hcl` when provider selections are resolved.

Example:

```bash
terraform init
```

### When should you run it again?

Common reasons include:

- Backend configuration changed.
- Provider requirements changed.
- Module sources changed.
- Provider version constraints changed.

---

## 7. What happens during `terraform plan`?

`terraform plan` evaluates the configuration against the known state and the current infrastructure information available through the providers, then proposes changes.

Typical result symbols:

| Symbol | Meaning |
|---|---|
| `+` | Create |
| `~` | Update in place |
| `-` | Destroy |
| `-/+` | Replace |
| No change | Resource remains unchanged |

Example:

```bash
terraform plan
```

Important: `terraform plan` does not normally apply the proposed infrastructure changes.

---

## 8. What does `terraform apply` do?

`terraform apply` executes a Terraform plan.

Without a saved plan file, Terraform generally creates a plan and asks for confirmation before applying it.

Example:

```bash
terraform apply
```

For an already reviewed plan:

```bash
terraform plan -out=tfplan
terraform apply tfplan
```

In CI/CD, a common pattern is:

```text
terraform fmt
      ↓
terraform validate
      ↓
terraform plan
      ↓
Review / approval
      ↓
terraform apply tfplan
```

---

## 9. What is `terraform destroy`?

`terraform destroy` removes resources managed by the current Terraform configuration/state.

Example:

```bash
terraform destroy
```

It should be treated carefully because it can remove production infrastructure and potentially cause data loss.

---

## 10. What is the standard Terraform workflow?

The common workflow is:

```text
Write
  ↓
terraform fmt
  ↓
terraform init
  ↓
terraform validate
  ↓
terraform plan
  ↓
Review
  ↓
terraform apply
```

When infrastructure is no longer required:

```bash
terraform destroy
```

---

## 11. What is the difference between `terraform validate` and `terraform plan`?

| `terraform validate` | `terraform plan` |
|---|---|
| Checks configuration syntax and internal consistency | Calculates proposed infrastructure changes |
| Does not require access to actual infrastructure for normal validation | Uses providers/state/infrastructure information |
| Faster | More involved |
| Useful early in CI | Used for change review |

Example:

```bash
terraform fmt -check
terraform init
terraform validate
terraform plan
```

---

## 12. What is the difference between a resource and a data source?

A resource is something Terraform **manages**.

A data source is something Terraform **reads**.

### Resource

```hcl
resource "aws_s3_bucket" "app" {
  bucket = "my-example-bucket"
}
```

Terraform manages that bucket.

### Data source

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
}
```

Terraform reads information about an existing AMI.

| Resource | Data Source |
|---|---|
| Creates/manages infrastructure | Reads existing information |
| `resource` block | `data` block |
| Managed lifecycle | Read-only from Terraform's perspective |

---

## 13. What is a provider block versus `required_providers`?

`required_providers` declares which provider plugins the configuration needs and can constrain versions/sources.

Example:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}
```

The `provider` block configures the provider:

```hcl
provider "aws" {
  region = "ap-south-1"
}
```

So:

```text
required_providers → Which provider/version/source?
provider            → How is the provider configured?
```

---

## 14. How do you configure multiple providers?

Provider aliases allow multiple configurations of the same provider.

Example:

```hcl
provider "aws" {
  region = "ap-south-1"
}

provider "aws" {
  alias  = "us_east"
  region = "us-east-1"
}
```

A resource can select the aliased provider:

```hcl
resource "aws_s3_bucket" "backup" {
  provider = aws.us_east
  bucket   = "my-backup-bucket-example"
}
```

This is useful for multi-Region or cross-account configurations.

---

## 15. What are input variables in Terraform?

Input variables allow a module to receive configurable values without hardcoding them.

```hcl
variable "instance_type" {
  type    = string
  default = "t3.micro"
}
```

Usage:

```hcl
resource "aws_instance" "web" {
  instance_type = var.instance_type
}
```

A value can be supplied using:

```bash
terraform apply -var="instance_type=t3.small"
```

---

## 16. What is the difference between `variables.tf`, `terraform.tfvars`, and `locals`?

| Item | Purpose |
|---|---|
| `variables.tf` | Declares input variables |
| `terraform.tfvars` | Supplies values for variables |
| `locals` | Defines reusable expressions/computed values inside the module |

Example:

```hcl
# variables.tf
variable "environment" {
  type = string
}
```

```hcl
# terraform.tfvars
environment = "dev"
```

```hcl
# locals.tf
locals {
  name_prefix = "otms-${var.environment}"
}
```

---

## 17. What are Terraform locals?

Locals assign names to expressions so they can be reused.

Example:

```hcl
locals {
  name_prefix = "${var.environment}-${var.application}"
}

resource "aws_s3_bucket" "app" {
  bucket = "${local.name_prefix}-data"
}
```

Locals do not represent user-supplied input in the same way variables do. They are calculated inside the module.

---

## 18. What are Terraform outputs?

Outputs expose selected values from a module.

Example:

```hcl
output "instance_id" {
  value = aws_instance.web.id
}
```

After apply:

```bash
terraform output instance_id
```

Outputs are useful for displaying important values and passing information from child modules to parent modules.

---

## 19. What are Terraform expressions?

Expressions calculate or reference values.

Examples:

```hcl
var.environment
```

```hcl
local.name_prefix
```

```hcl
aws_instance.web.id
```

```hcl
"${var.environment}-${var.application}"
```

Expressions can use operators, functions, conditionals, collections, references, and other Terraform language constructs.

---

## 20. What is implicit dependency in Terraform?

Terraform automatically creates a dependency when one resource references another.

Example:

```hcl
resource "aws_security_group" "web" {
  name = "web-sg"
}

resource "aws_instance" "web" {
  ami           = "ami-xxxxxxxx"
  instance_type = "t3.micro"

  vpc_security_group_ids = [
    aws_security_group.web.id
  ]
}
```

Terraform sees that the instance depends on the security group.

No explicit `depends_on` is required.

---

## 21. What is explicit dependency?

An explicit dependency is declared using `depends_on`.

Example:

```hcl
resource "aws_instance" "web" {
  depends_on = [
    aws_iam_role_policy_attachment.app
  ]

  ami           = "ami-xxxxxxxx"
  instance_type = "t3.micro"
}
```

Use `depends_on` when the dependency exists but is not visible through a direct expression reference.

---

## 22. What is `depends_on`?

`depends_on` tells Terraform that a resource or module depends on another object.

Example:

```hcl
resource "aws_instance" "app" {
  depends_on = [
    aws_iam_role_policy_attachment.app
  ]

  ami           = "ami-xxxxxxxx"
  instance_type = "t3.micro"
}
```

Prefer implicit dependencies when possible:

```hcl
subnet_id = aws_subnet.app.id
```

Use explicit `depends_on` only when Terraform cannot infer the dependency.

---

## 23. What are Terraform meta-arguments?

Meta-arguments are arguments understood by Terraform's resource/module model rather than by a particular provider resource schema.

| Meta-argument | Purpose |
|---|---|
| `count` | Create multiple instances using numeric indexes |
| `for_each` | Create multiple instances using keys |
| `depends_on` | Declare explicit dependencies |
| `provider` | Select a provider configuration |
| `lifecycle` | Control resource lifecycle behavior |

---

## 24. What is `count`?

`count` creates multiple instances of a resource using a numeric index.

Example:

```hcl
resource "aws_instance" "web" {
  count = 3

  ami           = "ami-xxxxxxxx"
  instance_type = "t3.micro"
}
```

Instances are addressed as:

```text
aws_instance.web[0]
aws_instance.web[1]
aws_instance.web[2]
```

`count` is useful when instances are interchangeable and index-based addressing is acceptable.

---

## 25. What is `for_each`?

`for_each` creates instances from a map or set.

Example:

```hcl
resource "aws_instance" "web" {
  for_each = {
    app1 = "t3.micro"
    app2 = "t3.small"
  }

  ami           = "ami-xxxxxxxx"
  instance_type = each.value
}
```

Resources are addressed by keys:

```text
aws_instance.web["app1"]
aws_instance.web["app2"]
```

---

## 26. What is the difference between `count` and `for_each`?

| `count` | `for_each` |
|---|---|
| Numeric indexes | Keys |
| `resource.x[0]` | `resource.x["app1"]` |
| Good for interchangeable instances | Good for uniquely identified instances |
| Index changes can cause address changes | Keys provide stable identity |

If individual instances have meaningful identities, `for_each` is often easier to manage.

---

## 27. What is the `lifecycle` meta-argument?

`lifecycle` controls how Terraform handles changes to a resource.

Common settings include:

```hcl
lifecycle {
  create_before_destroy = true
  prevent_destroy       = true
  ignore_changes        = [tags]
}
```

| Setting | Purpose |
|---|---|
| `create_before_destroy` | Create replacement before destroying old resource when possible |
| `prevent_destroy` | Prevent Terraform from destroying the resource through normal lifecycle operations |
| `ignore_changes` | Ignore selected configuration changes when planning |

---

## 28. What is `ignore_changes`?

`ignore_changes` tells Terraform to ignore changes to specified resource attributes when comparing configuration with remote state.

Example:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-xxxxxxxx"
  instance_type = "t3.micro"

  lifecycle {
    ignore_changes = [
      tags
    ]
  }
}
```

This can be useful when another system is intentionally responsible for modifying an attribute.

Do not use it simply to hide unexpected drift.

---

## 29. What is `create_before_destroy`?

It tells Terraform to create a replacement resource before destroying the existing one when the resource and provider support that lifecycle behavior.

Example:

```hcl
lifecycle {
  create_before_destroy = true
}
```

This can reduce downtime during replacement, but it may temporarily require additional capacity and can fail when uniqueness constraints prevent two copies from existing simultaneously.

---

## 30. What is `prevent_destroy`?

`prevent_destroy` prevents Terraform from planning a destroy for that resource through normal configuration changes.

Example:

```hcl
lifecycle {
  prevent_destroy = true
}
```

If Terraform would otherwise destroy the resource, the operation fails rather than proceeding.

It is useful for protecting critical resources, but it is not a replacement for backups or access controls.

---

## 31. What is a dynamic block?

A `dynamic` block generates repeated nested blocks from a collection.

Example:

```hcl
resource "aws_security_group" "web" {
  name = "web-sg"

  dynamic "ingress" {
    for_each = var.ingress_rules

    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}
```

It is useful when a provider resource has a repeatable nested block and the number/content of those blocks is variable.

---

## 32. What is the difference between `for_each` and `dynamic`?

| `for_each` | `dynamic` |
|---|---|
| Creates multiple resource/module instances | Generates repeated nested blocks |
| Resource/module level | Nested block level |
| `aws_instance.web["app1"]` | Repeated `ingress {}` blocks |
| Changes resource instance count | Changes nested configuration blocks |

A `dynamic` block does not create separate Terraform resources.

---

## 33. What is a Terraform module?

A module is a collection of Terraform configuration files grouped together and used as a reusable unit.

Example:

```text
modules/
└── ec2/
    ├── main.tf
    ├── variables.tf
    └── outputs.tf
```

Root module:

```hcl
module "web" {
  source        = "./modules/ec2"
  instance_type = "t3.micro"
}
```

Modules reduce duplication and provide reusable infrastructure building blocks.

---

## 34. What is the difference between a root module and a child module?

| Root Module | Child Module |
|---|---|
| Configuration in the working directory where Terraform is run | Module called by another module |
| Entry point for that Terraform operation | Reusable component |
| Can call child modules | Can itself call other modules |
| Commonly supplies environment-specific values | Commonly defines reusable infrastructure |

---

## 35. Why should we use modules?

Modules help with:

- Reuse
- Standardization
- Separation of concerns
- Smaller root configurations
- Centralized infrastructure patterns

Example:

```text
network module
      ↓
VPC + subnets + routes

compute module
      ↓
EC2 + security groups

database module
      ↓
RDS
```

---

## 36. How do you pass variables to a module?

Define a variable in the child module:

```hcl
variable "instance_type" {
  type = string
}
```

Pass it from the root module:

```hcl
module "web" {
  source        = "./modules/ec2"
  instance_type = "t3.small"
}
```

Inside the child module:

```hcl
resource "aws_instance" "web" {
  instance_type = var.instance_type
}
```

---

## 37. What is Terraform state?

Terraform state is Terraform's record of the infrastructure objects it manages and their relevant attributes.

It allows Terraform to associate configuration resources with real infrastructure objects and determine what needs to change.

Typical local state file:

```text
terraform.tfstate
```

For teams, state is commonly stored in a remote backend.

---

## 38. Why is Terraform state important?

State helps Terraform:

- Track managed resources.
- Detect changes.
- Calculate plans.
- Store resource identifiers and attributes.
- Manage resource instances.

State can contain sensitive information, so access must be controlled.

---

## 39. What is a Terraform backend?

A backend determines where Terraform stores state and how state operations are handled.

A common AWS setup uses an S3 backend.

Example:

```hcl
terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "network/terraform.tfstate"
    region = "us-east-1"
  }
}
```

---

## 40. Why use an S3 backend for Terraform state?

For team environments, remote state provides centralized storage rather than keeping state only on one developer's machine.

Benefits include:

- Centralized state
- Controlled access
- Durability
- Integration with CI/CD
- State versioning when bucket versioning is enabled

State locking/concurrency control should be configured according to the backend and Terraform version in use.

---

## 41. What is state locking and why is it important?

State locking prevents multiple Terraform operations from modifying the same state concurrently.

Example:

```text
Jenkins Job A
     ↓
terraform apply
     ↓
locks state

Jenkins Job B
     ↓
terraform apply
     ↓
waits/fails according to locking behavior
```

For shared CI/CD environments, state concurrency protection is essential.

---

## 42. What is state drift?

Drift occurs when real infrastructure changes outside Terraform's expected management process.

Example:

```text
Terraform configuration:
instance_type = "t3.micro"

Actual EC2:
instance_type = "t3.small"
```

If the change was made manually, Terraform may detect the difference during planning.

---

## 43. How do you detect Terraform drift?

A normal plan is commonly used:

```bash
terraform plan
```

Terraform reads relevant remote object information during planning and compares it with configuration and state.

If the real infrastructure differs, the plan may show a change.

Do not treat the older `terraform refresh` command as the primary modern workflow; normal plan/apply operations perform the necessary refresh behavior.

---

## 44. How do you fix Terraform drift?

The correct approach depends on who should own the changed value.

### If Terraform should own the change

Update the configuration to the desired value and run:

```bash
terraform plan
terraform apply
```

### If the manual change was intentional

Update Terraform configuration accordingly.

### If another system intentionally manages the attribute

Consider whether `ignore_changes` represents an intentional ownership boundary.

Do not blindly use `ignore_changes` to hide drift.

---

## 45. What happens if the Terraform state file is lost?

Terraform may lose its mapping between resource addresses and real infrastructure.

```text
Terraform state:
aws_instance.web → i-123456

State lost
     ↓
Terraform no longer has that mapping
```

Existing infrastructure may still exist, but Terraform cannot manage it correctly until state is recovered or reconstructed.

### Preferred recovery

1. Restore state from the remote backend/versioned backup.
2. If necessary, reconstruct state using import.
3. Run `terraform plan` and verify the result carefully.

---

## 46. What is `terraform import`?

Import associates an existing infrastructure object with a Terraform resource address.

Example:

```bash
terraform import aws_instance.web i-1234567890abcdef0
```

You normally need the corresponding resource block:

```hcl
resource "aws_instance" "web" {
  # configuration describing the imported object
}
```

After importing:

```bash
terraform plan
```

and adjust configuration until the plan matches the intended state.

---

## 47. What is the difference between refresh and import?

| Refresh/state refresh | Import |
|---|---|
| Updates Terraform's knowledge of an object already tracked in state | Brings an existing object under a Terraform resource address |
| Resource already has a state mapping | Resource mapping does not yet exist |
| Helps detect changes to managed infrastructure | Used to begin managing pre-existing infrastructure |

Simple example:

```text
Already managed:
aws_instance.web → i-123
        ↓
refresh/read current values


Not managed:
EC2 i-456
        ↓
terraform import
        ↓
aws_instance.web → i-456
```

---

## 48. What is the modern Terraform approach to importing resources?

Modern Terraform versions support declarative import blocks.

Example:

```hcl
import {
  to = aws_instance.web
  id = "i-1234567890abcdef0"
}
```

You still need the resource configuration and should run:

```bash
terraform plan
```

to review the proposed import.

Import blocks make import operations reviewable as configuration.

---

## 49. What is the Terraform dependency graph?

Terraform builds a dependency graph to determine which resources can be created, changed, or destroyed and in what order.

Example:

```text
VPC
 ↓
Subnet
 ↓
EC2
 ↓
Application
```

A direct reference creates an implicit dependency:

```hcl
subnet_id = aws_subnet.app.id
```

Terraform can execute independent resources in parallel when possible.

---

## 50. What is the `.terraform.lock.hcl` file?

`.terraform.lock.hcl` records selected provider versions and dependency checksums for a Terraform configuration.

It helps teams and CI/CD environments use consistent provider packages.

It should normally be committed to version control for a root module.

This is different from:

```text
terraform.tfstate
```

The lock file controls provider dependency selection/checksums; state tracks managed infrastructure.

---

## 51. What is `null_resource`?

`null_resource` is a resource that does not represent a real infrastructure object. It has historically been used with provisioners and triggers to run actions when its identity changes.

Example:

```hcl
resource "null_resource" "setup" {
  triggers = {
    version = var.app_version
  }

  provisioner "local-exec" {
    command = "./deploy.sh"
  }
}
```

Provisioners are generally a last resort because they can be harder to model reliably than provider resources.

---

## 52. What is `terraform_data` and how is it different from `null_resource`?

`terraform_data` is a built-in Terraform resource intended for storing values and expressing relationships in Terraform configurations without requiring the `null` provider.

Example:

```hcl
resource "terraform_data" "deployment" {
  input = var.app_version
}
```

| `terraform_data` | `null_resource` |
|---|---|
| Built into Terraform | Comes from the `null` provider |
| Useful for storing input/state and dependency relationships | Historically used for triggers/provisioners |
| Does not require the null provider | Requires the null provider |
| Modern choice for many data/dependency use cases | Older/common pattern |

Neither should replace a proper provider resource when an actual infrastructure object is required.

---

## 53. What are Terraform provisioners?

Provisioners allow Terraform to execute actions such as local or remote commands.

Example:

```hcl
provisioner "local-exec" {
  command = "echo hello"
}
```

Provisioners are generally discouraged when a provider resource, cloud-init/user data, configuration-management tool, or image-building tool can perform the job more reliably.

---

## 54. What is the difference between `local-exec` and `remote-exec`?

| `local-exec` | `remote-exec` |
|---|---|
| Runs on the machine executing Terraform | Runs commands on a remote target |
| Useful for local automation/integration | Requires remote connectivity/configuration |
| Example: invoke a local script | Example: configure a remote server |

Example:

```hcl
provisioner "local-exec" {
  command = "echo ${self.id}"
}
```

---

## 55. What are Terraform workspaces?

Terraform CLI workspaces allow multiple state instances to be associated with the same configuration.

Example:

```bash
terraform workspace new dev
terraform workspace new prod
terraform workspace select dev
```

Each workspace has separate state for the same configuration.

Workspaces are not a universal replacement for separate environment configurations. For complex environments with substantially different architecture, separate root modules/configuration and explicit state separation can be clearer.

---

## 56. What is the difference between workspaces and separate state/configurations?

| Workspaces | Separate root/state configurations |
|---|---|
| Same configuration with multiple state instances | Explicitly separated configurations/state |
| Convenient for similar environments | Better for substantially different environments |
| Easy to select the wrong workspace | Environment separation can be more explicit |
| Less configuration duplication | More explicit isolation |

Example:

```text
workspace:
dev
prod
```

versus:

```text
environments/
├── dev/
│   └── main.tf
└── prod/
    └── main.tf
```

---

## 57. What are important Terraform CI/CD best practices?

Use:

- Remote state.
- State locking/concurrency protection.
- Version-controlled Terraform code.
- `.terraform.lock.hcl`.
- `terraform fmt -check`.
- `terraform validate`.
- Plan review.
- Controlled approval for production.
- Least-privilege cloud credentials.
- Secret management outside source code.
- Separate state for independently managed environments.
- Module versioning.
- Policy/security scanning.

A strong pipeline makes the plan reviewable and the apply reproducible.

---

## 58. How would you use Terraform in a Jenkins CI/CD pipeline?

A common pipeline is:

```text
Git commit
   ↓
Jenkins
   ↓
terraform fmt -check
   ↓
terraform init
   ↓
terraform validate
   ↓
terraform plan
   ↓
Approval
   ↓
terraform apply tfplan
```

Example:

```groovy
stage('Terraform Plan') {
    steps {
        sh '''
            terraform init
            terraform validate
            terraform plan -out=tfplan
        '''
    }
}
```

For production, the apply stage should consume the reviewed plan artifact rather than silently creating a new plan.

---

## 59. What is Sentinel in Terraform?

Sentinel is HashiCorp's policy-as-code framework used with supported HashiCorp products to enforce governance policies.

Policies can check:

- Required tags
- Allowed instance types
- Restricted regions
- Security requirements
- Public resource restrictions

Conceptually:

```text
terraform plan
      ↓
Policy evaluation
      ↓
Allowed / rejected according to policy
```

---

## 60. What are Sentinel policy enforcement levels?

Common Sentinel enforcement levels are:

| Level | Meaning |
|---|---|
| Advisory | Reports the policy result but does not block |
| Soft Mandatory | Must pass unless an authorized override is used |
| Hard Mandatory | Must pass and cannot be overridden through the normal policy override mechanism |

Exact behavior depends on the HashiCorp product and policy configuration.

---

## 61. Give a real Sentinel policy example.

Suppose an organization requires an `Environment` tag.

```text
Terraform plan
      ↓
Check managed resources
      ↓
Environment tag present?
      ↓
YES → allow
NO  → policy failure
```

Another example is restricting EC2 instance types:

```text
Allowed:
t3.micro
t3.small
t3.medium

Blocked:
unapproved/high-cost types
```

---

## 62. What is Terragrunt?

Terragrunt is a wrapper/tooling layer around Terraform/OpenTofu configurations that can help reduce repetitive configuration and coordinate multiple infrastructure units.

Common concepts include:

- `terragrunt.hcl`
- Inputs
- Dependencies
- Remote-state configuration
- Environment/account organization

Terragrunt is not part of Terraform itself.

---

## 63. Why is Terragrunt used?

A common motivation is reducing repetition when managing many Terraform root modules/environments.

Example:

```text
live/
├── dev/
│   ├── vpc/
│   └── ec2/
└── prod/
    ├── vpc/
    └── ec2/
```

Terragrunt can help coordinate configuration and dependencies across these units.

| Terraform | Terragrunt |
|---|---|
| IaC engine | Wrapper/orchestration/configuration layer |
| Defines resources/modules | Helps organize and coordinate Terraform configurations |
| Uses `.tf` | Commonly uses `terragrunt.hcl` |
| Maintains state | Can generate/configure Terraform backend settings |

---

## 64. What is the difference between Terraform and Ansible in a real DevOps project?

A common architecture is:

```text
Terraform
   ↓
VPC
   ↓
Subnets
   ↓
Security Groups
   ↓
EC2
   ↓
Load Balancer
   ↓
Ansible
   ↓
Install packages
Configure services
Deploy application
```

Terraform is responsible for infrastructure lifecycle.

Ansible can configure the operating system and application environment.

---

## 65. What happens if two developers run `terraform apply` at the same time?

If both operations use the same properly configured remote state and locking/concurrency mechanism, one operation should acquire the lock while the other waits or fails according to the backend/tool behavior.

Without appropriate state concurrency protection, simultaneous operations can create race conditions.

---

## 66. What is the difference between `terraform plan -out=tfplan` and a normal plan?

A normal:

```bash
terraform plan
```

prints a proposed plan.

With:

```bash
terraform plan -out=tfplan
```

Terraform saves the generated plan to a plan file.

You can then apply that saved plan:

```bash
terraform apply tfplan
```

This is useful in CI/CD because the reviewed plan can be separated from the later apply step.

---

## 67. What is the difference between `terraform fmt`, `validate`, `plan`, and `apply`?

| Command | Main purpose | Changes infrastructure? |
|---|---|---:|
| `terraform fmt` | Format Terraform files | No |
| `terraform validate` | Validate configuration | No |
| `terraform plan` | Calculate proposed changes | No |
| `terraform apply` | Execute infrastructure changes | Yes |

Typical order:

```bash
terraform fmt
terraform init
terraform validate
terraform plan
terraform apply
```

---

## 68. How does Terraform achieve idempotency?

Terraform works toward the desired state described by configuration.

If infrastructure already matches the desired configuration, another plan should normally show no changes.

Example:

```text
Configuration
     ↓
Desired state = t3.micro
     ↓
Actual state = t3.micro
     ↓
terraform plan
     ↓
No changes
```

If actual state differs:

```text
Desired = t3.micro
Actual  = t3.small
     ↓
Terraform plans a corrective change
```

---

## 69. What is Terraform's declarative model?

In a declarative model, you describe what the desired infrastructure should look like rather than writing every API operation required to produce it.

Example:

```hcl
resource "aws_instance" "web" {
  instance_type = "t3.micro"
}
```

You don't normally write individual API calls for creation, dependency ordering, retries, and state tracking. Terraform determines the required actions.

---

## 70. What is the difference between `terraform state` commands and normal resource configuration?

Normal configuration describes the desired infrastructure:

```hcl
resource "aws_instance" "web" {
  instance_type = "t3.micro"
}
```

State commands operate on Terraform's state representation.

Examples:

```bash
terraform state list
terraform state show aws_instance.web
terraform state mv aws_instance.web aws_instance.application
```

These commands can be useful for state management and address changes, but they should be used carefully, especially against production state.

---

# Quick Reference

## Core Terraform Commands

| Command | Purpose |
|---|---|
| `terraform init` | Initialize backend/providers/modules |
| `terraform fmt` | Format configuration |
| `terraform validate` | Validate configuration |
| `terraform plan` | Preview changes |
| `terraform apply` | Apply changes |
| `terraform destroy` | Destroy managed infrastructure |
| `terraform output` | Read outputs |
| `terraform import` | Associate existing infrastructure with a resource address |
| `terraform state` | Inspect/manage Terraform state |
| `terraform workspace` | Manage CLI workspaces |

## Core Terraform Concepts

```text
Provider
   ↓
Resource / Data Source
   ↓
Dependency Graph
   ↓
State
   ↓
Plan
   ↓
Apply
```

## Common Terraform Files

```text
main.tf                 → Resources/modules
variables.tf            → Input declarations
terraform.tfvars        → Variable values
outputs.tf              → Output declarations
locals.tf               → Local expressions
versions.tf             → Terraform/provider requirements
backend.tf              → Backend configuration
.terraform.lock.hcl     → Provider dependency lock information
terraform.tfstate       → Terraform state when using local state
```

## Interview Mental Model

> **Terraform is a declarative IaC engine. Providers connect Terraform to APIs, resources represent managed infrastructure, data sources read existing information, state maps Terraform resources to real infrastructure, the dependency graph determines ordering, `plan` previews changes, and `apply` executes them.**

For team environments, remote state, concurrency protection, version control, testing, policy enforcement, and controlled CI/CD approvals are essential parts of a production Terraform workflow.
