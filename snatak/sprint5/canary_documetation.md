# <h1 align="center">Canary Deployment Strategy using Immutable Infrastructure</h1>

<div align="center">
<img width="120" alt="AWS" src="https://upload.wikimedia.org/wikipedia/commons/9/93/Amazon_Web_Services_Logo.svg" />
</div>

<br/>

---

<div align="center">

| Author | Created On | Version | Last Updated By | Last Edited On | Pre Reviewer | L0 Reviewer | L1 Reviewer | L2 Reviewer |
| ------------ | ---------- | ------- | --------------- | -------------- | ------------ | ----------- | ----------- | ----------- |
| Mukesh Kharb | 10/08/2026 | 1.0 | Mukesh Kharb | 10/08/2026| Team | Mohit Kumar | Faisal Khan | Mahesh Kumar |

</div>

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Pre-Requisites](#2-pre-requisites)
3. [Objective](#3-objective)
4. [Solution Overview](#4-solution-overview)
5. [v1 and v2 Architecture](#5-v1-and-v2-architecture)
6. [Traffic Distribution](#6-traffic-distribution)
7. [Terraform Implementation](#7-terraform-implementation)
8. [Deployment Flow](#8-deployment-flow)
9. [Monitoring and Validation](#9-monitoring-and-validation)
10. [Rollback Strategy](#10-rollback-strategy)
11. [Cost Considerations](#11-cost-considerations)
12. [Common Issues and Troubleshooting](#12-common-issues-and-troubleshooting)
13. [Conclusion](#13-conclusion)
14. [Contact Information](#14-contact-information)
15. [References](#15-references)

---

<a id="1-introduction"></a>

## 1. Introduction

Canary deployment is a progressive deployment strategy where a new application version is gradually introduced to production traffic instead of sending all traffic to the new version at once.

In this design:

- **v1** represents the existing application version.
- **v2** represents the new application version.
- v1 and v2 run independently during the rollout.
- Traffic is gradually shifted from v1 to v2.
- v1 remains available for rollback until v2 is successfully validated.

The infrastructure is managed using **Terraform** and follows the **Immutable Infrastructure** approach, where new infrastructure is created for v2 instead of modifying the existing v1 infrastructure.

---

<a id="2-pre-requisites"></a>

## 2. Pre-Requisites

| Component | Requirement |
| ------------ | ------------ |
| AWS Account | Configured AWS account |
| Terraform | Installed and configured |
| VPC | Required VPC and subnets |
| ALB | Application Load Balancer |
| Target Groups | v1 and v2 Target Groups |
| Auto Scaling | v1 and v2 Auto Scaling Groups |
| EC2 | Application instances |
| IAM | Required AWS permissions |

### Verify Terraform Installation

```bash
terraform version
```

### Verify AWS Access

```bash
aws sts get-caller-identity
```

---

<a id="3-objective"></a>

## 3. Objective

The objective of this design is to implement a **v1/v2 Canary deployment strategy using immutable infrastructure**.

The solution should provide:

- Gradual traffic shifting from v1 to v2.
- Independent v1 and v2 infrastructure.
- Monitoring of v2 before increasing traffic.
- Quick rollback to v1 in case of failure.
- Infrastructure management through Terraform.
- Controlled additional infrastructure cost during deployment.

---

<a id="4-solution-overview"></a>

## 4. Solution Overview

The proposed solution uses an **AWS Application Load Balancer (ALB)** to distribute traffic between the v1 and v2 Target Groups.

Terraform manages the infrastructure required for both versions.

```text
                         Users
                           |
                           v
                          ALB
                           |
                    ALB Listener
                           |
                  Weighted Routing
                    /             \
                   /               \
                 v1                v2
            Target Group      Target Group
                 |                  |
               ASG-v1             ASG-v2
                 |                  |
              EC2 v1             EC2 v2
```

The ALB controls the traffic percentage between v1 and v2, while the Auto Scaling Groups manage the capacity of each version.

---

<a id="5-v1-and-v2-architecture"></a>

## 5. v1 and v2 Architecture

v1 and v2 are maintained as separate infrastructure versions.

```text
v1 Target Group
       |
     ASG-v1
       |
   EC2 v1 Instances


v2 Target Group
       |
     ASG-v2
       |
   EC2 v2 Instances
```

### v1

- Represents the current production version.
- Continues serving traffic while v2 is being validated.
- Acts as the rollback version.

### v2

- Represents the new application version.
- Is deployed without modifying v1.
- Initially receives a small percentage of traffic.
- Receives increasing traffic after successful validation.

This separation supports the immutable infrastructure principle because v1 is not modified to become v2.

---

<a id="6-traffic-distribution"></a>

## 6. Traffic Distribution

Traffic distribution is controlled by the **ALB Listener** using weighted forwarding.

The traffic percentage is assigned to the **Target Groups**, not directly to individual EC2 instances.

### Traffic Rollout

| Stage | v1 Traffic | v2 Traffic |
| ------------ | ------------ | ------------ |
| Initial | 100% | 0% |
| Stage 1 | 90% | 10% |
| Stage 2 | 75% | 25% |
| Stage 3 | 50% | 50% |
| Final | 0% | 100% |

### Example

```text
                       ALB Listener
                            |
                    Weighted Forward
                       /          \
                    90%            10%
                     |               |
                   TG-v1           TG-v2
                     |               |
                   ASG-v1          ASG-v2
                     |               |
                  EC2 v1          EC2 v2
```

The traffic weight can be changed as v2 passes each validation stage.

---

<a id="7-terraform-implementation"></a>

## 7. Terraform Implementation

Terraform is used to create and manage the infrastructure.

### Terraform Resources

```text
Terraform
   |
   +-- ALB
   +-- ALB Listener
   +-- v1 Target Group
   +-- v2 Target Group
   +-- v1 Launch Template
   +-- v2 Launch Template
   +-- v1 ASG
   +-- v2 ASG
   +-- Security Groups
```

### Suggested Terraform Structure

```text
terraform/
├── provider.tf
├── variables.tf
├── terraform.tfvars
├── alb.tf
├── target-groups.tf
├── launch-template.tf
├── asg.tf
├── listener.tf
└── outputs.tf
```

### Weighted Forwarding

The ALB listener can use weighted Target Groups:

```hcl
forward {
  target_group {
    arn    = aws_lb_target_group.v1.arn
    weight = 90
  }

  target_group {
    arn    = aws_lb_target_group.v2.arn
    weight = 10
  }
}
```

The weights can be changed during the rollout:

```text
90/10 -> 75/25 -> 50/50 -> 0/100
```

Terraform can be used to apply each traffic change in a controlled manner.

---

<a id="8-deployment-flow"></a>

## 8. Deployment Flow

The deployment starts with v1 serving all production traffic.

```text
v1 -> 100%
v2 ->   0%
```

### Step 1: Deploy v2

Create the v2 infrastructure using Terraform.

```text
Create v2 Launch Template
          |
          v
       Create v2 ASG
          |
          v
    Create v2 Target Group
          |
          v
    Register v2 Instances
```

### Step 2: Start Canary Traffic

Route a small percentage of traffic to v2.

```text
v1 -> 90%
v2 -> 10%
```

### Step 3: Validate v2

Monitor the application and infrastructure.

If v2 is healthy, increase traffic.

```text
90/10 -> 75/25 -> 50/50 -> 0/100
```

### Step 4: Complete Rollout

After successful validation:

```text
v1 -> 0%
v2 -> 100%
```

v2 becomes the active production version.

---

<a id="9-monitoring-and-validation"></a>

## 9. Monitoring and Validation

v2 should be monitored at every traffic stage before increasing its traffic percentage.

### Metrics to Monitor

- HTTP 4xx errors
- HTTP 5xx errors
- Response latency
- EC2 CPU utilization
- EC2 memory utilization
- Target health
- Application health
- Application logs

### Validation Flow

```text
             v2
              |
          Monitoring
              |
        +-----+-----+
        |           |
     Healthy     Unhealthy
        |           |
        v           v
 Increase       Rollback
 Traffic
```

Traffic should only be increased after v2 meets the required health and performance criteria.

---

<a id="10-rollback-strategy"></a>

## 10. Rollback Strategy

If v2 shows errors or performance issues, traffic can be shifted back to v1.

### Example

Before rollback:

```text
v1 -> 50%
v2 -> 50%
```

After rollback:

```text
v1 -> 100%
v2 -> 0%
```

The rollback is performed through the ALB listener weights.

Since v1 infrastructure remains available during the rollout, the application can quickly return to the previous version without rebuilding v1.

---

<a id="11-cost-considerations"></a>

## 11. Cost Considerations

During the Canary rollout, v1 and v2 infrastructure run at the same time.

The main additional cost comes from the EC2 instances required for v2.

### Cost Optimization

- Start v2 with the minimum required capacity.
- Increase v2 capacity as traffic increases.
- Avoid unnecessary large instance types during validation.
- Remove v1 infrastructure after successful v2 promotion and the agreed rollback period.

The ASG itself is not the main cost driver; the compute resources running inside the ASGs are.

---

<a id="12-common-issues-and-troubleshooting"></a>

## 12. Common Issues and Troubleshooting

| Issue | Possible Cause | Resolution |
| ------------ | ------------ | ------------ |
| v2 receives no traffic | Incorrect listener weights | Verify ALB listener configuration |
| v2 target is unhealthy | Application or health check failure | Check target health and application logs |
| Terraform plan fails | Incorrect resource configuration | Run `terraform validate` |
| Unexpected traffic distribution | Incorrect target group weights | Verify weighted forwarding |
| Rollback does not work | Incorrect listener configuration | Set v1 to 100% and v2 to 0% |
| v2 instances are unavailable | ASG or Launch Template issue | Check ASG activity and EC2 health |

### Useful Terraform Commands

```bash
terraform fmt
```

```bash
terraform validate
```

```bash
terraform plan
```

```bash
terraform apply
```

---

<a id="13-conclusion"></a>

## 13. Conclusion

This design provides a controlled **v1/v2 Canary deployment strategy using immutable infrastructure**.

The responsibilities are clearly separated:

- **Terraform** manages the infrastructure.
- **ALB** controls traffic distribution.
- **Target Groups** separate v1 and v2 traffic.
- **ASGs** manage instance capacity.
- **Monitoring** validates v2.
- **ALB weights** provide quick rollback to v1.

The approach allows the new version to be introduced gradually while keeping the existing version available for rollback.

---

<a id="14-contact-information"></a>

## 14. Contact Information

| Name | Contact |
| ------------ | ------------ |
| Mukesh Kharb | mukesh.Kharb.snaatak@mygurukulam.co |

---

<a id="15-references"></a>

## 15. References

| S.No | Description | Reference |
| ------------ | ------------ | ------------ |
| 1 | AWS Application Load Balancer Documentation | [AWS ALB Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) |
| 2 | AWS Target Groups | [AWS Target Groups Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html) |
| 3 | AWS Auto Scaling | [AWS Auto Scaling Documentation](https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html) |
| 4 | Terraform AWS Provider | [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs) |
