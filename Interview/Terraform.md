# 1. Terraform Fundamentals

## Q1. What is Terraform and why is it used in DevOps?

Terraform is an **Infrastructure as Code (IaC)** tool from HashiCorp. It lets engineers define infrastructure declaratively in configuration files and then provision and manage that infrastructure through provider APIs.

Terraform is commonly used to:

- Provision cloud infrastructure.
- Version-control infrastructure configuration.
- Review infrastructure changes through `terraform plan`.
- Create repeatable environments.
- Manage dependencies between resources.
- Detect and reconcile infrastructure changes.
- Integrate infrastructure provisioning into CI/CD.

**Example:**

```hcl
provider "aws" {
  region = "ap-south-1"
}

resource "aws_instance" "web" {
  ami           = "ami-xxxxxxxx"
  instance_type = "t3.micro"

  tags = {
    Name = "web-server"
  }
}
```

The important interview distinction is:

```text
Terraform configuration
        ↓
Desired state
        ↓
Terraform plan
        ↓
Provider API
        ↓
Actual infrastructure
```

**Interview follow-up:** Why is Terraform called declarative?

Because you describe **what the final infrastructure should look like**, rather than writing every API operation required to create it.

---

## Q2. What is Infrastructure as Code (IaC)?

Infrastructure as Code means defining and managing infrastructure using machine-readable configuration instead of manually creating resources through a console.

### Main benefits

| Benefit | Meaning |
|---|---|
| Version control | Infrastructure changes can be reviewed and tracked |
| Repeatability | The same configuration can create consistent environments |
| Automation | Provisioning can be integrated into CI/CD |
| Reviewability | Changes can be reviewed before applying |
| Consistency | Reduces manual configuration differences |
| Recovery | Infrastructure can be recreated from code |

### Declarative vs procedural thinking

```text
Procedural:
Create VPC
Create subnet
Create route table
Attach route table
Create EC2
...

Declarative:
I want this VPC, subnet, route table and EC2 configuration.
Terraform determines the required operations.
```

**Interview follow-up:** What does idempotency mean?

Repeatedly applying the same desired configuration should converge on the same infrastructure rather than continually creating duplicate resources.

---

## Q3. How is Terraform different from Ansible and CloudFormation?

| Feature | Terraform | Ansible | CloudFormation |
|---|---|---|---|
| Primary use | Infrastructure provisioning | Configuration management / automation | AWS infrastructure provisioning |
| Model | Declarative | Task-oriented / procedural | Declarative |
| State | Terraform state | No Terraform-style state | AWS stack management |
| Cloud support | Multi-provider | Multi-platform | AWS |
| Dependency graph | Yes | Task ordering | Yes |
| Typical example | VPC, ALB, EC2 | Install/configure NGINX | AWS VPC/EC2 stack |

A common DevOps architecture is:

```text
Terraform
   ↓
Create VPC / EC2 / ALB
   ↓
Ansible
   ↓
Install packages and configure application
```

**Interview follow-up:** Why not use Ansible for everything?

Ansible can provision infrastructure, but Terraform is purpose-built around infrastructure lifecycle management, dependency graphs, planning, and state-based reconciliation.

---

# 2. Providers, Resources and Data Sources

## Q4. What are Terraform providers?

A provider is a plugin that lets Terraform communicate with an external API.

Examples include:

- AWS
- Azure
- Google Cloud
- Kubernetes
- GitHub
- many SaaS and infrastructure platforms

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}

provider "aws" {
  region = "ap-south-1"
}
```

### What a provider does

```text
Terraform
    ↓
Provider plugin
    ↓
External API
    ↓
AWS / Azure / GCP / Kubernetes / ...
```

Provider plugins are normally installed during:

```bash
terraform init
```

**Interview follow-up:** Why should provider versions be constrained?

To make builds predictable and prevent an unexpected provider upgrade from changing infrastructure behavior.

---

## Q5. What is a Terraform resource?

A resource represents an infrastructure object that Terraform manages.

Examples:

```text
aws_vpc
aws_subnet
aws_instance
aws_security_group
aws_lb
aws_s3_bucket
```

Example:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-xxxxxxxx"
  instance_type = "t3.micro"
}
```

Terraform tracks the resource in state and can create, update, replace, or destroy it according to configuration and lifecycle rules.

### Resource addressing

```text
aws_instance.web
```

With `count`:

```text
aws_instance.web[0]
```

With `for_each`:

```text
aws_instance.web["api"]
```

**Interview follow-up:** What is the difference between a resource type and a resource instance?

`aws_instance` is the resource type; `aws_instance.web` identifies a particular instance declared by the configuration.

---

## Q6. What is the difference between a resource and a data source?

A **resource** manages infrastructure. A **data source** reads information about infrastructure.

| Resource | Data source |
|---|---|
| Creates/manages objects | Reads existing information |
| Lifecycle is managed by Terraform | Read-only from Terraform's perspective |
| Can create/update/delete | Does not create/delete the object |
| Example: `aws_instance` | Example: `aws_ami` |

