# 🛡️ Sentinel Policies in Terraform

## 📘 What is Sentinel?

Sentinel is a **Policy as Code framework developed by HashiCorp** that allows organizations to enforce **security, compliance, governance, and operational rules** on infrastructure managed by Terraform.

Sentinel policies run during the **Terraform plan phase** and determine whether infrastructure changes should be allowed or blocked.

Sentinel is mainly available with:

* ☁️ Terraform Cloud
* 🏢 Terraform Enterprise

It ensures that infrastructure created by developers follows **organizational standards before deployment**.

---

# ⚙️ Where Sentinel Works in Terraform Workflow

Terraform workflow with policy enforcement:

👨‍💻 Developer Code → 📋 terraform plan → 🛡️ Sentinel Policy Check → 🚀 terraform apply

Behavior:

* ✅ If policy **passes** → Terraform continues
* ❌ If policy **fails** → Deployment is blocked

This allows companies to implement **governance before infrastructure is provisioned**.

---

# 📜 Types of Sentinel Policies

## 1️⃣ Advisory Policy

### 📖 Definition

Advisory policies only **provide warnings** if rules are violated but **do not stop the Terraform run**.

### ⚙️ Behavior

* Terraform run continues
* ⚠️ Warning is displayed
* Infrastructure still gets created

### 🎯 Use Case

Used for **best practice recommendations**.

Example:

All resources should contain tags.

If tags are missing → ⚠️ Warning is shown but deployment continues.

---

## 2️⃣ Soft Mandatory Policy

### 📖 Definition

Soft mandatory policies **enforce rules but can be overridden by administrators**.

### ⚙️ Behavior

* ❌ Terraform run initially fails
* 👨‍💼 Admin can override the policy
* ✅ Infrastructure deployment continues after override

### 🎯 Use Case

Used when companies want **control but still allow exceptions**.

Example:

Only approved AWS regions should be used.

If another region is used → Terraform blocks the run but **admin may approve the override**.

---

## 3️⃣ Hard Mandatory Policy

### 📖 Definition

Hard mandatory policies are **strict policies that cannot be overridden**.

### ⚙️ Behavior

* ❌ Terraform run fails
* 🚫 No override allowed
* ⛔ Deployment completely blocked

### 🎯 Use Case

Used for **critical security and compliance rules**.

Example:

Public S3 buckets are not allowed.

If violated → Deployment stops.

---

# 📊 Summary of Policy Types

| Policy Type    | Blocks Deployment | Override Allowed | Use Case                |
| -------------- | ----------------- | ---------------- | ----------------------- |
| Advisory       | ❌ No              | Not required     | Best practice warnings  |
| Soft Mandatory | ⚠️ Initially      | ✅ Yes            | Controlled enforcement  |
| Hard Mandatory | ⛔ Yes             | ❌ No             | Critical security rules |

---

# 🏭 Real Sentinel Policies Used in Production

Below are **common real-world Sentinel policies used in enterprise DevOps environments**.

---

# 🏷️ 1. Mandatory Resource Tagging Policy

### 🎯 Purpose

Ensure every infrastructure resource contains required tags.

Common required tags:

* Environment
* Owner
* Project
* CostCenter

### 🏢 Why Companies Use It

* 💰 Cost tracking
* 👤 Resource ownership
* 📊 Governance
* 🧾 Billing transparency

### 📌 Example Scenario

If a developer creates an EC2 instance without required tags, the Sentinel policy **blocks the Terraform deployment**.

---

# 🌐 2. Restrict Public S3 Buckets

### 🎯 Purpose

Prevent accidental **public exposure of data**.

### ⚠️ Example Violation

```
acl = "public-read"
```

or

```
public_access_block = false
```

### 🔐 Why Important

Many cloud data leaks happen due to **misconfigured public S3 buckets**.

Sentinel blocks deployment if public access is enabled.

---

# 💻 3. Restrict EC2 Instance Types

### 🎯 Purpose

Allow only **approved instance types**.

### ✅ Example Allowed Instances

* t3.micro
* t3.small
* t3.medium

### ❌ Example Blocked Instances

* m5.4xlarge
* c5.9xlarge

### 💰 Why Companies Use It

* Prevent excessive cloud costs
* Standardize infrastructure

---

# 🌍 4. Restrict Deployment Regions

### 🎯 Purpose

Allow infrastructure deployment only in **approved regions**.

### ✅ Example Allowed Regions

* us-east-1
* us-west-2
* eu-west-1

### 📜 Why Companies Enforce It

* Data residency regulations
* Compliance requirements
* Cost optimization

Example:

Block deployment in `ap-south-1` if not approved.

---

# 🔒 5. Enforce Encryption Policy

### 🎯 Purpose

Ensure storage services are **encrypted**.

Resources typically enforced:

* EBS volumes
* S3 buckets
* RDS databases

### ⚠️ Example Violation

```
encrypted = false
```

### 📋 Compliance Standards

* SOC2
* HIPAA
* PCI-DSS

Sentinel blocks infrastructure without encryption.

---

# 🚫 6. Restrict Open Security Groups

### 🎯 Purpose

Prevent infrastructure from being exposed to the internet.

### ⚠️ Example Violation

```
0.0.0.0/0
```

Especially dangerous for:

* SSH (22)
* RDP (3389)
* Databases

### 🔥 Real Security Risk

Open SSH ports are one of the **most common cloud misconfigurations**.

Sentinel blocks such rules.

---

# 📦 7. Limit Number of Resources

### 🎯 Purpose

Prevent accidental large infrastructure creation.

Example rule:

Maximum allowed EC2 instances in a Terraform plan:

```
<= 10
```

### 💸 Why Companies Use It

* Prevent cost explosions
* Avoid Terraform loop mistakes

---

# 🖥️ 8. Approved AMI Policy

### 🎯 Purpose

Allow only **company-approved machine images**.

Example approved AMIs:

* ami-0abc123
* ami-09xyz456

### 🔐 Why Important

Ensures:

* Secure OS images
* Patched systems
* Standardized environment

This prevents developers from launching insecure or unpatched images.

---

# ⚖️ Sentinel vs Open Policy Agent (OPA)

| Feature               | Sentinel                     | OPA                      |
| --------------------- | ---------------------------- | ------------------------ |
| Developed by          | HashiCorp                    | CNCF                     |
| Language              | Sentinel                     | Rego                     |
| Terraform Integration | Native                       | External                 |
| Usage                 | Terraform Cloud / Enterprise | Kubernetes, CI/CD, Cloud |

---

# ✅ Advantages of Sentinel

* 🔐 Enforces security automatically
* 🚫 Prevents misconfiguration
* 🏛️ Improves governance
* 📦 Standardizes infrastructure deployment
* ☁️ Integrates directly with Terraform Cloud

---

# ⚠️ Limitations

* Mainly available with Terraform Cloud/Enterprise
* Requires learning Sentinel language
* Not commonly used in small teams

---

# 🎤 Quick Interview Definition

Sentinel is a **policy-as-code framework used with Terraform Cloud and Terraform Enterprise to enforce governance, security, and compliance rules on infrastructure before it is deployed.**

It evaluates Terraform execution plans and blocks changes that violate organizational policies.
