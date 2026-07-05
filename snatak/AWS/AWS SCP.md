# AWS Service Control Policies (SCP)

## What is an SCP?

A **Service Control Policy (SCP)** is an AWS Organizations policy that defines the **maximum permissions** available for AWS accounts within an organization.

SCPs **do not grant permissions**; they only **allow or deny** what IAM users and roles can do.

---

## Example

Suppose your company doesn't want developers launching expensive GPU instances.

SCP:

```text
Deny

ec2:RunInstances

Instance Type = p5.*
```

Even if an IAM user has **AdministratorAccess**, they cannot launch those instances because the SCP denies it.

---

## Benefits

- Centralized governance
- Restrict AWS services
- Prevent accidental resource creation
- Improve security
- Reduce unnecessary cloud costs

---

## Real DevOps Example

```text
AWS Organization

│

├── Development Account

├── Testing Account

└── Production Account

        │

        ▼

Apply SCP

↓

Deny

- Delete CloudTrail
- Disable GuardDuty
- Launch GPU Instances
- Create resources outside ap-south-1
```

---

## DevOps Interview Answer

> AWS Service Control Policies (SCPs) are organization-level policies that define the maximum permissions for AWS accounts. They enforce governance by restricting actions regardless of IAM permissions.
