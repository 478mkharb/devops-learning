# 04 — VictoriaMetrics Storage

## 1. What Are We Trying to Understand?

> **What happens after a metric reaches VictoriaMetrics?**

The simplified answer is:

```text
Metric + Labels
       ↓
Identify the time series
       ↓
TSID
       ↓
Index
       ↓
Samples
       ↓
Blocks
       ↓
Parts
       ↓
Compression
       ↓
Background merge
       ↓
Disk
```

VictoriaMetrics buffers recently ingested data in memory, writes it into storage parts, organizes samples into blocks, maintains an inverted `indexDB` for finding matching series, and periodically merges parts in the background. [Official documentation](https://docs.victoriametrics.com/victoriametrics/)

---

# 2. Why Is Time-Series Storage Different?

A normal database may store records such as:

```text
employee_id | name   | department
------------+--------+-----------
101         | Rahul  | IT
102         | Priya  | HR
103         | Amit   | Finance
```

Monitoring data behaves differently.

A metric is repeatedly sampled:

```text
10:00:00 → 100
10:00:15 → 105
10:00:30 → 109
10:00:45 → 112
```

And there may be millions of different time series:

```text
CPU
Memory
HTTP requests
HTTP errors
Latency
Network
Kubernetes metrics
Application metrics
...
```

Therefore, a time-series database has to optimize for:

```text
Continuous writes
+
Time-based data
+
Large numbers of series
+
Fast time-range queries
+
Efficient disk usage
```

VictoriaMetrics is specifically designed around this workload.

---

# 3. Start With One Metric

Consider:

```text
http_requests_total{
    service="employee-api",
    method="GET",
    status="200"
} 1532
```

A sample can be thought of as:

```text
Metric identity
       +
Timestamp
       +
Value
```

The metric identity is:

```text
http_requests_total{
    service="employee-api",
    method="GET",
    status="200"
}
```

This complete metric-and-label combination identifies one time series.

Then we may receive:

```text
10:00:00 → 1532
10:00:15 → 1541
10:00:30 → 1550
10:00:45 → 1557
```

So conceptually:

```text
One Time Series
       |
       +---- Timestamp 1 → Value 1
       +---- Timestamp 2 → Value 2
       +---- Timestamp 3 → Value 3
       +---- Timestamp 4 → Value 4
```

---

# 4. Metric Identity and TSID

VictoriaMetrics internally identifies time series using a **TSID (time series ID)**.

Conceptually:

```text
Metric + Labels
      |
      v
    TSID
```

For example:

```text
http_requests_total{
    service="employee-api",
    method="GET",
    status="200"
}
```

might internally correspond to:

```text
TSID = 42
```

The exact numeric value is an internal implementation detail.

The important point is:

> **TSID is an internal identifier for a time series.**

VictoriaMetrics documentation states that raw samples are sorted by TSID and that TSID is used internally; it is not exposed to clients. [Official documentation](https://docs.victoriametrics.com/victoriametrics/)

---

# 5. Why Use a TSID?

Imagine a series produces thousands of samples.

Without an internal series identifier, conceptually every sample would repeatedly need to be associated with the full metric identity:

```text
Metric + Labels + Timestamp + Value
Metric + Labels + Timestamp + Value
Metric + Labels + Timestamp + Value
...
```

Instead, the storage layer can conceptually work with:

```text
TSID = 42

42 → timestamp → value
42 → timestamp → value
42 → timestamp → value
42 → timestamp → value
```

This separates:

```text
Series identity
```

from:

```text
Samples belonging to that series
```

This is one of the foundational ideas behind VictoriaMetrics' storage layout.

---

# 6. TSID Is Not a User-Facing Metric Identifier

Do not confuse:

```text
TSID
```

with:

```text
metric name
```

or:

```text
label set
```

For example:

```text
http_requests_total{
    service="employee-api",
    status="200"
}
```

is the user-visible metric identity.

Internally:

```text
Metric + Labels
       ↓
     TSID 42
```

The client normally never sees:

```text
TSID=42
```

VictoriaMetrics explicitly documents TSID as an internal-only identifier. [Official documentation](https://docs.victoriametrics.com/victoriametrics/)

---

# 7. Why Do We Need an Index?

Now suppose Grafana asks:

```promql
http_requests_total{
    service="employee-api",
    status="500"
}
```

VictoriaMetrics cannot efficiently scan every stored sample looking for matching labels.

Instead, it needs an index that can answer:

> Which time series match these metric and label selectors?

That is the job of **indexDB**.

Conceptually:

```text
Query selector
      |
      v
    indexDB
      |
      v
Matching TSIDs
```

For example:

```text
service="employee-api"
        |
        +----> TSID 42
        +----> TSID 73
        +----> TSID 99
```

VictoriaMetrics uses an inverted index to map metric names, label names, and label values to the corresponding TSIDs. [Official documentation](https://docs.victoriametrics.com/victoriametrics/)

---

# 8. What Is an Inverted Index?

The word **inverted** can sound confusing.

Think of it as:

```text
Label value
    ↓
Which series contain this value?
```

For example:

```text
status="500"
     |
     +----> TSID 42
     +----> TSID 73
     +----> TSID 91
```

Another lookup:

```text
service="employee-api"
     |
     +----> TSID 42
     +----> TSID 73
     +----> TSID 99
```

The query can then intersect the matching series.

Conceptually:

```text
service="employee-api"
        ↓
{42, 73, 99}

status="500"
        ↓
{42, 73, 91}

Intersection
        ↓
{42, 73}
```

The exact query execution is more involved, but this is the right mental model for understanding why an inverted index exists.

---

# 9. Global Index and Per-Day Index

VictoriaMetrics maintains two types of inverted indexes:

```text
Global index
+
Per-day index
```

Both contain mappings useful for finding series, but they are used differently depending on the query time range.

### Global index

Conceptually:

```text
Partition
   |
   v
Global index
   |
   v
Mappings across the partition
```

### Per-day index

Conceptually:

```text
Partition
   |
   +---- Day 1 index
   +---- Day 2 index
   +---- Day 3 index
```

For shorter query ranges, the per-day index can reduce the amount of index data that needs to be searched.

VictoriaMetrics chooses between the per-day and global index based on the query time range. [Official documentation](https://docs.victoriametrics.com/victoriametrics/)

---

# 10. Why Can IndexDB Become Large?

Indexing makes queries faster, but the index itself consumes storage.

VictoriaMetrics stores index information for labels and registered time series.

High churn can therefore make the index significantly larger.

For example:

```text
Series A created
Series A disappears

Series B created
Series B disappears

Series C created
Series C disappears

...
```

Even if the number of currently active series is not enormous, many different historical series identities may have been registered.

VictoriaMetrics documents that `indexdb` can become larger than the data directory under high churn. [Official FAQ](https://docs.victoriametrics.com/victoriametrics/faq/)

This gives us an important relationship:

```text
High churn
    ↓
Many historical series identities
    ↓
More index entries
    ↓
Larger indexDB
```

---

# 11. Data Partitions

VictoriaMetrics organizes stored data into **time-based partitions**.

For single-node storage, data is split into per-month partitions under the storage directory.

Conceptually:

```text
Storage
  |
  +-- 2026_07
  |
  +-- 2026_08
  |
  +-- 2026_09
```

Each partition contains data parts and its associated index information.

Time partitioning helps with:

```text
Retention
Index management
Storage organization
Background maintenance
```

VictoriaMetrics documents monthly partitions for its standard storage layout. [Official documentation](https://docs.victoriametrics.com/victoriametrics/)

---

# 12. Where Does Fresh Data Go?

When metrics arrive, VictoriaMetrics does not immediately create a final optimized disk layout for every sample.

The simplified process is:

```text
Incoming samples
       ↓
Memory buffer
       ↓
In-memory parts
       ↓
Disk parts
```

VictoriaMetrics buffers ingested data in memory for a short period, writes it into in-memory parts, and periodically persists those parts to disk. [Official documentation](https://docs.victoriametrics.com/victoriametrics/)

---

# 13. Why Buffer Data in Memory?

Suppose metrics arrive continuously:

```text
Sample
Sample
Sample
Sample
Sample
...
```

Writing every individual sample directly to disk would cause excessive I/O.

Instead:

```text
Incoming samples
       ↓
Memory buffer
       ↓
Group samples
       ↓
Write larger unit
       ↓
Disk
```

This improves write efficiency.

The flush interval can be configured using:

```text
-inmemoryDataFlushInterval
```

However, setting the flush interval too aggressively can significantly increase disk I/O. [Official documentation](https://docs.victoriametrics.com/victoriametrics/)

---

# 14. What Is a Storage Part?

A **part** is a unit of stored time-series data.

Conceptually:

```text
Storage
   |
   +-- Part A
   +-- Part B
   +-- Part C
```

Each part contains:

```text
Blocks
+
Indexes for those blocks
+
Metadata
```

A part also has metadata such as:

```text
RowsCount
BlocksCount
MinTimestamp
MaxTimestamp
MinDedupInterval
```

VictoriaMetrics stores this information in `metadata.json` inside each part directory. [Official documentation](https://docs.victoriametrics.com/victoriametrics/)

---

# 15. What Is a Block?

A block is a smaller unit inside a part.

For a single time series, imagine:

```text
TSID = 42

Block 1
---------
10:00 → 100
10:15 → 105
10:30 → 109

Block 2
---------
10:45 → 112
11:00 → 118
11:15 → 120
```

VictoriaMetrics documents that each block contains up to **8K raw samples** belonging to a single time series, with samples sorted by timestamp. [Official documentation](https://docs.victoriametrics.com/victoriametrics/)

The exact number of samples in a block may be smaller than the maximum.

---

# 16. Why Divide Data Into Blocks?

Suppose a query asks for:

```text
10:00 → 10:30
```

and the database has months of data.

It should not need to read every sample from every time series.

Instead:

```text
Query time range
       ↓
Find matching TSIDs
       ↓
Find relevant blocks
       ↓
Read those blocks
```

So blocks provide a useful unit for:

```text
Storage
Compression
Indexing
Query access
```

---

# 17. Timestamps and Values Are Stored Separately

VictoriaMetrics stores compressed timestamps and values in separate files inside a part:

```text
timestamps.bin
values.bin
```

Conceptually:

```text
Part
 |
 +-- timestamps.bin
 |
 +-- values.bin
 |
 +-- index.bin
 |
 +-- metaindex.bin
 |
 +-- metadata.json
```

The exact filesystem layout is an implementation detail and can evolve, but these names are useful when learning the current storage structure.

VictoriaMetrics documents `timestamps.bin` and `values.bin` for compressed timestamp/value storage, plus `index.bin` and `metaindex.bin` for fast block lookup. [Official documentation](https://docs.victoriametrics.com/victoriametrics/)

---

# 18. Column-Oriented Thinking

A useful conceptual way to understand the layout is to separate similar types of information.

Instead of thinking:

```text
TSID | Timestamp | Value
42   | T1        | V1
42   | T2        | V2
42   | T3        | V3
```

think:

```text
TSID:
42 42 42

Timestamps:
T1 T2 T3

Values:
V1 V2 V3
```

This is useful because timestamps have timestamp-like patterns and values may have numerical patterns.

### Important precision

For learning, it is useful to describe this as **column-oriented/columnar-style organization of timestamp and value data**.

However, don't memorize "VictoriaMetrics is just a generic column-store" as the full storage model. Its actual storage engine is built around:

```text
TSID
+
blocks
+
parts
+
indexes
+
compressed timestamps/values
```

That is the more accurate mental model.

---

# 19. Compression

Compression is one of the most important reasons VictoriaMetrics can store large amounts of metric data efficiently.

Consider timestamps:

```text
10:00:00
10:00:15
10:00:30
10:00:45
10:01:00
```

There is a predictable interval:

```text
+15 seconds
+15 seconds
+15 seconds
+15 seconds
```

Instead of treating every timestamp as completely independent information, compression can exploit the structure.

Values may also show patterns:

```text
100
101
101
102
103
104
```

or:

```text
1000
1000
1000
1001
1000
1000
```

These patterns can often compress well.

VictoriaMetrics stores timestamps and values in compressed form. [Official documentation](https://docs.victoriametrics.com/victoriametrics/)

---

# 20. Why Compression Matters

Suppose raw data requires:

```text
100 GB
```

After compression:

```text
30 GB
```

The exact ratio depends heavily on the workload.

A useful conceptual relationship is:

```text
Better compression
       ↓
Less disk I/O
       ↓
Less storage required
       ↓
More data fits in cache/storage
```

VictoriaMetrics' sizing documentation notes that average sample storage can be around or below one byte per sample after compression for some workloads, but actual usage varies and high churn can reduce compression efficiency. [Official documentation](https://docs.victoriametrics.com/guides/understand-your-setup-size/)

Do not treat "1 byte per sample" as a universal fixed storage cost.

---

# 21. High Churn Can Hurt Compression

Recall:

```text
Cardinality
=
How many unique series?

Churn
=
How frequently new series are created?
```

High churn can reduce compression efficiency.

Why?

Stable series provide more opportunities to store and compress similar data together.

With high churn:

```text
New series
New labels
New TSIDs
New blocks
New index entries
```

This can increase storage overhead.

VictoriaMetrics explicitly notes that high churn can negatively affect compression efficiency. [Official documentation](https://docs.victoriametrics.com/guides/understand-your-setup-size/)

This connects Part 4 back to Part 5:

```text
Cardinality / Churn
        ↓
Storage behavior
        ↓
Index size
        ↓
Compression efficiency
        ↓
Disk usage
```

---

# 22. Index Files

A part can contain files such as:

```text
metadata.json
timestamps.bin
values.bin
index.bin
metaindex.bin
```

Conceptually:

```text
                  Part
                   |
       +-----------+-----------+
       |           |           |
       v           v           v
   metadata     indexes     data
                            |
                      +-----+-----+
                      |           |
                      v           v
               timestamps       values
```

### metadata.json

Describes metadata about the part, such as:

```text
RowsCount
BlocksCount
MinTimestamp
MaxTimestamp
```

### index.bin

Helps locate blocks associated with TSIDs and time ranges.

### metaindex.bin

Provides higher-level information that helps locate relevant index data efficiently.

### timestamps.bin

Contains compressed timestamps.

### values.bin

Contains compressed values.

These are implementation details of the current storage format, not a public API contract. [Official documentation](https://docs.victoriametrics.com/victoriametrics/)

---

# 23. Querying the Index

Suppose the query is:

```promql
rate(
  http_requests_total{
    service="employee-api",
    status="500"
  }[5m]
)
```

The first problem is:

> Which time series match these selectors?

Conceptually:

```text
service="employee-api"
        ↓
indexDB
        ↓
TSIDs

status="500"
        ↓
indexDB
        ↓
TSIDs
```

The matching sets are combined.

Then:

```text
Matching TSIDs
      ↓
Find relevant blocks
      ↓
Read timestamps + values
```

This is why indexing is so important.

---

# 24. Query Path — Detailed Mental Model

A simplified query path is:

```text
Grafana
   |
   | MetricsQL / PromQL-like query
   v
VictoriaMetrics
   |
   v
Find matching metric/labels
   |
   v
indexDB
   |
   v
Matching TSIDs
   |
   v
Find relevant blocks
   |
   v
Read compressed timestamps
   +
Read compressed values
   |
   v
Decompress
   |
   v
Execute query functions
   |
   v
Return result
```

For a query such as:

```promql
rate(http_requests_total[5m])
```

the engine still needs the underlying samples before it can calculate the rate.

The detailed query engine and execution behavior will be covered in:

**Part 6 — Query Engine**

---

# 25. Background Merging

Incoming data creates parts.

For example:

```text
Part A
Part B
Part C
Part D
```

If these remain separate forever, queries may need to inspect many different parts.

VictoriaMetrics therefore performs **background merges**.

Conceptually:

```text
Part A ----\
Part B -----+----> Merge ----> Part E
Part C -----/
Part D ----/
```

The new part is generally larger and more optimized.

VictoriaMetrics performs these compactions independently on its time partitions. [Official documentation](https://docs.victoriametrics.com/victoriametrics/)

---

# 26. Why Merge Parts?

Background merging has several important purposes.

### 1. Fewer files

```text
Many small parts
       ↓
Fewer larger parts
```

### 2. Better compression

Larger parts can often compress more efficiently.

### 3. Faster queries

Queries generally have fewer parts to inspect.

### 4. Background maintenance

Merges also perform maintenance work such as:

```text
Deduplication
Downsampling where configured
Freeing disk space for deleted series
```

VictoriaMetrics explicitly documents these benefits of background merges. [Official documentation](https://docs.victoriametrics.com/victoriametrics/)

---

# 27. Why Can Too Many Parts Be a Problem?

Imagine:

```text
1,000 small parts
```

A query may need to inspect many separate structures.

Compared with:

```text
50 larger parts
```

the second situation can be more efficient.

Therefore:

```text
Too many parts
      ↓
More files
      ↓
More work during queries
      ↓
More overhead
```

This is one reason background merging is important.

---

# 28. Merging Needs Free Disk Space

Merging is not free.

Imagine:

```text
Part A = 10 GB
Part B = 10 GB
```

To produce a merged part, VictoriaMetrics needs enough free disk space to safely write the new result before old parts are removed.

Therefore, keeping the disk almost completely full can interfere with merging.

VictoriaMetrics recommends keeping at least **20% free storage space** in the storage directory. [Official documentation](https://docs.victoriametrics.com/victoriametrics/)

Conceptually:

```text
Disk
+-----------------------------+
| Existing data               |
|                             |
|                             |
|       Free space            |
|       >= ~20% recommended   |
+-----------------------------+
```

This is an important operational consequence of understanding the storage engine.

---

# 29. Atomic Part Registration

VictoriaMetrics takes care to avoid leaving partially registered parts as valid storage.

Conceptually:

```text
Write new part
     ↓
Finish writing
     ↓
fsync
     ↓
Register part
```

If the process or machine fails before the part is safely registered, the incomplete part can be cleaned up.

VictoriaMetrics documents atomic registration through `parts.json`, including similar behavior for merges. [Official documentation](https://docs.victoriametrics.com/victoriametrics/)

The important learning point is:

> A part is not treated as valid storage simply because some files were created.

---

# 30. Retention

Monitoring data cannot normally be stored forever on finite disks.

VictoriaMetrics supports retention through:

```text
-retentionPeriod
```

For example:

```text
-retentionPeriod=30d
```

means approximately:

```text
Keep data for 30 days
        ↓
Older data becomes eligible for deletion
```

The default retention for VictoriaMetrics single-node and `vmstorage` is one month. [Official documentation](https://docs.victoriametrics.com/victoriametrics/)

---

# 31. How Retention Relates to Partitions

Suppose:

```text
Retention = 30 days
```

and the storage contains monthly partitions:

```text
2026_07
2026_08
2026_09
```

As data becomes older than the retention period:

```text
Old partition
      ↓
Outside retention
      ↓
Deleted eventually
```

Retention is therefore closely connected to the time-partitioned storage layout.

VictoriaMetrics documents that data partitions outside the configured retention are deleted as part of the retention process. [Official documentation](https://docs.victoriametrics.com/victoriametrics/)

---

# 32. Retention Is Not the Same as Instant Deletion

This distinction is important.

If data becomes older than the retention boundary, do not assume every byte disappears immediately.

VictoriaMetrics documents that data outside retention is deleted eventually through its partition/part lifecycle and background merge behavior.

So think:

```text
Data becomes too old
       ↓
Eligible for removal
       ↓
Storage maintenance
       ↓
Space eventually reclaimed
```

This is different from:

```text
Timestamp crosses boundary
       ↓
Instantly delete every byte
```

---

# 33. Why Retention Saves Storage

Suppose:

```text
Ingestion = 1 TB/month
```

With:

```text
30 days retention
```

the system stores roughly one month of historical data plus the additional storage overhead required by partitions, indexes, merges, and free-space requirements.

If retention becomes:

```text
180 days
```

the storage requirement increases substantially.

Conceptually:

```text
More retention
      ↓
More historical samples
      ↓
More disk required
```

VictoriaMetrics' sizing guidance explicitly factors retention period into disk capacity planning. [Official documentation](https://docs.victoriametrics.com/guides/understand-your-setup-size/)

---

# 34. Storage Size Is Not Just Raw Samples

A common beginner mistake is:

```text
Storage needed
=
samples × value size
```

The actual system also needs space for:

```text
Compressed sample data
+
IndexDB
+
Parts
+
Temporary merge requirements
+
Metadata
+
Free space
```

A better conceptual model is:

```text
Total disk
=
Data
+
Index
+
Operational overhead
+
Merge headroom
```

VictoriaMetrics' capacity guidance explicitly recommends accounting for index size and keeping free disk space for merges. [Official documentation](https://docs.victoriametrics.com/guides/understand-your-setup-size/)

---

# 35. Index Size and Churn

VictoriaMetrics provides an important real-world observation:

```text
High churn
     ↓
Large indexDB
```

In high-churn workloads, indexDB can sometimes become larger than the actual compressed metric data.

Why?

Because the index stores information about many registered series and label values.

So:

```text
Low churn
    ↓
Stable series identities
    ↓
Smaller index overhead

High churn
    ↓
Many changing identities
    ↓
More index entries
    ↓
Potentially much larger indexDB
```

This is why cardinality and churn are not just "metrics theory"; they directly affect storage.

---

# 36. Caching

VictoriaMetrics also uses caching to reduce repeated expensive work.

At a high level:

```text
Query
  |
  v
Cache?
 /   \
Yes   No
 |     |
 v     v
Return  Read/process
        |
        v
      Cache
```

Caching can help with:

```text
Repeated queries
Index lookups
Frequently accessed data
```

The exact cache types and internal behavior depend on the VictoriaMetrics version and component.

For this fundamentals README, remember:

> **Caching reduces repeated disk or computation work by keeping useful information in memory.**

---

# 37. Why RAM Matters to Storage

It is tempting to think:

```text
VictoriaMetrics storage
=
Disk only
```

That is incorrect.

The storage engine also uses RAM for:

```text
Recent data
Indexes
Caches
Metadata
Query processing
```

VictoriaMetrics recommends keeping substantial free RAM to reduce the probability of OOM conditions and excessive cache eviction/I/O. [Official documentation](https://docs.victoriametrics.com/victoriametrics/)

So the storage architecture is:

```text
               VictoriaMetrics
                     |
          +----------+----------+
          |                     |
         RAM                   Disk
          |                     |
     Buffers/Caches        Parts/Indexes
     Query state           Compressed data
```

---

# 38. The Complete Storage Flow

Now combine everything.

```text
                Incoming Metric
                       |
                       v
              Metric + Labels
                       |
                       v
                    TSID
                       |
              +--------+--------+
              |                 |
              v                 v
           indexDB          Samples
              |                 |
              |            Timestamp
              |                 +
              |              Value
              |                 |
              |                 v
              |              Blocks
              |                 |
              |                 v
              |               Parts
              |                 |
              |            Compression
              |                 |
              |                 v
              |                Disk
              |
              +------> Fast lookup
```

Then:

```text
Many Parts
    ↓
Background Merge
    ↓
Fewer / larger optimized Parts
    ↓
Better compression
    ↓
Better query efficiency
```

---

# 39. Complete Query Flow

When Grafana sends:

```promql
rate(
  http_requests_total{
    service="employee-api"
  }[5m]
)
```

the conceptual flow is:

```text
Grafana
   |
   v
Query
   |
   v
Find matching metric/labels
   |
   v
indexDB
   |
   v
Matching TSIDs
   |
   v
Find relevant blocks
   |
   v
Read compressed timestamps
+
Read compressed values
   |
   v
Decompress
   |
   v
Calculate rate()
   |
   v
Return result
```

Part 6 will explain the query engine in much more detail.

---

# 40. Complete Write Flow

The write side can be visualized as:

```text
Application
    |
    v
Prometheus / vmagent
    |
    | Remote Write
    v
VictoriaMetrics / vminsert
    |
    v
Ingestion
    |
    v
Memory buffer
    |
    v
In-memory part
    |
    v
Disk part
    |
    v
Blocks
    |
    v
Compressed timestamps + values
    |
    v
Background merge
    |
    v
Optimized storage
```

The exact path differs between single-node and cluster deployments, but the storage concepts remain similar at the `vmstorage` layer.

---

# 41. How Storage Connects to Architecture

From Part 2:

```text
vmagent
    |
    v
vminsert
    |
    v
vmstorage
```

Now we can expand `vmstorage`:

```text
vminsert
    |
    v
vmstorage
    |
    +--> Memory buffer
    |
    +--> Parts
    |      |
    |      +--> Blocks
    |      +--> timestamps
    |      +--> values
    |      +--> indexes
    |
    +--> indexDB
    |
    +--> Background merge
    |
    +--> Retention
    |
    v
   Disk
```

This is the bridge between:

```text
Part 2 — Architecture
```

and:

```text
Part 4 — Storage
```

---

# 42. Storage Efficiency — The Big Picture

VictoriaMetrics' storage efficiency comes from multiple mechanisms working together.

```text
Efficient storage
       |
       +--> TSID-based organization
       |
       +--> Efficient indexes
       |
       +--> Blocks
       |
       +--> Compression
       |
       +--> Parts
       |
       +--> Background merges
       |
       +--> Caching
       |
       +--> Time partitioning
       |
       +--> Retention
```

Do not attribute storage efficiency to a single feature.

It is the combination that matters.

---

# 43. Capacity Planning Connection

Storage planning depends on more than disk size.

Important variables include:

```text
Ingestion rate
Active time series
Cardinality
Churn
Retention
Replication
Query workload
Index size
Compression efficiency
```

VictoriaMetrics' sizing guidance provides a formula based on bytes per sample, ingestion rate, replication factor, retention, and a free-space factor. It also notes that index size is workload-dependent and can become particularly large under high churn. [Official documentation](https://docs.victoriametrics.com/guides/understand-your-setup-size/)

A simplified mental model is:

```text
More samples
      ↓
More data

More retention
      ↓
More historical data

More cardinality
      ↓
More series/index entries

More churn
      ↓
More index overhead
      ↓
Potentially worse compression

Replication
      ↓
More copies
      ↓
More storage
```

---

# 44. Example: Employee API

Suppose:

```text
employee-api
```

exports:

```text
http_requests_total{
    service="employee-api",
    method="GET",
    status="200"
}
```

At 15-second intervals:

```text
10:00 → 1532
10:15 → 1538
10:30 → 1546
10:45 → 1554
```

Conceptually:

```text
Metric + Labels
      ↓
TSID = 42
      ↓
Timestamp + Value
      ↓
Blocks
      ↓
Part
      ↓
Compression
      ↓
Disk
```

Meanwhile the index provides:

```text
service="employee-api"
        ↓
TSID 42
```

So when Grafana asks for:

```promql
http_requests_total{service="employee-api"}
```

the index helps find the relevant series, and the storage blocks provide the actual samples.

---

# 45. Example: High-Churn Kubernetes Pods

Suppose a Kubernetes deployment repeatedly creates Pods:

```text
employee-api-abc123
employee-api-def456
employee-api-ghi789
...
```

If the Pod name becomes part of metric identity:

```text
pod="employee-api-abc123"
pod="employee-api-def456"
pod="employee-api-ghi789"
```

then many unique series may be created.

Conceptually:

```text
Pod churn
   ↓
Series churn
   ↓
More TSIDs
   ↓
More index entries
   ↓
Potentially larger indexDB
   ↓
More storage pressure
```

This is why Part 5 — Cardinality & Churn — is directly connected to storage.

---

# 46. Important Storage Distinctions

## TSID vs IndexDB

```text
TSID
=
Internal identity of one time series

indexDB
=
Lookup structure that maps metric/labels to TSIDs
```

---

## Part vs Block

```text
Part
=
Larger storage unit

Block
=
Smaller unit inside a part
```

Conceptually:

```text
Part
 |
 +-- Block
 +-- Block
 +-- Block
```

---

## Data vs Index

```text
Data
=
timestamps + values

Index
=
information used to find relevant series/blocks
```

---

## Compression vs Merging

```text
Compression
=
Represent data using fewer bytes

Merging
=
Combine parts and perform storage maintenance
```

Merging can also improve compression.

---

## Retention vs Merge

```text
Retention
=
How long data should be kept

Merge
=
How stored parts are compacted/maintained
```

They are related but not the same mechanism.

---

# 47. Interview Questions

## Q1. How does VictoriaMetrics store time-series data efficiently?

> VictoriaMetrics uses internal TSIDs to identify time series, inverted indexes to quickly find matching series, blocks and parts to organize samples, compressed timestamps and values to reduce storage usage, and background merges to compact parts, improve compression, reduce file counts, and improve query efficiency.

---

## Q2. What is TSID?

> TSID is VictoriaMetrics' internal time-series identifier. It represents a unique metric-and-label combination and is used internally for organizing and retrieving raw samples.

---

## Q3. Is TSID exposed to clients?

> No. TSID is an internal identifier and is not exposed to clients.

---

## Q4. What is indexDB?

> indexDB is VictoriaMetrics' inverted index. It maps metric names, label names, and label values to TSIDs so the system can efficiently find time series matching a query.

---

## Q5. Why is an index necessary?

> Without an index, the storage engine would need to scan large amounts of stored data to find series matching metric and label selectors. The index allows the system to narrow the search to relevant TSIDs.

---

## Q6. What is a storage part?

> A part is a storage unit containing blocks of time-series samples and associated indexes and metadata.

---

## Q7. What is a block?

> A block is a smaller storage unit inside a part. In the documented storage format, a block contains up to 8K raw samples belonging to one time series, sorted by timestamp.

---

## Q8. Why does VictoriaMetrics use blocks?

> Blocks provide manageable units for storing, indexing, compressing, and reading time-series data during queries.

---

## Q9. Why are timestamps and values compressed?

> Time-series timestamps and values often contain patterns that can be represented efficiently, reducing disk usage and I/O.

---

## Q10. Why does VictoriaMetrics merge parts?

> Background merges reduce the number of parts, improve compression, improve query performance, and perform storage maintenance such as deduplication and freeing space from deleted data.

---

## Q11. What happens if there are too many small parts?

> Queries may need to inspect more parts, increasing query overhead. Background merging helps reduce this fragmentation.

---

## Q12. What is retention?

> Retention defines how long VictoriaMetrics keeps stored data. It is configured with `-retentionPeriod`.

---

## Q13. Does retention mean old data is deleted instantly?

> No. Data outside retention is eventually removed through VictoriaMetrics' partition and background maintenance mechanisms.

---

## Q14. Why does cardinality affect storage?

> Higher cardinality means more unique time series, which means more TSIDs and more index information, increasing memory and disk requirements.

---

## Q15. Why does churn affect indexDB?

> High churn creates many changing time-series identities, which can produce many index entries. VictoriaMetrics documents that high churn can make indexDB unexpectedly large.

---

## Q16. Why should disk space not be allowed to reach 100%?

> Background merges require free disk space to safely create merged parts. VictoriaMetrics recommends keeping at least 20% free storage space.

---

# 48. Interview Answer — Short Version

> VictoriaMetrics identifies each unique metric-and-label combination internally with a TSID. It uses an inverted index, called indexDB, to map metric names and labels to those TSIDs so queries can find matching series efficiently. Samples are organized into blocks and storage parts, with timestamps and values stored in compressed form. Parts are periodically merged in the background to reduce the number of files, improve compression and query performance, and perform maintenance. Time-based partitions and retention control how historical data is managed.

---

# 49. Revision Notes

## Storage in One Line

> **TSID identifies the series, indexDB finds the series, blocks hold samples, parts organize blocks, compression saves space, and merging keeps storage optimized.**

---

## Basic Storage Flow

```text
Metric + Labels
      ↓
     TSID
      ↓
   indexDB
      ↓
Matching Series
      ↓
   Blocks
      ↓
    Parts
      ↓
Compression
      ↓
    Disk
```

---

## TSID

```text
Metric + Labels
      ↓
    TSID
```

Remember:

> **TSID = internal identity of a time series**

---

## indexDB

```text
Metric / Label selector
        ↓
     indexDB
        ↓
      TSIDs
```

Remember:

> **indexDB = find the series**

---

## Block

```text
TSID
 ↓
Timestamp + Value
 ↓
Block
```

A documented block contains up to **8K raw samples** for one time series.

---

## Part

```text
Part
 |
 +-- Block
 +-- Block
 +-- Block
```

Remember:

> **Part = collection of blocks**

---

## Important Files

```text
timestamps.bin
values.bin
index.bin
metaindex.bin
metadata.json
```

Conceptually:

```text
metadata
    ↓
index
    ↓
blocks
 /    \
time  value
```

---

## Compression

```text
Predictable timestamps
        +
Patterned values
        ↓
Better compression
        ↓
Less disk usage
```

---

## Background Merge

```text
Part A
Part B
Part C
Part D
   ↓
 Merge
   ↓
Part E
```

Benefits:

```text
Fewer parts
Better compression
Better query efficiency
Storage maintenance
```

---

## Retention

```text
-retentionPeriod=30d
```

means approximately:

```text
Keep recent 30 days
       ↓
Older data
       ↓
Eventually removed
```

---

## Cardinality Connection

```text
More cardinality
      ↓
More TSIDs
      ↓
More index entries
      ↓
More storage pressure
```

---

## Churn Connection

```text
More churn
      ↓
More changing series identities
      ↓
More index entries
      ↓
Potentially larger indexDB
      ↓
Potentially worse compression
```

---

## Storage Mental Model

```text
             TIME SERIES
                  |
                  v
                TSID
                  |
          +-------+-------+
          |               |
          v               v
       indexDB         Samples
          |               |
          |         Timestamp + Value
          |               |
          |               v
          |             Blocks
          |               |
          |               v
          |             Parts
          |               |
          |               v
          |          Compression
          |               |
          +---------------+
                  |
                  v
                 Disk
```

---

# 50. Final Mental Model

If you are asked:

> **"How does VictoriaMetrics store metrics?"**

Think through this sequence:

```text
1. Metric + Labels
        ↓
2. Identify Time Series
        ↓
3. Assign internal TSID
        ↓
4. Use indexDB for lookup
        ↓
5. Store samples
        ↓
6. Organize samples into blocks
        ↓
7. Organize blocks into parts
        ↓
8. Compress timestamps and values
        ↓
9. Merge parts in background
        ↓
10. Apply retention over time
```

And remember the shortest version:

```text
TSID
  ↓
INDEX
  ↓
BLOCKS
  ↓
PARTS
  ↓
COMPRESSION
  ↓
MERGE
  ↓
DISK
```

That is the storage mental model you should carry into **Part 5 — Cardinality & Churn** and **Part 6 — Query Engine**.
