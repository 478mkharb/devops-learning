### Q1. What is Terraform and why is it used in DevOps?

**Terraform** is an **Infrastructure as Code (IaC)** tool developed by **HashiCorp** that allows DevOps teams to define, provision, and manage infrastructure using declarative configuration files (`.tf`).

### Why Terraform is used in DevOps

- **Infrastructure as Code (IaC)**  
  Infrastructure is written as code, enabling version control, code review, and repeatability.

- **Cloud Agnostic**  
  Supports multiple providers like AWS, Azure, GCP, Kubernetes, etc., with a single workflow.

- **Declarative Configuration**  
  You describe the desired state, and Terraform determines how to achieve it.

- **Automation & Consistency**  
  Reduces manual errors and ensures consistent environments (dev, stage, prod).

- **State Management**  
  Maintains a state file to track resources and detect configuration drift.

- **CI/CD Integration**  
  Easily integrates with tools like Jenkins, GitHub Actions, and GitLab CI.

### Q2. What is Infrastructure as Code (IaC)?

**Infrastructure as Code (IaC)** is a practice of **managing and provisioning infrastructure using code instead of manual processes**. The infrastructure setup is defined in configuration files, which can be version-controlled and automated.

### Key Characteristics of IaC

- **Code-Based Infrastructure**  
  Servers, networks, databases, and other resources are defined using code files.

- **Version Control**  
  Infrastructure code can be stored in Git, enabling tracking, rollback, and collaboration.

- **Automation**  
  Infrastructure provisioning is automated, reducing manual errors.

- **Consistency**  
  Ensures the same infrastructure is created every time across environments.

- **Idempotency**  
  Running the same code multiple times produces the same result.

### Q3. How is Terraform different from Ansible and CloudFormation?

Terraform, Ansible, and CloudFormation are all infrastructure tools, but they serve different purposes and follow different approaches.

#### Terraform
- Declarative IaC tool
- Cloud-agnostic (AWS, Azure, GCP, etc.)
- Manages infrastructure lifecycle (create, update, delete)
- Uses a state file to track resources
- Best suited for infrastructure provisioning

#### Ansible
- Procedural / task-based configuration management tool
- Agentless (uses SSH)
- Mainly used for configuration management and application deployment
- Does not manage infrastructure state by default
- Best suited for server configuration and automation

#### CloudFormation
- Declarative IaC tool
- AWS-specific service
- Manages AWS resources only
- Uses AWS-managed state (stacks)
- Best suited for AWS-only environments

### Q4. What are Terraform providers?

**Terraform providers** are plugins that allow Terraform to interact with external platforms such as cloud providers, SaaS services, and other APIs.

- Providers act as a bridge between Terraform and the target infrastructure.
- Each provider exposes resource types and data sources.
- Terraform uses providers to create, update, and delete resources.

#### Key Points
- Examples: AWS, Azure, Google Cloud, Kubernetes, GitHub
- Providers are defined in the `provider` block
- Provider versions can be pinned for stability
- Terraform downloads providers during `terraform init`

#### Example
```hcl
provider "aws" {
  region = "ap-south-1"
}
```
### Q5. What is a Terraform resource?

A **Terraform resource** represents a **infrastructure object** that Terraform manages, such as an EC2 instance, S3 bucket, VPC, or Kubernetes pod.

- Resources are the **core building blocks** in Terraform
- Each resource maps to an API object of a provider
- Terraform can create, update, and delete resources
- Resources are declared using the `resource` block

#### Example
```hcl
resource "aws_instance" "web" {
  ami           = "ami-0abcdef123"
  instance_type = "t2.micro"
}
```
### Q6. What is the purpose of `terraform init`?

`terraform init` is the **first command** run in any Terraform project. It initializes the working directory and prepares it for Terraform operations.

- Downloads and installs required providers
- Initializes the backend for state storage
- Downloads modules used in the configuration
- Prepares the `.terraform` directory

#### Key Points
- Must be run before `plan` or `apply`
- Re-run when providers or backend configuration changes
- Safe to run multiple times


