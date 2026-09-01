# OT-Microservices — EC2 Database Backup & Recovery

## Scope

This guide covers backup and recovery when PostgreSQL, ScyllaDB, and Redis run directly on separate EC2 instances.

```text
PostgreSQL EC2 ──┐
ScyllaDB EC2 ────┼──> Backup ──> Amazon S3
Redis EC2 ───────┘
```

> Store disaster-recovery backups outside the database EC2 instance. A backup kept only on the same server can be lost with that server.

---

## 1. PostgreSQL Backup

### Check PostgreSQL

```bash
sudo systemctl status postgresql
sudo -u postgres psql -c "\l"
```

### Create a backup directory

```bash
sudo mkdir -p /var/backups/postgresql
sudo chown postgres:postgres /var/backups/postgresql
```

### Backup `attendance_db`

```bash
sudo -u postgres pg_dump \
  -d attendance_db \
  -F c \
  -f /var/backups/postgresql/attendance_db_$(date +%F_%H-%M-%S).dump
```

`-F c` creates PostgreSQL custom format, which can be restored with `pg_restore`.

Check:

```bash
sudo ls -lh /var/backups/postgresql/
```

### Backup all PostgreSQL databases

```bash
sudo -u postgres pg_dumpall \
  > /var/backups/postgresql/postgres_all_$(date +%F_%H-%M-%S).sql
```

### Restore

Create the target database if required:

```bash
sudo -u postgres createdb attendance_db
```

Restore:

```bash
sudo -u postgres pg_restore \
  -d attendance_db \
  /var/backups/postgresql/attendance_db_YYYY-MM-DD_HH-MM-SS.dump
```

---

## 2. PostgreSQL Backup to S3

Use an EC2 IAM role instead of hardcoding AWS access keys.

Test AWS identity:

```bash
aws sts get-caller-identity
```

Upload:

```bash
aws s3 cp \
  /var/backups/postgresql/attendance_db_YYYY-MM-DD_HH-MM-SS.dump \
  s3://otms-database-backups/postgresql/
```

Verify:

```bash
aws s3 ls s3://otms-database-backups/postgresql/
```

For production, enable appropriate S3 encryption, versioning, lifecycle retention, and restricted IAM permissions.

---

## 3. ScyllaDB Backup

### Check ScyllaDB

```bash
sudo systemctl status scylla-server
nodetool status
```

### Create a snapshot

All keyspaces:

```bash
sudo nodetool snapshot
```

Specific keyspace:

```bash
sudo nodetool snapshot employee
```

Find snapshots:

```bash
sudo find /var/lib/scylla/data -type d -name snapshots
```

> A snapshot on the same EC2 disk is not sufficient as a disaster-recovery backup. Copy it to external durable storage.

For a simple DEV/manual workflow, archive the required snapshot data and upload it:

```bash
sudo tar -czf /tmp/scylla-snapshot-$(date +%F_%H-%M-%S).tar.gz \
  /var/lib/scylla/data
```

Then:

```bash
aws s3 cp \
  /tmp/scylla-snapshot-YYYY-MM-DD_HH-MM-SS.tar.gz \
  s3://otms-database-backups/scylladb/
```

> For large or production ScyllaDB deployments, use Scylla's supported backup/restore tooling rather than blindly archiving the entire data directory. Follow the procedure appropriate to your ScyllaDB version and topology.

### Recovery

High-level recovery flow:

```text
S3 backup
   |
   v
Replacement ScyllaDB node/cluster
   |
   v
Restore using supported Scylla procedure
   |
   v
Validate keyspace/tables
   |
   v
Application reconnects
```

Validate:

```bash
nodetool status
cqlsh
```

Then query the required keyspaces/tables.

---

## 4. Redis Backup

Redis is different because it is commonly used as a cache.

Persistence options include:

- RDB snapshots
- AOF (Append Only File)
- Both, depending on the recovery requirement

Check configuration:

```bash
redis-cli CONFIG GET save
redis-cli CONFIG GET appendonly
redis-cli CONFIG GET dir
redis-cli CONFIG GET dbfilename
```

### Create an RDB snapshot

```bash
redis-cli BGSAVE
```

Check the last successful save:

```bash
redis-cli LASTSAVE
```

Find the Redis data directory:

```bash
redis-cli CONFIG GET dir
```

Find the RDB filename:

```bash
redis-cli CONFIG GET dbfilename
```

Then copy the RDB file to a backup directory. For example, if the configured directory is `/var/lib/redis` and filename is `dump.rdb`:

```bash
sudo mkdir -p /var/backups/redis

sudo cp /var/lib/redis/dump.rdb \
  /var/backups/redis/dump-$(date +%F_%H-%M-%S).rdb
```

