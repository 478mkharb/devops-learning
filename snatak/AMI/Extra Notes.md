# AMI in DevOps - Interview Notes

---

# 1. Security Benefits of AMI

## What are the security benefits of using AMIs?

An **Amazon Machine Image (AMI)** improves security by ensuring every EC2 instance is launched from a **pre-approved, secure, and standardized image** instead of being configured manually.

### Security Benefits

| Benefit | Explanation |
|----------|-------------|
| **Pre-patched OS** | Latest operating system patches are already installed before deployment. |
| **Security Hardening** | CIS benchmarks, firewall rules, SSH configuration, and security policies are pre-configured. |
| **Consistent Security Configuration** | Every EC2 instance has identical security settings. |
| **Reduced Human Errors** | Eliminates manual server configuration mistakes. |
| **No Configuration Drift** | Running servers remain identical throughout their lifecycle. |
| **Pre-installed Security Agents** | CloudWatch Agent, CrowdStrike, Trend Micro, Defender, etc., are already installed. |
| **Compliance** | Helps meet PCI DSS, HIPAA, ISO 27001, CIS, and other compliance requirements. |
| **Faster Incident Recovery** | Compromised instances can be replaced quickly with a trusted AMI. |

---

### Interview Answer

> AMIs improve security by providing pre-patched, security-hardened, and standardized server images. They eliminate manual configuration, reduce configuration drift, ensure compliance, and enable rapid recovery by replacing compromised instances with trusted images.

---

# 2. Standardization Using AMI

## What is Standardization?

Standardization means **every server is created with exactly the same operating system, software, configurations, security settings, and monitoring agents**.

Instead of manually configuring every EC2 instance,

all instances are launched from the same AMI.

---

### Without Standardization

```text
EC2-1
Java 17
Docker 26

EC2-2
Java 21
Docker 25

EC2-3
Java 11
No Monitoring Agent
```

Problems

- Different software versions
- Configuration drift
- Difficult troubleshooting

---

### With Standardized AMI

```text
Golden AMI

Ubuntu 22.04

Java 21

Docker 28

CloudWatch Agent

Security Agent

Monitoring Configuration
```

Every EC2 launched from this AMI is identical.

---

### Benefits

| Benefit | Explanation |
|----------|-------------|
| Consistent environments | All servers have the same configuration. |
| Faster deployments | No manual installation after launch. |
| Easier troubleshooting | Every server behaves the same way. |
| Better compliance | Uniform security policies across servers. |
| Reduced operational errors | Eliminates manual setup inconsistencies. |

---

### Interview Answer

> Standardization using AMIs ensures every EC2 instance is built from the same approved image, providing consistent operating system versions, software, security settings, and monitoring tools across the infrastructure.

---

# 3. Golden AMI

## What is a Golden AMI?

A **Golden AMI** is a **pre-built, security-hardened, fully tested, and organization-approved machine image** used as the standard image for launching production EC2 instances.

It is usually built from a **Generic (Base) AMI** using tools such as:

- AWS EC2 Image Builder
- Packer
- Jenkins
- Ansible

---

### Typical Golden AMI Contents

```text
Ubuntu 22.04

↓

Latest Security Patches

↓

Java

↓

Docker

↓

CloudWatch Agent

↓

Security Agent

↓

Application Dependencies

↓

Organization Standards
```

---

### Benefits

| Benefit | Explanation |
|----------|-------------|
| Faster deployments | Software is already installed. |
| Better security | Latest patches and hardening are included. |
| Consistency | Every server is identical. |
| Easier rollback | Previous AMI versions can be relaunched. |
| Supports Immutable Infrastructure | Replace servers instead of modifying them. |

---

### Interview Answer

> A Golden AMI is a pre-configured, security-hardened, and organization-approved image containing the operating system, required software, security patches, monitoring agents, and standard configurations, ensuring consistent and secure deployments.

---

# 4. Impact on Patching

## Traditional (Mutable Infrastructure)

Without AMIs

```text
Launch EC2

↓

SSH into Server

↓

Install Updates

↓

Install Java

↓

Restart Services

↓

Repeat on Every Server
```

Problems

- Time-consuming
- Human errors
- Downtime
- Configuration drift

---

## AMI-Based (Immutable Infrastructure)

```text
Update Base AMI

↓

Install Latest Security Patches

↓

Run Security Tests

↓

Create New Golden AMI

↓

Launch New EC2 Instances

↓

Terminate Old EC2 Instances
```

---

### Benefits

| Traditional Patching | AMI-Based Patching |
|----------------------|--------------------|
| Patch each server individually | Patch the AMI once |
| Configuration drift | No configuration drift |
| Slow rollout | Faster rollout |
| Manual updates | Automated CI/CD |
| Difficult rollback | Roll back by launching the previous AMI |
| Higher risk | Lower risk |

---

### Example

Old Production

```text
AMI v1

↓

100 EC2 Instances
```

Critical security patch released

Instead of patching all 100 servers:

```text
Create AMI v2

↓

Launch 100 New EC2 Instances

↓

Terminate Old Instances
```

This follows the **Immutable Infrastructure** model.

---

### Interview Answer

> AMI-based patching simplifies maintenance by applying updates to a new AMI instead of patching running servers. New instances are launched from the updated AMI, and old instances are replaced. This reduces downtime, eliminates configuration drift, and makes rollbacks easier.

---

# Frequently Asked Interview Questions

| Question | Answer |
|----------|--------|
| What are the security benefits of AMIs? | Pre-patched OS, security hardening, consistent configurations, compliance, reduced human errors, and easier recovery. |
| What is standardization using AMIs? | Creating all EC2 instances from the same approved image to ensure identical software, configurations, and security settings. |
| What is a Golden AMI? | A pre-configured, tested, and security-hardened AMI used as the standard image for production deployments. |
| How do AMIs simplify patch management? | Patch the AMI once, create a new version, launch new instances, and replace the old ones instead of patching servers individually. |
| Why are Golden AMIs preferred in production? | They provide consistency, security, faster deployments, easier rollbacks, and support immutable infrastructure. |

---

# One-Line Interview Answers

| Topic | Answer |
|-------|--------|
| **Security Benefits of AMI** | AMIs provide secure, patched, and standardized images, reducing manual configuration and improving compliance. |
| **Standardization Using AMI** | Every EC2 instance is launched with identical software, configurations, and security settings from a common image. |
| **Golden AMI** | A fully tested, hardened, and organization-approved image used for consistent production deployments. |
| **Impact on Patching** | Instead of patching running servers, organizations build a new patched AMI, deploy new instances, and replace the old ones, enabling immutable infrastructure. |
