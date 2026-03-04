# 🚀 Terragrunt – Complete DevOps Guide

## 📘 What is Terragrunt?

**Terragrunt** is a **wrapper tool built on top of Terraform** that helps manage large Terraform infrastructures more efficiently.

It was created by **Gruntwork** to solve several common Terraform challenges such as:

* Repetition of backend configuration
* Managing multiple environments
* Handling dependencies between modules
* Reducing duplicate code

In simple terms:

👉 **Terragrunt = Terraform + automation + DRY infrastructure management**

Terragrunt still uses Terraform internally to provision infrastructure.

---

# 🎯 Why Terragrunt is Used

When infrastructure grows, Terraform projects become difficult to manage.

Common problems in large Terraform projects:

* 🔁 Repeated backend configuration
* 🌍 Multiple environments (dev / stage / prod)
* 🔗 Managing dependencies between modules
* 📂 Complex folder structures
* ⚙️ Difficult CI/CD orchestration

Terragrunt solves these problems by adding an **orchestration layer on top of Terraform**.

---

# 🧱 Terraform Problem Without Terragrunt

Example Terraform structure:

```
terraform-project

 dev
   vpc
     main.tf
   ec2
     main.tf

 stage
   vpc
     main.tf
   ec2
     main.tf

 prod
   vpc
     main.tf
   ec2
     main.tf
```

Problems:

* Backend repeated everywhere
* Variables duplicated
* Hard to manage dependencies
* Difficult to scale

---

# 🧩 Terraform With Terragrunt

Terragrunt introduces a **clean separation** between:

* Infrastructure modules
* Environment configuration

Example structure:

```
infrastructure

 modules
   vpc
   ec2

 live
   dev
     vpc
       terragrunt.hcl
     ec2
       terragrunt.hcl

   stage
     vpc
       terragrunt.hcl
     ec2
       terragrunt.hcl

   prod
     vpc
       terragrunt.hcl
     ec2
       terragrunt.hcl
```

Benefits:

* Modules reused
* Configuration separated
* Clean environment structure

---

# ⚙️ How Terragrunt Works

Workflow:

Developer
↓
Terragrunt
↓
Terraform CLI
↓
Cloud Provider (AWS / Azure / GCP)

Terragrunt reads **terragrunt.hcl** configuration and then runs Terraform commands automatically.

---

# 📄 What is `terragrunt.hcl`?

`terragrunt.hcl` is the **main configuration file used by Terragrunt**.

It defines:

* Terraform module source
* Backend configuration
* Variables
* Dependencies

Example:

```
terraform {
  source = "../../modules/vpc"
}

inputs = {
  cidr_block = "10.0.0.0/16"
}
```

Terragrunt will automatically download the module and execute Terraform.

---

# 🧠 Important Components of Terragrunt

## 1️⃣ include Block

Used to **inherit configuration from a parent file**.

Example:

```
include {
  path = find_in_parent_folders()
}
```

Use case:

* Share backend configuration
* Share common variables

---

## 2️⃣ remote_state Block

Automatically configures **Terraform backend**.

Example:

```
remote_state {
  backend = "s3"

  config = {
    bucket         = "terraform-state-bucket"
    key            = "vpc/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-lock"
  }
}
```

Benefits:

* No need to repeat backend configuration
* Enables **state locking**

---

## 3️⃣ dependency Block

Used to **fetch outputs from other Terraform modules**.

Example:

```
dependency "vpc" {
  config_path = "../vpc"
}

inputs = {
  vpc_id = dependency.vpc.outputs.vpc_id
}
```

Use case:

VPC must be created before EC2.

---

## 4️⃣ inputs Block

Used to **pass variables to Terraform modules**.

Example:

```
inputs = {
  instance_type = "t3.micro"
  instance_count = 2
}
```

---

## 5️⃣ locals Block

Defines reusable local variables.

Example:

```
locals {
  environment = "dev"
}
```

---

## 6️⃣ generate Block

Used to automatically generate Terraform files.

Example:

```
generate "provider" {
  path      = "provider.tf"
  if_exists = "overwrite"

  contents = <<EOF
provider "aws" {
  region = "us-east-1"
}
EOF
}
```

---

# 🧪 Example: Deploy VPC + EC2 Using Terragrunt

## Step 1 — Create Modules

```
modules

 vpc
 ec2
```

---

## Step 2 — Create Environment

```
live

 dev
   vpc
   ec2
```

---

## Step 3 — VPC Terragrunt File

```
terraform {
  source = "../../../modules/vpc"
}

inputs = {
  cidr_block = "10.0.0.0/16"
}
```

---

## Step 4 — EC2 Terragrunt File

```
dependency "vpc" {
  config_path = "../vpc"
}

terraform {
  source = "../../../modules/ec2"
}

inputs = {
  vpc_id = dependency.vpc.outputs.vpc_id
}
```

Terragrunt ensures **VPC is deployed before EC2**.

---

# 🧾 Common Terragrunt Commands

Initialize:

```
terragrunt init
```

Plan:

```
terragrunt plan
```

Apply:

```
terragrunt apply
```

Run across multiple modules:

```
terragrunt run-all apply
```

---

# ⚖️ Terraform vs Terragrunt

| Feature                      | Terraform | Terragrunt     |
| ---------------------------- | --------- | -------------- |
| Infrastructure provisioning  | ✅ Yes     | Uses Terraform |
| DRY configuration            | ❌ Limited | ✅ Strong       |
| Multi environment management | ⚠️ Manual | ✅ Automated    |
| Dependency management        | ⚠️ Manual | ✅ Built-in     |
| Remote state automation      | ❌ Manual  | ✅ Automatic    |
| Infrastructure orchestration | ❌ No      | ✅ Yes          |

---

# 📊 Advantages of Terragrunt

* 🔁 Eliminates duplicate Terraform code
* 🌍 Simplifies multi-environment infrastructure
* 🔗 Handles module dependencies
* ⚙️ Simplifies CI/CD pipelines
* 📦 Improves Terraform project structure

---

# ⚠️ Limitations

* Adds an extra layer to manage
* Requires learning new configuration
* Debugging may involve both Terragrunt and Terraform

---

# 🎤 Quick Interview Definition

**Terragrunt is a wrapper tool for Terraform that simplifies infrastructure management by enabling DRY configurations, managing multiple environments, handling module dependencies, and automating Terraform workflows.**

It improves Terraform scalability in large DevOps infrastructures.
