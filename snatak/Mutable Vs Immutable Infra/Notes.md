# Mutable Infrastructure vs Immutable Infrastructure

> **Interview Tip:** This is a very common DevOps interview question, especially when discussing **Packer, AMIs, Auto Scaling Groups, Kubernetes, Docker, and CI/CD**.

---

# Mutable Infrastructure vs Immutable Infrastructure

| Feature | Mutable Infrastructure | Immutable Infrastructure |
|---------|-------------------------|--------------------------|
| **Definition** | Existing servers are modified or updated after deployment. | Existing servers are never modified. A new server/image is created for every change. |
| **Approach** | Update the existing server. | Replace the existing server. |
| **Deployment** | In-place deployment. | Replace-and-deploy. |
| **Server Changes** | Allowed. | Not allowed. |
| **Configuration Drift** | Possible. | Eliminated. |
| **Rollback** | Difficult. | Easy (launch previous image). |
| **Consistency** | May vary between servers. | All servers are identical. |
| **Provisioning Time** | Faster updates but slower troubleshooting. | Slightly slower image build but faster deployments. |
| **Downtime** | Possible during updates. | Usually minimal with rolling or blue-green deployments. |
| **Automation** | Optional. | Essential. |
| **Best For** | Development, Testing, Legacy Applications. | Production, Cloud-Native Applications, Auto Scaling. |

---

# Mutable Infrastructure

## What is Mutable Infrastructure?

In Mutable Infrastructure, the existing server is **updated after deployment**.

### Example

```text
EC2 Instance

Ubuntu

↓

Install Java

↓

Install Nginx

↓

Update Java

↓

Install Monitoring Agent

↓

Update Application

↓

Install Security Patch

↓

Same Server Continues Running
```

The server keeps changing over time.

---

## Characteristics

- Existing server is modified.
- SSH access is common.
- Manual fixes are possible.
- Configuration drift can occur.

---

## Advantages

- Easy to make quick fixes.
- Faster for small changes.
- Useful during development.
- Lower initial setup effort.

---

## Disadvantages

- Configuration drift.
- Difficult troubleshooting.
- Hard to reproduce environments.
- Rollbacks are complex.
- Servers become inconsistent over time.

---

# Immutable Infrastructure

## What is Immutable Infrastructure?

In Immutable Infrastructure, **servers are never modified after deployment**.

Whenever a change is needed:

- Build a new image.
- Launch a new server.
- Destroy the old server.

---

### Example

```text
Version 1

AMI v1

↓

EC2 Instance


Need Java Update

↓

Create AMI v2

↓

Launch New EC2

↓

Terminate Old EC2
```

No changes are made to the running server.

---

## Characteristics

- No SSH changes in production.
- Every deployment creates a new image.
- Old servers are discarded.
- Infrastructure remains identical.

---

## Advantages

- No configuration drift.
- Easy rollback.
- Highly reproducible.
- Better security.
- Excellent for Auto Scaling.
- Ideal for CI/CD.

---

## Disadvantages

- Requires automation.
- Image creation takes time.
- Slightly more storage for AMIs/images.

---

# Visual Comparison

## Mutable Infrastructure

```text
EC2 Instance

Version 1

↓

Update Packages

↓

Install New Software

↓

Patch Server

↓

Same Server
```

---

## Immutable Infrastructure

```text
AMI v1

↓

EC2 Instance

↓

Need Change

↓

Create AMI v2

↓

Launch New EC2

↓

Terminate Old EC2
```

---

# Real DevOps Example

### Mutable

```text
EC2

↓

SSH into Server

↓

sudo apt update

↓

Install Java

↓

Restart Application
```

---

### Immutable

```text
Git Push

↓

CI Pipeline

↓

Build Application

↓

Build New AMI (Packer)

↓

Launch New EC2

↓

Terminate Old EC2
```

---

# Common Tools

| Mutable Infrastructure | Immutable Infrastructure |
|-------------------------|--------------------------|
| Ansible | Packer |
| Chef | Docker |
| Puppet | Kubernetes |
| SaltStack | EC2 Image Builder |
| SSH | Auto Scaling Groups |
| Bash Scripts | Terraform + Packer |

---

# Which One is Better for OT-Microservices?

## Current OT-Microservices Setup

From our discussions, your project consists of:

- React Frontend
- Go Employee API
- Python Attendance API
- Spring Boot Salary API
- Notification Service
- Nginx
- ScyllaDB
- PostgreSQL
- Redis
- Elasticsearch

### Current Approach

You currently:

- Install packages on an EC2 instance.
- Clone repositories.
- Build applications on the server.
- Start services manually.

This is a **Mutable Infrastructure** approach because the server is modified after it is created.

---

## Recommended Production Approach

For a production-ready OT-Microservices deployment, an **Immutable Infrastructure** model is better.

Example:

```text
Git Push
     │
     ▼
Jenkins Pipeline
     │
     ▼
Run Tests
     │
     ▼
Build Applications
     │
     ▼
Create Golden AMI using Packer
     │
     ▼
Launch New EC2 from Golden AMI
     │
     ▼
Deploy New Version
     │
     ▼
Terminate Old EC2
```

Benefits:

- Consistent deployments
- Easy rollback
- No configuration drift
- Faster recovery
- Better scalability
- Supports Auto Scaling

---

# Interview Questions

| Question | Answer |
|----------|--------|
| What is Mutable Infrastructure? | Infrastructure where existing servers are modified after deployment by applying updates, patches, or configuration changes. |
| What is Immutable Infrastructure? | Infrastructure where servers are never modified after deployment; any change requires creating a new image and replacing the existing server. |
| Which approach eliminates configuration drift? | Immutable Infrastructure. |
| Which approach supports easier rollback? | Immutable Infrastructure, by redeploying a previous image or AMI. |
| Which approach is commonly used with Packer? | Immutable Infrastructure. |
| Which approach is better for Auto Scaling Groups? | Immutable Infrastructure. |
| Which approach is easier for manual debugging? | Mutable Infrastructure. |

---

# Which is Better?

| Scenario | Recommended Approach |
|----------|----------------------|
| Development | Mutable Infrastructure |
| Testing | Mutable Infrastructure |
| Learning | Mutable Infrastructure |
| Legacy Applications | Mutable Infrastructure |
| Production | Immutable Infrastructure |
| Auto Scaling | Immutable Infrastructure |
| CI/CD Pipelines | Immutable Infrastructure |
| Kubernetes | Immutable Infrastructure |

---

# One-Line Interview Answer

> **Mutable Infrastructure updates existing servers after deployment, whereas Immutable Infrastructure never modifies running servers. Instead, it creates a new image (such as a Golden AMI) and replaces the old server. For modern DevOps and production systems like OT-Microservices, Immutable Infrastructure is the preferred approach because it provides consistency, repeatability, easier rollbacks, and eliminates configuration drift.**