### Upload to S3

```bash
aws s3 cp \
  /var/backups/redis/dump-YYYY-MM-DD_HH-MM-SS.rdb \
  s3://otms-database-backups/redis/
```

Verify:

```bash
aws s3 ls s3://otms-database-backups/redis/
```

### Redis restore

Stop Redis:

```bash
sudo systemctl stop redis-server
```

Download the backup:

```bash
aws s3 cp \
  s3://otms-database-backups/redis/dump-YYYY-MM-DD_HH-MM-SS.rdb \
  /var/lib/redis/dump.rdb
```

Fix ownership:

```bash
sudo chown redis:redis /var/lib/redis/dump.rdb
```

Start Redis:

```bash
sudo systemctl start redis-server
```

Verify:

```bash
redis-cli ping
```

Expected:

```text
PONG
```

> If Redis is only a rebuildable cache in OTMS, loss of Redis should not mean loss of source-of-truth data. The application should be able to repopulate the cache from PostgreSQL/ScyllaDB.

---

## 5. Recommended S3 Layout

```text
s3://otms-database-backups/
├── postgresql/
│   ├── attendance_db_2026-09-01_01-00-00.dump
│   └── ...
├── scylladb/
│   ├── snapshot-2026-09-01/
│   └── ...
└── redis/
    ├── dump-2026-09-01_01-00-00.rdb
    └── ...
```

---

## 6. Automate the Backups

For DEV, cron can run a backup script.

A proper backup script should:

1. Create the backup.
2. Validate the backup.
3. Upload it to S3.
4. Verify the upload.
5. Remove old local copies.
6. Log success/failure.
7. Alert when a backup fails.

Do not store AWS access keys directly in scripts. Prefer an EC2 instance profile/IAM role with least-privilege S3 permissions.

---

## 7. Backup vs High Availability

These are different concepts.

### Backup

Protects against:

- Accidental deletion
- Data corruption
- Disaster
- Loss of an EC2 instance

### High Availability

Protects against:

- Node failure
- Hardware failure
- Service interruption

A single-node DEV database with S3 backups provides **recoverability**, but it is not highly available.

---

## 8. OTMS DEV Architecture

```text
              OT-Microservices
                     |
        +------------+------------+
        |            |            |
        v            v            v
   PostgreSQL     ScyllaDB       Redis
      EC2            EC2          EC2
        |             |            |
     pg_dump       snapshot      RDB/AOF
        |             |            |
        +-------------+------------+
                      |
                      v
                  Amazon S3
               Backup / DR copy
```

---

## 9. Recovery Flow

### PostgreSQL

```text
PostgreSQL EC2 failure
        |
        v
Provision replacement EC2
        |
        v
Install PostgreSQL
        |
        v
Download backup from S3
        |
        v
pg_restore
        |
        v
Validate database
        |
        v
Application reconnects
```

### ScyllaDB

```text
ScyllaDB EC2 failure
        |
        v
Provision replacement node/cluster
        |
        v
Restore using supported Scylla backup procedure
        |
        v
Validate keyspace/data
        |
        v
Application reconnects
```

### Redis

```text
Redis EC2 failure
        |
        +-----------------------------+
        |                             |
        v                             v
Cache is rebuildable            Persistent Redis data
        |                             |
        v                             v
Reinstall Redis                 Restore RDB/AOF
        |                             |
        +-------------+---------------+
                      |
                      v
                Redis available
```

---

## 10. Reviewer Questions and Answers

### Q: Why are PostgreSQL and ScyllaDB single-node in DEV?

> "This is a DEV environment, so I intentionally kept the database deployment simple to control cost and infrastructure complexity. I understand that a single node is a single point of failure. Production would use appropriate replication, HA, and automated failover."

### Q: What happens if the database EC2 instance dies?

> "The application will temporarily lose database connectivity. I recover the database using the backup stored outside the failed EC2 instance, preferably in S3. I provision a replacement instance and restore the data."

### Q: What if the EC2 instance and its local backup are both lost?

> "The local copy is not my disaster-recovery copy. The backup is uploaded to S3, which is independent of the database EC2 instance."

### Q: Does backup provide HA?

> "No. Backup provides recoverability, while HA provides availability during failures. They are complementary controls."

### Q: How do you back up PostgreSQL?

> "I use `pg_dump` for logical database backups and `pg_restore` for recovery. The backup is then stored in S3."

### Q: How do you back up ScyllaDB?

> "I use ScyllaDB snapshots and the supported backup/restore procedure for the ScyllaDB version and topology. The backup is stored outside the database host."

