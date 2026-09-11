# Database L2 Interview Questions — MySQL, PostgreSQL, ScyllaDB, Redis, MongoDB & Elasticsearch

> **Purpose:** L2 DevOps / SRE / Platform / Database interview

> **Example convention:** Database-specific answers include a short practical example where it helps. Use the example to explain the concept, then state the operational trade-off or failure mode.

> preparation.
>
> Focus: database fundamentals, SQL, administration, performance,
> reliability, HA, backup/restore, security, observability, and
> production troubleshooting.

## Database Comparison — Quick Interview Reference

| Database | Type / Model | Best Known For | Scaling Model | Query Language / Interface | Transactions | Primary Use Cases | Key DevOps Concern |
|---|---|---|---|---|---|---|---|
| **MySQL** | Relational (RDBMS) | Widely used OLTP workloads | Primarily vertical + read replicas / sharding patterns | SQL | ✅ ACID with InnoDB | Web apps, OLTP, transactional services | Replication, slow queries, InnoDB buffer pool, backup/restore |
| **PostgreSQL** | Relational (RDBMS) | Advanced SQL, MVCC, JSONB, extensibility | Vertical + replicas + partitioning/sharding solutions | SQL | ✅ Strong transactional support | OLTP, complex queries, reporting, geospatial/JSON workloads | VACUUM, bloat, WAL, replication lag, long transactions |
| **ScyllaDB** | Wide-column NoSQL | High throughput and predictable low latency | ✅ Horizontal scale-out | CQL | Distributed consistency model | High-volume distributed services, event/time-series style workloads | Partition design, hot shards/partitions, compaction, repair, RF |
| **Redis** | In-memory key-value/data structures | Very low-latency access and caching | Scale-out via replication/cluster patterns | Redis commands | Limited compared with RDBMS; supports transactions/scripts | Caching, sessions, counters, queues, rate limiting | Memory pressure, eviction, persistence, cache stampede |
| **MongoDB** | Document NoSQL | Flexible document model | ✅ Horizontal scaling with sharding | MongoDB query API | ✅ Multi-document transactions supported | Document-oriented applications, APIs, rapidly evolving schemas | Indexes, replica sets, shard-key design |
| **Elasticsearch** | Distributed search/analytics | Full-text search and aggregations | ✅ Horizontal via shards | Query DSL / REST API | Not an RDBMS transaction system | Search, logs, observability, analytics | Shard sizing, refresh, heap, mappings, ingestion pressure |

### When would you choose which?

| Requirement | Typical Choice | Why |
|---|---|---|
| Complex joins, strong relational constraints | **PostgreSQL / MySQL** | SQL + ACID + relational model |
| Advanced SQL + JSONB + rich extensions | **PostgreSQL** | Strong SQL and extensibility |
| Huge distributed write/read throughput with predictable latency | **ScyllaDB** | Horizontal scale + query-driven NoSQL model |
| Very fast temporary/cache/session data | **Redis** | In-memory access and TTL/eviction support |
| Flexible JSON-like application documents | **MongoDB** | Document model and schema flexibility |
| Full-text search and log analytics | **Elasticsearch** | Inverted-index search + aggregations |
| Transactional source of truth + search | **PostgreSQL/MySQL + Elasticsearch** | DB owns truth; Elasticsearch serves search |
| Transactional source of truth + low-latency cache | **PostgreSQL/MySQL + Redis** | DB owns truth; Redis accelerates reads |

> **Interview rule:** Do not answer "Which database is best?" in isolation. State the **workload, consistency requirement, query pattern, scale, latency target, and operational trade-off** first.

## Question Priority

| Priority | Questions | Focus |
|---|---|---|
| **P1** | Q1–Q70 | Core concepts, SQL, PostgreSQL, MySQL, ScyllaDB, performance, HA, backup, troubleshooting |
| **P2** | Q71–Q105 | Redis, MongoDB, Elasticsearch, advanced operations, security, scaling |
| **P3** | Q106–Q120 | Cross-database architecture and advanced production scenarios |

## 1. Database Fundamentals

### Q1. What is a database?

A system for storing, organizing, retrieving, and managing data.
Examples include MySQL, PostgreSQL, ScyllaDB, Redis, MongoDB, and
Elasticsearch.

