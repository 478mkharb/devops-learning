# AWS Cost Management

### Q1. What is AWS Cost Explorer?

**Answer:** AWS Cost Explorer provides interactive analysis of AWS costs and usage over time. It helps identify cost drivers, trends, and service-level spending.# AWS Cost Management — Senior DevOps Notes

## 1. Cost Management Mindset

AWS cost optimization is not simply reducing the bill. It is the continuous process of delivering the required reliability, performance, security, and scalability at the lowest appropriate total cost.

| Dimension | DevOps focus |
|---|---|
| Visibility | Identify cost by account, service, application, environment, team, and owner |
| Allocation | Attribute spending to the team or workload responsible |
| Optimization | Remove waste and select appropriate resource sizes and pricing models |
| Governance | Prevent unapproved or unexpected spending |
| Automation | Detect idle resources and enforce lifecycle policies |
| Reliability | Never reduce cost by removing required redundancy or recovery capability |

---

## 2. Core AWS Cost Tools

| Tool | Purpose | DevOps use case |
|---|---|---|
| **Cost Explorer** | Analyze historical and current cost and usage | Find cost spikes and service-level cost drivers |
| **AWS Budgets** | Define cost, usage, or commitment thresholds | Alert teams before spending exceeds expectations |
| **Pricing Calculator** | Estimate expected cost before deployment | Compare architecture and instance choices during design |
| **Cost and Usage Report (CUR)** | Detailed billing and usage data delivered to S3 | Build chargeback, FinOps, and cost analytics pipelines |
| **Cost Anomaly Detection** | Detect unusual spending patterns | Identify unexpected resource creation or traffic increases |
| **Trusted Advisor** | Provides recommendations across several AWS areas | Find idle or underutilized resources where supported |
| **Billing Console** | Central billing and payment visibility | Review invoices, credits, and account-level spend |
| **Compute Optimizer** | Provides rightsizing recommendations for supported resources | Detect over-provisioned EC2, EBS, and other resources |

> Cost Explorer explains **where money is being spent**. Budgets and anomaly detection help identify **when spending becomes abnormal**. CUR supports detailed reporting and automation.

---

## 3. Cost Allocation and Tagging

Cost allocation tags allow AWS spending to be grouped by business or technical dimensions.

### Recommended tag standard

| Tag | Example | Purpose |
|---|---|---|
| `Application` | `otms` | Identify the workload |
| `Environment` | `dev`, `stage`, `prod` | Separate lifecycle environments |
| `Owner` | `platform-team` | Identify responsibility |
| `CostCenter` | `Snaatak` | Chargeback or accounting |
| `ManagedBy` | `terraform` | Identify infrastructure ownership |
| `Criticality` | `high`, `medium`, `low` | Support optimization decisions |
| `DataClass` | `standard`, `sensitive` | Support security and retention decisions |

### Tagging principles

- Use a consistent naming convention across Terraform, EC2, EBS, load balancers, and other taggable resources.
- Activate required user-defined cost allocation tags in the Billing console.
- Do not depend on tags alone for security enforcement; use IAM conditions, SCPs, and policy controls where appropriate.
- Treat missing tags as a governance failure that should be detected automatically.

---

## 4. EC2 Cost Optimization

| Technique | When to use | Important consideration |
|---|---|---|
| **Rightsizing** | Instance is over-provisioned | Validate CPU, memory, network, and application latency |
| **Auto Scaling** | Demand varies over time | Scale on useful workload metrics, not only CPU |
| **Spot Instances** | Workload tolerates interruption | Use checkpointing, retries, and replacement capacity |
| **Savings Plans** | Predictable eligible compute usage | Requires an hourly spend commitment |
| **Reserved Instances** | Stable, predictable usage with suitable scope | Less flexible than general compute commitments in some cases |
| **Scheduled shutdown** | Non-production resources used only during working hours | Avoid shutting down stateful systems without recovery planning |
| **Right instance family** | Workload has specific CPU, memory, storage, or network needs | Choose based on measured workload characteristics |
| **Golden AMIs** | Faster and more consistent provisioning | Reduce bootstrapping time and configuration drift |