### Q: How do you back up Redis?

> "I use Redis RDB snapshots, or AOF where the recovery requirements call for it. If Redis is only a cache, the source data remains in PostgreSQL or ScyllaDB and the cache can be rebuilt."

---

## 11. Key Interview Statement

> **"In DEV, PostgreSQL and ScyllaDB are single-node deployments for cost and simplicity, so they don't provide database HA. I provide recoverability through database-level backups stored outside the EC2 instances, preferably in S3. PostgreSQL uses `pg_dump`, ScyllaDB uses snapshots and supported backup tooling, and Redis uses RDB/AOF depending on the persistence requirement. In production I would add replication, automated failover, backup retention, encryption, monitoring, and regular restore testing."**


---

## 12. AWS EBS Snapshots vs Database-Native Backups

When databases run directly on EC2, their data is normally stored on EBS volumes.

An AWS EBS snapshot is a **block-level volume snapshot**. It is useful for infrastructure-level recovery, but it should not automatically be treated as a replacement for a database-native backup.

### EBS snapshot flow

```text
Database EC2
     |
     v
  EBS Volume
     |
     | EBS Snapshot
     v
AWS Snapshot Storage
     |
     | Restore
     v
New EBS Volume
     |
     v
Replacement EC2
```

Create a snapshot manually:

```bash
aws ec2 create-snapshot \
  --volume-id vol-xxxxxxxx \
  --description "OTMS PostgreSQL EBS backup"
```

Check snapshots:

```bash
aws ec2 describe-snapshots \
  --owner-ids self \
  --query 'Snapshots[*].[SnapshotId,VolumeId,StartTime,State,Description]' \
  --output table
```

### EBS snapshot vs database backup

| | EBS Snapshot | Database-Native Backup |
|---|---|---|
| Backup level | Block/device | Database |
| PostgreSQL-aware | No | Yes |
| Granular DB/table restore | Limited | Yes |
| Full volume/server recovery | Yes | No |
| Fast volume recovery | Yes | Usually slower |
| Database-consistent by default | Not necessarily | Yes, when using the database's backup mechanism |
| Disaster recovery | Yes | Yes |
| Recommended use | Infrastructure/volume recovery | Database recovery |

For PostgreSQL, do not assume that an EBS snapshot taken during active writes is equivalent to a clean logical database backup. Database-native backups should remain part of the recovery strategy.

---

## 13. Should EBS Snapshots Replace `pg_dump`?

No.

A stronger production strategy is:

```text
                 Backup Strategy
                       |
          +------------+------------+
          |                         |
          v                         v
 Database-native backup       EBS Snapshot
          |                         |
          v                         v
         S3                    AWS Snapshot
          |                         |
          v                         v
 DB-level recovery          Volume-level recovery
```

### PostgreSQL

Use:

```bash
pg_dump
```

for database-level recovery.

Also consider EBS snapshots for faster infrastructure-level recovery.

### ScyllaDB

Use ScyllaDB's supported backup/snapshot mechanism for the deployed ScyllaDB version and topology.

EBS snapshots can additionally be used for volume-level disaster recovery where appropriate.

### Redis

Use RDB/AOF according to the persistence requirement.

If Redis is only a cache in OTMS, the primary data should remain in PostgreSQL/ScyllaDB, so Redis can be rebuilt after a failure.

---

## 14. Automating EBS Snapshots

You do **not** have to use Linux cron to schedule EBS snapshots.

For AWS-native automation, use **Amazon Data Lifecycle Manager (DLM)**.

Example concept:

```text
                 DLM Policy
                     |
               Scheduled execution
                     |
                     v
                 EBS Volume
                     |
                     v
                  Snapshot
                     |
                     v
             Retention policy
                     |
                     v
             Older snapshots
                 deleted
```

For example:

```text
Daily snapshot
      |
      v
Keep last 7 snapshots
      |
      v
Automatically expire older snapshots
```

This avoids maintaining a cron job solely for EBS snapshot scheduling.

> DLM is particularly useful for EC2/EBS infrastructure backups. Database-native backups still require an appropriate database backup mechanism and schedule.

---

## 15. Do I Need Cron?

It depends on what you are automating.

### Database-native backup

For a simple DEV environment, cron is perfectly reasonable:

```bash
crontab -e
```

Example:

```cron
0 2 * * * /usr/local/bin/otms-backup.sh
```

The script can:

```text
cron
  |
  v
backup script
  |
  +--> PostgreSQL pg_dump
  |
  +--> ScyllaDB backup
  |
  +--> Redis RDB/AOF handling
  |
  v
Upload backup to S3
  |
  v
Verify upload
```

