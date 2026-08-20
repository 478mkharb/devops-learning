# <h1 align="center">POC - PostgreSQL and ScyllaDB Observability</h1>

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
    <td align="center">18/08/2026</td>
    <td align="center">1.0</td>
    <td align="center">Mukesh Kharb</td>
    <td align="center">18/08/2026</td>
    <td align="center">Team</td>
    <td align="center">Mohit Kumar</td>
    <td align="center">Faisal Khan</td>
    <td align="center">Mahesh Kumar</td>
  </tr>
</table>

</div>

---

# Table of Contents

1. [Introduction](#1-introduction)
2. [Architecture](#2-architecture)
3. [Environment and Components](#3-environment-and-components)
4. [Grafana Alloy](#4-grafana-alloy)
5. [PostgreSQL Monitoring](#5-postgresql-monitoring)
6. [ScyllaDB Monitoring](#6-scylladb-monitoring)
7. [Logs and Traces](#7-logs-and-traces)
8. [Grafana Dashboards](#8-grafana-dashboards)
9. [POC Validation](#9-poc-validation)
10. [Troubleshooting](#10-troubleshooting)
11. [Conclusion](#11-conclusion)
12. [Contact Information](#12-contact-information)
13. [References](#13-references)

---

## 1. Introduction

This POC demonstrates observability for **PostgreSQL and ScyllaDB** using:

- **Prometheus** for metrics
- **Grafana Alloy + Loki** for logs
- **OpenTelemetry Collector + Tempo** for traces
- **Grafana** for visualization

Only PostgreSQL and ScyllaDB are in scope. Other application services and Redis monitoring are handled by separate tickets.

---

## 2. Architecture

PostgreSQL and ScyllaDB run as independent database services on separate servers; they are not containerized in this POC. The complete observability stack runs as containers on a **dedicated observability server**.

| Service | Private DNS | Database Port | Metrics |
|---|---|---:|---:|
| PostgreSQL | `otms.postgres.internal` | `5432` | `9187` via PostgreSQL Exporter |
| ScyllaDB | `otms.scylladb.internal` | `9042` | `9180` |


<img width="1536" height="1024" alt="ChatGPT Image Aug 21, 2026, 12_11_48 AM" src="https://github.com/user-attachments/assets/8986ea5a-a7b7-4ca3-a486-0bcea3bcf223" />


---

## 3. Environment and Components

| Component | Purpose | Port |
|---|---|---:|
| PostgreSQL Exporter | PostgreSQL metrics | `9187` |
| ScyllaDB Metrics Endpoint | ScyllaDB metrics | `9180` |
| Prometheus | Metrics collection | `9090` |
| Loki | Log storage/query | `3100` |
| Tempo | Trace storage/query | `3200` |
| Grafana | Visualization | `3001` |

Prometheus targets:

```yaml
- job_name: "postgres-exporter"
  static_configs:
    - targets:
        - otms.postgresql.internal:9187

- job_name: "scylladb"
  static_configs:
    - targets:
        - otms.scylladb.internal:9180
```
---

## 4. Grafana Alloy

**Grafana Alloy** is used as the **log collection agent** in this POC. Alloy is installed directly on the **PostgreSQL server** and **ScyllaDB server**, where it reads the local database logs and pushes them to the centralized Loki instance on the observability server.

### 4.1 Why Grafana Alloy

In this POC, Alloy is deployed as a lightweight **agent on each database server**. Its responsibility is to collect local logs, apply the required labels/pipeline processing, and push those logs to the centralized Loki server. Alloy is also a broader telemetry collection platform, but this implementation uses it specifically in its agent role for log forwarding.

For this POC, Alloy is used specifically as the **log collection agent**:

```text
PostgreSQL Server
    PostgreSQL logs
          │
          ▼
        Alloy
          │
          ▼
        Loki
          │
          ▼
       Grafana


ScyllaDB Server
    systemd / journald logs
          │
          ▼
        Alloy
          │
          ▼
        Loki
          │
          ▼
       Grafana
```

### 4.2 Alloy Agent Features Used in This POC

| Feature | Purpose |
|---|---|
| `loki.source.file` | Reads PostgreSQL log files |
| `loki.source.journal` | Reads ScyllaDB systemd/journald logs |
| `loki.write` | Sends collected logs to Loki |
| Component-based pipelines | Separates discovery, processing, and delivery |
| Labeling | Adds consistent labels such as `service_name` |
| OpenTelemetry support | Supports broader telemetry pipelines when required |
| Built-in observability | Alloy exposes its own operational status and debugging information |

### 4.3 Why Promtail Is Disregarded

Promtail is **not used in this POC** because Grafana Labs has deprecated it and it reached **End-of-Life (EOL) on March 2, 2026**. Grafana states that no future support or updates are provided for Promtail and that future feature development is in Grafana Alloy.

Alloy is preferred as the **agent** in this POC because:

- **Promtail is EOL** and is no longer a supported choice for a new implementation.
- Alloy provides the log collection and forwarding capabilities required here while also supporting broader telemetry pipelines when needed.
- Alloy can handle **logs, metrics, traces, and profiles** through one agent/collector technology, reducing the need to introduce separate telemetry agents.
- Alloy has native **OTLP/OpenTelemetry** support and Prometheus-compatible collection pipelines.
- Grafana provides an official migration path and a converter for moving Promtail configurations to Alloy.
- Using Alloy keeps this POC aligned with the current Grafana observability direction and avoids introducing an EOL component.

> **Decision:** Promtail is intentionally excluded. **Grafana Alloy is deployed as the log collection agent on the PostgreSQL and ScyllaDB servers, pushing logs to the centralized Loki server.**

### 4.4 Log Agent Flow

```text
PostgreSQL
    │
    │ PostgreSQL log file
    ▼
Alloy Agent
    │
    │ Push logs
    ▼
Loki :3100
    │
    ▼
Grafana


ScyllaDB
    │
    │ systemd / journald
    ▼
Alloy Agent
    │
    │ Push logs
    ▼
Loki :3100
    │
    ▼
Grafana
```

## 5. PostgreSQL Monitoring

### 5.1 Relevant PostgreSQL Metrics

| Metric | Description | Monitoring |
|---|---|---|
| `up{job="postgres-exporter"}` | Exporter scrape availability | `0` = Critical |
| `pg_database_size_bytes` | Database size | Monitor growth/capacity |
| `pg_stat_database_numbackends` | Active database connections | Compare with maximum connections |
| `pg_settings_max_connections` | Maximum configured connections | Capacity reference |
| `pg_locks_count` | Locks by database and lock mode | Investigate sustained increase |
| `pg_replication_lag_seconds` | Replication lag | Alert according to SLA |
| `pg_exporter_last_scrape_error` | Last exporter scrape error | `> 0` = Investigate |
| `pg_scrape_collector_success` | Exporter collector success | `0` = Investigate |

These metrics are based on the PostgreSQL exporter metrics verified during the POC.

### 5.2 Validate PostgreSQL Metrics

```bash
curl -s http://otms.postgresql.internal:9187/metrics | grep '^pg_' | head -100
```

<details>
<summary>📸 <strong>Screenshot - PostgreSQL Metrics</strong></summary>

<img width="1833" height="987" alt="image" src="https://github.com/user-attachments/assets/2a84c6b7-c141-4a19-853f-c00470d7729c" />

</details>

---

## 6. ScyllaDB Monitoring

### 6.1 Relevant ScyllaDB Metrics

The following are selected from the actual `scylla_*` metrics available in Prometheus during the POC. fileciteturn8file0

| Metric | Description | Monitoring |
|---|---|---|
| `up{job="scylladb"}` | ScyllaDB metrics target availability | `0` = Critical |
| `scylla_reactor_utilization` | Reactor/shard utilization | Warning > 0.70; Critical > 0.85 |
| `scylla_database_total_reads` | Total database reads | Monitor workload/rate |
| `scylla_database_total_writes` | Total database writes | Monitor workload/rate |
| `scylla_database_total_reads_failed` | Failed reads | Alert on sustained increase |
| `scylla_database_total_writes_failed` | Failed writes | Alert on sustained increase |
| `scylla_database_total_writes_timedout` | Timed-out writes | Alert on sustained increase |
| `scylla_database_requests_blocked_memory_current` | Requests currently blocked by memory pressure | Sustained `> 0` = Investigate |
| `scylla_compaction_manager_pending_compactions` | Pending compactions | Monitor sustained backlog |
| `scylla_compaction_manager_failed_compactions` | Failed compactions | Alert on increase |
| `scylla_memory_allocated_memory` | Memory allocated by ScyllaDB | Monitor utilization |
| `scylla_column_family_live_disk_space` | Live disk space used by column families | Monitor storage growth |
| `scylla_storage_proxy_coordinator_read_latency_bucket` | Read latency histogram | Calculate P95/P99 |
| `scylla_storage_proxy_coordinator_write_latency_bucket` | Write latency histogram | Calculate P95/P99 |

### 6.2 Validate ScyllaDB Metrics

Verify the Prometheus target:

```bash
curl -s http://localhost:9090/api/v1/targets | jq '.data.activeTargets[] | select(.labels.job=="scylladb") | {
    job: .labels.job,
    instance: .labels.instance,
    health: .health,
    scrapeUrl: .scrapeUrl,
    lastError: .lastError
}'
```

Validate a ScyllaDB metric:

```bash
curl -s 'http://localhost:9090/api/v1/query?query=scylla_reactor_utilization' | jq .
```

<details>
<summary>📸 <strong>Screenshot - ScyllaDB Metrics</strong></summary>

<img width="1852" height="548" alt="image" src="https://github.com/user-attachments/assets/e2bf2d36-ec0d-428f-b599-e2acaa4f562e" />

<img width="1852" height="933" alt="image" src="https://github.com/user-attachments/assets/7a4eabae-0884-45a4-972c-98b70bad10f2" />

</details>

---

## 7. Logs and Traces

### 7.1 PostgreSQL Logs

```logql
{service_name="postgres"}
```

Validation:

```bash
curl -sG 'http://localhost:3100/loki/api/v1/query_range' \
  --data-urlencode 'query={service_name="postgres"}' \
  --data-urlencode 'limit=10' | jq
```
> [!NOTE]
> `-sG` = silent GET request, while `--data-urlencode` safely encodes Loki/LogQL query parameters.

<details>
<summary>📸 <strong>Screenshot - PostgreSQL Logs</strong></summary>
<img width="1840" height="882" alt="image" src="https://github.com/user-attachments/assets/c75e1cb4-a1d5-4472-942b-39db06312ddc" />

</details>

### 7.2 ScyllaDB Logs

```logql
{service_name="scylladb"}
```

Validation:

```bash
curl -sG 'http://localhost:3100/loki/api/v1/query_range' \
  --data-urlencode 'query={service_name="scylladb"}' \
  --data-urlencode 'limit=10' \
| jq -r '.data.result[]?.values[]? | .[1]'
```

<details>
<summary>📸 <strong>Screenshot - ScyllaDB Logs</strong></summary>

<img width="1840" height="882" alt="image" src="https://github.com/user-attachments/assets/b7085b41-9c4e-4f45-bfb3-97d25ba5010c" />

</details>

### 7.3 PostgreSQL Database Trace

Database spans appear under the **Attendance API trace**, not as a separate PostgreSQL trace service.

```traceql
{ resource.service.name = "attendance-api" && span.db.system = "postgresql" }
```

Validated structure:

```text
POST /api/v1/attendance/create
└── INSERT
    ├── db.system = postgresql
    ├── db.name = attendance_db
    ├── db.user = postgres
    ├── net.peer.name = postgres
    └── net.peer.port = 5432
```

<details>
<summary>📸 <strong>Screenshot - PostgreSQL Database Trace</strong></summary>

<img width="1840" height="882" alt="image" src="https://github.com/user-attachments/assets/6cb8c72a-4a89-4a53-9756-5f1d5bfa064c" />
<img width="1840" height="882" alt="image" src="https://github.com/user-attachments/assets/957acfab-63fb-4003-85d1-62e0d4b28961" />

</details>

### 7.4 ScyllaDB Database Trace

ScyllaDB database operations appear under the **Salary API trace**. Do not search for `service.name=scylladb`.

```traceql
{ resource.service.name = "salary-api" && name =~ ".*INSERT.*" }
```

<details>
<summary>📸 <strong>Screenshot - ScyllaDB Database Trace</strong></summary>
<img width="1840" height="882" alt="image" src="https://github.com/user-attachments/assets/adbe3978-4a17-4a8a-9bad-fc69404a9030" />
<img width="1840" height="882" alt="image" src="https://github.com/user-attachments/assets/72c8f204-50de-4e42-99af-c8d8340644fd" />

</details>

---

## 8. Grafana Dashboards

Two separate dashboards should be maintained.

### PostgreSQL Dashboard

| Panel | Source |
|---|---|
| Availability | Prometheus |
| Database Size | Prometheus |
| Connections | Prometheus |
| Locks | Prometheus |
| Replication Lag | Prometheus |
| Logs | Loki |
| Database Traces | Tempo |

<details>
<summary>📸 <strong>Screenshot - PostgreSQL Grafana Dashboard</strong></summary>

<img width="1840" height="882" alt="image" src="https://github.com/user-attachments/assets/7fec674c-dc4a-424e-a963-372071053f9c" />

</details>

### ScyllaDB Dashboard

| Panel | Source |
|---|---|
| Availability | Prometheus |
| Reactor Utilization | Prometheus |
| Read/Write Workload | Prometheus |
| Failed/Timed-out Writes | Prometheus |
| Memory Pressure | Prometheus |
| Compaction Backlog | Prometheus |
| Read/Write P95/P99 Latency | Prometheus |
| Logs | Loki |
| Database Traces | Tempo |

<details>
<summary>📸 <strong>Screenshot - ScyllaDB Grafana Dashboard</strong></summary>

<img width="1840" height="882" alt="image" src="https://github.com/user-attachments/assets/ad352054-701d-4ef6-b743-81156c9be438" />
<img width="1840" height="882" alt="image" src="https://github.com/user-attachments/assets/10ee70a8-6f28-45ad-ab2f-5f1ef15afa65" />

</details>

---

## 9. POC Validation

| Validation | Expected Result |
|---|---|
| PostgreSQL exporter target | `UP` |
| ScyllaDB metrics target | `UP` |
| PostgreSQL metrics | Available in Prometheus |
| ScyllaDB metrics | Available in Prometheus |
| PostgreSQL logs | Available in Loki |
| ScyllaDB logs | Available in Loki |
| PostgreSQL database span | Visible under Attendance API trace |
| ScyllaDB database operation | Visible under Salary API trace |
| PostgreSQL dashboard | Loads successfully |
| ScyllaDB dashboard | Loads successfully |


---

## 10. Troubleshooting

| Issue | Solution |
|---|---|
| PostgreSQL target DOWN | Check PostgreSQL Exporter and port `9187` |
| ScyllaDB target DOWN | Check ScyllaDB metrics endpoint and Prometheus target |
| PostgreSQL logs blank | Use `{service_name="postgres"}` |
| ScyllaDB logs blank | Use `{service_name="scylladb"}` |
| PostgreSQL DB span missing | Verify OpenTelemetry PostgreSQL instrumentation |
| ScyllaDB DB operation missing | Verify Salary API OpenTelemetry instrumentation |
| Grafana panel blank | Validate PromQL/LogQL/TraceQL directly |
| Metrics not changing | Generate database workload |

---

## 11. Conclusion

This POC demonstrates database observability for PostgreSQL and ScyllaDB using:

```text
Metrics + Logs + Traces
```

It provides database availability, workload and performance monitoring, operational logs, application-to-database trace visibility, and separate Grafana dashboards.

---

## 12. Contact Information

| Name | Contact |
|---|---|
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

## 13. References

| S.No | Description | Reference |
|---:|---|---|
| 1 | Prometheus Documentation | [Prometheus](https://prometheus.io/docs/) |
| 2 | Grafana Documentation | [Grafana](https://grafana.com/docs/) |
| 3 | Loki Documentation | [Loki](https://grafana.com/docs/loki/latest/) |
| 4 | Tempo Documentation | [Tempo](https://grafana.com/docs/tempo/latest/) |
| 5 | OpenTelemetry Documentation | [OpenTelemetry](https://opentelemetry.io/docs/) |
| 6 | Grafana Alloy Documentation | [Grafana Alloy](https://grafana.com/docs/alloy/latest/) |
| 7 | Grafana Alloy - Why Alloy | [Why Alloy](https://grafana.com/docs/alloy/latest/introduction/why-alloy/) |
| 8 | Migrate from Promtail to Grafana Alloy | [Alloy Migration Guide](https://grafana.com/docs/alloy/latest/set-up/migrate/from-promtail/) |
| 9 | Promtail Lifecycle / EOL | [Promtail Documentation](https://grafana.com/docs/loki/latest/send-data/promtail/) |
| 10 | PostgreSQL Documentation | [PostgreSQL](https://www.postgresql.org/docs/) |
| 11 | ScyllaDB Documentation | [ScyllaDB](https://docs.scylladb.com/) |