### Spot Instances

Spot Instances use spare AWS capacity at a discounted rate. They are suitable for interruptible workloads such as:

- Batch processing
- CI/CD build agents
- Distributed data processing
- Stateless workers
- Test environments
- Render or simulation jobs

They should not be the only capacity for workloads that cannot tolerate interruption.

### Rightsizing workflow

```text
Collect utilization data
        ↓
Identify over-provisioned resources
        ↓
Check performance and SLO impact
        ↓
Test smaller or better-suited instance type
        ↓
Apply through Terraform or approved change process
        ↓
Measure cost and performance again
```

---

## 5. Savings Plans vs Reserved Instances

| Feature | Savings Plans | Reserved Instances |
|---|---|---|
| Commitment | Hourly spend commitment or eligible usage commitment | Commitment to a specific reservation configuration |
| Flexibility | Generally more flexible for eligible compute usage | Depends on RI type and scope |
| Best fit | Teams with predictable compute spending but changing instance choices | Stable workloads with predictable instance requirements |
| Main risk | Commit too much usage | Reserve the wrong type, Region, or scope |
| DevOps practice | Review commitment coverage and utilization regularly | Review reservation utilization before purchasing or renewing |

**Rule:** Do not purchase commitments only because they offer a discount. First verify that the workload is stable enough to use the commitment.

---

## 6. S3 Cost Optimization

| Technique | Purpose |
|---|---|
| Lifecycle transitions | Move older objects to cheaper storage classes |
| Lifecycle expiration | Delete temporary files, logs, and obsolete artifacts |
| Intelligent-Tiering | Automatically optimize changing access patterns |
| Version cleanup | Expire noncurrent versions when retention allows |
| Multipart-upload cleanup | Remove incomplete multipart uploads |
| Compression | Reduce stored data and transfer volume where appropriate |
| Prefix and tag filters | Apply different retention policies to different data classes |
| Storage Lens | Analyze storage usage and optimization opportunities |

### Example lifecycle policy

```text
Day 0       → S3 Standard
Day 30      → S3 Standard-IA or Intelligent-Tiering
Day 90      → Glacier Flexible Retrieval
Day 365     → Expiration, if retention permits
```

The actual policy must be based on access frequency, minimum storage duration, retrieval cost, compliance, and recovery requirements.

> S3 lifecycle rules belong in the S3 README. This document covers only their cost-management impact.

---

## 7. Cost Control in CI/CD

CI/CD systems can generate significant cost through build agents, artifacts, logs, test environments, and temporary infrastructure.

| Cost area | Optimization approach |
|---|---|
| Jenkins agents | Use ephemeral agents and terminate them after builds |
| EC2 build workers | Use Spot where builds can retry safely |
| Build duration | Cache dependencies and optimize pipeline stages |
| Artifacts | Store only required artifacts and apply retention policies |
| Logs | Set CloudWatch retention periods instead of indefinite retention |
| Test environments | Create on demand and destroy automatically |
| Terraform environments | Use scheduled cleanup and owner-based expiration |
| AMI creation | Remove unused AMIs and associated snapshots |
| Container images | Remove obsolete image tags and unused layers where applicable |

### Recommended CI/CD controls

- Apply `Application`, `Environment`, `Owner`, and `CostCenter` tags to temporary resources.
- Enforce automatic cleanup for preview and test environments.
- Keep artifact retention aligned with release and rollback requirements.
- Avoid running expensive integration tests unnecessarily on every branch.
- Track build duration and infrastructure cost per pipeline where practical.

---

## 8. Budgets and Alerts

AWS Budgets can monitor cost, usage, and certain commitment-related metrics.

| Threshold type | Example | Purpose |
|---|---|---|
| Actual cost | Alert at 80% of monthly budget | Detect spending already incurred |
| Forecasted cost | Alert when projected spend exceeds budget | Provide early warning |
| Usage | Alert when a service exceeds expected usage | Detect abnormal consumption |
| Commitment coverage/utilization | Alert on poor commitment efficiency | Prevent unused commitments |

### Practical budget model

