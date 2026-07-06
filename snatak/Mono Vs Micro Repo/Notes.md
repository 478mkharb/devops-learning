# Monorepo vs Multi-Repo (Micro-Repo)

> **Interview Tip:** There are two common repository strategies in modern software development:
>
> - **Monorepo** → One repository contains all services/projects.
> - **Multi-Repo (Micro-Repo)** → Each microservice has its own repository.

---

# Monorepo vs Multi-Repo

| Feature | Monorepo | Multi-Repo (Micro-Repo) |
|---------|----------|-------------------------|
| **Definition** | All applications, microservices, libraries, and infrastructure are stored in a single Git repository. | Each microservice, library, or application has its own separate Git repository. |
| **Repository Count** | One | Multiple |
| **Codebase** | Shared | Independent |
| **Versioning** | Entire project shares one version history. | Each service has its own versioning. |
| **CI/CD Pipeline** | One pipeline with path-based triggers or multiple pipelines. | Separate pipeline for every repository. |
| **Deployment** | Can deploy individual services using path filters. | Each service is deployed independently. |
| **Dependency Management** | Easier to share internal libraries. | Shared libraries usually require package management. |
| **Code Sharing** | Very easy | Requires publishing or Git submodules/packages. |
| **Build Time** | Can be longer without optimization. | Faster because only one service is built. |
| **Repository Size** | Large | Small |
| **Access Control** | Harder to restrict access per service. | Easier to manage permissions per repository. |
| **Merge Conflicts** | More likely in shared files. | Less likely across services. |
| **Scalability** | Excellent with proper tooling. | Excellent for independently managed teams. |
| **Best For** | Small to medium teams, tightly coupled services, shared libraries. | Large organizations with independent microservice teams. |

---

# Monorepo

## Structure

```text
OT-Microservices/

├── frontend/
├── employee-api/
├── attendance-api/
├── salary-api/
├── notification-api/
├── shared-library/
├── infrastructure/
├── Jenkinsfile
└── README.md
```

Everything is stored in **one Git repository**.

---

## Advantages

- Single source of truth
- Easy refactoring across services
- Shared libraries are simple
- Easier dependency management
- One place for documentation
- Easier code search

---

## Disadvantages

- Repository becomes large
- Build time may increase
- More complex CI/CD
- More merge conflicts
- Access control is harder

---

# Multi-Repo (Micro-Repo)

## Structure

```text
employee-api
│
└── Git Repository

attendance-api
│
└── Git Repository

salary-api
│
└── Git Repository

notification-api
│
└── Git Repository

frontend
│
└── Git Repository
```

Each microservice has **its own Git repository**.

---

## Advantages

- Independent development
- Independent deployments
- Smaller repositories
- Better security and access control
- Faster CI/CD pipelines
- Independent versioning

---

## Disadvantages

- Harder to share common code
- More repositories to manage
- Cross-service refactoring is more difficult
- Documentation may be duplicated

---

# Visual Comparison

## Monorepo

```text
Git Repository

│

├── Frontend

├── Employee API

├── Attendance API

├── Salary API

├── Notification API

├── Infrastructure

└── Shared Library
```

---

## Multi-Repo

```text
Git

├── Frontend Repository

├── Employee Repository

├── Attendance Repository

├── Salary Repository

├── Notification Repository

├── Infrastructure Repository

└── Shared Library Repository
```

---

# CI/CD Comparison

| Monorepo | Multi-Repo |
|-----------|------------|
| One repository | Many repositories |
| One Jenkins organization | Multiple Jenkins jobs |
| Path-based builds | Repository-based builds |
| More complex pipeline logic | Simpler pipelines |
| Shared CI configuration | Independent CI per service |

---

# Real Example (OT-Microservices)

Your project contains:

- Frontend (React)
- Employee API (Go)
- Attendance API (Python)
- Salary API (Spring Boot)
- Notification API (Python)
- Infrastructure
- Jenkins Shared Library

### Monorepo Structure

```text
OT-Microservices

├── frontend
├── employee-api
├── attendance-api
├── salary-api
├── notification-api
├── infrastructure
└── shared-library
```

One repository.

---

### Multi-Repo Structure

```text
Frontend Repo

Employee Repo

Attendance Repo

Salary Repo

Notification Repo

Infrastructure Repo

Shared Library Repo
```

Six or more repositories.

---

# Which is Better for OT-Microservices?

## For Learning / Small Teams

✅ **Monorepo**

Reason:

- Easier to manage
- Easier documentation
- Easier CI/CD setup
- Easier code sharing
- Single Pull Request can update multiple services

---

## For Enterprise Production

✅ **Multi-Repo (Micro-Repo)**

Reason:

- Independent deployments
- Independent releases
- Better scalability
- Better security
- Each team owns one service
- Faster pipelines
- Better access control

---

# Recommendation for OT-Microservices

### If OT-Microservices is a training or small-team project

**Monorepo** is a good choice because it simplifies development and collaboration.

### If OT-Microservices evolves into a production-grade enterprise application with dedicated teams for each microservice

**Multi-Repo** is generally the better architecture because each service can be versioned, built, tested, and deployed independently.

---

# Interview Questions

| Question | Answer |
|----------|--------|
| What is a Monorepo? | A single Git repository containing multiple applications, services, libraries, or infrastructure code. |
| What is a Multi-Repo (Micro-Repo)? | A repository strategy where each microservice or application has its own independent Git repository. |
| Which repository strategy is easier for sharing code? | Monorepo. |
| Which repository strategy provides better access control? | Multi-Repo. |
| Which strategy scales better for large organizations? | Multi-Repo. |
| Which strategy simplifies dependency management? | Monorepo. |
| Which strategy allows independent releases? | Multi-Repo. |
| Which strategy usually has smaller repositories? | Multi-Repo. |

---

# One-Line Interview Answer

> **A Monorepo stores all applications and microservices in a single Git repository, making collaboration and code sharing easier. A Multi-Repo (Micro-Repo) stores each microservice in its own repository, enabling independent versioning, CI/CD pipelines, deployments, and team ownership. For enterprise microservices, Multi-Repo is generally preferred due to its scalability and independent service lifecycle.**
