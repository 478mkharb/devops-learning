# Types of AMIs (DevOps Interview)

| AMI Type | Description | Typical Use Case |
|----------|-------------|------------------|
| **Generic AMI (Base AMI)** | Base operating system image provided by AWS or AWS Marketplace. | Starting point for custom images. |
| **Golden AMI** | Organization-approved, hardened AMI with required software, patches, and agents. | Production deployments. |
| **Hardened AMI** | AMI configured according to security standards (CIS, STIG, etc.). | Security-sensitive workloads. |
| **Application AMI** | AMI with a specific application pre-installed (e.g., Nginx, Tomcat, Jenkins). | Faster application deployment. |
| **Baked AMI** | AMI with the application and all dependencies pre-installed. | Immutable infrastructure deployments. |
| **Pipeline AMI** | AMI automatically built and tested by a CI/CD pipeline using tools like Packer. | Automated infrastructure provisioning. |
| **Marketplace AMI** | Pre-built AMIs published by AWS partners or third-party vendors. | Commercial software deployment. |
| **Custom AMI** | Any user-created AMI based on a customized EC2 instance. | Organization-specific requirements. |
| **Recovery/Backup AMI** | Snapshot-based AMI created for disaster recovery. | Backup and disaster recovery. |

---

# Most Important AMIs for DevOps Interviews

| AMI | Interview Importance | Notes |
|------|----------------------|-------|
| Generic AMI | ⭐⭐⭐⭐⭐ | Starting point |
| Golden AMI | ⭐⭐⭐⭐⭐ | Most commonly asked |
| Baked AMI | ⭐⭐⭐⭐ | Immutable Infrastructure |
| Hardened AMI | ⭐⭐⭐⭐ | Security-focused organizations |
| Custom AMI | ⭐⭐⭐⭐ | Built by organizations |
| Marketplace AMI | ⭐⭐⭐ | Vendor-provided software |
| Application AMI | ⭐⭐⭐ | Pre-installed applications |

---

# Relationship Between Them

```text
AWS Base AMI
(Generic AMI)
       │
       ▼
Customize
       │
       ▼
Custom AMI
       │
       ├──────────────┐
       ▼              ▼
Hardened AMI     Application AMI
       │              │
       └──────┬───────┘
              ▼
         Golden AMI
              │
              ▼
      Baked Production AMI
              │
              ▼
      Auto Scaling / EC2
```

---

# Difference Between Custom AMI and Golden AMI

| Custom AMI | Golden AMI |
|------------|------------|
| Any user-created AMI | Organization-approved standard AMI |
| May not be tested | Fully tested and validated |
| May not be patched | Fully patched |
| May not follow standards | Follows security and compliance standards |
| Used by individuals or teams | Used organization-wide |

---

# Difference Between Baked AMI and Golden AMI

| Golden AMI | Baked AMI |
|------------|-----------|
| Contains OS, patches, agents, and common software | Contains everything in the Golden AMI plus the application itself |
| Reusable across many applications | Built specifically for one application/version |
| Base image for deployments | Ready-to-run application image |

Example:

Golden AMI

```text
Ubuntu
+ Java
+ Docker
+ CloudWatch Agent
+ Security Agent
```

Baked AMI

```text
Golden AMI
+ Employee API
+ Configuration
+ Application Dependencies
```

---

# Interview Questions

| Question | Answer |
|----------|--------|
| Which AMI does AWS provide? | Generic (Base) AMI. |
| Which AMI is used in production? | Golden AMI. |
| Which AMI contains the application? | Baked AMI or Application AMI. |
| Which AMI is security hardened? | Hardened AMI. |
| Which AMI is built automatically by CI/CD? | Pipeline AMI. |
| Which AMI is sold by third-party vendors? | Marketplace AMI. |
| Which AMI is created by users? | Custom AMI. |

> **Interview Tip:** In DevOps interviews, focus primarily on **Generic AMI, Custom AMI, Golden AMI, Baked AMI, and Hardened AMI**. These are the terms you're most likely to encounter when discussing Packer, EC2 Image Builder, immutable infrastructure, or AMI pipelines.