| Budget | Scope |
|---|---|
| Organization budget | Total AWS spending |
| Account budget | Spending per AWS account |
| Environment budget | Dev, stage, or production |
| Application budget | Spending for one workload |
| Service budget | EC2, S3, RDS, or another service |
| Team budget | Cost-center or owner-based allocation |

A budget alert is a notification mechanism. It does not automatically stop all spending unless additional automation or control mechanisms are configured.

---

## 9. Organizations and Consolidated Billing

AWS Organizations groups multiple AWS accounts under centralized management.

| Capability | Cost-management value |
|---|---|
| Consolidated billing | Central view of member-account charges |
| Account separation | Isolate production, development, security, and shared services |
| Cost allocation | Attribute spending by account and workload |
| Volume pricing | May provide benefits across eligible consolidated usage |
| SCPs | Restrict services, Regions, or actions that could create unwanted cost |
| Central governance | Standardize tagging, budgets, and account controls |

### Recommended account structure

```text
AWS Organization
├── Management account
├── Security account
├── Log archive account
├── Shared services account
├── Production account
├── Staging account
└── Development account
```

Account separation improves governance and blast-radius control, but it does not replace tagging or detailed cost allocation.

---

## 10. Cost Anomaly Detection

Cost Anomaly Detection identifies unusual spending patterns using historical cost behavior.

Typical signals include:

- Unexpected EC2 instance creation
- Sudden data transfer increase
- S3 request or storage growth
- RDS or OpenSearch scaling
- Forgotten development resources
- Accidental high-volume logging
- Compromised credentials creating resources

### Incident response flow

```text
Anomaly detected
        ↓
Identify account, service, Region, and owner
        ↓
Compare CloudTrail activity and deployment history
        ↓
Validate whether usage is expected
        ↓
Stop or restrict unauthorized resources
        ↓
Fix the root cause
        ↓
Add preventive controls
```

---

## 11. Senior DevOps Cost Optimization Checklist

- [ ] Every production resource has an owner and cost center.
- [ ] Non-production resources have an automatic expiration or shutdown strategy.
- [ ] EC2 utilization is reviewed before rightsizing.
- [ ] Spot is used only for interruption-tolerant workloads.
- [ ] Savings Plans or Reserved Instances are purchased from measured baseline usage.
- [ ] S3 lifecycle policies exist for logs, artifacts, backups, and temporary data.
- [ ] CloudWatch log retention is explicitly configured.
- [ ] Unused EBS volumes, snapshots, AMIs, and Elastic IPs are detected.
- [ ] Budgets exist for accounts, environments, or applications.
- [ ] Cost anomaly alerts reach the responsible operations team.
- [ ] Terraform tags and naming conventions are standardized.
- [ ] Cost changes are reviewed as part of architecture and pull requests.

---

## 12. Common Cost Failure Scenarios

| Scenario | Likely cause | Corrective action |
|---|---|---|
| EC2 bill suddenly increases | Forgotten instances or scaling event | Check Cost Explorer, ASG activity, and CloudTrail |
| S3 bill increases | Version accumulation, large objects, requests, or data transfer | Review Storage Lens, versions, lifecycle, and access patterns |
| EBS cost remains after instance termination | Volumes were not deleted | Find unattached volumes and verify retention requirements |
| Snapshot cost grows continuously | Excessive or old snapshots | Apply snapshot retention and review backup policy |
| NAT Gateway cost is high | Large data transfer through NAT | Use VPC endpoints where appropriate and review routing |
| CI/CD cost increases | Long builds or persistent agents | Use ephemeral workers, caching, and cleanup |
| Budget alert arrives too late | Only actual-cost threshold configured | Add forecasted-cost and anomaly alerts |
| Savings Plan is underutilized | Commitment exceeds actual eligible usage | Review utilization before renewal or additional purchase |

---

## 13. Useful AWS CLI Commands

### Cost and billing permissions

```bash
aws sts get-caller-identity
```

### List EC2 instances

```bash
aws ec2 describe-instances \
  --query 'Reservations[].Instances[].{Id:InstanceId,Type:InstanceType,State:State.Name,Name:Tags[?Key==`Name`].Value|[0]}' \
  --output table
```

