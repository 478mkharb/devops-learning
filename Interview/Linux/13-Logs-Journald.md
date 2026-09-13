# Logs, Journald and Application Troubleshooting for DevOps

## Scope

This topic covers production log handling from a DevOps Engineer perspective:

- Linux logs and application logs
- `journalctl` and systemd journals
- Log levels and structured logging
- Finding failures quickly
- Correlating logs with deployments
- NGINX, Jenkins, Ansible and application troubleshooting
- Log rotation and retention basics
- Centralized logging concepts
- Practical incident scenarios

---

## 1. Why Logs Matter in DevOps

Logs help answer:

- What failed?
- When did it fail?
- Which host or service failed?
- Which deployment introduced the failure?
- Which request or user was affected?
- Was the issue application, infrastructure, network or dependency related?

A useful troubleshooting sequence is:

```text
Symptom
   ↓
Time window
   ↓
Affected host/service
   ↓
Relevant logs
   ↓
Error pattern
   ↓
Root cause
   ↓
Corrective action
```

Do not search random log files without first defining:

- Time range
- Service
- Host
- Request or deployment ID
- Expected behavior
- Actual behavior

---

## 2. Common Linux Log Locations

Common locations include:

```text
/var/log/
/var/log/syslog
/var/log/auth.log
/var/log/kern.log
/var/log/dmesg
/var/log/nginx/
/var/log/apache2/
/var/log/journal/
```

Examples:

```bash
sudo ls -lah /var/log
sudo tail -f /var/log/syslog
sudo tail -f /var/log/auth.log
```

On systemd systems, many service logs are stored in the journal instead of a separate text file.

Do not assume every service writes to:

```text
/var/log/<service>.log
```

Always check the service configuration and systemd unit.

---

## 3. Log Levels

Typical log levels:

| Level | Meaning | DevOps use |
|---|---|---|
| TRACE | Very detailed diagnostic data | Temporary deep debugging |
| DEBUG | Developer diagnostic information | Troubleshooting |
| INFO | Normal operational events | Deployment and service lifecycle |
| WARN | Unexpected but recoverable condition | Capacity or dependency warning |
| ERROR | Operation failed | Incident investigation |
| FATAL/CRITICAL | Severe failure | Service outage or crash |

Production logging should provide useful context without generating excessive noise.

Avoid logging:

- Passwords
- Private keys
- Access tokens
- Session cookies
- Database credentials
- Personal data unless required and protected

---

## 4. `journalctl` Basics

Show all journal entries:

```bash
sudo journalctl
```

Show newest entries first:

```bash
sudo journalctl -r
```

Follow logs live:

```bash
sudo journalctl -f
```

Show the last 100 entries:

```bash
sudo journalctl -n 100
```

Follow only the last 50 entries:

```bash
sudo journalctl -n 50 -f
```

Show logs for a service:

```bash
sudo journalctl -u nginx
```

Follow a service:

```bash
sudo journalctl -u nginx -f
```

Show only errors:

```bash
sudo journalctl -p err
```

Show warning and higher severity:

```bash
sudo journalctl -p warning
```

---

## 5. Time-Based Journal Queries

Since a specific time:

```bash
sudo journalctl --since "2026-09-13 10:00:00"
```

Until a specific time:

```bash
sudo journalctl --until "2026-09-13 11:00:00"
```

A time window:

```bash
sudo journalctl   --since "30 minutes ago"   --until "now"
```

Today:

```bash
sudo journalctl --since today
```

Yesterday:

```bash
sudo journalctl --since yesterday --until today
```

Combine service and time:

```bash
sudo journalctl   -u nginx   --since "15 minutes ago"   --no-pager
```

This is usually more useful than dumping the entire journal.

---

## 6. Useful Journal Filters

By boot:

```bash
sudo journalctl -b
```

Previous boot:

```bash
sudo journalctl -b -1
```

Kernel messages:

```bash
sudo journalctl -k
```

By process ID:

```bash
sudo journalctl _PID=1234
```

By executable:

```bash
sudo journalctl _COMM=sshd
```