#### Example
```bash
terraform init
```
### Q7. What happens during `terraform plan`?

`terraform plan` creates an **execution plan** by comparing the desired configuration with the current state and real infrastructure.

### 1. Loads configuration files
- Reads all `.tf` files in the working directory
- Parses resources, variables, providers, and modules

### 2. Refreshes the state
- Queries real infrastructure using providers
- Updates the state file with the current resource status

### 3. Compares desired state vs current state
- Determines what resources need to be:
  - Created
  - Updated
  - Deleted
  - Left unchanged

### 4. Resolves dependencies
- Builds a dependency graph
- Ensures correct resource creation order

### 5. Generates an execution plan
- Shows detailed actions Terraform will perform
- Marks changes using:
  - `+` create
  - `~` update
  - `-` destroy

### 6. No changes are applied
- Infrastructure is not modified
- Used mainly for validation and review before `apply`
  
### Q8. What does `terraform apply` do?

`terraform apply` **executes the actions proposed in the Terraform plan** and makes real changes to the infrastructure.

### 1. Creates an execution plan
- Generates a plan (if not provided explicitly)
- Shows proposed infrastructure changes

### 2. User confirmation
- Prompts for approval before making changes
- Can be auto-approved using `-auto-approve`

### 3. Applies changes to infrastructure
- Creates, updates, or deletes resources
- Follows dependency graph for correct order

### 4. Updates the state file
- Writes the new resource state to the backend
- Ensures state matches actual infrastructure

### 5. Handles partial failures
- Successfully created resources remain created
- Failed resources are reported for correction

### 6. Outputs results
- Displays output values defined in configuration
- Confirms successful application

### Q9. What is `terraform destroy`?

`terraform destroy` is used to **remove all infrastructure resources** managed by the current Terraform configuration.

### 1. Loads configuration and state
- Reads `.tf` files and the state file
- Identifies resources under Terraform management

### 2. Creates a destruction plan
- Determines which resources will be deleted
- Shows the order of destruction based on dependencies

### 3. User confirmation
- Requires explicit approval before deletion
- Can be auto-approved using `-auto-approve`

### 4. Destroys infrastructure
- Deletes resources in the correct dependency order
- Communicates with providers to remove resources

### 5. Updates the state
- Removes destroyed resources from the state file
- Keeps state consistent with infrastructure

### 6. Safety considerations
- Can cause data loss if used in production
- Often restricted in CI/CD pipelines

### Q10. What is the Terraform workflow?

The **Terraform workflow** defines the standard steps used to provision and manage infrastructure using Terraform.

### 1. Write
- Create Terraform configuration files (`.tf`)
- Define providers, resources, variables, and outputs

### 2. Initialize
- Run `terraform init`
- Downloads providers, initializes backend, and modules

### 3. Plan
- Run `terraform plan`
- Reviews proposed infrastructure changes without applying them

### 4. Apply
- Run `terraform apply`
- Provisions or updates infrastructure after approval

### 5. Manage & Update
- Modify configuration as needed
- Re-run `plan` and `apply` for changes

### 6. Destroy (Optional)
- Run `terraform destroy`
- Cleans up infrastructure when no longer required

### Q11. What is a provider block in Terraform?

A **provider block** is used to **configure how Terraform connects to an external service or platform** such as AWS, Azure, GCP, or Kubernetes.

### Key points

- Defines authentication and region/endpoint details
- Required before creating resources for that provider
- A configuration can have multiple provider blocks
- Provider settings apply to all associated resources

### Basic structure
```hcl
provider "aws" {
  region = "ap-south-1"
}
```
### Q12. How do you configure multiple providers in Terraform?

Terraform allows configuring **multiple providers** by defining multiple provider blocks or using **provider aliases**.

### 1. Multiple provider blocks (different regions/accounts)