The script should also log failures and clean up old local backup files.

### EBS snapshots

Prefer AWS Data Lifecycle Manager for scheduled EBS snapshots rather than writing your own cron job.

---

## 16. Recommended OTMS DEV Backup Architecture

For the current EC2-based DEV architecture:

```text
       PostgreSQL EC2
             |
          pg_dump
             |
             v
             S3
             ^
             |
     ScyllaDB EC2
             |
     Scylla backup/
       snapshot
             |
             v
             S3

       Redis EC2
             |
          RDB/AOF
             |
             v
             S3


Additional infrastructure protection:

PostgreSQL EBS ──> EBS Snapshot
ScyllaDB EBS ────> EBS Snapshot
Redis EBS ───────> EBS Snapshot
                         ^
                         |
                        DLM
```

---

## 17. Backup Retention

A practical DEV example could be:

```text
Database-native backups
    |
    +--> Daily
    +--> Keep 7 days
    +--> Store in S3

EBS snapshots
    |
    +--> Daily
    +--> Keep 7 snapshots
    +--> Managed by DLM
```

Production retention should be based on the organization's RPO, RTO, compliance, and data-retention requirements.

---

## 18. RPO and RTO

These are important concepts when discussing backups with reviewers.

### RPO — Recovery Point Objective

How much data loss is acceptable?

Example:

```text
RPO = 1 hour
```

means that, after a disaster, the organization accepts losing at most approximately one hour of changes.

### RTO — Recovery Time Objective

How long can the service be unavailable?

Example:

```text
RTO = 30 minutes
```

means the service should be restored within approximately 30 minutes.

Backups affect RPO/RTO, but **HA and replication are what reduce downtime during normal node failures**.

---

## 19. Backup vs HA vs Snapshot

Do not mix these concepts.

```text
                 Database Protection
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
       Backup          Snapshot          HA
          |              |              |
          v              v              v
      Data/DB         EBS volume      Availability
      recovery        recovery        / failover
```

### Backup

Protects against:

- Accidental deletion
- Corruption
- Logical errors
- Disaster recovery

### EBS Snapshot

Protects against:

- Volume-level failure
- Infrastructure recovery
- Fast volume restoration

### HA/Replication

Protects against:

- Node failure
- Service interruption
- Availability loss

Therefore:

> **EBS snapshots and database backups improve recoverability; replication and automated failover improve availability.**

---

## 20. How to Avoid Downtime During Recovery

Backups alone cannot provide zero downtime.

With the current DEV single-node setup:

```text
Database EC2 fails
       |
       v
Restore backup
       |
       v
Database available
```

There will be recovery downtime.

For production, use:

```text
                 DB Endpoint
                      |
             +--------+--------+
             |                 |
             v                 v
         DB Primary        DB Standby
             |                 ^
             +-- replication -+
                      |
                 Auto Failover
                      |
                      v
                 Application
```

The standby already contains the data, so failover does not require waiting for an S3 restore.

---

## 21. Reviewer Answer — AWS Snapshots

### Q: "Why aren't you only using EBS snapshots?"

> "EBS snapshots provide block-level volume recovery, but they are not a substitute for database-aware backups. For PostgreSQL I use pg_dump for database-level recovery and can additionally use EBS snapshots for volume-level recovery. For ScyllaDB I use its supported backup mechanism. I store database backups outside the database EC2 instance in S3."

### Q: "Do you use cron?"

> "For DEV database-native backups, I can use cron to execute the backup script and upload the result to S3. For EBS snapshots, I would prefer AWS Data Lifecycle Manager because AWS can manage the snapshot schedule and retention without a host-level cron job."

### Q: "Will an EBS snapshot give you zero downtime?"

> "No. A snapshot is a backup/recovery mechanism, not database HA. To minimize downtime, production needs replication and automated failover."

### Q: "What happens if your database EC2 dies?"

> "In DEV, I restore the database from the external S3 backup, so there is recovery downtime. In production, I would use a replicated standby with automated failover to minimize or avoid service interruption, while retaining S3 backups for disaster recovery."

---

## 22. Final Interview Statement

> **"My backup strategy has two layers. First, database-native backups such as PostgreSQL pg_dump and ScyllaDB-supported backups are stored in S3 for database-level recovery. Second, EBS snapshots provide infrastructure-level volume recovery and can be automated using AWS Data Lifecycle Manager. In DEV, cron is acceptable for scheduling database-native backup scripts. I don't consider snapshots a replacement for database backups, and neither provides HA by itself. For production, I would add database replication and automated failover to meet the required RPO and RTO."**
