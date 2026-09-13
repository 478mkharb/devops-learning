# PostgreSQL Interview Notes

## Table of Contents

1. [Introduction](#1-introduction)
2. [What is PostgreSQL?](#2-what-is-postgresql)
3. [What Type of Database is PostgreSQL?](#3-what-type-of-database-is-postgresql)
4. [Is PostgreSQL SQL or NoSQL?](#4-is-postgresql-sql-or-nosql)
5. [PostgreSQL Architecture](#5-postgresql-architecture)
6. [Database, Schema, Table, Row, and Column](#6-database-schema-table-row-and-column)
7. [Primary Key, Foreign Key, and Index](#7-primary-key-foreign-key-and-index)
8. [How Data is Stored](#8-how-data-is-stored)
9. [Why is PostgreSQL Powerful?](#9-why-is-postgresql-powerful)
10. [Transactions and ACID Properties](#10-transactions-and-acid-properties)
11. [PostgreSQL Concurrency and MVCC](#11-postgresql-concurrency-and-mvcc)
12. [PostgreSQL Persistence and WAL](#12-postgresql-persistence-and-wal)
13. [What Happens if PostgreSQL Goes Down?](#13-what-happens-if-postgresql-goes-down)
14. [PostgreSQL Use Cases](#14-postgresql-use-cases)
15. [Common PostgreSQL Commands](#15-common-postgresql-commands)
16. [Frequently Asked Interview Questions](#16-frequently-asked-interview-questions)
17. [PostgreSQL in OT-Microservices](#17-postgresql-in-ot-microservices)

---

# 1. Introduction

**PostgreSQL** is an open-source, object-relational database management system.

It is commonly called **Postgres**.

PostgreSQL supports:

* SQL queries
* Relational tables
* Transactions
* ACID properties
* Constraints
* Indexes
* Views
* Stored functions
* JSON and JSONB
* Replication
* Extensions
* Full-text search

PostgreSQL is widely used for:

* Banking applications
* Enterprise applications
* Attendance systems
* E-commerce
* Financial systems
* Web applications
* Analytics
* Microservices
* Data-intensive applications

---

# 2. What is PostgreSQL?

PostgreSQL is a **relational database management system**.

It stores data in tables consisting of rows and columns.

Example:

```text
Database: company_db

Table: employees

id | name   | department
---|--------|-----------
1  | Mukesh | DevOps
2  | Rahul  | Finance
```

Applications communicate with PostgreSQL using SQL.

```sql
SELECT *
FROM employees;
```

PostgreSQL is a server-based database.

```text
Client
   │
   ▼
PostgreSQL Server
   │
   ▼
Database
   │
   ▼
Tables
```

---

# 3. What Type of Database is PostgreSQL?

PostgreSQL is an:

* Open-source database
* Relational database
* Object-relational database
* SQL database
* ACID-compliant database
* Client-server database
* Persistent database

It supports both structured relational data and semi-structured data through features such as JSONB.

---

# 4. Is PostgreSQL SQL or NoSQL?

PostgreSQL is primarily a **SQL relational database**.

It uses:

* Tables
* Rows
* Columns
* Primary keys
* Foreign keys
* Constraints
* Joins
* SQL queries

Example:

```sql
SELECT e.name, d.department_name
FROM employees e
JOIN departments d
ON e.department_id = d.id;
```

PostgreSQL also supports JSON and JSONB, but this does not make PostgreSQL a NoSQL database.

---

# 5. PostgreSQL Architecture

## Client-Server Architecture

```text
Application
    │
    ▼
PostgreSQL Client
    │
    ▼
PostgreSQL Server
    │
    ▼
Database
```

Examples of clients:

* `psql`
* Python applications
* Java applications
* Go applications
* JDBC clients
* PostgreSQL GUI tools

## PostgreSQL Server Processes

PostgreSQL uses a process-based architecture.

```text
PostgreSQL Server
       │
       ├── Postmaster / Main Server
       ├── Client Backend Processes
       ├── Background Writer
       ├── WAL Writer
       ├── Checkpointer
       ├── Autovacuum Workers
       └── WAL Sender / Receiver
```

The exact process structure depends on the PostgreSQL version and configuration.

## Port

The default PostgreSQL port is:

```text
5432
```

---

# 6. Database, Schema, Table, Row, and Column

PostgreSQL organizes objects using the following hierarchy:

```text
PostgreSQL Server
   │
   ▼
Database
   │
   ▼
Schema
   │
   ▼
Table
   │
   ▼
Rows and Columns
```

## Database

A database is a container for schemas and database objects.

```sql
CREATE DATABASE attendance_db;
```

## Schema

A schema is a logical namespace inside a database.

```sql
CREATE SCHEMA attendance;
```

## Table

A table stores structured data.

```sql
CREATE TABLE attendance.employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    department VARCHAR(50)
);
```

## Row

A row represents one record.

```text
1 | Mukesh | DevOps
```

## Column

A column represents one attribute.

```text
id
name
department
```

## Fully Qualified Table Name

```sql
attendance.employees
```

Here:

* `attendance` = schema
* `employees` = table

---

# 7. Primary Key, Foreign Key, and Index

## Primary Key

A primary key uniquely identifies each row.

```sql
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);
```

A primary key:

* Must be unique
* Cannot be NULL
* Identifies a row

## Foreign Key

A foreign key creates a relationship between tables.

```sql
CREATE TABLE departments (
    id SERIAL PRIMARY KEY,
    department_name VARCHAR(100)
);

CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id)
        REFERENCES departments(id)
);
```

## Index

An index improves lookup performance.

```sql
CREATE INDEX idx_employee_name
ON employees(name);
```

An index can speed up reads but adds storage and write-maintenance overhead.

---

# 8. How Data is Stored

PostgreSQL stores data in database files on disk.

Important concepts include:

* Pages
* Shared buffers
* Heap tables
* Indexes
* WAL
* Checkpoints
* VACUUM

## Simplified Write Flow

```text
Application
    │
    ▼
PostgreSQL
    │
    ▼
Shared Buffers
    │
    ▼
WAL
    │
    ▼
Data Files
```

## Shared Buffers

Shared buffers are PostgreSQL's memory area for caching data pages.

## WAL

WAL stands for **Write-Ahead Log**.

PostgreSQL records changes in WAL before the corresponding data pages are considered safely persisted.

## Pages

PostgreSQL stores table and index data in fixed-size pages, commonly 8 KB by default.

---

# 9. Why is PostgreSQL Powerful?

PostgreSQL is powerful because it supports:

* Complex SQL queries
* Joins
* Transactions
* Strong data integrity
* Constraints
* Advanced indexing
* JSONB
* Window functions
* Common Table Expressions
* Extensions
* Full-text search
* Replication
* Partitioning
* Stored procedures and functions

Example:

```sql
SELECT department, COUNT(*)
FROM employees
GROUP BY department;
```

PostgreSQL is suitable when data relationships, correctness, and complex querying are important.

---

# 10. Transactions and ACID Properties

A transaction is a group of operations treated as one logical unit.

Example:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 1000
WHERE id = 1;

UPDATE accounts
SET balance = balance + 1000
WHERE id = 2;

COMMIT;
```

If an error occurs:

```sql
ROLLBACK;
```

## ACID

### Atomicity

All operations in a transaction succeed or none of them are applied.

### Consistency

The database moves from one valid state to another valid state while respecting constraints.

### Isolation

Concurrent transactions are isolated according to the configured isolation level.

### Durability

Committed data survives a successful commit even if the server later crashes, subject to the configured durability settings.

---

# 11. PostgreSQL Concurrency and MVCC

PostgreSQL uses **MVCC**, which stands for Multi-Version Concurrency Control.

MVCC allows transactions to work with consistent snapshots of data.

Example:

```text
Transaction A → Reads employee record
Transaction B → Updates employee record
Transaction A → Continues using its snapshot
```

PostgreSQL does not normally require every read to block every write.

## Isolation Levels

Common isolation levels include:

* Read Committed
* Repeatable Read
* Serializable

PostgreSQL also supports:

* Row-level locks
* Table-level locks
* Advisory locks
* `SELECT ... FOR UPDATE`

Example:

```sql
SELECT *
FROM employees
WHERE employee_id = 1
FOR UPDATE;
```

---

# 12. PostgreSQL Persistence and WAL

PostgreSQL is a persistent database.

## Write-Ahead Logging

The WAL principle is:

```text
Change
  │
  ▼
WAL Record
  │
  ▼
Data Page Flush
```

The WAL record is written before the modified data page is written to its final location.

## Checkpoint

A checkpoint helps ensure that modified data pages are flushed and recovery does not need to replay an unlimited amount of WAL.

## Recovery

After a crash, PostgreSQL uses WAL to recover committed changes and restore database consistency.

## Backup Types

Common backup approaches include:

* `pg_dump`
* `pg_dumpall`
* Physical base backups
* WAL archiving
* Point-in-time recovery

---

# 13. What Happens if PostgreSQL Goes Down?

PostgreSQL is commonly deployed with backups, replication, or high-availability solutions.

In a single-instance deployment:

```text
Application
    │
    ▼
PostgreSQL Down
    │
    ▼
Requests Fail
```

The application normally cannot read or write data until PostgreSQL becomes available again.

With a highly available setup:

```text
Application
    │
    ▼
Primary PostgreSQL
    │
    ▼
Standby PostgreSQL
```

A failover mechanism may promote a standby when the primary fails.

Important concepts include:

* Streaming replication
* Synchronous replication
* Asynchronous replication
* Read replicas
* Patroni
* Connection pooling
* Automated failover

---

# 14. PostgreSQL Use Cases

| Use Case | Description |
|---|---|
| Attendance Systems | Store attendance records |
| Banking | Store financial transactions |
| E-commerce | Store orders and payments |
| HR Applications | Store employee data |
| Enterprise Applications | Store relational business data |
| Reporting | Run analytical SQL queries |
| Microservices | Provide durable service-specific storage |
| Inventory | Track products and stock |
| Financial Systems | Maintain transactional consistency |
| JSON Applications | Store semi-structured data using JSONB |

---

# 15. Common PostgreSQL Commands

## Connect to PostgreSQL

```bash
psql -U postgres
```

Connect to a database:

```bash
psql -U postgres -d attendance_db
```

Connect using a host:

```bash
psql -h <HOST> -U <USER> -d <DATABASE>
```

## Database Commands

List databases:

```sql
\l
```

Create a database:

```sql
CREATE DATABASE attendance_db;
```

Connect to a database:

```sql
\c attendance_db
```

Drop a database:

```sql
DROP DATABASE attendance_db;
```

## Schema Commands

List schemas:

```sql
\dn
```

Create a schema:

```sql
CREATE SCHEMA attendance;
```

Set the search path:

```sql
SET search_path TO attendance;
```

## Table Commands

List tables:

```sql
\dt
```

Describe a table:

```sql
\d attendance.employees
```

Create a table:

```sql
CREATE TABLE attendance.employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    department VARCHAR(50)
);
```

Drop a table:

```sql
DROP TABLE attendance.employees;
```

## Insert

```sql
INSERT INTO attendance.employees
(name, department)
VALUES
('Mukesh', 'DevOps');
```

## Read

```sql
SELECT *
FROM attendance.employees;
```

Read selected columns:

```sql
SELECT name, department
FROM attendance.employees;
```

## Update

```sql
UPDATE attendance.employees
SET department = 'Cloud DevOps'
WHERE id = 1;
```

## Delete

```sql
DELETE FROM attendance.employees
WHERE id = 1;
```

## Index

```sql
CREATE INDEX idx_employee_department
ON attendance.employees(department);
```

## Transaction

```sql
BEGIN;

UPDATE attendance.employees
SET department = 'Platform Engineering'
WHERE id = 1;

COMMIT;
```

Rollback:

```sql
ROLLBACK;
```

## Server and Activity

```sql
SELECT version();
```

```sql
SELECT current_database();
```

```sql
SELECT *
FROM pg_stat_activity;
```

## PostgreSQL Service

```bash
sudo systemctl status postgresql
```

```bash
sudo systemctl start postgresql
```

```bash
sudo systemctl restart postgresql
```

---

# 16. Frequently Asked Interview Questions

| Question | Detailed Answer |
|---|---|
| **What is PostgreSQL?** | PostgreSQL is an open-source object-relational database management system that supports SQL, transactions, constraints, indexing, JSONB, and advanced querying. |
| **Is PostgreSQL SQL or NoSQL?** | PostgreSQL is primarily a SQL relational database. |
| **What is a schema?** | A schema is a logical namespace inside a database that organizes tables, views, functions, and other objects. |
| **What is a primary key?** | A primary key uniquely identifies each row and cannot contain NULL values. |
| **What is a foreign key?** | A foreign key enforces a relationship between a child table and a referenced parent table. |
| **What is an index?** | An index is a data structure that improves lookup performance but adds storage and write overhead. |
| **What is ACID?** | ACID stands for Atomicity, Consistency, Isolation, and Durability. |
| **What is a transaction?** | A transaction is a group of operations executed as one logical unit. |
| **What is MVCC?** | MVCC stands for Multi-Version Concurrency Control. It allows transactions to work with consistent snapshots while reducing read-write blocking. |
| **What is WAL?** | WAL stands for Write-Ahead Log. PostgreSQL records changes in WAL before writing modified data pages to their final storage locations. |
| **What is VACUUM?** | VACUUM reclaims space from obsolete row versions and helps maintain table health. |
| **What is the default PostgreSQL port?** | The default port is 5432. |
| **What is the difference between DELETE, TRUNCATE, and DROP?** | DELETE removes selected rows, TRUNCATE removes all rows from a table efficiently, and DROP removes the table object itself. |
| **What is the difference between PostgreSQL and Redis?** | PostgreSQL is a persistent relational source-of-truth database, while Redis is commonly used for in-memory caching and low-latency key-value operations. |
| **What is the difference between PostgreSQL and ScyllaDB?** | PostgreSQL is relational and supports joins and complex SQL. ScyllaDB is a distributed wide-column NoSQL database designed for high throughput and horizontal scalability. |
| **Can PostgreSQL store JSON?** | Yes. PostgreSQL supports JSON and JSONB data types. |
| **Can PostgreSQL be replicated?** | Yes. PostgreSQL supports streaming replication, logical replication, and other high-availability architectures. |

---

# 17. PostgreSQL in OT-Microservices

In the OT-Microservices project:

* **Attendance API (Python Flask)** uses PostgreSQL.
* The attendance database is named `attendance_db`.
* Redis may be used as a cache.
* Liquibase is used for database schema migration.

## Database Ownership

```text
Attendance API
      │
      ▼
PostgreSQL
      │
      ▼
attendance_db
      │
      ▼
Attendance Tables
```

PostgreSQL should remain the **source of truth** for attendance data.

Redis can cache frequently accessed attendance information, but the durable attendance records remain in PostgreSQL.

## Request Flow

```text
Client
   │
   ▼
NGINX / Frontend
   │
   ▼
Attendance API
   │
   ▼
Redis Cache
   │
   ├── Cache Hit → Return Response
   │
   └── Cache Miss
          │
          ▼
      PostgreSQL
          │
          ▼
      Store Result in Redis
          │
          ▼
      Return Response
```

## Database Migration

Liquibase can manage schema changes.

```bash
liquibase update
```

Typical migration tasks include:

* Creating tables
* Adding columns
* Creating indexes
* Adding constraints
* Updating database structure

This allows database changes to be version-controlled and applied consistently across environments.