```hcl
provider "aws" {
  region = "ap-south-1"
}
provider "aws" {
  alias  = "us"
  region = "us-east-1"
}
```
### 2. Using provider alias in resources
```
resource "aws_instance" "india_server" {
  ami           = "ami-0abcd"
  instance_type = "t2.micro"
}

resource "aws_instance" "us_server" {
  provider      = aws.us
  ami           = "ami-0efgh"
  instance_type = "t2.micro"
}
```
### Q13. What is the difference between a resource and a data source in Terraform?

Terraform uses **resources** and **data sources** for different purposes when managing infrastructure.

### Resource

A **resource** is used to **create, update, and delete infrastructure components**. Terraform fully manages the lifecycle of resources and tracks them in the state file.

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0abcd"
  instance_type = "t2.micro"
}
```

**Key points about resources**

* Create new infrastructure
* Modify existing infrastructure
* Delete infrastructure when required
* Fully managed and tracked by Terraform

---

### Data Source

A **data source** is used to **read information about existing infrastructure** that is not created by Terraform. Data sources are read-only.

```hcl
data "aws_ami" "latest" {
  most_recent = true
  owners      = ["amazon"]
}
```

**Key points about data sources**

* Do not create or modify resources
* Fetch existing resource details
* Used for dynamic values in resources
* Read-only access

---

### Key Differences

| Resource                      | Data Source                     |
| ----------------------------- | ------------------------------- |
| Manages lifecycle             | Read-only                       |
| Creates infrastructure        | Fetches existing infrastructure |
| Can be modified or destroyed  | Cannot be modified              |
| Actively managed by Terraform | Only referenced by Terraform    |

### Q14. Can multiple resources depend on one provider?

Yes, **multiple resources can depend on a single provider** in Terraform.

### How it works

- A provider defines **how Terraform connects** to a platform (AWS, Azure, GCP, etc.)
- All resources of that provider **automatically use it by default**
- Terraform creates a dependency graph so resources use the provider configuration correctly

### Example

```hcl
provider "aws" {
  region = "ap-south-1"
}

resource "aws_instance" "web" {
  ami           = "ami-0abcd"
  instance_type = "t2.micro"
}

resource "aws_s3_bucket" "logs" {
  bucket = "my-log-bucket"
}
```
### Q15. What is implicit dependency in Terraform?

**Implicit dependency** is when Terraform automatically determines the order of resource creation **based on references between resources**, without using `depends_on`.

### How implicit dependency works

- Terraform analyzes resource arguments
- If one resource references another, Terraform creates a dependency
- Ensures correct creation and destruction order

### Example

```hcl
resource "aws_security_group" "web_sg" {
  name = "web-sg"
}

resource "aws_instance" "web" {
  ami                    = "ami-0abcd"
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.web_sg.id]
}
```
### Q16. What is explicit dependency in Terraform?

**Explicit dependency** is when you manually tell Terraform that one resource **depends on another** using the `depends_on` argument.

### When explicit dependency is needed

- When no direct reference exists between resources
- When dependency is logical, not attribute-based
- When Terraform cannot automatically detect the order

### Example

```hcl
resource "aws_iam_role" "example" {
  name = "example-role"
}

resource "aws_instance" "web" {
  ami           = "ami-0abcd"
  instance_type = "t2.micro"

  depends_on = [
    aws_iam_role.example
  ]
}
```
### Q17. What is `depends_on` in Terraform?

`depends_on` is a **meta-argument** used to **explicitly define resource dependencies** when Terraform cannot automatically determine them.

### Why `depends_on` is used

- To control resource creation order
- To handle logical or indirect dependencies
- To avoid race conditions during provisioning

### How it works

- Terraform waits for all listed resources to be created or updated
- Then proceeds with the dependent resource

### Example

```hcl
resource "aws_security_group" "sg" {
  name = "web-sg"
}

resource "aws_instance" "web" {
  ami           = "ami-0abcd"
  instance_type = "t2.micro"

  depends_on = [
    aws_security_group.sg
  ]
}
```
### Q18. What are input variables in Terraform?

**Input variables** allow you to **parameterize Terraform configurations**, making them reusable and flexible across different environments.

### Purpose of input variables

- Avoid hardcoding values
- Reuse the same code for dev, stage, and prod
- Customize infrastructure without changing code
- Improve readability and maintainability

### Defining an input variable

```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

