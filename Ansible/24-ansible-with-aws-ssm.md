# How Ansible Works with AWS SSM via API – Deep Dive 🔐☁️

This document explains **in depth** how Ansible executes tasks on EC2 instances **without SSH**, using **AWS Systems Manager (SSM)** and **AWS APIs**.

This is a **senior‑level concept** frequently asked in interviews and heavily used in **locked‑down AWS environments**.

---

## The Core Idea (Big Picture)

> **Ansible does NOT connect to EC2 instances when using SSM.**
> **It connects to AWS APIs, and AWS delivers the commands to the instance via the SSM Agent.**

This is why it works even when:

* SSH (22) is blocked ❌
* Instances are in private subnets 🔒
* No inbound ports are allowed

---

## Key Components Involved

### 1️⃣ Ansible Control Node

* Runs Ansible playbooks
* Uses **AWS credentials** (IAM user/role)
* Calls **AWS SSM APIs**

---

### 2️⃣ AWS Systems Manager (SSM)

* Managed AWS service
* Receives commands via API
* Queues and delivers commands to instances

---

### 3️⃣ SSM Agent (CRITICAL COMPONENT)

Runs on each EC2 instance:

```text
amazon-ssm-agent
```

Responsibilities:

* Maintains **outbound HTTPS (443)** connection to AWS
* Polls for commands
* Executes commands locally (usually as root)
* Returns output back to AWS

📌 This is an **agent-based model**.

---

### 4️⃣ IAM (Security Layer)

Controls:

* Who can send commands
* Which instances can receive commands
* What actions are allowed

No SSH keys involved ❌

---

## Full Execution Flow (Step by Step)

```text
┌────────────────────┐
│ Ansible Control    │
│ Node               │
│ (aws_ssm plugin)   │
└─────────┬──────────┘
          │ 1️⃣ AWS API Call (SendCommand)
          ▼
┌────────────────────┐
│ AWS SSM Service    │
│ (Regional API)     │
└─────────┬──────────┘
          │ 2️⃣ Command queued
          ▼
┌────────────────────┐
│ SSM Agent on EC2   │◀───────┐
│ (Outbound HTTPS)   │        │
└─────────┬──────────┘        │
          │ 3️⃣ Execute command│
          ▼                   │
┌────────────────────┐        │
│ EC2 Instance       │        │
│ (No SSH access)    │        │
└─────────┬──────────┘        │
          │ 4️⃣ Output sent back
          ▼                   │
┌────────────────────┐        │
│ AWS SSM Service    │────────┘
└─────────┬──────────┘
          │ 5️⃣ Result returned
          ▼
┌────────────────────┐
│ Ansible Output     │
└────────────────────┘
```

---

## Important Detail: Where Modules Execute

With SSH:

* Ansible copies modules to the host

With SSM:

* **Modules run inside the SSM execution environment**
* Commands are wrapped as SSM documents

👉 Ansible never opens a shell on the instance.

---

## How Ansible Is Configured to Use SSM

### Inventory Example

```ini
[ec2]
i-0abc123def456
```

---

### Variables Required

```yaml
ansible_connection: aws_ssm
ansible_aws_ssm_region: us-east-1
ansible_aws_ssm_s3_bucket: my-ssm-logs
```

---

## Example Playbook (No SSH)

```yaml
- hosts: ec2
  gather_facts: false
  tasks:
    - name: Check uptime via SSM
      command: uptime

    - name: Ensure nginx is installed
      shell: yum install -y nginx
```

What actually happens:

* Ansible → AWS API (`SendCommand`)
* SSM Agent executes commands
* Output returned to Ansible

---

## Security Model Explained 🔐

### IAM on Control Node

```json
{
  "Effect": "Allow",
  "Action": [
    "ssm:SendCommand",
    "ssm:GetCommandInvocation"
  ],
  "Resource": "*"
}
```

### IAM Role on EC2

```json
{
  "Effect": "Allow",
  "Action": [
    "ssm:UpdateInstanceInformation",
    "ssm:ListCommands"
  ],
  "Resource": "*"
}
```

---

## Why Enterprises Prefer SSM 🔥

✔ No inbound ports
✔ No SSH key management
✔ Centralized audit logs
✔ IAM-based access control
✔ Works in private subnets

---

## Key Limitations (Important)

⚠️ Slower than SSH for large file transfers
⚠️ Requires AWS-only environment
⚠️ Some Ansible modules behave differently
⚠️ Debugging is less interactive

---

## Comparison: SSH vs SSM

| Feature                 | SSH | AWS SSM |
| ----------------------- | --- | ------- |
| Agent required          | ❌   | ✅       |
| Inbound ports           | ✅   | ❌       |
| Uses AWS APIs           | ❌   | ✅       |
| Works in private subnet | ⚠️  | ✅       |
| Audit logging           | ⚠️  | ✅       |

---

## Mental Model 🧠

> **With SSH, Ansible talks to the server.**
> **With SSM, Ansible talks to AWS, and AWS talks to the server.**

---

## Interview One‑Liner 🎯

> **Ansible works with AWS SSM by sending commands through AWS APIs, which are executed by the SSM agent on EC2 instances over outbound HTTPS, eliminating the need for SSH.**

---

## Summary

* This is an agent‑based execution model
* Ansible never connects to the instance
* AWS SSM acts as the secure command broker
* Ideal for locked‑down cloud environments

---
