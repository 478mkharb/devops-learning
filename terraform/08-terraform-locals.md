# 🧠 Terraform Locals – Concept and Use

## 📘 What Are Terraform `locals`?

`locals` in Terraform are **named values defined inside Terraform configuration that are used for internal calculations, reuse, and simplifying expressions**.

They are defined using a `locals` block and referenced using `local.<name>`.

Example:

```hcl
locals {
  instance_type = "t3.micro"
}

resource "aws_instance" "web" {
  instance_type = local.instance_type
}
```

Here:

```
local.instance_type
```

is a reusable value defined inside Terraform configuration.

---

# 🎯 Why Terraform Locals Are Used

Locals are mainly used to:

* Avoid repeating the same value or expression
* Create derived or computed values
* Improve code readability
* Centralize reusable values

---

# Example Without Locals

```hcl
resource "aws_instance" "web1" {
  instance_type = "t3.micro"
}

resource "aws_instance" "web2" {
  instance_type = "t3.micro"
}
```

The same value is repeated multiple times.

---

# Example With Locals

```hcl
locals {
  instance_type = "t3.micro"
}

resource "aws_instance" "web1" {
  instance_type = local.instance_type
}

resource "aws_instance" "web2" {
  instance_type = local.instance_type
}
```

Now the value is defined once and reused everywhere.

---

# ⚙️ Computed Values Using Locals

Locals can also compute values from variables.

Example:

```hcl
variable "environment" {}

locals {
  instance_name = "${var.environment}-web-server"
}

resource "aws_instance" "web" {
  tags = {
    Name = local.instance_name
  }
}
```

If:

```
environment = "prod"
```

Resulting tag:

```
prod-web-server
```

---

# 📘 What Is `variables.tf`?

`variables.tf` defines **input variables that Terraform expects from users or environments**.

Example:

```hcl
variable "instance_type" {
  description = "EC2 instance type"
}
```

Variables allow Terraform configurations to be **parameterized and reusable**.

---

# 📘 What Is `.tfvars`?

`.tfvars` files provide **values for variables defined in `variables.tf`**.

Example variable definition:

```hcl
variable "instance_type" {}
```

`terraform.tfvars` file:

```hcl
instance_type = "t3.micro"
```

Terraform reads this file and assigns values to variables.

---

# 🌍 Environment-Based tfvars Example

Projects often have multiple environments.

Example files:

```
dev.tfvars
stage.tfvars
prod.tfvars
```

Example:

**dev.tfvars**

```hcl
instance_type = "t3.micro"
```

**prod.tfvars**

```hcl
instance_type = "m5.large"
```

Run Terraform with:

```
terraform apply -var-file=prod.tfvars
```

---

# 🔄 How Variables, tfvars, and Locals Work Together

Typical flow in Terraform projects:

```
tfvars (user input)
      ↓
variables.tf (variable definitions)
      ↓
locals (derived values)
      ↓
resources
```

Example:

```hcl
variable "environment" {}

locals {
  name_prefix = "${var.environment}-app"
}

resource "aws_instance" "web" {
  tags = {
    Name = local.name_prefix
  }
}
```

---

# 📊 Difference: Locals vs variables.tf vs tfvars

| Feature                   | Locals                            | variables.tf            | tfvars                       |
| ------------------------- | --------------------------------- | ----------------------- | ---------------------------- |
| Purpose                   | Store reusable or computed values | Define input variables  | Provide values for variables |
| Defined by                | Terraform configuration           | Terraform configuration | User or environment          |
| Can be changed externally | No                                | Yes                     | Yes                          |
| Supports expressions      | Yes                               | Limited                 | No                           |
| Used for                  | Internal logic                    | Input definitions       | Environment configuration    |

---

# 🧠 Real DevOps Project Structure

Typical Terraform repository structure:

```
terraform-project

variables.tf
locals.tf
terraform.tfvars
main.tf
outputs.tf
```

Flow:

```
User input → tfvars
         ↓
variables.tf
         ↓
locals
         ↓
resources
```

---

# 🎤 Interview Answer

Terraform locals are used to define reusable or computed values within the Terraform configuration. Variables defined in `variables.tf` accept external input values, while `.tfvars` files provide those values for different environments. Locals help simplify expressions and avoid repeating logic across multiple resources.
