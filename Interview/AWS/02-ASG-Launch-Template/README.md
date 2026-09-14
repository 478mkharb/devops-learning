# AWS Launch Templates & Auto Scaling Groups — Interview and Practical Notes

> **Scope:** This document covers only **Amazon EC2 Launch Templates** and **EC2 Auto Scaling Groups (ASGs)**, including how they work together.  
> **Excluded:** Detailed EC2 fundamentals, EBS, IAM, VPC networking, ENI, Security Groups, NACLs, Load Balancers, CloudWatch, and other separate AWS services.

## Table of Contents

1. [Launch Templates](#1-launch-templates)
2. [Auto Scaling Groups](#2-auto-scaling-groups)
3. [Launch Template and ASG Relationship](#3-launch-template-and-asg-relationship)
4. [ASG Capacity Settings](#4-asg-capacity-settings)
5. [Scaling Policies](#5-scaling-policies)
6. [Health Checks and Instance Replacement](#6-health-checks-and-instance-replacement)
7. [Instance Refresh and Rolling Replacement](#7-instance-refresh-and-rolling-replacement)
8. [Common Operational Scenarios](#8-common-operational-scenarios)
9. [AWS CLI Examples](#9-aws-cli-examples)
10. [Interview Checkpoints](#10-interview-checkpoints)

---

# 1. Launch Templates

## What is a Launch Template?

An **EC2 Launch Template** is a reusable definition of how EC2 instances should be launched.

It stores launch parameters so that the same configuration can be reused consistently by an ASG, EC2 console, CLI, or other AWS services.

## Common Launch Template settings

| Setting | Purpose |
|---|---|
| AMI ID | Defines the operating system and initial software |
| Instance type | Defines the compute size, such as `t3.small` |
| Key pair | Optional SSH login configuration |
| Network settings | Defines subnet, network interfaces, and related launch settings |
| Security groups | Defines the security groups attached during launch |
| IAM instance profile | Defines the instance role, if required |
| User data | Runs bootstrap commands during first boot |
| Block device mappings | Defines attached storage devices |
| Monitoring | Enables detailed EC2 monitoring, where configured |
| Tags | Applies tags to launched resources |
| Market options | Supports On-Demand or Spot launch settings |

> A Launch Template can contain settings related to other AWS services. This document mentions them only as launch-template fields and does not explain those services separately.

## Why use a Launch Template?

- Avoids repeating the same launch configuration.
- Reduces configuration drift.
- Provides versioning.
- Allows controlled updates.
- Supports Auto Scaling Groups.
- Makes instance replacement more consistent.
- Helps standardize DevOps deployments.

## Launch Template versions

A Launch Template can have multiple versions.

Example:

```text
Launch Template: dev-otms-notification-lt

Version 1 → Ubuntu AMI + t3.small + old application configuration
Version 2 → New AMI + t3.small + updated application configuration
Version 3 → New AMI + t3.medium + updated configuration
```

One version can be marked as the **default version**.

An ASG can use:

- A specific Launch Template version.
- The default version.
- The latest version, when explicitly configured using the appropriate version reference.

### Important point

Creating a new Launch Template version does **not automatically replace existing ASG instances**. The ASG must be updated and, if required, an instance refresh or another replacement process must be initiated.

## Launch Template vs Launch Configuration

| Launch Template | Launch Configuration |
|---|---|
| Current recommended option | Older legacy option |
| Supports versioning | No versioning |
| Supports newer EC2 features | More limited |
| Can be used with ASGs | Can be used with ASGs |
| Preferred for new designs | Avoid for new designs |

---

# 2. Auto Scaling Groups

## What is an Auto Scaling Group?

An **Auto Scaling Group (ASG)** is a logical group of EC2 instances that maintains the desired number of instances and can automatically increase or decrease capacity according to scaling policies.

An ASG helps provide:

- Automatic instance replacement.
- Capacity management.
- Horizontal scaling.
- Better workload resilience.
- Controlled rolling replacement.
- Distribution of instances across configured Availability Zones.

## Main ASG settings

| Setting | Meaning |
|---|---|
| Minimum capacity | Lowest number of instances the ASG should maintain |
| Desired capacity | Target number of running instances |
| Maximum capacity | Highest number of instances the ASG may launch |
| Launch Template | Defines how new instances are launched |
| Health check type | Determines how instance health is evaluated |
| Health check grace period | Time allowed for a new instance to initialize |
| Scaling policies | Rules for increasing or decreasing capacity |
| Instance protection | Prevents selected instances from being terminated by scale-in |
| Availability Zones | Locations in which ASG instances may be launched |
| Termination policy | Helps select which instance to terminate during scale-in |

## ASG capacity example

```text
Minimum capacity = 1
Desired capacity = 2
Maximum capacity = 4
```

Meaning:

- The ASG tries to maintain 2 instances.
- It should not normally go below 1 instance.
- It can scale out up to 4 instances.
- If an instance becomes unhealthy, the ASG can replace it.

## ASG lifecycle

```text
ASG created
    ↓
Launches instances using Launch Template
    ↓
Instances enter service
    ↓
ASG checks desired capacity and health
    ↓
Scale out / scale in / replace unhealthy instances
    ↓
ASG maintains the required capacity
```

---

# 3. Launch Template and ASG Relationship

## How do they work together?

The Launch Template defines **how an instance should be created**.

The ASG defines **how many instances should exist and when capacity should change**.

```text
Launch Template
    │
    │ Defines instance configuration
    ▼
Auto Scaling Group
    │
    ├── Maintains desired capacity
    ├── Launches new instances
    ├── Replaces unhealthy instances
    ├── Scales out
    └── Scales in
```

## Example architecture

```text
Launch Template
    ├── AMI
    ├── Instance type
    ├── User data
    └── Instance configuration
             │
             ▼
Auto Scaling Group
    ├── Min = 1
    ├── Desired = 2
    ├── Max = 4
    └── Scaling policies
             │
             ▼
      EC2 Instances
```

## What happens when the Launch Template is updated?

Example:

```text
Current:
Launch Template version 1 → AMI v1

Updated:
Launch Template version 2 → AMI v2
```

The existing instances normally continue running with AMI v1.

New instances launched after the ASG uses version 2 will use AMI v2.

To replace existing instances with AMI v2, use an **instance refresh** or another controlled replacement strategy.

## Recommended update sequence

```text
Create new Launch Template version
            ↓
Test the new version
            ↓
Update ASG to use the new version
            ↓
Start instance refresh
            ↓
Replace instances gradually
            ↓
Validate application health
```

---

# 4. ASG Capacity Settings

## Minimum, Desired, and Maximum

| Setting | Example | Explanation |
|---|---:|---|
| Minimum | 1 | ASG should maintain at least one instance |
| Desired | 2 | ASG attempts to maintain two instances |
| Maximum | 4 | ASG cannot scale beyond four instances |

## Scale-out

Scale-out means increasing the number of instances.

```text
Desired capacity: 2
        ↓
High workload
        ↓
ASG launches instances
        ↓
Desired capacity: 3 or 4
```

## Scale-in

Scale-in means decreasing the number of instances.

```text
Desired capacity: 4
        ↓
Low workload
        ↓
ASG terminates selected instances
        ↓
Desired capacity: 2
```

## Important rules

- Desired capacity must be between minimum and maximum capacity.
- The ASG cannot normally scale below minimum capacity.
- The ASG cannot normally scale above maximum capacity.
- Scaling policies can change desired capacity.
- Manual desired-capacity changes can also affect the number of instances.
- Updating the desired capacity is not the same as changing the minimum or maximum limits.

---

# 5. Scaling Policies

## What is a scaling policy?

A scaling policy defines when and how an ASG should change its desired capacity.

## Common scaling policy types

| Policy | Explanation | Example |
|---|---|---|
| Manual scaling | Operator changes desired capacity | Increase desired capacity from 2 to 4 before a planned event |
| Simple scaling | Adds or removes a fixed amount after an alarm | CPU alarm triggers; add 2 instances |
| Step scaling | Uses different adjustments for different alarm levels | CPU 70% → add 1; CPU 90% → add 3 |
| Target tracking | Tries to maintain a target metric value | Maintain average CPU near 50% |
| Scheduled scaling | Changes capacity at a known time | 09:00 → 5 instances; 18:00 → 2 instances |
| Predictive scaling | Uses forecasting to plan capacity | Forecasted morning traffic causes capacity to increase in advance |

### Quick examples

```text
Manual:
Operator changes Desired capacity: 2 → 4

Simple:
CPU alarm → Add 2 instances

Step:
CPU 70% → Add 1 instance
CPU 90% → Add 3 instances

Target tracking:
Target CPU = 50% → ASG adds/removes instances to stay near 50%

Scheduled:
09:00 → Desired = 5
18:00 → Desired = 2

Predictive:
Forecasted traffic spike tomorrow → Increase capacity before the spike
```

## Scaling cooldown and warm-up

Scaling decisions should account for instance startup time.

If new instances need time to initialize, scaling too quickly can cause unnecessary launches.

Important concepts include:

- Instance warm-up.
- Cooldown behavior.
- Health check grace period.
- Application initialization time.

---

# 6. Health Checks and Instance Replacement

## Why does an ASG use health checks?

An ASG uses health checks to determine whether an instance should remain in service.

If an instance is unhealthy, the ASG can terminate it and launch a replacement.

## EC2 Health Check vs ELB Health Check

### ASG Health Check Settings

| Parameter | Meaning | Example |
|---|---|---|
| **Health check type** | Health information used by the ASG | `EC2` or `ELB` |
| **Health check grace period** | Time given to a new instance to initialize | `300 seconds` |

> With `EC2` health checks, the ASG uses EC2 status checks. With `ELB` health checks, it also considers the load balancer target health.

## ALB Health Check Parameters

An **Application Load Balancer (ALB)** checks whether a target can receive traffic.

| Parameter | Meaning | Example |
|---|---|---|
| Protocol | Protocol used for the health check | `HTTP` or `HTTPS` |
| Port | Port on which the check runs | `80` or `8080` |
| Path | Application endpoint to check | `/health` |
| Healthy threshold | Consecutive successful checks required | `5` |
| Unhealthy threshold | Consecutive failed checks required | `2` |
| Timeout | Time allowed for a response | `5 seconds` |
| Interval | Time between health checks | `30 seconds` |
| Success codes / Matcher | HTTP codes accepted as successful | `200` or `200-399` |


### Health Check Flow

```text
ALB sends GET /health
        ↓
Target returns HTTP 200
        ↓
After 5 successful checks → Healthy
        ↓
After 2 failed checks → Unhealthy
        ↓
ALB stops routing traffic to that target
```

> The health-check path must exist and the application must return an accepted success code.

## Health check grace period

The grace period gives a newly launched instance time to initialize before health evaluation can cause replacement.

Example:

```text
Instance launched
    ↓
Application starts
    ↓
Bootstrap completes
    ↓
Grace period ends
    ↓
Normal health evaluation
```

A grace period that is too short may cause healthy-but-slow instances to be replaced.

A grace period that is too long may delay replacement of genuinely unhealthy instances.

## Unhealthy instance replacement

```text
ASG desired capacity = 2

Instance A → Healthy
Instance B → Unhealthy
        ↓
ASG terminates Instance B
        ↓
ASG launches replacement Instance C
        ↓
ASG returns to desired capacity = 2
```

---

# 7. Instance Refresh and Rolling Replacement

## What is instance refresh?

**Instance refresh** is an ASG feature used to replace existing instances gradually, usually after changing the Launch Template version.

Common reasons:

- New AMI.
- Updated user data.
- New application image.
- New instance type.
- Updated launch configuration.
- Security or OS patching through a new image.

## Instance refresh flow

```text
Create new Launch Template version
            ↓
Update ASG
            ↓
Start instance refresh
            ↓
Launch replacement instance
            ↓
Validate replacement
            ↓
Terminate old instance
            ↓
Repeat until refresh completes
```

## Minimum healthy percentage

The minimum healthy percentage controls how much of the group should remain healthy during replacement.

Example:

```text
ASG desired capacity = 4
Minimum healthy percentage = 75%
```

The ASG should try to keep at least 3 instances healthy during the refresh.

## Important operational checks

Before starting an instance refresh:

- Confirm the new Launch Template version works.
- Verify bootstrap commands.
- Check application startup time.
- Ensure minimum healthy capacity is appropriate.
- Confirm the ASG maximum capacity allows the replacement strategy.
- Monitor refresh progress.
- Stop or cancel the refresh if the new version is unhealthy.

---

# 8. Common Operational Scenarios

## Scenario 1: ASG is launching the wrong AMI

Possible causes:

- ASG is using an older Launch Template version.
- The ASG is configured to use the default version.
- The new Launch Template version was created but not assigned to the ASG.
- Existing instances were not replaced after the update.

Check:

```bash
aws autoscaling describe-auto-scaling-groups \
  --auto-scaling-group-names dev-otms-notification-asg \
  --query "AutoScalingGroups[0].LaunchTemplate"
```

## Scenario 2: Launch Template was updated but running instances did not change

This is expected behavior.

A Launch Template version affects future launches. Existing instances are not automatically recreated merely because a new version exists.

Solution:

1. Update the ASG to the required version.
2. Start an instance refresh.
3. Monitor replacement health.

## Scenario 3: ASG keeps replacing instances

Possible causes:

- Incorrect AMI.
- User data failure.
- Application startup failure.
- Health check grace period too short.
- Load-balancer health check failure.
- Instance launch configuration problem.
- Insufficient capacity for the selected instance type.

## Scenario 4: ASG does not scale out

Check:

- Maximum capacity has not already been reached.
- Scaling policy is attached to the correct ASG.
- The scaling metric or alarm is functioning.
- The ASG can launch the configured instance.
- The Launch Template version is valid.
- The scaling policy adjustment is appropriate.

## Scenario 5: ASG does not scale in

Check:

- Desired capacity is already equal to minimum capacity.
- Scale-in protection is enabled.
- The scaling policy is not requesting scale-in.
- The ASG is in the middle of an instance refresh.
- Termination policies and lifecycle hooks are delaying termination.

---

# 9. AWS CLI Examples

## Create a Launch Template

```bash
aws ec2 create-launch-template \
  --launch-template-name dev-otms-notification-lt \
  --version-description "Initial version" \
  --launch-template-data '{
    "ImageId": "ami-xxxxxxxxxxxxxxxxx",
    "InstanceType": "t3.small",
    "UserData": "BASE64_ENCODED_USER_DATA"
  }'
```

## Create a new Launch Template version

```bash
aws ec2 create-launch-template-version \
  --launch-template-name dev-otms-notification-lt \
  --source-version 1 \
  --version-description "Updated AMI" \
  --launch-template-data '{
    "ImageId": "ami-yyyyyyyyyyyyyyyyy"
  }'
```

## View Launch Template versions

```bash
aws ec2 describe-launch-template-versions \
  --launch-template-name dev-otms-notification-lt
```

## Create an ASG

```bash
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name dev-otms-notification-asg \
  --launch-template \
    LaunchTemplateName=dev-otms-notification-lt,Version='$Latest' \
  --min-size 1 \
  --desired-capacity 1 \
  --max-size 1 \
  --vpc-zone-identifier "subnet-xxxxxxxx,subnet-yyyyyyyy"
```

## Update ASG to use a specific Launch Template version

```bash
aws autoscaling update-auto-scaling-group \
  --auto-scaling-group-name dev-otms-notification-asg \
  --launch-template \
    LaunchTemplateName=dev-otms-notification-lt,Version=2
```

## Start an instance refresh

```bash
aws autoscaling start-instance-refresh \
  --auto-scaling-group-name dev-otms-notification-asg \
  --preferences \
    MinHealthyPercentage=75,InstanceWarmup=300
```

## Check instance refresh status

```bash
aws autoscaling describe-instance-refreshes \
  --auto-scaling-group-name dev-otms-notification-asg
```

## Update desired capacity

```bash
aws autoscaling set-desired-capacity \
  --auto-scaling-group-name dev-otms-notification-asg \
  --desired-capacity 2
```

## Describe an ASG

```bash
aws autoscaling describe-auto-scaling-groups \
  --auto-scaling-group-names dev-otms-notification-asg
```

---

# 10. Interview Checkpoints

| Question | Interview-ready answer |
|---|---|
| What is a Launch Template? | A reusable, versioned definition of how EC2 instances should be launched. |
| What is an ASG? | A logical group that maintains EC2 capacity and can scale or replace instances automatically. |
| What is the difference between a Launch Template and an ASG? | The Launch Template defines instance configuration; the ASG manages instance count, health, and scaling. |
| Does creating a new Launch Template version replace running instances? | No. Existing instances continue running until an instance refresh or another replacement process is performed. |
| What are minimum, desired, and maximum capacity? | Minimum is the lower limit, desired is the target capacity, and maximum is the upper limit. |
| What is scale-out? | Increasing the number of EC2 instances in the ASG. |
| What is scale-in? | Decreasing the number of EC2 instances in the ASG. |
| Why is instance refresh used? | To replace existing instances gradually after a Launch Template or AMI update. |
| What happens when an ASG instance becomes unhealthy? | The ASG can terminate the unhealthy instance and launch a replacement to maintain desired capacity. |
| What is target tracking? | A scaling policy that attempts to maintain a selected metric near a target value. |
| What is the difference between Launch Template and Launch Configuration? | Launch Templates support versioning and newer features and are preferred for new deployments. |
| Why might an ASG repeatedly replace instances? | The instance may fail startup, bootstrap, EC2, or configured application health checks. |

---

## Final Revision

```text
Launch Template
    = How to launch an EC2 instance

Auto Scaling Group
    = How many instances should run

Scaling Policy
    = When capacity should change

Health Check
    = Whether an instance is healthy

Instance Refresh
    = How to replace existing instances safely
```
