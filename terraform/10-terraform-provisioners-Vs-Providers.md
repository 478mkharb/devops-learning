# 🔥 Terraform Providers vs Provisioners — Quick Comparison

---

## 🔹 Core Difference

* **Provider** → Defines *where* Terraform creates infrastructure (AWS, Azure, GCP)
* **Provisioner** → Defines *what to run* inside that infrastructure after creation

---

## 🔹 Key Differences

| Feature     | Provider             | Provisioner                 |
| ----------- | -------------------- | --------------------------- |
| Purpose     | Connect to cloud/API | Run scripts inside resource |
| Required    | Yes                  | No                          |
| Execution   | During plan & apply  | After resource creation     |
| Scope       | Global               | Resource-level              |
| Type        | Declarative          | Imperative                  |
| Reliability | High                 | Low                         |

---

## 🔹 Example

### Provider

```hcl
provider "aws" {
  region = "ap-south-1"
}
```

### Provisioner

```hcl
provisioner "remote-exec" {
  inline = [
    "sudo apt update"
  ]
}
```

---

## 🔥 One-Liner

> Provider = *Where infra is created*
> Provisioner = *What runs after creation*

---
