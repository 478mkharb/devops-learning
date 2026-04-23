<div align="center">

# Database Change Management | Liquibase POC

<img width="300" height="auto" alt="liquibase" src="https://github.com/user-attachments/assets/833a51d1-e27d-46a2-96dd-0cea2f8a2684" />


<br/>

[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Database-336791?logo=postgresql)](https://www.postgresql.org)
[![Liquibase Docs](https://img.shields.io/badge/Docs-Liquibase-navy)](https://docs.liquibase.com)

</div>

---

| Author       | Created on | Version | Last updated by | Last edited on | Pre Reviewer | L0 Reviewer | L1 Reviewer | L2 Reviewer  |
| ------------ | ---------- | ------- | --------------- | -------------- | ------------ | ----------- | ----------- | ------------ |
| Mukesh Kharb | 22/04/2026 | 1.0     | Mukesh Kharb    | 22/04/2026     | Team         | Mohit Kumar | Faisal Khan | Mahesh Kumar |

---

# 1. Objective

Demonstrate that Liquibase can manage database schema changes in a version-controlled, repeatable way.

---

# 2. Problem Statement

| Issue                | Impact                   |
| -------------------- | ------------------------ |
| Manual SQL execution | Errors and inconsistency |
| No version tracking  | Hard to manage changes   |
| No rollback          | Risky deployments        |

---

# 3. Proposed Solution

Use Liquibase to:

* Track database changes using changelog files
* Apply changes automatically
* Maintain execution history
* Support rollback

---

# 4. Database Release Management Flow

><img width="800" height="auto" alt="ChatGPT Image Apr 23, 2026, 07_22_47 AM" src="https://github.com/user-attachments/assets/9796f312-8805-4f45-9042-26bbebdea92c" />


---

# 5. Pre-requisites

| Component | Requirement         |
| --------- | ------------------- |
| OS        | Linux / Ubuntu      |
| Java      | 11+                 |
| Database  | PostgreSQL running  |
| Tool      | Liquibase installed |

---

# 6. Implementation (POC)

## Install Liquibase

```bash
wget https://github.com/liquibase/liquibase/releases/download/v4.27.0/liquibase-4.27.0.tar.gz
sudo tar -xvf liquibase-4.27.0.tar.gz
```

---

## Clone Project

```bash
git clone https://github.com/OT-MICROSERVICES/attendance-api.git
cd attendance-api
```

---

## Migration Directory Structure

```text
migration/
└── db.changelog-master.xml
```

* Main entry file for Liquibase execution

---

## Configure Liquibase

<img width="1012" height="auto" alt="image" src="https://github.com/user-attachments/assets/b9f20eee-f4b4-4dbb-bf91-28306d8920b4" />


---

## Changelog Details

<img width="1563" height="689" alt="image" src="https://github.com/user-attachments/assets/c074f299-6e98-4e9e-a523-285df939f298" />


---

## Run Migration

<img width="1500" height="auto" alt="image" src="https://github.com/user-attachments/assets/9bca6967-4162-4df2-88d1-1ba294434982" />


---

## Verify

```bash
psql -U postgres -d attendance_db
\dt
```
><img width="1268" height="522" alt="image" src="https://github.com/user-attachments/assets/6bbf811e-8dc2-4d5a-a1e7-848ed250ee2e" />

---

# 7. Testing Methodology

| Test          | Expected Result          |
| ------------- | ------------------------ |
| First run     | Table created            |
| Second run    | No duplicate execution   |
| New changeSet | Only new changes applied |

---

# 8. Predicted Outcome

* Consistent schema across environments
* Controlled DB changes
* Reduced manual effort

---

# 9. Summary

* Liquibase enables version-controlled database changes using changelog files
* Ensures consistent schema across environments by applying only new changes
* Provides reliable migration execution with built-in tracking

---

# Contact Information

| Name         | Email                                                                             |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

# References

| Source          | Link                                                                 |
| --------------- | -------------------------------------------------------------------- |
| Liquibase Docs  | [https://docs.liquibase.com](https://docs.liquibase.com)             |
| PostgreSQL Docs | [https://www.postgresql.org/docs/](https://www.postgresql.org/docs/) |

---
