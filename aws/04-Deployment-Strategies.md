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