By user ID:

```bash
sudo journalctl _UID=1000
```

By priority:

```bash
sudo journalctl -p 0..3
```

Priority values:

```text
0 emerg
1 alert
2 crit
3 err
4 warning
5 notice
6 info
7 debug
```

Show useful fields:

```bash
sudo journalctl -u myapp -o verbose
```

---

## 7. Output Formats

Short output:

```bash
sudo journalctl -u myapp -o short
```

ISO timestamps:

```bash
sudo journalctl -u myapp -o short-iso
```

JSON:

```bash
sudo journalctl -u myapp -o json
```

Pretty JSON:

```bash
sudo journalctl -u myapp -o json-pretty
```

Export format:

```bash
sudo journalctl -u myapp -o export
```

For automation and parsing, structured formats are preferable to fragile text parsing.

---

## 8. Check Service State Alongside Logs

Logs alone are insufficient. Always compare them with service state:

```bash
sudo systemctl status myapp
sudo journalctl -u myapp -n 100 --no-pager
```

Check whether a service is active:

```bash
systemctl is-active myapp
```

Check whether it is enabled:

```bash
systemctl is-enabled myapp
```

Show recent restart history:

```bash
systemctl show myapp   -p ActiveState   -p SubState   -p ExecMainStatus   -p NRestarts
```

A service may be:

- Active and healthy
- Active but functionally broken
- Failed
- Restarting continuously
- Inactive
- Running with stale configuration

---

## 9. Failed Services

List failed units:

```bash
systemctl --failed
```

Inspect a failed service:

```bash
sudo systemctl status myapp
sudo journalctl -u myapp -b --no-pager
```

Check the unit definition:

```bash
systemctl cat myapp
```

Show dependencies:

```bash
systemctl list-dependencies myapp
```

Show recent boot failures:

```bash
sudo journalctl -b -p err
```

Typical causes:

- Wrong executable path
- Missing environment variable
- Permission problem
- Port already in use
- Missing configuration file
- Invalid configuration syntax
- Dependency unavailable
- Incorrect working directory
- Wrong user or group

---

## 10. Search Logs Efficiently

Use `grep`:

```bash
sudo journalctl -u myapp --no-pager | grep -i error
```

Search multiple patterns:

```bash
sudo journalctl -u myapp --no-pager |   grep -Ei 'error|failed|exception|timeout|refused'
```

Search a text log:

```bash
grep -nEi 'error|failed|timeout' /var/log/app.log
```

Show context:

```bash
grep -n -B 3 -A 5 'Exception' /var/log/app.log
```

Count repeated errors:

```bash
sudo journalctl -u myapp --since "1 hour ago" --no-pager |   grep -i error | sort | uniq -c | sort -nr
```

Use `awk` for field-based processing:

```bash
awk '{print $1, $2, $3, $NF}' /var/log/app.log
```

Use `less` for large files:

```bash
less +G /var/log/app.log
```

Inside `less`:

```text
/error     search forward
n          next match
N          previous match
G          end of file
g          beginning
q          quit
```

---

## 11. Live Troubleshooting

Follow a service log:

```bash
sudo journalctl -u myapp -f
```

In another terminal, reproduce the issue:

```bash
curl -v http://127.0.0.1:8080/health
```

Watch NGINX access logs:

```bash
sudo tail -f /var/log/nginx/access.log
```

Watch NGINX errors:

```bash
sudo tail -f /var/log/nginx/error.log
```

A useful pattern:

```text
Terminal 1: application logs
Terminal 2: reverse proxy logs
Terminal 3: system resources
Terminal 4: reproduce request
```

Check resources:

```bash
top
free -h
df -h
ss -lntp
```

---

## 12. NGINX Troubleshooting

Check configuration:

```bash
sudo nginx -t
```

View service logs:

```bash
sudo journalctl -u nginx -n 100 --no-pager
```

View application-facing logs:

