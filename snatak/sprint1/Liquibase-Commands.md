# 📦 Liquibase Cheat Sheet

<div align="center">
<img width="180" alt="Liquibase" src="https://www.liquibase.com/hubfs/liquibase-logo.png" />
</div>

---

## 🔹 Version & Validation

```bash
# Check Liquibase version
liquibase --version

# Validate changelog
liquibase validate

# Check pending migrations
liquibase status

# View migration history
liquibase history
```

---

## 🔹 Migration Commands

```bash
# Run all pending migrations
liquibase update

# Run limited changesets
liquibase updateCount 2

# Generate SQL without execution
liquibase updateSQL
```

---

## 🔹 Rollback Commands

```bash
# Rollback last migration
liquibase rollbackCount 1

# Rollback last 3 migrations
liquibase rollbackCount 3

# Rollback to specific tag
liquibase rollback v1.0

# Rollback to specific date
liquibase rollbackToDate "2026-05-21 10:30:00"

# Generate rollback SQL
liquibase rollbackSQL v1.0
```

---

## 🔹 Tag Commands

```bash
# Create tag
liquibase tag v1.0

# Verify tag exists
liquibase tagExists v1.0
```

---

## 🔹 Lock Management

```bash
# Check active locks
liquibase listLocks

# Release stuck locks
liquibase releaseLocks
```

---

## 🔹 Checksum Commands

```bash
# Clear checksums
liquibase clearCheckSums

# Calculate checksum
liquibase calculateCheckSum migration.xml::1::mukesh
```

---

## 🔹 Changelog Commands

```bash
# Generate changelog from existing DB
liquibase generateChangeLog

# Compare database schemas
liquibase diff

# Generate diff changelog
liquibase diffChangeLog
```

---

## 🔹 Documentation & Snapshot

```bash
# Generate DB documentation
liquibase dbDoc ./db-docs

# Create DB snapshot
liquibase snapshot
```

---

# 🔹 Liquibase Tables

| Table                   | Purpose                        |
| ----------------------- | ------------------------------ |
| `databasechangelog`     | Stores migration history       |
| `databasechangeloglock` | Prevents concurrent migrations |

---

# 🔹 OT-Micro Attendance API Commands

```bash
cd ~/attendance-api

# Run migrations
make run-migrations

# Run liquibase directly
liquibase update

# Rollback last migration
liquibase rollbackCount 1

# Check migration history
liquibase history

# Check pending migrations
liquibase status

# Release stuck locks
liquibase releaseLocks
```

---

## 🔹 PostgreSQL Verification Commands

```bash
# Connect PostgreSQL
psql -h 127.0.0.1 -U postgres -d attendance_db

# Check executed migrations
SELECT id, author, filename, dateexecuted
FROM databasechangelog;

# Check lock status
SELECT * FROM databasechangeloglock;
```

---

## 🔹 Liquibase Flow

```text
Developer → ChangeSet → Liquibase Update → PostgreSQL
                               ↓
                  databasechangelog updated
```

---

## 🔹 Quick Notes

* Liquibase only executes NEW changesets.
* `databasechangelog` tracks executed migrations.
* `databasechangeloglock` prevents parallel execution.
* Rollback is safest using TAG-based strategy.

---

## 🔹 References

* [https://docs.liquibase.com](https://docs.liquibase.com)
* [https://www.liquibase.org/documentation](https://www.liquibase.org/documentation)

---

Prepared for OT-Microservices Liquibase Integration.