### Q2. DBMS vs RDBMS?

A DBMS is a general database-management system. An RDBMS uses relational
tables, keys, constraints, and SQL. MySQL and PostgreSQL are RDBMSs.

### Q3. What are table, row, column, and schema?

A table stores related records; a row is one record; a column is an
attribute; a schema is a namespace/organization for database objects.

### Q4. What is a primary key?

A primary key uniquely identifies a row and cannot be NULL. It can also
be composite: `PRIMARY KEY (department_id, employee_id)`.

### Q5. What is a foreign key?

A foreign key references a key in another table and helps enforce
referential integrity.

### Q6. What is normalization?

Normalization reduces unnecessary duplication and update anomalies by
separating related data into appropriate tables. Common forms include
1NF, 2NF, and 3NF.

### Q7. What is denormalization?

Intentional duplication used to reduce joins or improve reads. It trades
storage and consistency complexity for performance.

### Q8. What is an index?

A data structure that can make suitable lookups faster. Indexes consume
storage and add INSERT/UPDATE/DELETE overhead.

``` sql
CREATE INDEX idx_employee_email ON employees(email);
```

### Q9. What is a composite index?

An index on multiple columns, such as
`CREATE INDEX idx_orders ON orders(customer_id, status);`. Column order
matters.

### Q10. What is a transaction?

A logical unit of database work that is committed or rolled back as a
unit.

### Q11. Explain ACID.

Atomicity = all-or-nothing; Consistency = valid state transitions;
Isolation = controlled concurrency; Durability = committed data survives
failures according to the DB's durability guarantees.

``` sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

### Q12. What is an isolation level?

It controls what concurrent transactions can observe. Common levels are
Read Uncommitted, Read Committed, Repeatable Read, and Serializable.

### Q13. Dirty vs non-repeatable vs phantom read?

Dirty = reading uncommitted data. Non-repeatable = the same row changes
between reads. Phantom = the matching row set changes between repeated
queries.

### Q14. What is MVCC?

Multi-Version Concurrency Control lets readers see an appropriate row
version without requiring every reader to wait for writers. PostgreSQL
relies heavily on MVCC.

### Q15. What is a lock?

A concurrency-control mechanism that protects resources. Lock contention
can cause blocking and latency.

### Q16. Blocking vs deadlock?

Blocking is waiting for a conflicting lock. A deadlock is a cycle of
transactions waiting on each other; databases typically abort one
transaction.

### Q17. What is a connection pool?

A pool reuses and limits database connections. It avoids connection
setup for every request and controls concurrency.

### Q18. Can a connection pool be too large?

Yes. For example, 200 app instances × 50 connections = 10,000 possible
connections, which can overwhelm the database.

## 2. SQL and Query Performance

### Q19. WHERE vs HAVING?

`WHERE` filters rows before aggregation; `HAVING` filters groups after
aggregation.

### Q20. DELETE vs TRUNCATE vs DROP?

`DELETE` removes selected rows; `TRUNCATE` removes table rows
efficiently according to DB semantics; `DROP` removes the object itself.

### Q21. What is a JOIN?

A JOIN combines rows from related tables. Common types are INNER, LEFT,
RIGHT, FULL OUTER, and CROSS JOIN.

### Q22. INNER JOIN vs LEFT JOIN?

INNER returns matching rows. LEFT returns every left-side row plus
matching right-side data, otherwise NULLs.

### Q23. How do you troubleshoot a slow SQL query?

Capture the exact query; inspect EXPLAIN/EXPLAIN ANALYZE; check indexes,
estimates, locks, I/O, memory, statistics, concurrency, and recent
changes.

### Q24. What is an execution plan?

The database's chosen strategy for executing a query, including scans,
joins, sorts, aggregates, estimates, and sometimes actual runtime
information.

### Q25. Is a full table scan always bad?

No. A sequential scan can be optimal when the table is small or most
rows are needed.

PostgreSQL:

``` sql
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM employees WHERE department_id = 10;
```

MySQL:

``` sql
EXPLAIN SELECT * FROM employees WHERE department_id = 10;
```

### Q26. Why can an index make performance worse?

Indexes consume storage/cache and must be maintained during writes. Too
many indexes can increase write latency.

### Q27. What is a covering index?

An index that contains enough required columns for a query to avoid
additional base-table lookups, depending on the engine.

### Q28. What is selectivity?

How strongly a predicate narrows the result set. Highly selective
predicates often benefit more from indexes, subject to the optimizer's
cost model.

## 3. PostgreSQL

### Q29. What is PostgreSQL?

An open-source RDBMS known for strong SQL, transactions, MVCC,
extensibility, rich indexing, JSONB, and replication.

**Example:** An employee service can store employees in PostgreSQL when it needs foreign keys, transactions, joins, and reporting queries.

### Q30. Database vs schema in PostgreSQL?

A PostgreSQL cluster can contain multiple databases; a database contains
schemas that namespace objects.

**Example:** A single PostgreSQL database could have schemas such as `hr`, `payroll`, and `audit`, each containing its own tables.

### Q31. What is VACUUM?

It manages dead row versions created by MVCC, helps storage reuse, and
supports healthy visibility information. `VACUUM ANALYZE` also updates
statistics.

**Example:** After many `UPDATE`/`DELETE` operations, PostgreSQL creates dead row versions. `VACUUM employees;` helps reclaim reusable space and maintain visibility information.

### Q32. What is autovacuum?

Automatic PostgreSQL maintenance that vacuums tables and performs
analyze work. Poor tuning can contribute to bloat, stale statistics, and
transaction-ID problems.

**Example:** If an orders table changes heavily throughout the day, autovacuum can automatically vacuum/analyze it instead of requiring a DBA to run maintenance manually.

### Q33. What is PostgreSQL bloat?

Unused/dead space can accumulate because of MVCC and updates/deletes. It
can increase storage and I/O. Remedies depend on the case: VACUUM,
VACUUM FULL, REINDEX, or tools such as pg_repack.

``` sql
VACUUM ANALYZE employees;
```

**Example:** A table that repeatedly updates large rows may grow much larger than the currently live data because old row versions remain until vacuum/other maintenance can clean them up.

### Q34. What is WAL?

Write-Ahead Log records changes before corresponding data pages are
considered durable. It supports crash recovery, replication, and
point-in-time recovery.

**Example:** A committed `UPDATE` is recorded in WAL so PostgreSQL can recover changes after a crash before all corresponding data pages are flushed.

### Q35. What is streaming replication?

A primary sends WAL to a standby, which replays it to maintain a copy.

**Example:** `primary-db` sends WAL continuously to `standby-db`; the standby replays WAL and can be promoted during failover.

### Q36. Synchronous vs asynchronous PostgreSQL replication?

Asynchronous replication does not normally wait for standby
acknowledgement. Synchronous replication can wait for configured standby
acknowledgement, improving durability guarantees at the cost of
latency/availability trade-offs.

``` sql
SELECT pid, usename, state, wait_event_type, wait_event, query FROM pg_stat_activity;
```

**Example:** With asynchronous replication, the client can get a commit response before a standby has confirmed the WAL. With synchronous replication, the primary can wait for configured standby acknowledgement.

### Q37. What is a replication slot?

A slot makes PostgreSQL retain WAL needed by a consumer. A stalled slot
can cause WAL to grow and fill disk.

**Example:** If a logical replication consumer stops reading, PostgreSQL retains WAL for that slot. If it remains stalled, WAL can grow until disk space becomes a problem.

### Q38. What is logical replication?

Replication of logical changes that can support selective table
replication, migrations, and CDC-style integrations.

**Example:** During a migration, you can replicate selected tables from an old PostgreSQL system to a new PostgreSQL system instead of copying the entire cluster continuously.

### Q39. What is a PostgreSQL sequence?

A sequence generates numeric values, often for IDs. Sequence values are
not guaranteed to be gapless.

**Example:** `CREATE TABLE users (id bigint GENERATED BY DEFAULT AS IDENTITY, name text);` uses sequence-backed ID generation; deleting row 5 does not make the next ID become 5.

### Q40. What is JSONB?

PostgreSQL's binary/decomposed JSON representation, optimized for
querying and indexing. GIN indexes can support suitable JSONB queries.

**Example:** `SELECT profile->>'department' FROM employees WHERE profile->>'department' = 'IT';` can query JSONB data stored in a relational table.

### Q41. How do you troubleshoot PostgreSQL high CPU?

Inspect top SQL, `pg_stat_activity`, execution plans,
indexes/statistics, locks, connections, autovacuum, long transactions,
I/O, and host CPU.

**Example:** If CPU suddenly reaches 95%, first identify expensive SQL in `pg_stat_activity`/monitoring, then use `EXPLAIN (ANALYZE, BUFFERS)` on the offending query and check recent deployments or statistics changes.

### Q42. How do you find long-running PostgreSQL queries?

Use `pg_stat_activity`, for example:
`SELECT pid, now()-query_start AS duration, state, query FROM pg_stat_activity ORDER BY duration DESC;`.

**Example:** A query running for 25 minutes may be blocked, scanning too much data, or waiting on I/O. `pg_stat_activity` helps distinguish active execution from waits.

## 4. MySQL

### Q43. What is MySQL?

A widely used RDBMS. Production topics include InnoDB, transactions,
indexes, binary logs, replication, backups, and query optimization.

**Example:** A web application can use MySQL + InnoDB for user accounts, orders, payments, and other OLTP data.

### Q44. What is InnoDB?

MySQL's commonly used transactional storage engine, providing ACID
transactions, row-level locking, foreign keys, MVCC, and crash recovery.

**Example:** `orders` and `payments` stored in InnoDB can use transactions and row-level locking so concurrent updates remain controlled.

### Q45. What is the MySQL binary log?

A record of database changes/events used for common replication and
point-in-time recovery workflows.

**Example:** An `INSERT` into an orders table is recorded in the binary log and can be consumed by a replica or used during point-in-time recovery workflows.

### Q46. What is MySQL replication?

A source records changes in its binary log and a replica retrieves and
applies them.

**Example:** `mysql-primary` writes transactions; `mysql-replica-1` reads/applies the source's binary log and can serve read traffic.

### Q47. What is GTID?

Global Transaction Identifier. It gives transactions stable identifiers
and simplifies replication tracking and failover compared with only
file/position coordinates.

**Example:** During failover, GTIDs help identify which transactions a replica has already applied without relying only on a specific binlog filename and position.

### Q48. What is the slow query log?

A log of queries meeting configured slow-query criteria. It helps
identify expensive SQL and performance regressions.

**Example:** If queries taking more than 2 seconds are logged, the slow-query log can reveal a missing index or an inefficient join that is hurting production latency.

### Q49. How do you troubleshoot MySQL high CPU?

Check `SHOW FULL PROCESSLIST`, slow-query data, EXPLAIN plans, indexes,
locks, connections, buffer-pool behavior, temp tables/sorts,
replication, and host CPU.

**Example:** Check `SHOW FULL PROCESSLIST;`, identify the hottest SQL, run `EXPLAIN`, inspect indexes/joins, and verify whether a recent release changed the query pattern.

### Q50. What is the InnoDB buffer pool?

A major memory area caching InnoDB data and index pages, reducing disk
I/O for frequently accessed data.

**Example:** If frequently accessed index/data pages are already in the buffer pool, MySQL can serve them from memory instead of repeatedly reading them from disk.

### Q51. MySQL vs PostgreSQL?

Both are mature RDBMSs. PostgreSQL is particularly strong in advanced
SQL, extensibility, and JSONB; MySQL/InnoDB is widely deployed with a
strong ecosystem. Choose based on workload and requirements.

**Example:** Choose PostgreSQL when advanced SQL, JSONB, extensions, or complex relational querying dominate; choose MySQL when its ecosystem, existing standards, or operational fit better match the application.

## 5. ScyllaDB

### Q52. What is ScyllaDB?

A distributed NoSQL database using the Cassandra data model and designed
for high throughput and predictable low latency.

``` sql
SHOW FULL PROCESSLIST;
EXPLAIN SELECT * FROM orders WHERE customer_id = 100;
```

**Example:** A telemetry service receiving millions of events can distribute writes across a ScyllaDB cluster using a partition key designed for even traffic distribution.

### Q53. ScyllaDB vs PostgreSQL?

PostgreSQL is relational with SQL, joins, constraints, and rich
transactions. ScyllaDB is wide-column/NoSQL, query-driven, and designed
for horizontal distributed scaling.

**Example:** A payroll system requiring joins and multi-table transactions fits PostgreSQL better; a very high-volume event service with known access patterns may fit ScyllaDB better.

### Q54. What is CQL?

Cassandra Query Language, used by ScyllaDB for data-definition and
data-access operations.

**Example:** `SELECT * FROM employee_info WHERE employee_id = 'E101';` is a CQL-style query used with ScyllaDB.

### Q55. What is a partition key?

It determines how data is distributed across the cluster. Poor keys can
create hot partitions and uneven load.

**Example:** `PRIMARY KEY ((customer_id), event_time)` places all events for a customer in one logical partition. A customer with extreme traffic can become a hot partition.

### Q56. What is a clustering key?

It orders rows within a partition. Example:
`PRIMARY KEY ((employee_id), event_time)`.

**Example:** With `PRIMARY KEY ((employee_id), event_time)`, `employee_id` identifies the partition and `event_time` orders rows inside that partition.

### Q57. Why is ScyllaDB data modeling query-driven?

You start from required access patterns, then choose partition and
clustering keys to make those queries efficient. Denormalization is
often intentional.

``` sql
CREATE TABLE employee_info (
  employee_id text,
  name text,
  event_time timestamp,
  PRIMARY KEY ((employee_id), event_time)
);
```

**Example:** If the application needs `get latest 20 events for employee E101`, design the table so that exact query is efficient rather than relying on joins or ad-hoc scans.

### Q58. What is a hot partition?

A partition receiving disproportionate traffic or data, causing uneven
load and potentially high latency.

**Example:** If 40% of all writes go to one `customer_id`, the node/shards owning that partition can become much busier than the rest of the cluster.

### Q59. What is replication factor?

The number of replicas maintained for a partition under the configured
replication strategy. Higher RF improves redundancy but costs storage
and write resources.

**Example:** With RF=3, each partition is stored on three replicas. Losing one replica does not necessarily make the data unavailable if the consistency/read path still has enough healthy replicas.

### Q60. What is consistency level?

A request-level choice controlling replica participation/acknowledgement
semantics. It trades consistency guarantees, availability, and latency.

**Example:** With RF=3, a QUORUM-style operation requires agreement from enough replicas to meet the configured consistency requirement, trading latency/availability against stronger read/write guarantees.

### Q61. What is quorum?

Generally more than half of the relevant replicas. For RF=3, QUORUM is
2; for RF=5, it is 3.

**Example:** For RF=3, QUORUM = 2; for RF=5, QUORUM = 3. The exact request behavior still depends on the operation and consistency level used.

### Q62. What happens when a ScyllaDB node fails?

Replicas on other nodes can continue serving data subject to consistency
settings. Recovery and repair mechanisms restore replica consistency
after failures.

**Example:** If one node disappears, replicas on other nodes may continue serving requests. The cluster later needs the failed node's data to be repaired/rebalanced according to the architecture and failure-recovery process.

### Q63. What is repair?

A process that reconciles replicas and helps maintain consistency in a
distributed database.

**Example:** Repair can detect differences between replicas and stream the required data so replicas converge again after failures or inconsistencies.

### Q64. What is compaction?

Merging SSTables and removing obsolete storage information according to
the compaction strategy. It can consume significant CPU, disk, and I/O.

**Example:** Many SSTables created by ongoing writes are periodically merged. During heavy compaction, disk I/O and CPU can increase even though application traffic is unchanged.

### Q65. What are tombstones?

Markers for deleted data that remain until safe cleanup/compaction.
Excessive tombstones can hurt reads and increase compaction/storage
pressure.

**Example:** `DELETE` creates tombstone information rather than instantly erasing every physical copy. Large numbers of tombstones can make reads and compaction more expensive.

### Q66. What is an LSM tree?

Log-Structured Merge architecture: writes go through memory structures
and immutable SSTables that are periodically merged. ScyllaDB uses an
LSM-based storage design.

**Example:** A write is first handled in memory and later flushed to immutable SSTable files; background compaction merges these files over time.

### Q67. What is a hot shard?

A shard/core receiving disproportionate workload. Average node CPU can
look healthy while one shard is saturated, so shard-level metrics
matter.

**Example:** A Scylla node may report 40% average CPU while one shard/core is saturated because the workload is unevenly distributed.

## 6. Redis, MongoDB and Elasticsearch

### Q68. What is Redis?

An in-memory data store commonly used for caching, sessions, counters,
queues/streams, and fast key-value access.

**Example:** A login service can store `session:<user_id>` in Redis with a TTL so session lookup avoids hitting the relational database on every request.

### Q69. Why is Redis fast?

Primarily in-memory access, efficient data structures, and low
protocol/processing overhead. Persistence is still possible.

**Example:** A cache lookup such as `GET product:123` can be served from memory with very little processing compared with a disk-backed relational query.

### Q70. What is Redis persistence?

RDB snapshots and AOF are common persistence mechanisms. The choice
depends on durability, recovery, write workload, and storage
requirements.

**Example:** If Redis is used only as a disposable cache, losing it may be acceptable; if it stores important state, the persistence mode and recovery objectives become much more important.

### Q71. What is cache-aside?

Application checks Redis; on a miss it reads the database and populates
Redis. This keeps the database as the usual source of truth.

**Example:** Application → Redis `GET employee:101` → miss → query PostgreSQL → `SET employee:101 ... EX 300` → return response.

### Q72. What is cache stampede?

Many requests miss the same expired key simultaneously and overload the
database. Mitigate with locking/request coalescing, staggered expiry,
prewarming, or stale-while-revalidate patterns.

**Example:** A popular key expires at 10:00:00 and 5,000 requests simultaneously miss it, causing 5,000 database queries. Request coalescing or staggered expiry can reduce the spike.

### Q73. What is Redis eviction?

When a configured memory limit is reached, an eviction policy such as
`allkeys-lru`, `allkeys-lfu`, or `noeviction` controls behavior.

**Example:** With `allkeys-lru`, Redis may evict less-recently-used keys when the configured memory limit is reached.

### Q74. What is Redis TTL?

Time To Live. Example: `SET session:123 abc EX 300` creates an expiring
key.

``` text
Application → Redis → cache hit
             ↓ miss
          Database → populate Redis