```bash
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

Common HTTP status meanings:

| Status | Typical meaning |
|---|---|
| 400 | Bad request |
| 401 | Authentication required |
| 403 | Forbidden |
| 404 | Resource not found |
| 499 | Client closed request, commonly NGINX |
| 500 | Application/server error |
| 502 | Bad gateway, often upstream unavailable |
| 503 | Service unavailable |
| 504 | Gateway timeout |

For a `502`, check:

```bash
sudo ss -lntp
curl -v http://127.0.0.1:8080/health
sudo journalctl -u backend-service -n 100
```

For a `504`, investigate:

- Backend response time
- Network connectivity
- Upstream timeout settings
- Database or external API latency
- Thread or worker exhaustion

---

## 13. Jenkins Troubleshooting

Jenkins logs may be available through:

```bash
sudo journalctl -u jenkins -f
```

Check service state:

```bash
sudo systemctl status jenkins
```

Inspect recent errors:

```bash
sudo journalctl -u jenkins --since "1 hour ago" --no-pager
```

Pipeline logs should be checked for:

- Git checkout failures
- Credential errors
- Workspace permission errors
- Tool version mismatch
- Maven/Gradle failures
- Terraform validation errors
- Ansible SSH failures
- Host-key prompts
- Sudo password prompts
- Disk exhaustion
- Agent disconnects

Do not rely only on the final line:

```text
Build failed
```

Find the first meaningful error, not the last cascade of errors.

---

## 14. Ansible Troubleshooting

Run with verbosity:

```bash
ansible all -m ping -vvvv
```

Check inventory:

```bash
ansible-inventory --graph
ansible-inventory --list
```

Check remote logs using a task:

```yaml
- name: Read application log
  ansible.builtin.shell: tail -n 100 /var/log/myapp.log
  register: app_log
  changed_when: false

- name: Display application log
  ansible.builtin.debug:
    var: app_log.stdout_lines
```

Prefer purpose-built modules over shell commands where possible.

If Ansible reports a service failure:

```bash
sudo systemctl status myapp
sudo journalctl -u myapp -n 100 --no-pager
```

---

## 15. Application Log Design

A production application should ideally log:

- Timestamp
- Severity
- Service name
- Host or instance ID
- Environment
- Version or release ID
- Request ID
- Trace ID
- User or tenant identifier where safe
- Error type
- Error message
- Relevant operation
- Duration or latency

Example structured log:

```json
{
  "timestamp": "2026-09-13T10:30:00Z",
  "level": "ERROR",
  "service": "salary-api",
  "environment": "dev",
  "release": "2026.09.13-42",
  "request_id": "req-123",
  "operation": "generate_salary_slip",
  "error": "database_timeout",
  "duration_ms": 5000
}
```

Structured logs are easier to search and aggregate than unstructured text.

---

## 16. Correlation IDs

A correlation ID links related events across services.

Example:

```text
Browser request
   request_id=req-123
        ↓
NGINX log
        ↓
Employee API log
        ↓
Database operation log
        ↓
