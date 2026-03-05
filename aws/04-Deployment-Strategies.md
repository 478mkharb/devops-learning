# Deployment Strategies in AWS

Deployment strategies define **how a new version of an application is released to production**. AWS supports multiple deployment patterns using services such as **Auto Scaling, Load Balancers, CodeDeploy, ECS, and Kubernetes (EKS)**.

This document explains the **major deployment strategies used in AWS production environments**.

---

# 1. All-at-Once Deployment

## Definition

All instances are updated **simultaneously** with the new version.

## Flow

```
Old Version (v1)
   │ │ │ │
   ▼ ▼ ▼ ▼
Replace all instances
   │ │ │ │
   ▼ ▼ ▼ ▼
New Version (v2)
```

## Characteristics

| Feature  | Description        |
| -------- | ------------------ |
| Speed    | Fastest deployment |
| Downtime | Possible           |
| Risk     | High               |
| Rollback | Difficult          |

## Example

All EC2 instances in an Auto Scaling Group are updated at once.

---

# 2. Rolling Deployment

## Definition

Instances are updated **in batches** instead of all at once.

## Flow

```
Step 1
[v2] v1 v1 v1

Step 2
[v2 v2] v1 v1

Step 3
[v2 v2 v2] v1

Step 4
[v2 v2 v2 v2]
```

## Characteristics

| Feature  | Description |
| -------- | ----------- |
| Downtime | None        |
| Risk     | Medium      |
| Speed    | Moderate    |

## Use Case

Used with:

* Auto Scaling Groups
* Kubernetes (EKS)
* ECS

---

# 3. Blue-Green Deployment

## Definition

Two environments exist:

* **Blue** = current production
* **Green** = new version

Traffic switches from blue to green once the new environment is ready.

## Flow

```
        Users
          │
          ▼
     Load Balancer
        │     │
        ▼     ▼
      Blue   Green
     (v1)    (v2)

Traffic switch → Green
```

## Characteristics

| Feature  | Description                    |
| -------- | ------------------------------ |
| Downtime | None                           |
| Rollback | Instant                        |
| Cost     | Higher (duplicate environment) |

## Common AWS Services

* ALB
* Route53
* CodeDeploy

---

# 4. Canary Deployment

## Definition

A **small percentage of traffic** is routed to the new version first.

If successful, traffic gradually increases.

## Flow

```
Users
 │
 ├── 90% → Old Version
 │
 └── 10% → New Version
```

Later:

```
50% → Old
50% → New
```

Finally:

```
100% → New Version
```

## Characteristics

| Feature          | Description |
| ---------------- | ----------- |
| Risk             | Very low    |
| Monitoring       | Required    |
| Deployment speed | Slow        |

## Tools

* CodeDeploy
* Route53 weighted routing
* Service Mesh

---

# 5. Immutable Deployment

## Definition

Instead of updating existing servers, **new instances are created with the new version** and old ones are destroyed.

## Flow

```
Old Instances (v1)
      │
      ▼
Create new instances (v2)
      │
      ▼
Switch traffic
      │
      ▼
Terminate old instances
```

## Characteristics

| Feature          | Description |
| ---------------- | ----------- |
| Reliability      | High        |
| Rollback         | Easy        |
| Deployment speed | Moderate    |

## Common AWS Tools

* Auto Scaling
* Launch Templates
* AMI-based deployments

---

# 6. Linear Deployment

## Definition

Traffic shifts to the new version **at fixed intervals**.

Example:

```
10% every 5 minutes
```

## Flow

```
10% → v2
30% → v2
60% → v2
100% → v2
```

## AWS Service

AWS CodeDeploy supports **linear traffic shifting**.

---

# 7. Shadow Deployment (Traffic Mirroring)

## Definition

Production traffic is **copied to the new version**, but responses are ignored.

Used for testing production behavior safely.

## Flow

```
Users
 │
 ▼
Production (v1)
 │
 └── Mirror traffic → v2
```