* Using Variable
resource "aws_instance" "web" {
  ami           = "ami-0abcd"
  instance_type = var.instance_type
}
```
### Q19. What are meta-arguments in Terraform?

**Meta-arguments** are special arguments in Terraform that **change the behavior of resources and modules**, rather than defining infrastructure itself.

They are **provider-independent** and work the same way across all resources.

### Common Terraform meta-arguments

- `count`  
  Used to create multiple instances of a resource

- `for_each`  
  Used to create resources based on maps or sets

- `depends_on`  
  Explicitly defines resource dependencies

- `provider`  
  Specifies which provider configuration to use

- `lifecycle`  
  Controls resource lifecycle behavior

### Example

```hcl
resource "aws_instance" "web" {
  count         = 2
  ami           = "ami-0abcd"
  instance_type = "t2.micro"
}
```
### Q20. What is the `count` meta-argument in Terraform?

The `count` meta-argument is used to **create multiple instances of the same resource or module** using a single configuration block.

### How `count` works

- `count` accepts a numeric value
- Terraform creates resources indexed from `0`
- Each instance is accessed using `count.index`

### Example

```hcl
resource "aws_instance" "web" {
  count         = 3
  ami           = "ami-0abcd"
  instance_type = "t2.micro"
}
```
### Q21. What is the `for_each` meta-argument in Terraform?

The `for_each` meta-argument is used to **create multiple instances of a resource or module based on a map or a set of strings**.

### How `for_each` works

- Accepts a **map** or **set**
- Each instance is identified by a unique key
- Accessed using `each.key` and `each.value`

### Example using a map

```hcl
resource "aws_instance" "web" {
  for_each = {
    dev  = "t2.micro"
    prod = "t3.medium"
  }

  ami           = "ami-0abcd"
  instance_type = each.value
}
```
### Q22. What is the difference between `count` and `for_each`?

Both `count` and `for_each` are Terraform meta-arguments used to **create multiple resource instances**, but they differ in how resources are identified and managed.

### `count`

* Uses numeric index (`count.index`)
* Best for identical resources
* Order-dependent
* Removing an element can shift indexes and recreate resources

```hcl
resource "aws_instance" "web" {
  count = 2
}
```

---

### `for_each`

* Uses unique keys (`each.key`)
* Best for distinct resources
* Order-independent
* Safer for add/remove operations

```hcl
resource "aws_instance" "web" {
  for_each = toset(["dev", "prod"])
}
```

---

### Key Differences

| count                        | for_each                  |
| ---------------------------- | ------------------------- |
| Index-based                  | Key-based                 |
| Order matters                | Order does not matter     |
| Less safe for changes        | Safer for updates         |
| Best for identical resources | Best for unique resources |

### Q23. What are `locals` in Terraform?

**Locals** in Terraform are used to **define reusable named values** within a configuration to simplify expressions and avoid repetition.

### Purpose of locals

- Reduce duplication in Terraform code
- Improve readability and maintainability
- Store computed or derived values
- Centralize commonly used expressions

### Defining locals

```hcl
locals {
  environment = "dev"
  instance_name = "web-${local.environment}"
}
```
**Using locals**
```
resource "aws_instance" "web" {
  ami           = "ami-0abcd"
  instance_type = "t2.micro"
  tags = {
    Name = local.instance_name
  }
}
```
### Q24. What is the `lifecycle` meta-argument in Terraform?

The `lifecycle` meta-argument is used to **control how Terraform creates, updates, and destroys resources** beyond the default behavior.

### Why `lifecycle` is used

- Prevent accidental deletion of critical resources
- Reduce downtime during updates
- Ignore specific attribute changes
- Control replacement behavior

### Common `lifecycle` arguments

- `prevent_destroy`
- `create_before_destroy`
- `ignore_changes`

### Example

```hcl
resource "aws_db_instance" "db" {
  allocated_storage = 20
  engine            = "mysql"

  lifecycle {
    prevent_destroy = true
  }
}
```
### Q25. What is `ignore_changes` in Terraform?

`ignore_changes` is a **lifecycle argument** that tells Terraform to **ignore changes to specific resource attributes**, even if they differ from the configuration.

### When to use `ignore_changes`

- When attributes are modified outside Terraform
- When cloud providers update attributes automatically
- To avoid unnecessary resource updates
- To prevent drift-related re-creation

### Example

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0abcd"
  instance_type = "t2.micro"

  lifecycle {
    ignore_changes = [
      tags["LastModified"],
      user_data
    ]
  }
}
```
### Q26. What is `create_before_destroy` in Terraform?

