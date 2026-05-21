<h1 align="center">Documentation: PostgreSQL Setup & Run (POC)</h1>

<p align="center">
  <img src="https://www.postgresql.org/media/img/about/press/elephant.png" width="120"/>
</p>

<p align="center">
  <a href="https://www.postgresql.org/docs/">
    <img src="https://img.shields.io/badge/Database-PostgreSQL-blue?style=for-the-badge" />
  </a>
  <a href="https://docs.liquibase.com/">
    <img src="https://img.shields.io/badge/Tool-Liquibase-orange?style=for-the-badge" />
  </a>
  <a href="https://github.com/OT-MICROSERVICES/attendance-api">
    <img src="https://img.shields.io/badge/Service-AttendanceAPI-green?style=for-the-badge" />
  </a>
</p>

| Author       | Created on | Version | Last updated by | Last edited on | Pre Reviewer | L0 Reviewer | L1 Reviewer | L2 Reviewer  |
| ------------ | ---------- | ------- | --------------- | -------------- | ------------ | ----------- | ----------- | ------------ |
| Mukesh Kharb | 26/04/2026 | 1.0     | Mukesh Kharb    | 26/04/2026     | Team         | Mohit Kumar | Faisal Khan | Mahesh Kumar |

---

## Table of Contents

* [Introduction](#introduction)
* [What is PostgreSQL](#what-is-postgresql)
* [Why PostgreSQL in this Project](#why-postgresql-in-this-project)
* [Architecture Context](#architecture-context)
* [Prerequisites](#prerequisites)
* [Installation Steps](#installation-steps)
* [Database Setup](#database-setup)
* [Liquibase Configuration](#liquibase-configuration)
* [Run Migrations](#run-migrations)
* [Run Application](#run-application)
* [Verification](#verification)
* [Troubleshooting](#troubleshooting)
* [Summary](#summary)

---

## Introduction

> PostgreSQL is used as the primary relational database for the Attendance microservice.

In microservices architecture, each service owns its database. Here, the attendance-api uses PostgreSQL to store structured attendance records and supports SQL-based querying.

---

## What is PostgreSQL?

> PostgreSQL is an open-source relational database system.

* Supports SQL queries (SELECT, JOIN, GROUP BY)
* ACID compliant
* Highly reliable and production-ready

---

## Why PostgreSQL in this Project

| Reason          | Explanation                                      |
| --------------- | ------------------------------------------------ |
| Structured Data | Attendance has tabular format (ID, status, date) |
| SQL Queries     | Easy filtering using WHERE, GROUP BY             |
| Reliability     | Strong consistency guarantees                    |
| Compatibility   | Works well with Liquibase                        |

---

## Architecture Context
><img width="auto" height="500" alt="ChatGPT Image Apr 26, 2026, 11_20_27 PM" src="https://github.com/user-attachments/assets/98192cd0-1f38-4f11-ab44-37c29eb27361" />

As per project structure, PostgreSQL is used only by attendance-api.

---
## Installation Steps

### Step 1 — Update System

```bash
sudo apt update && sudo apt upgrade -y
```

### Step 2 — Install PostgreSQL

```bash
sudo apt install postgresql postgresql-contrib -y
```

### Step 3 — Start Service

```bash
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

### Step 4 — Verify

```bash
sudo systemctl status postgresql
```

---

## Database Setup

### Access PostgreSQL

```bash
sudo -u postgres psql
```

### Create Database

```sql
CREATE DATABASE attendance_db;
```

### Set Password

```sql
ALTER USER postgres PASSWORD 'password';
```

### Exit

```bash
\q
```

---

## Liquibase Configuration

PostgreSQL schema is managed using Liquibase.

Example configuration:

```properties
url=jdbc:postgresql://127.0.0.1:5432/attendance_db
username=postgres
password=password
driver=org.postgresql.Driver
changeLogFile=migration/db.changelog-master.xml
```

---

## Run Migrations

### Command

```bash
make run-migrations
```

OR

```bash
liquibase update --driver-properties-file=liquibase.properties
```

### What Happens

* Creates tables
* Maintains changelog
* Ensures version control

---

## Run Application

```bash
make setup
```

OR manually:

```bash
poetry install
liquibase update
poetry run python app.py
```

Project uses Poetry for dependency management.

---

## Verification

### Check Tables

```bash
psql -U postgres -h 127.0.0.1 -d attendance_db
```

```sql
\dt
```

Expected tables:

* databasechangelog
* databasechangeloglock
* records

### API Test

```bash
curl http://localhost:8000/api/v1/attendance/health
```

---

## Troubleshooting

### PostgreSQL not starting

```bash
sudo systemctl restart postgresql
```

### Connection refused

* Check port 5432
* Verify service is running

### Authentication error

```bash
psql -U postgres -h 127.0.0.1
```

(Use TCP instead of socket)

### Migration failure

* Verify DB name
* Check liquibase.properties

---

## Summary

* PostgreSQL is the source of truth for attendance data
* Liquibase manages schema changes
* Poetry manages dependencies
* Fully integrated with attendance-api

---

## FAQs

**Q1. Why use PostgreSQL instead of NoSQL here?**

> Attendance data is structured and benefits from SQL queries like filtering, grouping, and date-based operations.

**Q2. What happens if Liquibase migrations are not run?**

> Required tables will not be created, and the application will fail when trying to read/write data.

**Q3. Why use 127.0.0.1 instead of Docker IP (172.x)?**

> For local POC setup, services run on the same host, so localhost ensures stable connectivity.

**Q4. How to verify PostgreSQL is working correctly?**

> Use `psql` to connect and run `\dt` to check tables or test API endpoints using curl.

---

## Contact Information

| Name         | Email                                                                             | Role            |
| ------------ | --------------------------------------------------------------------------------- | --------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) | DevOps Engineer |

---

## References

| Resource                 | Link                                                                       | Description                                           |
| ------------------------ | -------------------------------------------------------------------------- | ----------------------------------------------------- |
| PostgreSQL Official Docs | [https://www.postgresql.org/docs/](https://www.postgresql.org/docs/)       | Official PostgreSQL documentation                     |
| Liquibase Documentation  | [https://docs.liquibase.com/](https://docs.liquibase.com/)                 | Database migration tool documentation                 |
| OT Microservices Repo    | [https://github.com/OT-MICROSERVICES](https://github.com/OT-MICROSERVICES) | Source code for microservices project                 |
| Attendance API Config    | Internal Project Files                                                     | PostgreSQL + Liquibase configuration used in this POC |

---
