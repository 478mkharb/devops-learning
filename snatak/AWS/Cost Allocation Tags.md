# AWS Cost Allocation Tags

## What are AWS Cost Allocation Tags?

AWS Cost Allocation Tags are **metadata (key-value pairs)** assigned to AWS resources (EC2, S3, RDS, Lambda, etc.) to categorize and track cloud spending.

They help organizations identify **which team, project, application, or environment is consuming AWS costs**.

---

## Example

| Resource | Tag Key | Tag Value |
|----------|---------|-----------|
| EC2 Instance | Project | Employee-API |
| EC2 Instance | Environment | Production |
| S3 Bucket | Team | DevOps |
| RDS | CostCenter | Finance |

---

## Benefits

- Track costs by project or department
- Simplify chargeback/showback
- Improve cost visibility
- Generate cost reports
- Support FinOps practices

---

## Common Cost Allocation Tags

| Tag Key | Example Value |
|----------|---------------|
| Project | Employee-API |
| Environment | Dev |
| Team | DevOps |
| Owner | Mukesh |
| Application | OT-Microservices |
| CostCenter | Finance |

---

## DevOps Interview Answer

> AWS Cost Allocation Tags are key-value pairs assigned to AWS resources to categorize and track cloud costs. They help organizations analyze spending by project, team, environment, or application using AWS Cost Explorer and Cost and Usage Reports.

---