`create_before_destroy` is a **lifecycle meta-argument** that instructs Terraform to **create a new resource before destroying the old one**, helping to avoid downtime.

### When to use `create_before_destroy`

- Zero-downtime deployments
- Updating critical resources
- Replacing immutable infrastructure
- Blue/green style updates

### Example

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0abcd"
  instance_type = "t2.micro"

  lifecycle {
    create_before_destroy = true
  }
}
```
### Q27. What is a `dynamic` block in Terraform?

A `dynamic` block is used to **generate nested blocks dynamically** inside a resource based on variables, maps, or lists.

It is mainly used when:
- A resource requires **repeating nested blocks**
- The number of blocks is **not fixed**
- You want to avoid duplicate code

### Why `dynamic` blocks are used

- Reduce repetitive configuration
- Handle optional or variable nested blocks
- Improve maintainability

### Example

```hcl
resource "aws_security_group" "example" {
  name = "web-sg"

  dynamic "ingress" {
    for_each = [80, 443]
    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
}
```
### Q28. What are Terraform expressions?

**Terraform expressions** are used to **compute values dynamically** within Terraform configurations using variables, functions, operators, and references.

### Types of Terraform expressions

- **Literal expressions**  
  Fixed values such as strings, numbers, and booleans

- **Variable expressions**  
  Values referenced from input variables

- **Local expressions**  
  Values derived from `locals`

- **Function expressions**  
  Built-in functions like `length`, `lookup`, `join`, etc.

- **Conditional expressions**  
  Logic-based value selection

### Example

```hcl
variable "env" {
  default = "dev"
}

locals {
  instance_type = var.env == "prod" ? "t3.medium" : "t2.micro"
}
```
### Q29. What are output values in Terraform?

**Output values** are used to **expose information about resources** created by Terraform after `apply` so that it can be viewed, reused, or passed to other configurations or tools.

### Why output values are used

- Display important information after deployment
- Share values between modules
- Pass data to CI/CD pipelines
- Reference values from remote state

### Defining an output

```hcl
output "instance_public_ip" {
  value = aws_instance.web.public_ip
}
```
### Q30. What is Terraform state?

**Terraform state** is a file where Terraform **stores information about the infrastructure it manages**, mapping Terraform resources to real-world resources.

### Purpose of Terraform state

- Keeps track of created resources
- Maps configuration to real infrastructure
- Helps Terraform determine what to change
- Enables dependency management

### How state is used

- Terraform reads the state during `plan` and `apply`
- Compares desired configuration with current state
- Decides whether to create, update, or delete resources

### State file details

- Default file name: `terraform.tfstate`
- Can be stored locally or remotely
- Contains resource IDs and metadata

### Key points

- State is critical for Terraform operations
- Should be stored securely
- Loss of state can cause resource recreation
- Remote state is recommended for teams

### Q31. Why is the Terraform state file important?

The **Terraform state file** is critical because it acts as the **source of truth** for mapping Terraform configurations to real infrastructure.

### Why the state file matters

- **Resource Mapping**  
  Links Terraform resources to actual cloud resource IDs.

- **Change Detection**  
  Helps Terraform determine what needs to be created, updated, or destroyed.

- **Dependency Management**  
  Stores relationships between resources to apply changes in the correct order.

- **Performance Optimization**  
  Avoids querying the provider for every resource on each run.

- **Collaboration Support**  
  Enables teams to work together using a shared remote state.

### Key points

- Without state, Terraform cannot safely manage infrastructure
- State enables idempotent operations
- Remote state is essential for team environments
- State must be protected due to sensitive data

### Q32. Why use an S3 backend for Terraform state?

Using an **S3 backend** allows Terraform to store its state file **remotely, securely, and reliably**, which is essential for team-based and production environments.

### Reasons to use S3 backend

- **Centralized state storage**  
  All team members use the same state file.

- **State locking support**  
  When combined with DynamoDB, it prevents concurrent `apply` operations.

- **High durability and availability**  
  S3 provides reliable storage for critical state data.

- **Versioning and recovery**  
  Enables rollback to previous state versions if needed.

- **Security**  
  Supports encryption at rest and IAM-based access control.

### Example configuration

```hcl
terraform {
  backend "s3" {
    bucket         = "terraform-state-bucket"
    key            = "prod/terraform.tfstate"
    region         = "ap-south-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```
### Q33. Difference between `terraform.tfvars` and `variables.tf`

**`variables.tf`**
- Declares variables
- Defines type, description, default
- Acts as variable schema
- Environment-independent

**`terraform.tfvars`**
- Assigns values to variables
- Environment-specific values
- Automatically loaded by Terraform
- Overrides default values

**In short:**  
- `variables.tf` → *what variables exist*  
- `terraform.tfvars` → *what values they use*
### Q34. What is state drift?

**State drift** occurs when the **actual infrastructure changes outside Terraform**, causing a mismatch between the Terraform state file and real resources.

**Common causes**
- Manual changes in cloud console
- Changes by other automation tools
- Auto-scaling or cloud-managed updates

---

### Q35. How do you detect drift?

Terraform detects drift by **refreshing the state** and comparing it with real infrastructure.

**Ways to detect drift**
- `terraform plan`
- `terraform refresh`
- CI pipelines running plan regularly

Terraform will show differences between:
- Terraform configuration
- Terraform state
- Actual infrastructure

---

### Q36. How do you fix drift?

Drift is fixed by **reconciling Terraform state with actual infrastructure**.

**Ways to fix drift**
- Run `terraform apply` to enforce configuration
- Update Terraform code to match real changes
- Use `terraform import` for unmanaged changes
- Use `ignore_changes` for intentional external updates
### Q37. What is a Terraform module?

A **Terraform module** is a **collection of Terraform configuration files** that are grouped together to create reusable and manageable infrastructure components.

---

### Q38. Difference between root module and child module

- **Root module**
  - Main working directory
  - Executed directly using Terraform commands

- **Child module**
  - Called by another module
  - Used for reuse and abstraction

---

### Q39. Why should we use modules?

- Reusability
- Better code organization
- Consistency across environments
- Easier maintenance and scaling

---

### Q40. How do you create a reusable module?

- Create a separate directory
- Define `main.tf`, `variables.tf`, `outputs.tf`
- Avoid hardcoded values
- Use variables and outputs

---

### Q41. How do you pass variables to modules?

Variables are passed from the calling module.

```hcl
module "vpc" {
  source = "./modules/vpc"
  cidr   = "10.0.0.0/16"
}
```
### Q42. What is a workspace in Terraform?

A **Terraform workspace** allows you to **manage multiple, isolated state files** using the same Terraform configuration.

### Key points

- Each workspace has its **own state**
- Uses the same `.tf` configuration
- Useful for managing multiple environments (dev, stage, prod)
- Default workspace is named `default`

### Common commands

```bash
terraform workspace list
terraform workspace new dev
terraform workspace select prod
```
### Key Differences
| Workspace          | Separate State Files      |
| ------------------ | ------------------------- |
| Single backend     | Separate backend or keys  |
| Easy to switch     | More configuration needed |
| Limited isolation  | Strong isolation          |
| Not ideal for prod | Recommended for prod      |