Notification worker log
```

Without correlation IDs, distributed troubleshooting becomes guesswork.

Recommended fields:

```text
request_id
trace_id
span_id
deployment_id
instance_id
```

Do not expose sensitive information in IDs or log values.

---

## 17. Log Rotation

Logs can fill disks and cause outages.

Check log sizes:

```bash
sudo du -sh /var/log/*
sudo journalctl --disk-usage
```

Traditional text logs are commonly managed by `logrotate`.

Check configuration:

```bash
cat /etc/logrotate.conf
ls -l /etc/logrotate.d/
```

Test configuration:

```bash
sudo logrotate -d /etc/logrotate.conf
```

Force rotation for testing:

```bash
sudo logrotate -f /etc/logrotate.conf
```

Do not force rotation blindly in production.

For journald retention, inspect:

```bash
sudo journalctl --disk-usage
```

Configuration is commonly in:

```text
/etc/systemd/journald.conf
```

Example settings:

```text
SystemMaxUse=1G
MaxRetentionSec=14day
```

After changing journald configuration:

```bash
sudo systemctl restart systemd-journald
```

Retention settings should match compliance and incident-investigation requirements.

---

## 18. Persistent versus Volatile Journals

A journal may be stored:

- Persistently under `/var/log/journal`
- Volatilely under `/run/log/journal`

Check:

```bash
ls -ld /var/log/journal
ls -ld /run/log/journal
```

Persistent logs survive reboot. Volatile logs may not.

Create persistent journal storage:

```bash
sudo mkdir -p /var/log/journal
sudo systemd-tmpfiles --create --prefix /var/log/journal
sudo systemctl restart systemd-journald
```

Confirm:

```bash
sudo journalctl --disk-usage
```

---

## 19. Log Security

Logs may contain sensitive data.

Controls:

- Restrict file permissions.
- Avoid secrets in logs.
- Protect centralized log transport.
- Apply retention limits.
- Mask personal information.
- Control who can query logs.
- Audit access to sensitive logs.
- Prevent log injection where user input is logged.
- Synchronize system clocks.

Check permissions:

```bash
ls -l /var/log/auth.log
```

Check time synchronization:

```bash
timedatectl
systemctl status systemd-timesyncd
```

Incorrect time makes incident correlation difficult.

---

## 20. Centralized Logging

Local logs are insufficient for large environments because:

- Instances are ephemeral.
- Auto Scaling replaces hosts.
- Logs disappear when instances terminate.
- Searching many servers manually is slow.
- Cross-service correlation is difficult.

Typical architecture:

```text
Application / NGINX / systemd
             ↓
      Log collector
             ↓
 Central log platform
             ↓
 Search, dashboards, alerts
```

Common solutions include:

- Elasticsearch and Kibana
- OpenSearch
- Loki and Grafana
- CloudWatch Logs
- Fluent Bit
- Fluentd
- Filebeat
- Vector

Centralized logging should address:

- Collection
- Parsing
- Transport
- Storage
- Retention
- Indexing
- Access control
- Alerting
- Cost

---

## 21. Logs and Metrics Are Different

| Logs | Metrics |
|---|---|
| Detailed event records | Numeric time-series measurements |
| Good for individual failures | Good for trends and alert thresholds |
| High-cardinality details | Aggregated measurements |
| Often expensive to search at scale | Efficient for dashboards |
| Example: stack trace | Example: request rate |

Use both:

```text
Metric detects abnormal behavior
        ↓
Alert fires
        ↓
Logs explain the event
```

Example:

- Prometheus detects HTTP 5xx increase.
- Grafana shows the affected service.
- NGINX logs show `502`.
- Application journal shows database connection failure.

---

## 22. Practical Incident Workflow

### Step 1: Confirm the symptom

```bash
curl -I https://app.example.com
```

### Step 2: Identify the affected service

```bash
systemctl --failed
systemctl status nginx
systemctl status myapp
```

### Step 3: Check recent logs

```bash
sudo journalctl -u myapp --since "15 minutes ago" --no-pager
```

### Step 4: Search for patterns

```bash
sudo journalctl -u myapp --since "15 minutes ago" --no-pager |   grep -Ei 'error|failed|timeout|refused|exception'
```

### Step 5: Check dependencies

```bash
ss -lntp
curl -v http://127.0.0.1:8080/health
df -h
free -h
```

### Step 6: Compare deployment timing

```bash
journalctl --since "1 hour ago" | grep -Ei 'deploy|restart|reload'
```

### Step 7: Mitigate safely

Possible actions:

- Roll back deployment
- Restart a failed service
- Remove a bad configuration
- Increase capacity
- Restore dependency connectivity

### Step 8: Preserve evidence

Before deleting or rotating logs:

- Save relevant entries.
- Record timestamps.
- Record commands and findings.
- Capture service status.
- Record deployment version.

---

## 23. Common Scenarios

### Scenario 1: Service is active but application returns 502

Check:

```bash
systemctl status myapp
ss -lntp
curl -v http://127.0.0.1:8080/health
journalctl -u myapp -n 100 --no-pager
```

The process may be running but listening on the wrong port or failing health checks.

### Scenario 2: Service repeatedly restarts

Check:

```bash
systemctl status myapp
journalctl -u myapp -b --no-pager
systemctl show myapp -p NRestarts -p ExecMainStatus
```

Look for:

- Invalid configuration
- Missing environment variable
- Permission error
- Port conflict
- Dependency failure
- Out-of-memory termination

### Scenario 3: Disk is full

Check:

```bash
df -h
sudo du -xhd1 /var/log | sort -h
sudo journalctl --disk-usage
```

Do not immediately delete logs. Determine:

- Which file is growing
- Whether rotation is working
- Whether an application is logging in a loop
- Whether retention is configured correctly

### Scenario 4: Deployment succeeded but service is broken

Compare:

```bash
journalctl -u myapp --since "30 minutes ago"
systemctl status myapp
systemctl cat myapp
```

Check:

- Release version
- Environment variables
- File ownership
- Configuration syntax
- Dependency versions
- Database migrations
- Port changes

### Scenario 5: Logs show connection timeout

A timeout may indicate:

- Network route problem
- Security group/NACL issue
- Service not listening
- Dependency overload
- DNS resolution issue
- Application connection pool exhaustion

Validate independently:

```bash
getent hosts database.internal
nc -vz database.internal 5432
curl -v http://dependency.internal:8080/health
```

---

## 24. Commands to Memorize

```bash
journalctl -f
journalctl -u SERVICE
journalctl -u SERVICE -n 100
journalctl -u SERVICE --since "15 minutes ago"
journalctl -p err
journalctl -b -1
journalctl -k
journalctl --disk-usage
systemctl status SERVICE
systemctl --failed
systemctl cat SERVICE
systemctl show SERVICE
systemctl is-active SERVICE
systemctl is-enabled SERVICE
grep -nEi 'error|failed|timeout' FILE
tail -f FILE
less +G FILE
df -h
du -sh /var/log/*
ss -lntp
nginx -t
timedatectl
```

---

## 25. Interview Questions

### Q1. What is the difference between logs and metrics?

Logs contain detailed event records. Metrics are numeric measurements used for trends, dashboards and alerting.

### Q2. How do you view logs for a systemd service?

```bash
sudo journalctl -u service-name
```

### Q3. How do you view logs from the previous boot?

```bash
sudo journalctl -b -1
```

### Q4. How do you follow logs in real time?

```bash
sudo journalctl -u service-name -f
```

### Q5. How do you troubleshoot a service that is repeatedly restarting?

Inspect `systemctl status`, journal entries, exit status, restart count, configuration, permissions, ports and dependencies.

### Q6. Why can a service be active but still unavailable?

The process may be alive but listening on the wrong address or port, failing health checks, or unable to access a dependency.

### Q7. Why are correlation IDs important?

They connect events across multiple services and make distributed troubleshooting possible.

### Q8. Why is centralized logging important in cloud environments?

Instances are ephemeral and logs may disappear when hosts are replaced. Centralized logging provides durable, searchable evidence.

### Q9. How can logs cause an outage?

Unbounded logs can fill the disk, preventing applications and system services from writing files or operating normally.

### Q10. What should never be logged?

Passwords, private keys, tokens, credentials, session cookies and unnecessary sensitive personal data.

---

## Final DevOps Checklist

- [ ] Define the time window before searching logs.
- [ ] Identify the affected host and service.
- [ ] Check service state and logs together.
- [ ] Use `journalctl -u SERVICE`.
- [ ] Filter by time and priority.
- [ ] Search for the first meaningful error.
- [ ] Correlate proxy, application and system logs.
- [ ] Check deployment timing.
- [ ] Check ports, disk, memory and dependencies.
- [ ] Protect secrets and personal data.
- [ ] Configure log rotation and retention.
- [ ] Use persistent or centralized logging.
- [ ] Preserve evidence during incidents.
- [ ] Connect metrics alerts to detailed logs.

## Key DevOps Principle

> Metrics tell you that something is wrong. Logs help explain why. Effective DevOps troubleshooting uses both, correlated with service state, deployment history, infrastructure signals and application context.