## Characteristics

| Feature     | Description             |
| ----------- | ----------------------- |
| User impact | None                    |
| Testing     | Real production traffic |
| Risk        | Very low                |

---

# Deployment Strategy Comparison

| Strategy    | Downtime | Risk     | Cost   | Rollback |
| ----------- | -------- | -------- | ------ | -------- |
| All-at-once | Possible | High     | Low    | Hard     |
| Rolling     | None     | Medium   | Low    | Medium   |
| Blue-Green  | None     | Low      | High   | Instant  |
| Canary      | None     | Very Low | Medium | Easy     |
| Immutable   | None     | Very Low | Medium | Easy     |
| Shadow      | None     | Very Low | Medium | N/A      |

---

# Quick Interview Summary

```
All-at-once → fastest but risky
Rolling → update instances in batches
Blue-Green → two environments switch traffic
Canary → small % traffic testing
Immutable → replace servers instead of updating
Linear → gradual traffic shift
Shadow → mirror production traffic
```

These deployment strategies are commonly implemented using:

* AWS CodeDeploy
* Auto Scaling Groups
* Application Load Balancers
* Route53
* ECS / EKS

---

# Role of ALB and ASG in Deployment Strategies

In AWS production environments, **Application Load Balancer (ALB)** and **Auto Scaling Groups (ASG)** are key components that enable safe deployments with minimal downtime.

They work together to:

* distribute traffic
* launch new instances
* remove unhealthy instances
* shift traffic gradually between versions

---

# 1. Auto Scaling Group (ASG) Role in Deployments

ASG manages the **lifecycle of EC2 instances** during deployments.

Responsibilities:

* Launch new instances with the new version
* Maintain desired capacity
* Replace unhealthy instances
* Terminate old instances after deployment

Example deployment flow:

```
ASG launches new EC2 instances (v2)
        │
        ▼
Instances pass health checks
        │
        ▼
Old instances (v1) are terminated
```

ASG also supports:

* Rolling updates
* Instance refresh
* Lifecycle hooks

These features help automate deployments safely.

---

# 2. Application Load Balancer (ALB) Role in Deployments

ALB controls **how traffic is routed between application versions**.

It helps ensure that users only reach **healthy instances**.

Responsibilities:

* Distribute traffic across instances
* Perform health checks
* Route traffic to specific target groups
* Enable gradual traffic shifting

Example ALB routing:

```
Users
  │
  ▼
Application Load Balancer
  │
  ├── Target Group Blue (v1)
  │
  └── Target Group Green (v2)
```

ALB can also support:

* path based routing
* host based routing
* weighted traffic shifting

---

# 3. Example: Blue-Green Deployment with ALB and ASG

```
Users
   │
   ▼
Application Load Balancer
   │
   ├── Target Group Blue
   │        │
   │        ▼
   │     ASG Blue
   │     EC2 (v1)
   │
   └── Target Group Green
            │
            ▼
         ASG Green
         EC2 (v2)
```

Deployment process:

1. ASG launches new instances (Green environment).
2. Instances pass ALB health checks.
3. Traffic is switched from Blue → Green.
4. Old instances are terminated.

---

# 4. Example: Rolling Deployment with ASG

```
Step 1
ASG replaces 1 instance

[v2] v1 v1 v1

Step 2

[v2 v2] v1 v1

Step 3

[v2 v2 v2] v1

Step 4

[v2 v2 v2 v2]
```

ALB ensures traffic only goes to **healthy instances during each step**.

---

# 5. Example: Canary Deployment Using ALB

```
Users
 │
 ▼
Application Load Balancer
 │
 ├── 90% → Target Group v1
 │
 └── 10% → Target Group v2
```

If monitoring shows no errors:

```
50% → v2
100% → v2
```

---

# Key Takeaway

```
ASG = manages instances
ALB = manages traffic
```

Together they enable:

* rolling deployments
* blue green deployments
* canary releases
* zero downtime deployments