### Find unattached EBS volumes

```bash
aws ec2 describe-volumes \
  --filters Name=status,Values=available \
  --query 'Volumes[].{VolumeId:VolumeId,Size:Size,Type:VolumeType,AZ:AvailabilityZone}' \
  --output table
```

### Find snapshots owned by the account

```bash
aws ec2 describe-snapshots \
  --owner-ids self \
  --query 'Snapshots[].{Id:SnapshotId,Start:StartTime,Size:VolumeSize,Description:Description}' \
  --output table
```

### List S3 buckets

```bash
aws s3api list-buckets \
  --query 'Buckets[].Name' \
  --output table
```

### Review S3 lifecycle configuration

```bash
aws s3api get-bucket-lifecycle-configuration \
  --bucket <bucket-name>
```

### Review bucket tagging

```bash
aws s3api get-bucket-tagging \
  --bucket <bucket-name>
```

> Billing APIs may require specific permissions and may be limited by Region or account configuration. Use the Billing and Cost Management console or CUR for detailed financial analysis.

---

## 14. Senior Interview Questions

### Q1. What is the difference between Cost Explorer, Budgets, CUR, and Cost Anomaly Detection?

| Tool | Main role |
|---|---|
| Cost Explorer | Interactive cost and usage analysis |
| Budgets | Threshold-based alerts |
| CUR | Detailed billing dataset for reporting and analytics |
| Cost Anomaly Detection | Detect unusual spending patterns |

### Q2. How would you reduce EC2 cost without affecting availability?

First measure utilization and workload behavior. Then rightsize safely, use Auto Scaling, select suitable instance families, apply Savings Plans to stable baseline usage, and use Spot only for interruptible capacity. Required multi-AZ and recovery controls must remain intact.

### Q3. Why is Spot unsuitable as the only capacity for a critical service?

Spot capacity can be interrupted. A critical service needs reliable replacement capacity, usually through On-Demand or committed capacity combined with Auto Scaling and multi-AZ design.

### Q4. What is the difference between rightsizing and autoscaling?

| Concept | Meaning |
|---|---|
| Rightsizing | Select a better resource size or family for the workload |
| Autoscaling | Change the number of running resources based on demand |

They solve different problems and are often used together.

### Q5. How would you investigate an unexpected AWS bill increase?

1. Identify the account, Region, service, and usage type in Cost Explorer.
2. Compare the increase with deployment and scaling history.
3. Check CloudTrail for resource creation or configuration changes.
4. Review data transfer, NAT Gateway, logging, storage, and request charges.
5. Identify the owner and contain unauthorized or unnecessary resources.
6. Add a preventive control such as a budget, SCP, tag policy, or cleanup automation.

### Q6. Why should cost optimization be part of Terraform and CI/CD?

Infrastructure code determines resource size, count, storage, retention, and lifecycle. Reviewing cost-impacting changes during pull requests prevents expensive configuration from reaching production unnoticed.

### Q7. What is the danger of deleting resources only to reduce cost?

Deletion may remove required availability, backup, compliance, or recovery capability. Cost optimization must be evaluated against SLOs, RTO, RPO, security, and data-retention requirements.

---

## 15. One-Line Interview Answers

| Question | One-line answer |
|---|---|
| Cost Explorer? | Interactive analysis of AWS cost and usage. |
| AWS Budgets? | Threshold-based alerts for cost, usage, and commitments. |
| Pricing Calculator? | Estimates cost before deploying an architecture. |
| CUR? | Detailed billing and usage data for analytics and reporting. |
| Cost allocation tags? | Tags used to attribute cost to applications, teams, or environments. |
| Rightsizing? | Matching resource capacity to measured workload requirements. |
| Spot Instance? | Discounted spare capacity that can be interrupted. |
| Savings Plan? | Discount obtained by committing to eligible compute spend or usage. |
| Consolidated billing? | Centralized billing across AWS Organization accounts. |
| Cost anomaly detection? | Detection of unusual spending patterns. |
| FinOps? | Collaboration between engineering, finance, and business to optimize cloud value. |

