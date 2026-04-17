# 📘 Database Migration Guide (Liquibase + OT-Micro Context)

<p align="center">
  <img width="120" src="https://cdn-icons-png.flaticon.com/512/919/919836.png" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Tool-Liquibase-blue?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Database-MySQL-orange?style=for-the-badge" />
  <img src="https://img.shields.io/badge/DevOps-Migrations-green?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Project-OT--Microservices-purple?style=for-the-badge" />
</p>

---

## 📌 Introduction

In real-world applications, database structure (schema) keeps changing over time. Instead of manually writing SQL again and again, we use **migrations** to manage these changes in a controlled and versioned way.

This guide explains:

* What is run migration
* How Liquibase works
* How OT-Micro uses SQL migrations
* Role of Makefile and configuration files

---

## 🔹 What is Run Migration?

Run migration means:

👉 Applying database changes automatically using predefined files.

Instead of doing this manually:

```sql
CREATE TABLE employee (...);
```

We write migration files and run:

```bash
make run-migrations
```

👉 This executes all pending database changes.

---

## 🔹 Why Migrations are Important

* Keeps database changes version-controlled
* Avoids manual mistakes
* Ensures same DB structure across environments
* Supports team collaboration

---

## 🔹 OT-Micro Migration Approach (SQL Based)

In OT-Micro, migrations are written as SQL files:

```
migration/
 ├── 000001_create_employee_info_table.up.sql
 ├── 000001_create_employee_info_table.down.sql
 ├── 000002_create_salary_table.up.sql
```

### Types:

* `.up.sql` → Apply change
* `.down.sql` → Rollback change

---

## 🔹 Manual Execution vs Run Migration (Important)

### 👉 Manual Execution (Without Migration Tool)

You directly login to database and run SQL:

```bash
mysql -u root -p
```

```sql
CREATE TABLE employee (
  id INT PRIMARY KEY,
  name VARCHAR(100)
);
```

👉 Problems with manual approach:

* No tracking of changes
* Easy to forget what was executed
* Not repeatable across environments
* Team members may have different DB states

---

### 👉 Using Run Migration (Automated Way)

You write migration file:

```sql
-- 000001_create_employee_table.up.sql
CREATE TABLE employee (
  id INT PRIMARY KEY,
  name VARCHAR(100)
);
```

Then run:

```bash
make run-migrations
```

👉 What happens:

1. Tool reads migration folder
2. Checks already executed files
3. Runs only new ones
4. Updates database automatically

---

### 🔑 Key Difference

| Feature          | Manual Execution | Run Migration |
| ---------------- | ---------------- | ------------- |
| Tracking         | ❌ No             | ✅ Yes         |
| Automation       | ❌ No             | ✅ Yes         |
| Team consistency | ❌ No             | ✅ Yes         |
| Rollback         | ❌ Hard           | ✅ Easy        |

---

## 🔹 What Happens When You Run Migration

```bash
make run-migrations
```

👉 Internally:

1. Migration tool reads SQL files
2. Checks which ones are already applied
3. Runs only new ones
4. Updates database

---

## 🔹 What is Liquibase?

Liquibase is a tool used to manage database schema changes in a structured and version-controlled way.

Instead of raw SQL files, it uses **changeSets**.

---

## 🔹 Liquibase Example

```xml
<changeSet id="1" author="mukesh">
    <createTable tableName="employee">
        <column name="id" type="int"/>
        <column name="name" type="varchar(100)"/>
    </createTable>
</changeSet>
```

👉 Each changeSet acts like a commit.

---

## 🔹 How Liquibase Works

When you run:

```bash
liquibase update
```

👉 It:

1. Reads changelog file
2. Connects to database
3. Checks tracking table
4. Executes only new changes

---

## 🔹 Tracking Tables (Very Important)

Liquibase creates:

```
DATABASECHANGELOG
DATABASECHANGELOGLOCK
```

👉 Purpose:

* Track executed migrations
* Prevent duplicate execution
* Ensure idempotency

---

## 🔹 Idempotency in Migrations

👉 Meaning:

Running migration multiple times should not break system.

Liquibase ensures:

* Same changeSet runs only once
* No duplicate tables or columns

---

## 🔹 Liquibase vs SQL Migration (OT-Micro)

| Feature  | SQL Migration  | Liquibase |
| -------- | -------------- | --------- |
| Format   | SQL files      | XML/YAML  |
| Tracking | Tool dependent | Built-in  |
| Rollback | Manual         | Built-in  |

---

## 🔹 Role of Makefile

Makefile is used to simplify commands.

Instead of writing long commands:

```bash
migrate -source file://migration -database "..." up
```

We use:

```bash
make run-migrations
```

### Example Makefile

```makefile
run-migrations:
	migrate -source file://migration -database "DB_URL" up
```

👉 Benefits:

* Easy to remember
* Standardized commands
* Cleaner workflow

---

## 🔹 Role of liquibase.properties

This file stores database configuration.

Example:

```properties
url=jdbc:mysql://localhost:3306/employee_db
username=root
password=root
changeLogFile=db.changelog.xml
```

👉 Purpose:

* Avoid writing DB details every time
* Centralized configuration

---

## 🔹 Full Migration Flow

1. Developer creates migration file
2. Pushes code
3. DevOps runs migration
4. Database updated
5. Application uses new schema

---

## 🔹 Rollback Concept

If something goes wrong:

* SQL → run `.down.sql`
* Liquibase → use rollback command

```bash
liquibase rollbackCount 1
```

👉 Reverts last change

---

## 🔹 Real DevOps Workflow

```
Code Change → Migration File → Run Migration → DB Updated → App Runs
```

---

## 🔹 FAQs (Scenario-Based)

### 1. Migration ran twice — what happens?

Liquibase prevents duplicate execution using tracking table.

---

### 2. New developer joins — how DB sync happens?

They run migration → DB becomes latest automatically.

---

### 3. Column added but not visible?

Migration not executed → run migration again.

---

### 4. Wrong migration applied?

Use rollback or fix with new migration.

---

### 5. Why not run SQL manually?

Manual execution causes inconsistency and errors.

---

### 6. Can migrations fail?

Yes, due to syntax errors or DB conflicts.

---

### 7. Why use Makefile?

To simplify and standardize commands.

---

### 8. Is Liquibase required always?

No. Some projects use raw SQL (like OT-Micro), others use Liquibase.

---

## 🔹 Final Understanding

Migrations are a structured way to manage database changes. Tools like Liquibase automate and track these changes, ensuring safe and consistent database updates across environments.

---

## 🔗 References

* [https://www.liquibase.org/docs](https://www.liquibase.org/docs)
* [https://docs.liquibase.com/concepts/basic.html](https://docs.liquibase.com/concepts/basic.html)
* [https://github.com/golang-migrate/migrate](https://github.com/golang-migrate/migrate)
* [https://www.atlassian.com/devops/database-change-management](https://www.atlassian.com/devops/database-change-management)

---
