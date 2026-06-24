# <h1 align="center">Documentation - Ansible Role | PostgreSQL Deployment</h1>

<div align="center">
<img width="100" alt="PostgreSQL" src="https://www.postgresql.org/media/img/about/press/elephant.png" />
</div>

<br/>

---

<div align="center">

<table>
  <tr>
    <th align="center">Author</th>
    <th align="center">Created On</th>
    <th align="center">Version</th>
    <th align="center">Last Updated By</th>
    <th align="center">Last Edited On</th>
    <th align="center">Pre Reviewer</th>
    <th align="center">L0 Reviewer</th>
    <th align="center">L1 Reviewer</th>
    <th align="center">L2 Reviewer</th>
  </tr>

  <tr>
    <td align="center">Mukesh Kharb</td>
    <td align="center">24/06/2026</td>
    <td align="center">1.0</td>
    <td align="center">Mukesh Kharb</td>
    <td align="center">24/06/2026</td>
    <td align="center">Team</td>
    <td align="center">Mohit Kumar</td>
    <td align="center">Faisal Khan</td>
    <td align="center">Mahesh Kumar</td>
  </tr>

</table>

</div>

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [What is PostgreSQL Ansible Role](#2-what-is-postgresql-ansible-role)
3. [Why PostgreSQL Automation is Required](#3-why-postgresql-automation-is-required)
4. [OT-Microservices Architecture Integration](#4-ot-microservices-architecture-integration)
5. [Proposed Role Directory Structure](#5-proposed-role-directory-structure)
6. [Role File Description](#6-role-file-description)
7. [Variable Management Strategy](#7-variable-management-strategy)
8. [Workflow Diagram](#8-workflow-diagram)
9. [Advantages](#9-advantages)
10. [Best Practices](#10-best-practices)
11. [Recommendation / Conclusion](#11-recommendation--conclusion)
12. [FAQs](#12-faqs)
13. [Contact Information](#13-contact-information)
14. [References](#14-references)

---

<a id="1-introduction"></a>

## 1. Introduction

PostgreSQL is the relational database used within the OT-Microservices architecture for storing attendance-related information. The Attendance API uses PostgreSQL as its primary persistence layer while Liquibase manages database schema creation and versioning.

To ensure consistent deployments across environments, PostgreSQL infrastructure can be provisioned and managed through an Ansible role. This role standardizes installation, configuration, database creation, user management, privilege assignment, and validation activities.

> [!NOTE]
> The PostgreSQL role is responsible only for database infrastructure provisioning. Application schema creation remains the responsibility of Liquibase migrations executed by the Attendance API.

---

<a id="2-what-is-postgresql-ansible-role"></a>

## 2. What is PostgreSQL Ansible Role

A PostgreSQL Ansible Role is a reusable automation component that manages PostgreSQL deployment and configuration through Infrastructure as Code (IaC).

The role automates:

* PostgreSQL installation
* Service enablement
* Service startup
* Database creation
* User creation
* Privilege assignment
* Configuration management
* Validation checks

This approach reduces manual intervention and ensures consistent deployments across environments.

---

<a id="3-why-postgresql-automation-is-required"></a>

## 3. Why PostgreSQL Automation is Required

| Requirement         | Description                               |
| ------------------- | ----------------------------------------- |
| Standardization     | Ensures consistent PostgreSQL deployments |
| Automation          | Eliminates manual setup effort            |
| Reliability         | Reduces configuration drift               |
| Scalability         | Enables reusable deployments              |
| Faster Provisioning | Accelerates infrastructure setup          |
| Maintainability     | Simplifies operational management         |

---

<a id="4-ot-microservices-architecture-integration"></a>

## 4. OT-Microservices Architecture Integration

### Database Consumer Mapping

| Microservice   | Database   |
| -------------- | ---------- |
| Attendance API | PostgreSQL |
| Employee API   | ScyllaDB   |
| Salary API     | ScyllaDB   |

### Architecture Flow

```text
User Request
      │
      ▼
Attendance API
      │
      ▼
PostgreSQL
      │
      ▼
attendance_db
```

### Integration Scope

The PostgreSQL role provisions:

* PostgreSQL Server
* attendance_db database
* Database users
* Required privileges

The role does not provision:

* Application tables
* Liquibase migrations
* Attendance API deployment

---

<a id="5-proposed-role-directory-structure"></a>

## 5. Proposed Role Directory Structure

```text
roles/
└── postgresql/
    ├── defaults/
    │   └── main.yml
    ├── files/
    ├── handlers/
    │   └── main.yml
    ├── meta/
    │   └── main.yml
    ├── tasks/
    │   ├── install.yml
    │   ├── configure.yml
    │   ├── database.yml
    │   ├── validate.yml
    │   └── main.yml
    ├── templates/
    │   └── postgresql.conf.j2
    ├── vars/
    │   └── main.yml
    └── README.md
```

---

<a id="6-role-file-description"></a>

## 6. Role File Description

| File/Directory               | Purpose                           |
| ---------------------------- | --------------------------------- |
| defaults/main.yml            | Contains default variables        |
| vars/main.yml                | Stores role-specific variables    |
| tasks/main.yml               | Entry point for role execution    |
| tasks/install.yml            | PostgreSQL package installation   |
| tasks/configure.yml          | PostgreSQL configuration tasks    |
| tasks/database.yml           | Database and user creation tasks  |
| tasks/validate.yml           | Validation and verification tasks |
| handlers/main.yml            | Service restart handlers          |
| templates/postgresql.conf.j2 | PostgreSQL configuration template |
| meta/main.yml                | Role metadata and dependencies    |
| README.md                    | Role documentation                |

---

<a id="7-variable-management-strategy"></a>

## 7. Variable Management Strategy

| Variable                  | Description             | Example         |
| ------------------------- | ----------------------- | --------------- |
| postgresql_version        | PostgreSQL version      | 16              |
| postgresql_service_name   | PostgreSQL service name | postgresql      |
| postgresql_port           | Database listening port | 5432            |
| postgresql_database_name  | Database name           | attendance_db   |
| postgresql_username       | Database user           | postgres        |
| postgresql_password       | Database password       | Vault Protected |
| postgresql_listen_address | Allowed interface       | localhost       |

### Sample Variables

```yaml
postgresql_version: "16"

postgresql_service_name: "postgresql"

postgresql_database_name: "attendance_db"

postgresql_username: "postgres"

postgresql_port: 5432
```

---

<a id="8-workflow-diagram"></a>

## 8. Workflow Diagram

```text
Start
  │
  ▼
Install PostgreSQL Packages
  │
  ▼
Configure PostgreSQL
  │
  ▼
Enable Service
  │
  ▼
Start Service
  │
  ▼
Create Database
  │
  ▼
Create User
  │
  ▼
Assign Privileges
  │
  ▼
Validate Deployment
  │
  ▼
Success
```

---

<a id="9-advantages"></a>

## 9. Advantages

| Advantage    | Description                    |
| ------------ | ------------------------------ |
| Consistency  | Standardized deployments       |
| Automation   | Reduced manual effort          |
| Reusability  | Supports multiple environments |
| Reliability  | Reduced human errors           |
| Faster Setup | Rapid provisioning             |
| Scalability  | Easy environment expansion     |

---

<a id="10-best-practices"></a>

## 10. Best Practices

| Best Practice              | Description                       |
| -------------------------- | --------------------------------- |
| Use Ansible Vault          | Protect sensitive credentials     |
| Parameterize Variables     | Improve flexibility               |
| Use Least Privilege        | Restrict database access          |
| Separate Schema Management | Use Liquibase for schema creation |
| Validate Deployment        | Verify service and connectivity   |
| Maintain Idempotency       | Ensure repeatable executions      |

---

<a id="11-recommendation--conclusion"></a>

## 11. Recommendation / Conclusion

The recommended approach is to manage PostgreSQL infrastructure using a dedicated Ansible role while maintaining database schema management through Liquibase migrations. This separation of responsibilities improves maintainability, scalability, and operational consistency across OT-Microservices environments.

The PostgreSQL role should focus exclusively on infrastructure provisioning, service management, database creation, user management, and validation activities.

---

<a id="12-faqs"></a>

## 12. FAQs

### Q1. Does the PostgreSQL role create application tables?

**Answer:**
No. Application tables are created through Liquibase migrations executed separately by the Attendance API deployment workflow.

### Q2. Why should PostgreSQL be managed through Ansible?

**Answer:**
Ansible ensures repeatable, automated, and standardized PostgreSQL deployments.

### Q3. Which microservice uses PostgreSQL?

**Answer:**
Attendance API uses PostgreSQL as its backend database.

### Q4. Does the role manage Liquibase?

**Answer:**
No. Liquibase remains part of the application deployment process.

### Q5. Can the role be reused across multiple environments?

**Answer:**
Yes. Variable-driven configuration enables deployment across development, staging, and production environments.

### Q6. Why separate infrastructure and schema management?

**Answer:**
Separation improves maintainability, simplifies troubleshooting, and follows Infrastructure as Code best practices.

---

<a id="13-contact-information"></a>

## 13. Contact Information

| Name         | ✉️ Contact                                                                        |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

<a id="14-references"></a>

## 14. References

| S.No | Description                           | Click to View                                      |
| ---- | ------------------------------------- | -------------------------------------------------- |
| 1    | PostgreSQL Official Documentation     | https://www.postgresql.org/docs/                   |
| 2    | Ansible Official Documentation        | https://docs.ansible.com/                          |
| 3    | Liquibase Documentation               | https://docs.liquibase.com/                        |
| 4    | Infrastructure as Code Best Practices | https://www.ansible.com/resources/get-started      |
| 5    | PostgreSQL Administration Guide       | https://www.postgresql.org/docs/current/admin.html |
