# Terraform Dynamic Block

## 1. What is a Dynamic Block?

A **dynamic block** in Terraform allows you to **generate repeated nested blocks inside a resource automatically using loops**.

It is useful when a resource requires multiple nested configurations like:

* security group ingress rules
* EBS volumes
* load balancer listeners
* IAM policy statements

Instead of writing these blocks manually many times, Terraform can generate them dynamically.

---

# 2. Why Dynamic Blocks Are Needed

Many Terraform resources contain **nested blocks**.

Example (security group):

```
ingress {
  from_port = 80
  to_port   = 80
}

ingress {
  from_port = 443
  to_port   = 443
}
```

If you have **10–20 rules**, the configuration becomes repetitive and hard to maintain.

Dynamic blocks solve this by creating blocks using loops.

---

# 3. Basic Syntax

```hcl
dynamic "<BLOCK_NAME>" {
  for_each = <COLLECTION>

  content {
    # block configuration
  }
}
```

## Components

| Component  | Meaning                                        |
| ---------- | ---------------------------------------------- |
| dynamic    | Terraform keyword to create blocks dynamically |
| BLOCK_NAME | Name of nested block                           |
| for_each   | Loop through list or map                       |
| content    | Actual configuration of generated block        |

---

# 4. Example – Security Group Ingress Rules

## Without Dynamic Block

```hcl
resource "aws_security_group" "web_sg" {

  ingress {
    from_port = 80
    to_port   = 80
    protocol  = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port = 443
    to_port   = 443
    protocol  = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

}
```

This becomes difficult to manage when many rules exist.

---

## With Dynamic Block

```hcl
variable "ports" {
  default = [80, 443, 8080]
}

resource "aws_security_group" "web_sg" {

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

Terraform will automatically generate multiple ingress blocks.

---

# 5. Internal Behaviour

Terraform expands the configuration internally like this:

```
ingress { port = 80 }
ingress { port = 443 }
ingress { port = 8080 }
```

---

# 6. Example – Dynamic EBS Volumes

```hcl
variable "ebs_volumes" {
  default = [
    { device = "/dev/sdb", size = 10 },
    { device = "/dev/sdc", size = 20 }
  ]
}

resource "aws_instance" "server" {

  ami           = "ami-12345"
  instance_type = "t2.micro"

  dynamic "ebs_block_device" {

    for_each = var.ebs_volumes

    content {
      device_name = ebs_block_device.value.device
      volume_size = ebs_block_device.value.size
    }

  }

}
```

Terraform will attach multiple EBS volumes to the instance.

---

# 7. Real DevOps Use Cases

| Use Case           | Example                    |
| ------------------ | -------------------------- |
| Security groups    | Multiple ingress rules     |
| EC2 instances      | Multiple EBS volumes       |
| Load balancers     | Listener rules             |
| IAM policies       | Multiple policy statements |
| Kubernetes configs | Multiple container ports   |

---

# 8. Difference: for_each vs dynamic

| Feature  | for_each                  | dynamic                      |
| -------- | ------------------------- | ---------------------------- |
| Works on | Resources                 | Nested blocks                |
| Example  | Multiple EC2 instances    | Multiple ingress rules       |
| Purpose  | Create multiple resources | Create nested configurations |

---

# 9. Interview Answer

**Question:** What is a dynamic block in Terraform?

**Answer:**

A dynamic block in Terraform is used to generate repeated nested blocks inside a resource using a loop such as for_each. It helps avoid repetitive configuration and is commonly used for scenarios like multiple security group ingress rules, EBS volumes, and load balancer listeners.

---

# 10. Simple Analogy

Without dynamic block:

```
Write rule 1
Write rule 2
Write rule 3
```

With dynamic block:

```
Loop rules list → automatically generate rules
```
