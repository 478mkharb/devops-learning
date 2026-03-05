# 🔄 Terraform `refresh` vs `import`

Many engineers get confused between **`terraform refresh`** and **`terraform import`** because both commands update the **Terraform state file**. However, they solve **completely different problems**.

The key idea is:

| Situation                                  | Command             |
| ------------------------------------------ | ------------------- |
| Resource already exists in Terraform state | `terraform refresh` |
| Resource exists in cloud but NOT in state  | `terraform import`  |

---

# 🧠 First Understand Terraform State

Terraform tracks infrastructure using the state file:

```
terraform.tfstate
```

The state stores mappings like:

```
aws_instance.web → i-123456
```

Meaning:

* Terraform resource: `aws_instance.web`
* Real infrastructure ID: `i-123456`

Terraform relies on this mapping to manage infrastructure.

---

# 🔄 Terraform Refresh

## Purpose

`terraform refresh` **updates attributes of resources that are already tracked in the Terraform state**.

Terraform queries the cloud provider API and updates the state file with the **latest infrastructure values**.

Example command:

```
terraform refresh
```

---

## Example Scenario

State file contains:

```
aws_instance.web → i-123456
instance_type = t3.micro
```

Terraform already knows this EC2 instance exists.

Now someone manually changes the instance in AWS console:

```
t3.micro → t3.small
```

Now:

| Location         | Value    |
| ---------------- | -------- |
| State file       | t3.micro |
| Actual AWS infra | t3.small |

State is outdated.

---

### Run Refresh

```
terraform refresh
```

Terraform does:

```
Read resource from state
        ↓
Call AWS API
        ↓
Read actual infrastructure
        ↓
Update terraform.tfstate
```

State becomes:

```
instance_type = t3.small
```

Important:

✔ Refresh **only works for resources already in state**.

---

# 📥 Terraform Import

## Purpose

`terraform import` is used when **infrastructure exists in the cloud but Terraform does not know about it yet**.

It creates a mapping between an existing resource and the Terraform state.

Example command:

```
terraform import aws_instance.web i-123456
```

---

## Example Scenario

Suppose an EC2 instance already exists in AWS:

```
i-123456
```

But Terraform state contains:

```
(empty)
```

Terraform thinks **no infrastructure exists**.

---

### If You Run Refresh

```
terraform refresh
```

Terraform checks state and finds:

```
No resources in state
```

Result:

```
Nothing happens
```

Terraform cannot refresh something it does not know about.

---

### Now Run Import

```
terraform import aws_instance.web i-123456
```

Terraform does:

```
Existing AWS resource
        ↓
Create mapping in state
        ↓
aws_instance.web → i-123456
```

Now Terraform state contains the resource.

After this you can run:

```
terraform plan
```

Terraform will compare configuration with real infrastructure.

---

# 📊 Visual Comparison

## Before Refresh

```
State
aws_instance.web → i-123456
```

Refresh updates **attributes only**.

---

## Before Import

```
State
(empty)
```

Import creates **a new state entry**.

---

# 📊 Refresh vs Import Summary

| Feature                 | Terraform Refresh          | Terraform Import              |
| ----------------------- | -------------------------- | ----------------------------- |
| Works on                | Resources already in state | Resources not in state        |
| Purpose                 | Sync state with real infra | Add existing infra to state   |
| Infrastructure creation | ❌ No                       | ❌ No                          |
| Updates state           | ✅ Yes                      | ✅ Yes                         |
| Requires resource block | Already exists             | Must be defined before import |

---

# 🧠 Simple Analogy

Think of Terraform state like a **database table**.

Refresh:

```
Update existing rows
```

Import:

```
Insert new row into table
```

---

# 🎤 Interview Answer

`terraform refresh` updates the attributes of resources that are already tracked in the Terraform state by querying the cloud provider APIs, while `terraform import` is used to add existing infrastructure resources that are not yet tracked into the Terraform state so Terraform can begin managing them.