Example data source:

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true

  owners = ["099720109477"]

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/*"]
  }
}
```

Then:

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
}
```

The data source dynamically obtains a value that the resource consumes.

**Interview follow-up:** When would you use a data source?

When the required object/value already exists or is managed elsewhere and Terraform needs to read information about it.

---

## Q7. Can multiple resources use one provider?

Yes. A provider configuration can be used by many resources.

```hcl
provider "aws" {
  region = "ap-south-1"
}

resource "aws_instance" "web" {
  ami           = "ami-xxxxxxxx"
  instance_type = "t3.micro"
}

resource "aws_s3_bucket" "logs" {
  bucket = "example-log-bucket"
}
```

Both resources use the default AWS provider configuration.

---

## Q8. How do you configure multiple providers or multiple AWS regions?

Use **provider aliases**.

```hcl
provider "aws" {
  region = "ap-south-1"
}

provider "aws" {
  alias  = "us"
  region = "us-east-1"
}

resource "aws_instance" "india" {
  ami           = "ami-xxxxxxxx"
  instance_type = "t3.micro"
}

resource "aws_instance" "usa" {
  provider      = aws.us
  ami           = "ami-yyyyyyyy"
  instance_type = "t3.micro"
}
```

### Why aliases are useful

- Multiple AWS regions.
- Multiple AWS accounts.
- Cross-region resources.
- Disaster-recovery configurations.

**Interview follow-up:** What happens if you forget `provider = aws.us`?

The resource uses the default provider configuration, which may cause it to be created in the wrong region/account.

---

# 3. Terraform Workflow and Commands

## Q9. What is the Terraform workflow?

The standard workflow is:

```text
Write
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
  ↓
Manage / modify
  ↓
terraform plan
  ↓
terraform apply
```

`destroy` is used only when the managed infrastructure should be removed.

### Common commands

| Command | Purpose |
|---|---|
| `terraform init` | Initialize directory/backend/providers/modules |
| `terraform fmt` | Format configuration |
| `terraform validate` | Validate configuration syntax and consistency |
| `terraform plan` | Preview changes |
| `terraform apply` | Apply changes |
| `terraform destroy` | Destroy managed infrastructure |
| `terraform output` | Display outputs |
| `terraform state` | Inspect/manage state |
| `terraform import` | Associate existing infrastructure with state |

---

## Q10. What does `terraform init` do?

`terraform init` prepares the working directory for Terraform operations.

It can:

1. Initialize the backend.
2. Download provider plugins.
3. Download modules.
4. Create/update `.terraform`.
5. Create/update `.terraform.lock.hcl`.
6. Prepare the dependency environment.

Typical command:

```bash
terraform init
```

### Simplified internal flow

```text
terraform init
     |
     +--> Read Terraform configuration
     |
     +--> Configure backend
     |
     +--> Resolve providers
     |
     +--> Install providers
     |
     +--> Download modules
     |
     +--> Update lock file
```

**Interview follow-up:** When should you run `terraform init` again?

Typically after changes to backend configuration, required providers/provider constraints, or module sources, or when initializing a fresh working directory.

---

## Q11. What happens during `terraform plan`?

`terraform plan` determines what Terraform **would change** without applying those changes.

Conceptually:

```text
Configuration
     +
State
     +
Current infrastructure information
     ↓
Terraform
     ↓
Execution plan
```

The plan can show:

```text
+ create
~ update in-place
- destroy
-/+ replace
```

### Important point

`plan` is a **preview**. It does not normally make the requested infrastructure changes.

Example:

```bash
terraform plan
```

For CI/CD, a useful pattern is:

```bash
terraform plan -out=tfplan
terraform apply tfplan
```

This lets the apply use the reviewed saved plan.

---

## Q12. What does `terraform apply` do?

`terraform apply` executes the planned changes.

Typical flow:

```text
Read configuration
      ↓
Build plan
      ↓
Approval
      ↓
Execute changes
      ↓
Update state
      ↓
Show outputs
```

Example:

```bash
terraform apply
```

For automation:

```bash
terraform apply -auto-approve
```

Use `-auto-approve` carefully, especially for production infrastructure.

---

## Q13. What is `terraform destroy`?

`terraform destroy` removes resources managed by the current Terraform configuration/state.

```bash
terraform destroy
```

Terraform evaluates dependencies so that resources are destroyed in an appropriate order.

### Example

```text
EC2 / ALB dependency
       ↓
Dependent resource removed
       ↓
Underlying resource removed
```

**Interview warning:** `destroy` is a destructive operation and should be protected in production CI/CD.

---

## Q14. Why use `terraform validate` and `terraform fmt`?

### `terraform fmt`

Formats Terraform configuration consistently.

```bash
terraform fmt
```

### `terraform validate`

Checks whether the configuration is syntactically valid and internally consistent.

```bash
terraform validate
```

A good CI pipeline often does:

```text
terraform fmt
terraform validate
terraform plan
approval
terraform apply
```

---

# 4. Dependencies

## Q15. What is an implicit dependency?

Terraform automatically creates a dependency when one resource references another.

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

Terraform sees:

```text
aws_security_group.web
          ↓
aws_instance.web
```

No `depends_on` is required.

### Why implicit dependencies are preferred

They express the actual data relationship in the configuration and let Terraform build an accurate dependency graph.

---

## Q16. What is an explicit dependency?

An explicit dependency is manually declared with `depends_on`.

```hcl
resource "aws_iam_role" "app" {
  name = "app-role"
  # ...
}

resource "aws_instance" "app" {
  ami           = "ami-xxxxxxxx"
  instance_type = "t3.micro"

  depends_on = [
    aws_iam_role.app
  ]
}
```

Use this when the dependency is real but **not represented by a direct attribute reference**.

---

## Q17. What is `depends_on`?

`depends_on` is a Terraform meta-argument used to declare an explicit dependency.

### Implicit vs explicit

| Type | How dependency is created | Preferred? |
|---|---|---|
| Implicit | Resource references another resource | Yes, normally |
| Explicit | `depends_on` | Use when necessary |

### Interview trap

Do not use `depends_on` everywhere simply to force ordering.

Terraform already understands many dependencies through references.

**Interview follow-up:** What is the problem with unnecessary `depends_on`?

It can make Terraform's dependency graph more conservative than necessary and can cause broader ordering/replacement behavior.

---

# 5. Variables, tfvars, Locals and Outputs

## Q18. What are input variables?

Input variables parameterize Terraform configuration.

```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}
```

Use it:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-xxxxxxxx"
  instance_type = var.instance_type
}
```

### Why variables matter

Without variables:

```text
Hardcoded configuration
       ↓
Difficult reuse
```

With variables:

```text
Same configuration
       ↓
Different inputs
       ↓
dev / stage / prod
```

---

## Q19. What is the difference between `variables.tf` and `terraform.tfvars`?

| File | Purpose |
|---|---|
| `variables.tf` | Declares variables and their schema |
| `terraform.tfvars` | Supplies variable values |

Example `variables.tf`:

```hcl
variable "environment" {
  type = string
}
```

Example `terraform.tfvars`:

```hcl
environment = "dev"
```

### Easy interview answer

> `variables.tf` defines **what variables exist**; `terraform.tfvars` defines **what values they receive**.

---

## Q20. What are Terraform locals?

Locals define reusable values within a module.

```hcl
locals {
  environment = "dev"
  application = "otms"

  common_name = "${local.environment}-${local.application}"
}
```

Use:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-xxxxxxxx"
  instance_type = "t3.micro"

  tags = {
    Name        = local.common_name
    Environment = local.environment
  }
}
```

### Variables vs locals

| Variables | Locals |
|---|---|
| Input from outside the module | Internal calculated/reusable values |
| Can be supplied by caller/environment | Defined by configuration |
| `var.name` | `local.name` |

---

## Q21. How do variables, tfvars and locals work together?

A common environment pattern is:

```text
variables.tf
     ↓
Defines inputs
     ↓
terraform.tfvars
     ↓
Supplies values
     ↓
locals
     ↓
Calculates common values
     ↓
resources
```

Example:

```hcl
variable "environment" {
  type = string
}

variable "application" {
  type = string
}

locals {
  name_prefix = "${var.environment}-${var.application}"
}
```

`terraform.tfvars`:

```hcl
environment = "dev"
application = "otms"
```

Resource:

```hcl
resource "aws_security_group" "app" {
  name = "${local.name_prefix}-sg"
}
```

Result:

```text
dev-otms-sg
```

---

## Q22. What are Terraform output values?

Outputs expose useful values after Terraform operations.

```hcl
output "instance_private_ip" {
  value = aws_instance.web.private_ip
}
```

Outputs are useful for:

- Showing deployment information.
- Passing values from child modules to parent modules.
- CI/CD automation.
- Reading values from remote state.

Example:

```hcl
output "vpc_id" {
  value = aws_vpc.main.id
}
```

---

# 6. Meta-Arguments

## Q23. What are Terraform meta-arguments?

Meta-arguments change how Terraform manages resources/modules rather than describing a provider-specific infrastructure property.

Important examples:

| Meta-argument | Purpose |
|---|---|
| `count` | Multiple instances by index |
| `for_each` | Multiple instances by key |
| `depends_on` | Explicit dependency |
| `provider` | Select provider configuration |
| `lifecycle` | Control resource lifecycle |

---

## Q24. What is `count`?

`count` creates multiple instances using a numeric index.

```hcl
resource "aws_instance" "web" {
  count         = 3
  ami           = "ami-xxxxxxxx"
  instance_type = "t3.micro"

  tags = {
    Name = "web-${count.index}"
  }
}
```

Terraform addresses them as:

```text
aws_instance.web[0]
aws_instance.web[1]
aws_instance.web[2]
```

### Best use case

Use `count` when instances are essentially interchangeable.

---

## Q25. What is `for_each`?

`for_each` creates resource instances from a map or set of strings.

```hcl
resource "aws_instance" "web" {
  for_each = {
    api = "t3.small"
    web = "t3.micro"
  }

  ami           = "ami-xxxxxxxx"
  instance_type = each.value

  tags = {
    Name = each.key
  }
}
```

Addresses become:

```text
aws_instance.web["api"]
aws_instance.web["web"]
```

### Why keys matter

The identity of each instance is based on its key rather than a numeric position.

---

## Q26. What is the difference between `count` and `for_each`?

| `count` | `for_each` |
|---|---|
| Numeric index | Key-based identity |
| `count.index` | `each.key`, `each.value` |
| Good for interchangeable instances | Good for distinct instances |
| Index changes can cause address changes | Stable keys are generally safer |
| Works with a number | Works with map/set of strings |

Example:

```hcl
count = 3
```

vs:

```hcl
for_each = toset(["web", "api", "worker"])
```

### Interview scenario

If you have:

```text
web
api
worker
```

and may later remove `api`, `for_each` is usually more suitable because each object has a meaningful identity.

---

# 7. Lifecycle Management

## Q27. What is the `lifecycle` meta-argument?

`lifecycle` changes Terraform's default create/update/destroy behavior.

Common arguments:

- `prevent_destroy`
- `create_before_destroy`
- `ignore_changes`

Example:

```hcl
resource "aws_db_instance" "db" {
  # ...

  lifecycle {
    prevent_destroy = true
  }
}
```

This protects the resource from Terraform destroy/replacement actions that would violate the lifecycle rule.

---

## Q28. What is `prevent_destroy`?

`prevent_destroy` prevents Terraform from destroying a resource while the lifecycle rule remains enabled.

```hcl
lifecycle {
  prevent_destroy = true
}
```

Useful for:

- Critical databases.
- Production infrastructure.
- Important stateful resources.

It is a safety mechanism, **not a replacement for backups**.

---

## Q29. What is `ignore_changes`?

`ignore_changes` tells Terraform not to act on selected attribute changes.

```hcl
resource "aws_instance" "web" {
  # ...

  lifecycle {
    ignore_changes = [
      tags["LastModified"]
    ]
  }
}
```

### Appropriate use

Use it when an attribute is intentionally controlled outside Terraform.

### Interview warning

Do not use `ignore_changes` merely to hide unwanted drift. If Terraform should own the attribute, the better solution is normally to reconcile the infrastructure and configuration.

---

## Q30. What is `create_before_destroy`?

It instructs Terraform to create the replacement resource before destroying the old one when replacement is required.

```hcl
lifecycle {
  create_before_destroy = true
}
```

Useful for:

- Reducing downtime.
- Immutable infrastructure.
- Certain blue/green replacement patterns.

### Important caveat

It does not magically guarantee zero downtime. The resource must support having old and new instances coexist, and dependencies, names, quotas, and capacity can affect the result.

---

# 8. Expressions and Dynamic Blocks

## Q31. What are Terraform expressions?

Expressions calculate or reference values.

Common categories:

- Literal values.
- Variable references.
- Resource references.
- Local references.
- Function calls.
- Conditional expressions.
- Collection expressions.

Example:

```hcl
variable "environment" {
  type    = string
  default = "dev"
}

locals {
  instance_type = var.environment == "prod" ? "t3.medium" : "t3.micro"
}
```

The conditional expression is:

```text
condition ? true_value : false_value
```

---

## Q32. What is a dynamic block?

A dynamic block generates repeated **nested blocks** inside a resource.

Example:

```hcl
variable "ports" {
  type    = list(number)
  default = [80, 443]
}

resource "aws_security_group" "web" {
  name = "web-sg"

  dynamic "ingress" {
    for_each = var.ports

    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
}
```

Conceptually:

```text
var.ports
   ↓
dynamic "ingress"
   ↓
ingress block for 80
ingress block for 443
```

---

## Q33. What is the difference between `for_each` and a dynamic block?

This is a common interview question.

| `for_each` on resource | `dynamic` block |
|---|---|
| Creates multiple resource instances | Creates repeated nested blocks |
| Resource identity is created | No separate resource identity |
| Example: multiple EC2 instances | Example: multiple ingress rules |
| Addressable in state | Nested inside parent resource |

Think:

```text
for_each
   ↓
resource A
resource B
resource C
```

while:

```text
dynamic
   ↓
one resource
   ├── nested block
   ├── nested block
   └── nested block
```

---

# 9. Terraform State

## Q34. What is Terraform state?

Terraform state records information that Terraform uses to map configuration resources to real infrastructure objects.

Typical local state:

```text
terraform.tfstate
```

A simplified relationship is:

```text
Terraform configuration
        ↓
aws_instance.web
        ↓
State
        ↓
i-0123456789
        ↓
Actual AWS EC2 instance
```

State can contain resource IDs, attributes, dependencies, and other information required for Terraform to manage infrastructure.

---

## Q35. Why is Terraform state important?

State is important because Terraform needs a reliable mapping between its resource addresses and real infrastructure.

It helps Terraform:

- Track resource identities.
- Determine what has changed.
- Build/update infrastructure.
- Avoid treating known resources as completely unmanaged.
- Coordinate team workflows when stored remotely.

### Important security point

Terraform state can contain sensitive information depending on the resources/configuration.

Therefore:

- Protect access.
- Encrypt remote storage.
- Use IAM permissions.
- Avoid committing state to Git.
- Treat state as sensitive infrastructure data.

---

## Q36. What is a remote backend?

A backend determines where Terraform stores state and, depending on the backend, how state operations are coordinated.

For a team, remote state is generally preferred over a local `terraform.tfstate`.

Example AWS S3 backend:

```hcl
terraform {
  backend "s3" {
    bucket  = "company-terraform-state"
    key     = "dev/network/terraform.tfstate"
    region  = "ap-south-1"
    encrypt = true
  }
}
```

### Why remote state?

| Requirement | Local state | Remote state |
|---|---:|---:|
| Shared team access | Poor | Good |
| Centralized state | No | Yes |
| Central security controls | Limited | Stronger |
| Recovery/versioning | Manual | Can be designed robustly |
| CI/CD use | Possible | Preferred |

---

## Q37. What is state locking and why is it important?

State locking prevents multiple Terraform operations from modifying the same state concurrently when the backend supports locking.

Without appropriate locking:

```text
Engineer A → terraform apply
Engineer B → terraform apply
             ↓
        concurrent state changes
             ↓
       risk of corruption/conflict
```

With locking:

```text
Engineer A → lock → apply → unlock
Engineer B → waits
```

For AWS S3-based designs, the exact locking mechanism depends on the Terraform/backend capabilities and version in use. Do not blindly assume older DynamoDB locking guidance applies to every current Terraform setup.

---

# 10. Drift, Refresh and Import

## Q38. What is Terraform state drift?

Drift occurs when infrastructure changes outside Terraform's expected management path.

Example:

```text
Terraform says:
instance_type = t3.micro

AWS console changed it to:
instance_type = t3.small
```

Now configuration/state/real infrastructure may no longer agree.

### Common causes

- Manual console changes.
- Another automation system changes resources.
- Cloud-managed behavior.
- Out-of-band operational changes.

---

## Q39. How do you detect drift?

A common approach is:

```bash
terraform plan
```

Terraform reads current infrastructure information and compares it with its configuration/state model.

A CI/CD system can periodically run plans to identify unexpected changes.

### Interview answer

> I detect drift by running a plan and reviewing differences between the desired configuration and the observed infrastructure. For important environments, I can automate periodic plan checks.

---

## Q40. How do you fix drift?

The correct fix depends on whether the external change was intentional.

### Case 1 — Terraform should win

If someone manually changed:

```text
t3.micro → t3.small
```

but Terraform configuration says:

```text
t3.micro
```

then apply Terraform configuration to reconcile the resource.

### Case 2 — External change is intentional

Update Terraform configuration to represent the new desired state.

### Case 3 — Resource is not managed by Terraform

Use import/configuration adoption when appropriate.

### Case 4 — Attribute is intentionally externally managed

Consider `ignore_changes`, but only when that ownership boundary is deliberate.

---

## Q41. What is the difference between refresh and import?

### Refresh / state synchronization

The idea of refreshing is to update Terraform's knowledge of resources it already manages by reading current remote values.

Modern Terraform workflows generally perform refresh-style reconciliation as part of planning/applying; `terraform refresh` as a standalone command is no longer the normal recommended workflow.

### Import

Import associates an **existing external resource** with a Terraform resource address.

Conceptually:

```text
Existing AWS EC2
       ↓
Terraform import
       ↓
Terraform state
       ↓
Terraform configuration manages it
```

### Key difference

| Refresh | Import |
|---|---|
| Resource is already tracked | Resource is not yet tracked at that Terraform address |
| Updates observed state information | Adds/adopts an existing resource into management |
| Does not create the resource | Does not create the resource |
| Used for reconciliation | Used for adoption |

**Interview follow-up:** Does import automatically create the full Terraform configuration?

No. Import establishes the resource in state; you still need appropriate Terraform configuration so that future plans represent the intended resource configuration.

---

# 11. State Recovery

## Q42. What happens if Terraform state is lost?

If state is lost, Terraform may no longer know the mapping between resource addresses and real infrastructure.

Potential consequences:

- Terraform may consider existing resources unmanaged.
- Future plans may be incorrect.
- Recreating resources can cause duplication.
- Destroy/update operations become risky.
- Dependencies represented in state can be lost.

### Best recovery

If state is stored remotely with versioning/backups:

```text
Remote backend
      ↓
Previous state version
      ↓
Restore state
      ↓
Run plan
      ↓
Verify
```

### Alternative

Reconstruct resource configuration and import existing resources.

### Last resort

Recreate infrastructure only when appropriate and safe.

### Prevention

- Use a remote backend.
- Enable appropriate versioning/recovery controls.
- Restrict state access.
- Protect backend credentials.
- Review state operations.
- Test recovery procedures.

---

# 12. Modules

## Q43. What is a Terraform module?

A module is a collection of Terraform configuration files used as a reusable infrastructure component.

Typical module:

```text
modules/
└── vpc/
    ├── main.tf
    ├── variables.tf
    └── outputs.tf
```

Called from the root module:

```hcl
module "vpc" {
  source = "./modules/vpc"

  cidr_block = "10.0.0.0/16"
}
```

---

## Q44. What is the difference between a root module and a child module?

| Root module | Child module |
|---|---|
| Main working directory | Called by another module |
| Terraform commands are normally run here | Reusable component |
| Supplies inputs to child modules | Defines resources/outputs |
| Consumes module outputs | Exposes module outputs |

Conceptually:

```text
Root module
   |
   +--- VPC child module
   |
   +--- ALB child module
   |
   +--- EC2 child module
```

---

## Q45. Why should we use modules?

Modules provide:

- Reusability.
- Standardization.
- Separation of concerns.
- Easier maintenance.
- Consistent infrastructure patterns.
- Reduced duplication.

For example, instead of writing an EC2 setup five times:

```text
module "app_dev"
module "app_stage"
module "app_prod"
```

the same reusable module can receive different inputs.

---

## Q46. How do you create a reusable module?

A practical structure:

```text
modules/
└── ec2/
    ├── main.tf
    ├── variables.tf
    └── outputs.tf
```

### `variables.tf`

```hcl
variable "instance_type" {
  type = string
}
```

### `main.tf`

```hcl
resource "aws_instance" "this" {
  ami           = var.ami
  instance_type = var.instance_type
}
```

### `outputs.tf`

```hcl
output "instance_id" {
  value = aws_instance.this.id
}
```

The caller supplies values:

```hcl
module "app" {
  source = "./modules/ec2"

  ami           = "ami-xxxxxxxx"
  instance_type = "t3.small"
}
```

---

## Q47. How do you pass variables to modules?

The caller assigns arguments that correspond to variables declared inside the child module.

```hcl
module "vpc" {
  source = "./modules/vpc"

  cidr_block = "10.0.0.0/16"
}
```

Inside the child module:

```hcl
variable "cidr_block" {
  type = string
}
```

The child module uses:

```hcl
var.cidr_block
```

Outputs travel in the opposite direction:

```text
Root
  ↓ inputs
Child module
  ↓ outputs
Root
```

---

# 13. Workspaces

## Q48. What is a Terraform workspace?

A workspace provides a separate state instance for the same Terraform configuration.

Conceptually:

```text
Same configuration
      |
      +--- workspace: dev
      |       ↓
      |    dev state
      |
      +--- workspace: test
      |       ↓
      |    test state
      |
      +--- workspace: prod
              ↓
           prod state
```

Commands:

```bash
terraform workspace list
terraform workspace new dev
terraform workspace select dev
```

### Important interview nuance

Workspaces are useful, but they are **not automatically the best way to represent every environment**. For larger environments, separate root configurations, accounts, directories, or Terragrunt structures may provide clearer isolation.

---

# 14. Provisioners

## Q49. What is a Terraform provisioner?

Provisioners execute commands or scripts during resource creation or destruction.

Examples include:

- `local-exec`
- `remote-exec`

Example:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-xxxxxxxx"
  instance_type = "t3.micro"

  provisioner "local-exec" {
    command = "echo ${self.public_ip}"
  }
}
```

### Important interview point

Provisioners are generally a **last-resort mechanism**. Prefer:

- Native Terraform resources.
- Cloud-init/user data where appropriate.
- Configuration-management tools such as Ansible.
- Purpose-built deployment mechanisms.

---

## Q50. What is the difference between a provider and a provisioner?

| Provider | Provisioner |
|---|---|
| Connects Terraform to an external platform/API | Executes commands/scripts |
| Fundamental Terraform mechanism | Specialized mechanism |
| Provides resources/data sources | Runs actions around resource lifecycle |
| Used for infrastructure management | Often used for bootstrapping/workarounds |

Simple interview answer:

> A provider tells Terraform how to communicate with an external system; a provisioner executes commands associated with resource lifecycle events.

---

# 15. `null_resource` and `terraform_data`

## Q51. What is `null_resource`?

`null_resource` is a resource-like mechanism historically used to trigger provisioners or represent orchestration logic without creating a normal infrastructure object.

Example:

```hcl
resource "null_resource" "bootstrap" {
  provisioner "local-exec" {
    command = "echo bootstrap"
  }
}
```

It has historically been used for:

- Running scripts.
- Triggering external commands.
- Workarounds where no native resource existed.

### Why it should be used carefully

It can make infrastructure behavior harder to reason about because Terraform is primarily designed to model infrastructure resources and their relationships.

---

## Q52. What is `terraform_data` and how is it different from `null_resource`?

`terraform_data` is a built-in Terraform resource designed for storing data and participating in Terraform dependency/trigger relationships without requiring a provider-specific infrastructure resource.

Example:

```hcl
resource "terraform_data" "example" {
  input = "bootstrap-v1"
}
```

### Comparison

| `null_resource` | `terraform_data` |
|---|---|
| Legacy/common orchestration pattern | Built-in Terraform resource |
| Often paired with provisioners | Useful for data/dependency relationships |
| Requires null provider in historical usage | Built into Terraform |
| Can become script-centric | Better fit for Terraform-native dependency patterns |

### Interview takeaway

If the requirement is to model a Terraform dependency/data relationship, consider `terraform_data`. Do not automatically reach for `null_resource` plus provisioners.

---

# 16. Terraform vs Ansible — Deeper Interview Questions

## Q53. How does Terraform differ from Ansible operationally?

Terraform is primarily focused on **desired infrastructure state** and resource lifecycle.

Ansible is commonly focused on **tasks and configuration state on machines/services**.

Example:

```text
Terraform
  ↓
VPC
Subnet
Security Group
EC2
Load Balancer

Ansible
  ↓
Install Java
Install NGINX
Create users
Deploy application
Configure services
```

### Direct vs indirect cloud interaction

Terraform commonly works directly through provider APIs:

```text
Terraform → AWS provider → AWS API
```

Ansible can also interact with cloud APIs through its modules, but its common operational strength is configuration/automation on systems.

---

## Q54. What is declarative vs procedural automation?

### Declarative

You specify the desired result.

```text
I want:
2 EC2 instances
1 load balancer
3 subnets
```

Terraform determines the operations required.

### Procedural/task-oriented

You describe tasks/actions.

```text
1. Install package
2. Create file
3. Start service
4. Copy configuration
5. Restart service
```

### Interview answer

> Terraform is primarily declarative: I describe the desired infrastructure and Terraform determines the required changes. Ansible is commonly task-oriented and excels at configuring systems and executing operational tasks.

---

# 17. `terraform init` Internals

## Q55. What is `.terraform.lock.hcl`?

The dependency lock file records selected provider versions and checksums so that Terraform can reproduce provider installations more consistently.

It is normally committed to version control.

```text
Terraform configuration
       ↓
Provider constraints
       ↓
Selected provider
       ↓
.lock.hcl records selection/checksums
```

### Interview point

The lock file is not the same thing as Terraform state.

| Lock file | State |
|---|---|
| Provider dependency selections/checksums | Managed infrastructure information |
| `.terraform.lock.hcl` | `terraform.tfstate` / remote state |
| Helps provider reproducibility | Helps resource management |

---

## Q56. Why can `terraform init` take time?

It may need to:

- Connect to the backend.
- Download providers.
- Verify provider packages.
- Download modules.
- Resolve dependencies.
- Access remote systems.

In CI/CD, repeated initialization can be optimized with appropriate provider/plugin caching and efficient workspace handling, while preserving correctness.

---

# 18. Terragrunt

## Q57. What is Terragrunt?

Terragrunt is a wrapper/tooling layer commonly used to make larger Terraform codebases more DRY and easier to manage.

It can help with:

- Reusing configuration.
- Managing multiple environments.
- Remote-state configuration.
- Module dependencies.
- Common inputs.
- Generated Terraform configuration.

Conceptually:

```text
Terragrunt
    ↓
Terraform modules/configuration
    ↓
Infrastructure
```

---

## Q58. Why is Terragrunt useful?

Without additional structure, large Terraform repositories can repeat:

```text
backend configuration
provider configuration
common variables
module sources
environment configuration
```

Terragrunt can centralize or generate some of this configuration.

A typical structure might be:

```text
live/
├── dev/
│   ├── vpc/
│   └── app/
└── prod/
    ├── vpc/
    └── app/

modules/
├── vpc/
└── app/
```

---

## Q59. What are common Terragrunt concepts?

| Concept | Purpose |
|---|---|
| `include` | Reuse parent configuration |
| `remote_state` | Configure remote state |
| `dependency` | Obtain outputs from another unit |
| `inputs` | Pass variables to Terraform |
| `locals` | Define reusable Terragrunt values |
| `generate` | Generate Terraform files/configuration |

### Interview nuance

Terragrunt is not a replacement for understanding Terraform. Terraform concepts such as state, modules, providers, dependencies, and plans must be understood first.

---

## Q60. Terraform vs Terragrunt — what is the difference?

| Terraform | Terragrunt |
|---|---|
| IaC engine | Wrapper/orchestration/tooling layer around Terraform |
| Manages infrastructure | Helps organize/manage Terraform at scale |
| Providers/resources/state are core concepts | DRY config, dependencies, environment structure |
| Can be used independently | Commonly invokes Terraform |

---

# 19. Sentinel and Policy as Code

## Q61. What is Sentinel in Terraform?

Sentinel is HashiCorp's policy-as-code framework used with supported HashiCorp products to enforce organizational policies.

Conceptually:

```text
Terraform configuration
        ↓
terraform plan
        ↓
Policy evaluation
        ↓
Allowed / denied
        ↓
Apply
```

Example policies might require:

- Mandatory resource tags.
- Approved regions.
- Encryption.
- Approved AMIs.
- Restricted security groups.
- Approved instance types.

---

## Q62. What are common Sentinel policy levels?

The supplied material covers three common policy behaviors:

| Policy | Behavior |
|---|---|
| Advisory | Reports policy violation but does not necessarily block |
| Soft Mandatory | Blocks unless an authorized override mechanism is used |
| Hard Mandatory | Must pass; cannot be overridden in normal policy workflow |

The exact behavior depends on the Terraform/HashiCorp product workflow and policy configuration.

---

## Q63. Give production examples of Terraform policies.

### Mandatory tagging

```text
Every resource must contain:
Environment
Owner
CostCenter
```

### Restrict public storage

```text
Public S3 configuration → reject
```

### Restrict regions

```text
Allowed:
ap-south-1
us-east-1

Other regions:
reject
```

### Restrict security groups

```text
0.0.0.0/0 → port 22
reject
```

### Encryption

```text
Unencrypted production storage
→ reject
```

These policies shift security/compliance left into the infrastructure pipeline.

---

## Q64. Sentinel vs OPA — what is the difference?

Both are policy-as-code technologies, but they differ in ecosystem, language, integrations, and deployment model.

At interview level:

> Sentinel is strongly associated with HashiCorp's ecosystem, while Open Policy Agent (OPA) is a general-purpose policy engine used across many platforms and tools.

Do not claim that one is universally "better"; the right choice depends on the organization's architecture and policy enforcement points.

---

# 20. Scenario-Based Interview Questions

## Q65. Your EC2 instance was manually changed from `t3.micro` to `t3.small`. Terraform plan shows a change. What do you do?

First determine ownership.

If Terraform should manage the value:

```text
terraform configuration = t3.micro
AWS = t3.small
        ↓
plan detects difference
        ↓
apply
        ↓
reconcile to t3.micro
```

If the manual change was intentional, update Terraform configuration instead.

Do not blindly add `ignore_changes`.

---

## Q66. Two engineers run `terraform apply` at the same time. What can happen?

If the backend supports locking correctly:

```text
Engineer A → obtains lock → apply
Engineer B → waits/fails depending on behavior
```

This protects concurrent state operations.

Without proper coordination, concurrent operations can create race conditions and state conflicts.

---

## Q67. You need three EC2 instances, but each has a different instance type and name. `count` or `for_each`?

Prefer `for_each`.

```hcl
for_each = {
  web    = "t3.micro"
  api    = "t3.small"
  worker = "t3.medium"
}
```

The keys provide stable, meaningful identities.

---

## Q68. You need 10 identical EC2 instances. `count` or `for_each`?

`count` is a reasonable choice when the instances are genuinely interchangeable.

```hcl
count = 10
```

If each instance has meaningful identity, use `for_each`.

---

## Q69. You need multiple security-group ingress blocks generated from a list. What do you use?

A `dynamic` block.

```hcl
dynamic "ingress" {
  for_each = var.ports

  content {
    from_port = ingress.value
    to_port   = ingress.value
    protocol  = "tcp"
  }
}
```

---

## Q70. A resource exists in AWS but Terraform does not manage it. What do you do?

1. Write the corresponding Terraform resource configuration.
2. Import/adopt the existing resource into Terraform state using the appropriate import mechanism.
3. Run `terraform plan`.
4. Reconcile configuration until the plan represents the intended state.

---

## Q71. Terraform state is deleted but the AWS resources still exist. Should you run `terraform apply` immediately?

No.

First recover or reconstruct state.

Preferred order:

```text
Check remote backend/version history
        ↓
Restore state if possible
        ↓
terraform plan
        ↓
Verify carefully
```

If recovery is impossible, import existing resources into reconstructed Terraform configuration.

---

## Q72. Terraform creates an EC2 before another resource that it logically depends on. What do you check?

First check whether the dependency is represented by an attribute reference.

If it is not, and the dependency is genuinely required, use `depends_on`.

Example:

```hcl
depends_on = [
  aws_iam_role.app
]
```

Avoid adding `depends_on` without a real dependency.

---

# 21. Rapid-Fire Revision Table

| Question | Short interview answer |
|---|---|
| What is Terraform? | Declarative IaC tool for provisioning/managing infrastructure |
| What is IaC? | Managing infrastructure through version-controlled code |
| Provider? | Plugin/API integration used by Terraform |
| Resource? | Infrastructure object Terraform manages |
| Data source? | Read-only information lookup |
| `init`? | Initializes backend, providers, modules and working directory |
| `plan`? | Shows proposed changes |
| `apply`? | Executes changes |
| `destroy`? | Removes managed infrastructure |
| Implicit dependency? | Dependency inferred from references |
| Explicit dependency? | Dependency declared with `depends_on` |
| Variable? | External/configurable input |
| Local? | Internal reusable/calculated value |
| Output? | Exposes useful Terraform values |
| `count`? | Multiple instances by numeric index |
| `for_each`? | Multiple instances by stable keys |
| Dynamic block? | Generates repeated nested blocks |
| Lifecycle? | Controls resource lifecycle behavior |
| State? | Maps Terraform configuration to managed infrastructure |
| Backend? | Defines where/how state is stored |
| Drift? | Difference between expected and actual infrastructure |
| Import? | Adopts an existing resource into Terraform state |
| Module? | Reusable Terraform configuration |
| Workspace? | Separate state instance for the same configuration |
| Provisioner? | Executes commands/scripts around resource lifecycle |
| `null_resource`? | Historical orchestration/provisioner pattern |
| `terraform_data`? | Built-in resource for data/dependency relationships |
| Terragrunt? | Tooling layer that helps organize Terraform at scale |
| Sentinel? | HashiCorp policy-as-code framework |

---

# 22. Important Interview Traps

## Trap 1 — "Terraform state is the infrastructure."

Incorrect.

State is Terraform's record/mapping of managed infrastructure. The actual infrastructure exists in the target platform.

---

## Trap 2 — "A data source creates infrastructure."

Incorrect.

A data source reads information. Resources manage infrastructure.

---

## Trap 3 — "`for_each` is just a different syntax for `count`."

Not exactly.

The biggest practical difference is **resource identity**:

```text
count    → numeric indexes
for_each → keys
```

---

## Trap 4 — "Always use `depends_on`."

Incorrect.

Prefer implicit dependencies through references. Use explicit dependencies only when Terraform cannot infer a real dependency.

---

## Trap 5 — "`ignore_changes` fixes drift."

Not necessarily.

It tells Terraform to ignore selected differences. It does not actually reconcile infrastructure.

---

## Trap 6 — "Import creates the Terraform code."

Not by itself.

Import establishes the relationship with existing infrastructure in Terraform's state/management model. You still need correct configuration.

---

## Trap 7 — "Provisioners are the normal way to configure servers."

No.

Provisioners are generally a last resort. Prefer native resources, cloud-init/user data where appropriate, or configuration-management/deployment tools.

---

## Trap 8 — "Terraform workspaces are always the best way to manage dev/stage/prod."

No.

They can be useful, but environment isolation often benefits from separate root configurations, accounts, directories, or other organizational patterns.

---

# 23. Practical Terraform Interview Exercise

Be able to explain this configuration without looking at notes:

```hcl
variable "environment" {
  type = string
}

variable "instances" {
  type = map(string)
}

locals {
  name_prefix = "otms-${var.environment}"
}

provider "aws" {
  region = "ap-south-1"
}

resource "aws_security_group" "web" {
  name = "${local.name_prefix}-sg"

  dynamic "ingress" {
    for_each = toset([80, 443])

    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
}

resource "aws_instance" "web" {
  for_each = var.instances

  ami           = "ami-xxxxxxxx"
  instance_type = each.value

  vpc_security_group_ids = [
    aws_security_group.web.id
  ]

  tags = {
    Name        = "${local.name_prefix}-${each.key}"
    Environment = var.environment
  }

  lifecycle {
    create_before_destroy = true
  }
}

output "instance_ids" {
  value = {
    for name, instance in aws_instance.web :
    name => instance.id
  }
}
```

### Interviewer may ask:

1. What is the provider?
2. What are the variables?
3. Why is `locals` used?
4. Why is `for_each` used?
5. Why is a dynamic block used?
6. Is the security group an implicit dependency?
7. What does `create_before_destroy` do?
8. What does the output return?
9. Where would state be stored in production?
10. How would you detect manual infrastructure changes?
11. How would you import an existing instance?
12. Why might you avoid putting secrets directly into Terraform configuration?

If you can explain all 12 clearly, your Terraform fundamentals are in good shape.

---

# 24. Final Interview Preparation Checklist

Before the interview, make sure you can explain each of these **without memorizing a paragraph**:

### Core

- [ ] Terraform
- [ ] IaC
- [ ] Declarative model
- [ ] Providers
- [ ] Resources
- [ ] Data sources
- [ ] `init`
- [ ] `validate`
- [ ] `plan`
- [ ] `apply`
- [ ] `destroy`

### Configuration

- [ ] Variables
- [ ] `variables.tf`
- [ ] `.tfvars`
- [ ] Locals
- [ ] Outputs
- [ ] Expressions
- [ ] Functions
- [ ] Conditional expressions

### Resource behavior

- [ ] Implicit dependencies
- [ ] `depends_on`
- [ ] Meta-arguments
- [ ] `count`
- [ ] `for_each`
- [ ] `count` vs `for_each`
- [ ] `lifecycle`
- [ ] `prevent_destroy`
- [ ] `ignore_changes`
- [ ] `create_before_destroy`
- [ ] Dynamic blocks

### State

- [ ] Terraform state
- [ ] State mapping
- [ ] Remote backend
- [ ] S3 backend
- [ ] State locking
- [ ] State security
- [ ] Drift
- [ ] Drift detection
- [ ] Drift reconciliation
- [ ] Import
- [ ] Refresh/reconciliation
- [ ] State recovery

### Reuse and scale

- [ ] Modules
- [ ] Root module
- [ ] Child module
- [ ] Module inputs
- [ ] Module outputs
- [ ] Workspaces
- [ ] Terragrunt

### Advanced operational concepts

- [ ] Providers vs provisioners
- [ ] Provisioners
- [ ] `null_resource`
- [ ] `terraform_data`
- [ ] Terraform vs Ansible
- [ ] Declarative vs procedural automation
- [ ] `terraform init` internals
- [ ] `.terraform.lock.hcl`
- [ ] Sentinel
- [ ] Policy as Code
- [ ] Sentinel policy levels
- [ ] Sentinel vs OPA

---

# 25. Interview Answer Formula

For almost every Terraform interview question, use this structure:

```text
1. Definition
      ↓
2. Why it exists
      ↓
3. Small example
      ↓
4. Real DevOps use case
      ↓
5. Important caveat / comparison
```

For example:

> **What is `for_each`?**

**Definition:** It is a Terraform meta-argument for creating multiple resource instances from a map or set.

**Why:** It gives each instance a stable key-based identity.

**Example:** `for_each = { web = "t3.micro", api = "t3.small" }`.

**Use case:** Creating different application servers with different configurations.

**Caveat:** Use `for_each` when resource identity matters; for interchangeable instances, `count` may be simpler.

This style produces a much stronger interview answer than giving only a one-line definition.

---

# End

The objective is not to memorize Terraform syntax. The objective is to understand:

```text
Configuration
     ↓
Terraform
     ↓
Dependency Graph
     ↓
Plan
     ↓
Provider/API
     ↓
Infrastructure
     ↓
State
     ↓
Next Plan
     ↓
Reconciliation
```

If you understand this lifecycle and can connect it to **variables, dependencies, lifecycle rules, state, modules, drift, and real DevOps scenarios**, you can handle most Terraform interview discussions confidently.