```

**Example:** `SET otp:user101 849251 EX 300` stores an OTP that expires after 300 seconds.

### Q75. What is MongoDB?

A document-oriented NoSQL database storing BSON documents.

**Example:** A product catalog can store each product as a document such as `{ "sku": "P100", "name": "Laptop", "tags": ["IT"] }`.

### Q76. MongoDB vs PostgreSQL?

MongoDB is document/access-pattern oriented; PostgreSQL is relational
and strong for joins, constraints, and complex SQL. MongoDB also
supports transactions.

**Example:** A rapidly changing product document may fit MongoDB naturally, while a banking workflow with strong relational constraints and joins is often a better PostgreSQL fit.

### Q77. What is a MongoDB index?

An index helps locate documents efficiently. It improves suitable reads
but costs storage and write work.

**Example:** `db.users.createIndex({ email: 1 })` can make equality searches on `email` much faster, at the cost of additional storage and write maintenance.

### Q78. What is a MongoDB replica set?

A group of MongoDB nodes with a primary and secondaries that provide
redundancy and automatic election/failover.

**Example:** A three-node replica set can have one primary and two secondaries; after primary failure, an eligible secondary can be elected as the new primary.

### Q79. What is Elasticsearch?

A distributed search and analytics engine commonly used for full-text
search, log analytics, observability, and aggregations.

**Example:** Application logs can be indexed into Elasticsearch so operators can search for `HTTP 500`, trace IDs, or error messages across many hosts quickly.

### Q80. Elasticsearch vs PostgreSQL?

PostgreSQL is generally a transactional source of truth; Elasticsearch
is optimized for search and analytics. They can be used together.

**Example:** Keep an order record in PostgreSQL as the source of truth and index a searchable representation in Elasticsearch for fast text search and filtering.

### Q81. What is an Elasticsearch index?

A logical collection of documents that is divided into shards for
distributed storage and search.

**Example:** An index named `application-logs-2026.09.11` can contain many log documents and be split across multiple primary shards.

### Q82. What is an Elasticsearch shard?

A distributed unit of an index. Primary and replica shards provide
distribution and redundancy. Excessive shard counts create overhead.

**Example:** An index with 3 primary shards distributes documents across three shards; replicas can provide redundant searchable copies on other nodes.

### Q83. What is Elasticsearch refresh?

Refresh makes recently indexed documents searchable. It is distinct from
durability/commit and has performance implications.

**Example:** A document written at 10:00:00 may not be immediately searchable until a refresh makes the new segment visible to search operations.

## 7. Backup, Recovery, HA and Scaling

### Q84. What is RPO?

Recovery Point Objective: the acceptable amount of data loss, expressed
as a time window. Example: RPO=5 minutes.

### Q85. What is RTO?

Recovery Time Objective: the target time to restore service. Example:
RTO=30 minutes.

### Q86. RPO vs RTO?

RPO asks 'how much data can we lose?'; RTO asks 'how long can we be
unavailable?'

### Q87. Logical vs physical backup?

Logical backups represent objects/data logically and can be selective
but may restore slowly at scale. Physical backups copy database storage
through database-specific physical mechanisms and are often efficient
for large recovery operations.

### Q88. Why test restores?

Because a backup file alone does not prove recoverability. Restore it,
validate data/schema/application behavior, and measure recovery time.

### Q89. What is point-in-time recovery?

Recovery to a chosen time using a base backup plus change logs such as
PostgreSQL WAL or MySQL binary logs.

### Q90. Full vs incremental vs differential backup?

Full copies the selected set; incremental copies changes since a prior
backup according to the backup system; differential copies changes since
the last full backup.

### Q91. What is HA?

High availability combines redundancy, health detection, failover,
application reconnection, consistency strategy, monitoring, and tested
recovery.

### Q92. What is failover?

Moving service responsibility from a failed component to a healthy
component, often by promoting a replica.

### Q93. What is replication lag?

Delay between source changes and their receipt/application on a replica.
It can cause stale reads and reduce failover readiness.

``` text
Backup → Restore → Validate data/schema → Validate application → Measure RTO
```

### Q94. What causes replication lag?

CPU/disk saturation, network delay, large transactions, locks, write
spikes, slow apply, or excessive replica read workload.

### Q95. What is read scaling?

Distributing read workload to replicas while accepting their consistency
and lag characteristics.

### Q96. What is write scaling?

Increasing write capacity through partitioning, sharding, application
distribution, or distributed databases. It is generally harder than read
scaling for traditional RDBMSs.

### Q97. Partitioning vs sharding?

Partitioning divides a logical dataset/table into partitions; sharding
distributes data across nodes. They solve different scaling/management
problems.

## 8. Security and Operations

### Q98. How should DB credentials be managed?

Use a secret-management system, least privilege, rotation, TLS, and
separate application accounts. Never commit passwords to Git.

### Q99. What is least privilege?

Give each user/application only the permissions required. Avoid
application use of superuser/admin accounts.

### Q100. Encryption at rest vs in transit?

At-rest encryption protects stored database files/disks/backups.
In-transit encryption such as TLS protects network traffic.

### Q101. What is SQL injection?

Untrusted input changes SQL structure. Prevent it with parameterized
queries/prepared statements, validation, and least privilege.

### Q102. What database metrics should you monitor?

CPU, memory, disk usage/latency, IOPS, throughput, connections, pool
usage, query latency/rate, errors, locks, deadlocks, replication lag,
cache behavior, WAL/binlog growth, database size, and storage growth.

### Q103. What is connection saturation?

Available connections are exhausted or nearly exhausted, causing
requests to wait/fail. Increasing max connections blindly can worsen the
problem.

### Q104. What is cache hit ratio?

A measure of how often data is served from cache/buffer rather than
slower storage. It is useful but must be interpreted with latency and
workload metrics.

### Q105. What is disk pressure?

High or rapidly growing disk usage caused by data, WAL/binlog, logs,
backups, bloat, compaction, tombstones, or stalled replication.

### Q106. What happens when a DB runs out of disk?

Writes can fail, logs cannot grow, replication can stop, and recovery
can become difficult. Never blindly delete database files.

### Q107. What is a connection leak?

Application code acquires connections but fails to return/close them,
eventually exhausting the pool or database connection capacity.

## 9. Production Scenarios and Brain Teasers

### Q108. Application latency increases but DB CPU is low. What do you check?

Query latency, locks, connection-pool waits, disk latency, network
latency, replication lag, wait events, application timeouts, and recent
changes. Low CPU does not prove the DB is healthy.

### Q109. Database CPU is 95%. What is your first approach?

Identify top SQL/processes first; inspect execution plans, indexes,
statistics, locks, concurrency, and recent deployments. Do not restart
or add CPU without evidence.

### Q110. Scylla latency is high but average node CPU is 40%. Why?

Average CPU can hide a hot shard or partition. Check shard-level CPU,
partition distribution, disk latency, compaction, tombstones, and
workload distribution.

### Q111. PostgreSQL replicas are minutes behind. What do you investigate?

Primary write/WAL rate, replica CPU/disk/network, WAL receive/replay
position, long-running queries, replication slots, and replica workload.

### Q112. MySQL replication is broken. What do you check?

Replica status, I/O and SQL/apply errors, source connectivity, binlog
availability, GTID state, replication threads, and data consistency
before deciding on remediation.

### Q113. 200 EC2 instances each have a DB pool of 50. Is it safe?

Potentially not: 200×50=10,000 possible connections. Size pools against
DB capacity, workload, latency, autoscaling, and connection limits.

### Q114. Why can increasing DB connections make performance worse?

More connections consume memory and CPU and increase contention/locking.
Higher concurrency can increase latency instead of throughput.

### Q115. Cache returns stale data. Is Redis broken?

Not necessarily. Check TTL, invalidation, cache-aside logic, write
ordering, race conditions, and whether reads are routed to stale
replicas.

### Q116. Why is a replica not automatically a backup?

Bad deletes/updates can replicate too. Replicas improve availability;
independent backups provide recovery points.

### Q117. DB is at 90% disk. Is that immediately an outage?

It is a warning. Check what consumes space and the growth rate: data,
WAL/binlog, backups, logs, bloat, compaction, tombstones, or replication
retention.

### Q118. Can an index solve every slow query?

No. Slow queries can result from SQL, plans, statistics, locks, I/O,
memory, result size, network, data distribution, or concurrency.

### Q119. Query returns no rows after deployment although data exists. What do you check?

Query change, schema, environment/database, credentials,
filters/collation, replica lag, transaction visibility, configuration,
and feature flags.

### Q120. What is the most important database production principle?

Measure first, change second. Observe the symptom, gather evidence,
identify the bottleneck, mitigate safely, validate, find root cause, and
prevent recurrence.

## 10. Frequently Used Database Commands

## PostgreSQL

``` bash
psql -h <host> -U <user> -d <database>
```

``` sql
SELECT version();
SELECT * FROM pg_stat_activity;
SELECT pg_size_pretty(pg_database_size(current_database()));
```

## MySQL

``` bash
mysql -h <host> -u <user> -p
```

``` sql
SELECT VERSION();
SHOW FULL PROCESSLIST;
SHOW VARIABLES LIKE 'max_connections';
```

## ScyllaDB / CQL

``` bash
cqlsh <host> 9042
```

``` sql
DESCRIBE KEYSPACES;
USE employee;
SELECT * FROM employee_info WHERE employee_id = '100';
```

## 11. Quick Revision Table

| Concept | Remember |
|---|---|
| Primary key | Uniquely identifies a row |
| Index | Speeds suitable reads; adds write/storage cost |
| ACID | Atomicity, Consistency, Isolation, Durability |
| MVCC | Multiple row versions/visibility for concurrency |
| Blocking | Waiting for a conflicting lock |
| Deadlock | Cyclic waiting; DB normally aborts one transaction |
| Connection pool | Reuses and limits DB connections |
| PostgreSQL VACUUM | MVCC/storage maintenance |
| PostgreSQL WAL | Recovery and replication log |
| MySQL binlog | Common replication/recovery log |
| Scylla partition key | Determines data distribution |
| Scylla clustering key | Orders rows within a partition |
| Scylla RF | Replica count |
| Hot partition | Uneven traffic/data distribution |
| Compaction | Merges LSM/SSTable data |
| Redis TTL | Expiration time |
| MongoDB replica set | HA topology |
| Elasticsearch shard | Distributed unit of an index |
| RPO | Acceptable data-loss window |
| RTO | Target recovery time |
| Replication | Availability/data-copy mechanism, not a backup replacement |

## 12. L2 Interview Rule

``` text
Symptom → Metrics/evidence → Database state → Query/workload analysis → Root cause → Safe mitigation → Validation → Prevention
```