---

## 16. Ownership Boundaries

| Topic | Canonical README |
|---|---|
| EC2 sizing and Spot implementation | `01-EC2` |
| AMI and golden-image optimization | `02-AMI` |
| Auto Scaling policies | `03-Launch-Template-ASG` |
| S3 lifecycle and storage classes | `09-S3` |
| CloudWatch log retention and alarms | `13-CloudWatch` |
| Organizations and SCP security controls | `23-AWS-Security-Architecture` |
| Terraform cost review and tagging | `22-Terraform-AWS-IaC` |
| Incident investigation and AWS CLI troubleshooting | `24-AWS-Troubleshooting` |

---

## Official References

- [AWS Cost Management](https://aws.amazon.com/aws-cost-management/)
- [AWS Cost Explorer](https://aws.amazon.com/aws-cost-management/aws-cost-explorer/)
- [AWS Budgets](https://aws.amazon.com/aws-cost-management/aws-budgets/)
- [AWS Pricing Calculator](https://calculator.aws/)
- [AWS Cost and Usage Reports](https://docs.aws.amazon.com/cur/latest/userguide/what-is-cur.html)
- [AWS Organizations](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html)

---

### Q2. What is AWS Budgets?

**Answer:** AWS Budgets lets you define cost, usage, or commitment thresholds and receive alerts when actual or forecast values exceed configured limits.

---

### Q3. What is the AWS Pricing Calculator?

**Answer:** AWS Pricing Calculator estimates AWS costs before deployment by modeling services, configurations, and expected usage.

---

### Q4. What is Cost and Usage Report?

**Answer:** The AWS Cost and Usage Report provides detailed AWS billing and usage data that can be delivered to S3 for analysis.

---

### Q5. What is consolidated billing?

**Answer:** Consolidated billing combines billing information for accounts in an AWS Organization and can provide centralized cost visibility and applicable volume pricing benefits.

---

### Q6. How can EC2 costs be reduced?

**Answer:** EC2 costs can be reduced through rightsizing, Auto Scaling, using Spot for interruptible workloads, Savings Plans or Reserved Instances for predictable usage, shutting down unused non-production resources, and selecting appropriate instance families.

---

### Q7. When should Spot be considered?

**Answer:** Spot Instances should be considered for workloads that can tolerate interruption and restart, such as batch processing, CI/CD workers, distributed analytics, and stateless compute. They are not appropriate as the only capacity for interruption-intolerant workloads.

---

### Q8. How do rightsizing recommendations help?

**Answer:** Rightsizing compares actual workload utilization with the provisioned resource size and identifies opportunities to use a more appropriate instance type or size. It can reduce cost from over-provisioning while preserving required performance.

---

### Q9. How can S3 lifecycle policies reduce cost?

**Answer:** Amazon S3 is an object-storage service. Data is stored as objects in buckets and can be protected, versioned, replicated, transitioned between storage classes, and accessed through APIs.

---

### Q10. How can Savings Plans reduce compute cost?

**Answer:** Savings Plans reduce eligible compute costs by providing discounted rates in exchange for a committed hourly spend over the commitment term. The discount applies automatically to eligible usage up to the committed amount according to the selected Savings Plan type.

---

### Q11. What are cost allocation tags?

**Answer:** Cost allocation tags categorize AWS costs by dimensions such as application, environment, team, or owner so spending can be analyzed and allocated.

---

### Q12. What is a budget alert?

**Answer:** A budget alert is a notification triggered when actual or forecast AWS cost or usage reaches a configured threshold. It helps teams detect unexpected spending early and can notify through supported AWS notification mechanisms.

---

### Q13. How can Organizations help control spend?

**Answer:** AWS Organizations centrally manages multiple AWS accounts. It supports account grouping, consolidated billing, governance policies, and centralized security controls.

---

### Q14. What is AWS Trusted Advisor cost optimization?

**Answer:** Cost optimization focuses on delivering business value at the lowest appropriate cost through rightsizing, elasticity, pricing models, storage optimization, and ongoing cost visibility.

---
