# Linux Performance Monitoring and Troubleshooting for DevOps

## Scope

This guide is written from a **DevOps Engineer perspective**.

The objective is not merely to memorize Linux commands. The objective is to determine:

- Why an application is slow.
- Whether the bottleneck is CPU, memory, disk, network, or the application itself.
- Whether the issue affects one process, one host, one availability zone, or the whole service.
- What evidence should be collected before changing production.
- How Linux host metrics connect to Prometheus, Grafana, alerting, CI/CD, EC2, Jenkins, Ansible, and cloud operations.

---

# 1. The Performance Troubleshooting Methodology

A reliable production investigation should follow this sequence:

```text
User symptom
    ↓
Define the impact
    ↓
Check recent changes
    ↓
Check service health
    ↓
Check host saturation
    ↓
Identify the responsible process
    ↓
Correlate with application metrics and logs
    ↓
Apply the smallest safe mitigation
    ↓
Verify recovery
    ↓
Document the root cause
```

## 1.1 First questions to ask

Before running commands, establish:

1. What is slow or failing?
2. When did it start?
3. Is the problem constant or intermittent?
4. Are all users affected or only some users?
5. Is one instance affected or all instances?
6. Was there a recent deployment, configuration change, traffic spike, or infrastructure change?
7. Is the symptom high latency, high CPU, memory exhaustion, disk full, packet loss, or application errors?
8. What is the business impact?
9. What changed immediately before the incident?
10. What metric would prove that the system has recovered?

## 1.2 Never begin with random commands

A poor investigation looks like:

```bash
top
df -h
free -m
systemctl restart myapp
```

A better investigation is hypothesis-driven:

```text
Hypothesis: Application is slow because the host is CPU saturated.
Evidence required: CPU utilization, load average, run queue, process CPU usage.
Validation: top, mpstat, vmstat, pidstat, application latency.
Mitigation: Reduce traffic, scale out, stop runaway process, or optimize workload.
Verification: CPU saturation falls and latency returns to baseline.
```

---

# 2. Establish a Baseline

A performance metric is useful only when compared with a baseline.

Examples:

- Normal CPU utilization: 25–40%.
- Normal load average: 1–2 on a 4-vCPU host.
- Normal application latency: 100 ms.
- Normal disk await: 5 ms.
- Normal memory pressure: low.
- Normal network retransmission rate: near zero.

These values are examples only. Every service needs its own baseline.

## 2.1 Capture a quick snapshot

```bash
date
hostname
uptime
nproc
free -h
df -h
df -ih
ip -s link
systemctl --failed
ps -eo pid,ppid,user,%cpu,%mem,stat,etime,cmd --sort=-%cpu | head -20
```

## 2.2 Save evidence to a file

```bash
{
  date
  hostname
  uptime
  nproc
  free -h
  df -h
  systemctl --failed
  ps -eo pid,ppid,user,%cpu,%mem,stat,etime,cmd --sort=-%cpu | head -20
} | tee /tmp/performance-snapshot-$(date +%F-%H%M%S).txt
```

Do not collect only one point in time for an intermittent issue. Capture multiple samples:

```bash
vmstat 5 12
```

This collects 12 samples at 5-second intervals.

---

# 3. Understand Load Average

## 3.1 View load average

```bash
uptime
```

Example:

```text
11:00:00 up 10 days,  2 users,  load average: 8.00, 7.50, 6.20
```

The three values represent approximately:

- 1-minute load average.
- 5-minute load average.
- 15-minute load average.

Linux load average includes tasks that are:

- Runnable and waiting for CPU.
- In uninterruptible sleep, commonly waiting for I/O.

Therefore, load average is **not identical to CPU utilization**.

## 3.2 Interpret load relative to CPU count

Check CPU count:

```bash
nproc
```

If the system has 4 logical CPUs:

| Load average | Possible interpretation |
|---|---|
| 0–2 | Usually comfortable |
| Around 4 | CPU queues may be forming |
| Above 4 | Sustained CPU pressure is possible |
| High load with low CPU usage | Possible I/O wait or blocked tasks |

These are rough indicators, not universal thresholds.

## 3.3 Example

A 4-vCPU host has:

```text
load average: 8.0, 7.5, 6.2
CPU idle: 2%
```

Likely interpretation:

- More runnable work exists than the CPU can execute.
- CPU saturation is probable.
- Identify CPU-consuming processes.
- Check whether the workload is expected or caused by a runaway process.

Another host has:

```text
load average: 8.0, 7.5, 6.2
CPU idle: 70%
iowait: 25%
```

Likely interpretation:

- Load is high, but CPU is not the primary bottleneck.
- Tasks may be blocked on storage.
- Investigate disk latency and I/O queue depth.

---

# 4. CPU Performance

## 4.1 Use `top`

```bash
top
```

Useful fields:

- `%us`: User-space CPU time.
- `%sy`: Kernel/system CPU time.
- `%ni`: Time running processes with adjusted nice priority.
- `%id`: Idle CPU time.
- `%wa`: I/O wait.
- `%hi`: Hardware interrupt time.
- `%si`: Software interrupt time.
- `%st`: Steal time from a virtualized CPU.

Sort processes by CPU inside `top`:

```text
Shift + P
```

Show threads:

```text
H
```

Exit:

```text
q
```

## 4.2 Use `ps` for a quick process ranking

```bash
ps -eo pid,ppid,user,%cpu,%mem,stat,etime,cmd --sort=-%cpu | head -20
```

Find a specific process:

```bash
pgrep -af java
pgrep -af gunicorn
pgrep -af nginx
```

## 4.3 Use `mpstat`

Install the package if required:

```bash
sudo apt install sysstat
```

View per-CPU statistics:

```bash
mpstat -P ALL 5 5
```

Important fields:

- `%usr`: User CPU.
- `%sys`: System CPU.
- `%iowait`: Time waiting for I/O.
- `%irq`: Hardware interrupts.
- `%soft`: Software interrupts.
- `%steal`: CPU stolen by the hypervisor.
- `%idle`: Idle CPU.

## 4.4 Use `vmstat`

```bash
vmstat 5 10
```

Important columns:

| Column | Meaning |
|---|---|
| `r` | Runnable tasks waiting for CPU |
| `b` | Tasks blocked, commonly on I/O |
| `si` | Swap-in |
| `so` | Swap-out |
| `us` | User CPU |
| `sy` | System CPU |
| `id` | Idle CPU |
| `wa` | I/O wait |
| `st` | Steal time |

Interpretation examples:

- High `r`, low `id`: CPU contention.
- High `b`, high `wa`: I/O blocking.
- High `si` and `so`: memory pressure and swapping.
- High `st`: virtual CPU contention or cloud host pressure.

## 4.5 Use `pidstat`

```bash
pidstat -u -p ALL 5 5
```

For a particular process:

```bash
pidstat -u -p 1234 5 10
```

For threads:

```bash
pidstat -t -p 1234 5 5
```

For context switches:

```bash
pidstat -w -p 1234 5 5
```

For I/O:

```bash
pidstat -d -p 1234 5 5
```

## 4.6 CPU user time versus system time

### High user CPU

Possible causes:

- Application computation.
- Compression or encryption.
- JSON parsing.
- Regular expression processing.
- JVM garbage collection.
- Python CPU-bound code.
- Busy loops.
- Excessive worker count.

### High system CPU

Possible causes:

- Excessive system calls.
- Network packet processing.
- File operations.
- Context switching.
- Kernel overhead.
- Interrupt storms.
- High connection churn.

Investigate with:

```bash
pidstat -u -p ALL 5 5
sar -w 5 5
```

Advanced tools:

```bash
strace -p 1234 -c
perf top
perf stat -p 1234
```

Use tracing carefully in production because it can add overhead and may expose sensitive data.

## 4.7 CPU steal time in cloud VMs

`%st` indicates CPU time that the virtual machine wanted but the hypervisor allocated elsewhere.

Check:

```bash
mpstat 5 5
```

If steal time is persistently high:

- The underlying host may be contended.
- The VM may be oversubscribed.
- Investigate the cloud provider's instance metrics.
- Consider moving to a different instance family or scaling out.

## 4.8 EC2 burstable CPU credits

For burstable EC2 instances such as T-family instances, CPU performance can depend on CPU credits.

Symptoms of exhausted credits may include:

- Application latency increases.
- CPU utilization appears capped.
- Performance degrades after sustained load.
- Cloud monitoring shows low CPU credit balance.

DevOps investigation:

1. Check OS CPU metrics.
2. Check EC2 CPU credit metrics.
3. Check whether the workload is continuously CPU-intensive.
4. Consider a larger instance, unlimited mode, or a non-burstable instance family.
5. Compare cost against performance requirements.

---

# 5. Memory Performance

## 5.1 Use `free`

```bash
free -h
```

Example:

```text
               total   used   free  shared  buff/cache  available
Mem:             8Gi    5Gi   500Mi   200Mi       2.5Gi       2Gi
Swap:            2Gi   100Mi   1.9Gi
```

The most useful field for many operational decisions is:

```text
available
```

Linux uses unused memory for filesystem cache. Therefore:

```text
used memory != application memory
```

Low `free` memory alone does not prove memory exhaustion.

## 5.2 Check memory consumers

```bash
ps -eo pid,ppid,user,%mem,rss,vsz,cmd --sort=-%mem | head -20
```

`RSS` is resident memory currently held in RAM.

`VSZ` is virtual memory size and can be much larger than actual RAM usage.

## 5.3 Check memory pressure

```bash
vmstat 5 10
```

Watch:

- `si`: Swap in.
- `so`: Swap out.
- `free`: Free memory.
- `r`: Runnable tasks.
- `b`: Blocked tasks.

Repeated non-zero swap activity may indicate memory pressure, but occasional swap use is not automatically a problem.

## 5.4 Check OOM killer events

```bash
dmesg -T | grep -i -E 'out of memory|oom|killed process'
```

On systemd systems:

```bash
journalctl -k -g 'oom|out of memory|killed process'
```

Check memory-related kernel messages:

```bash
journalctl -k --since "1 hour ago"
```

Typical evidence:

```text
Out of memory: Killed process 1234 (java)
```

This means the kernel terminated a process to recover memory.

## 5.5 Memory leak indicators

Possible indicators:

- RSS increases continuously.
- Memory does not return after traffic falls.
- OOM kills occur repeatedly.
- Swap usage increases.
- Application restarts temporarily fix the issue.
- Heap or object counts grow over time.

Investigation steps:

1. Record process RSS over time.
2. Correlate with request rate.
3. Check application heap metrics.
4. Check garbage collection behavior.
5. Compare deployed version with the previous version.
6. Capture a heap profile only with an approved production procedure.

Example:

```bash
while true; do
  date
  ps -o pid,ppid,%mem,rss,vsz,cmd -p 1234
  sleep 60
done | tee /tmp/process-memory.log
```

## 5.6 Linux cache and buffers

Linux uses RAM for:

- Page cache.
- Directory entries.
- Inode cache.
- Block device buffers.

Do not routinely run:

```bash
sudo sync
sudo sysctl -w vm.drop_caches=3
```

This is generally a diagnostic action, not a performance optimization. Dropping caches can make applications slower and should not be used as a normal fix.

---

# 6. Swap and Swapping

## 6.1 Check swap

```bash
swapon --show
free -h
cat /proc/swaps
```

## 6.2 Why excessive swap is harmful

Swap allows memory pages to move from RAM to disk.

If an application repeatedly needs swapped-out pages:

- Page faults increase.
- Disk I/O increases.
- Latency rises.
- The system may appear frozen.
- Load average may increase.

## 6.3 Check swappiness

```bash
sysctl vm.swappiness
```

Do not change it blindly. The correct value depends on workload, RAM, storage, and operating-system policy.

## 6.4 Production response to memory pressure

Prefer:

1. Identify the memory-consuming process.
2. Check whether usage is expected.
3. Check recent deployments.
4. Check service limits.
5. Scale vertically or horizontally.
6. Restart only as a controlled mitigation.
7. Fix the leak or excessive allocation permanently.

---

# 7. Disk I/O Performance

Disk capacity and disk performance are different problems.

A disk can have:

- Plenty of free space but very high latency.
- Low latency but no free space.
- High throughput but poor random I/O performance.
- Enough IOPS but insufficient throughput.

## 7.1 Use `iostat`

```bash
iostat -xz 5 5
```

Important fields:

| Field | Meaning |
|---|---|
| `r/s` | Reads per second |
| `w/s` | Writes per second |
| `rkB/s` | Read throughput |
| `wkB/s` | Write throughput |
| `await` | Average I/O request wait time |
| `%util` | Device busy percentage |
| `avgqu-sz` | Average queue size |

High `await` indicates high I/O latency.

High `%util` can indicate a busy device, but interpretation depends on the device type and workload.

## 7.2 Identify disks

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
findmnt
df -hT
```

## 7.3 Identify I/O-heavy processes

```bash
sudo iotop -oPa
```

Alternative:

```bash
pidstat -d 5 10
```

## 7.4 Check filesystem usage

```bash
df -h
df -ih
```

`df -h` checks blocks.

`df -ih` checks inode consumption.

A filesystem can fail to create files even when block space is available if all inodes are exhausted.

## 7.5 Find large files

```bash
sudo du -xhd1 /var | sort -h
sudo du -xhd1 /var/log | sort -h
sudo find /var/log -type f -size +500M -ls
```

Find recently modified large files:

```bash
sudo find /var -xdev -type f -size +500M -printf '%s %TY-%Tm-%Td %p\n' \
  | sort -n | tail -20
```

## 7.6 Deleted files still consuming space

A process can keep a deleted file open.

Check:

```bash
sudo lsof +L1
```

Common example:

- Log file is deleted.
- Application still has the file descriptor open.
- `df` shows space consumed.
- `du` cannot find the file.

Possible mitigation:

- Reload or restart the responsible service using the approved procedure.
- Configure correct log rotation.
- Avoid deleting active logs manually.

## 7.7 Disk latency versus disk capacity

| Symptom | Likely area |
|---|---|
| `df -h` near 100% | Capacity |
| `df -ih` near 100% | Inodes |
| High `await` | I/O latency |
| High queue size | I/O contention |
| High write throughput | Write-heavy workload |
| High read latency | Storage or workload bottleneck |
| Deleted files in `lsof` | Open file descriptors |

---

# 8. Network Performance

## 8.1 Check listening ports and connections

```bash
ss -tulpen
ss -s
```

View established TCP connections:

```bash
ss -tan state established
```

View connections to a port:

```bash
ss -tan '( sport = :8080 or dport = :8080 )'
```

Count connections by state:

```bash
ss -tan | awk 'NR>1 {print $1}' | sort | uniq -c
```

## 8.2 Check interface statistics

```bash
ip -s link
```

Look for:

- RX errors.
- TX errors.
- Dropped packets.
- Overruns.
- Carrier errors.

A rising drop or error counter may indicate:

- Interface problems.
- Buffer exhaustion.
- Driver issues.
- Network congestion.
- Cloud networking limits.
- Incorrect MTU.
- Security or routing problems.

## 8.3 Check routes and MTU

```bash
ip route
ip addr
ip link
```

Test path:

```bash
tracepath example.com
```

Test DNS separately:

```bash
dig example.com
```

Test TCP connectivity:

```bash
nc -vz example.com 443
```

## 8.4 Check retransmissions

```bash
nstat -az | grep -i retrans
ss -ti
```

TCP retransmissions may indicate:

- Packet loss.
- Congestion.
- Faulty network path.
- Receiver overload.
- MTU problems.

## 8.5 Check network throughput

If installed:

```bash
sar -n DEV 5 5
sar -n TCP,ETCP 5 5
```

Other tools:

```bash
nload
iftop
```

Use packet capture only when necessary and approved:

```bash
sudo tcpdump -ni any port 443
```

Avoid capturing sensitive production traffic without authorization.

## 8.6 Network latency investigation

Break the request into layers:

```text
Client
  → DNS resolution
  → TCP connection
  → TLS handshake
  → Load balancer
  → Reverse proxy
  → Application
  → Database/cache
```

A slow HTTP request does not automatically mean Linux CPU is high.

---

# 9. Context Switches and Interrupts

## 9.1 Context switches

A context switch occurs when the CPU changes from one task to another.

Check:

```bash
vmstat 5 5
pidstat -w 5 5
sar -w 5 5
```

High context switching may be caused by:

- Too many threads.
- Excessive worker processes.
- Lock contention.
- High connection churn.
- Very small tasks.
- Oversized application concurrency.

## 9.2 Interrupts

Check:

```bash
cat /proc/interrupts
```

High interrupt activity can be related to:

- Network traffic.
- Storage activity.
- Hardware devices.
- Virtualized device processing.

Use `mpstat` to examine:

```text
%irq
%soft
```

---

# 10. Process-Level Troubleshooting

## 10.1 Process state

```bash
ps -eo pid,stat,cmd
```

Common states:

| State | Meaning |
|---|---|
| `R` | Running or runnable |
| `S` | Interruptible sleep |
| `D` | Uninterruptible sleep, often I/O |
| `T` | Stopped |
| `Z` | Zombie |
| `I` | Idle kernel thread |

A process stuck in `D` state often requires investigation of the underlying I/O or filesystem.

## 10.2 Process tree

```bash
pstree -ap
```

For a specific service:

```bash
systemctl status myapp
systemctl show myapp -p MainPID
pstree -ap 1234
```

## 10.3 Open files and sockets

```bash
sudo lsof -p 1234
sudo lsof -p 1234 | wc -l
sudo lsof -i -P -n
```

Too many open files may cause:

```text
Too many open files
```

Check limits:

```bash
ulimit -n
cat /proc/1234/limits | grep -i open
```

## 10.4 File descriptor usage

```bash
ls /proc/1234/fd | wc -l
cat /proc/sys/fs/file-nr
```

Check service configuration:

```bash
systemctl show myapp -p LimitNOFILE
```

## 10.5 Threads

```bash
ps -eLf | grep myapp
top -H -p 1234
```

A high thread count can increase:

- Memory consumption.
- Context switching.
- Scheduling overhead.
- Lock contention.

---

# 11. Application Performance Versus Host Performance

## 11.1 Host metrics alone are insufficient

A host may show:

```text
CPU: 30%
Memory: 50%
Disk: normal
Network: normal
```

Yet the application can be slow because of:

- Database query latency.
- External API latency.
- Lock contention.
- Connection pool exhaustion.
- Thread pool exhaustion.
- Garbage collection pauses.
- Cache misses.
- DNS delays.
- Application-level rate limits.

## 11.2 Use the four golden signals

| Signal | Question |
|---|---|
| Latency | How long do requests take? |
| Traffic | How much demand is arriving? |
| Errors | How many requests fail? |
| Saturation | Which resource is near its limit? |

Examples:

- HTTP p95 latency.
- Requests per second.
- HTTP 5xx rate.
- CPU saturation.
- Database connection pool usage.
- Worker queue depth.

## 11.3 Percentiles

Average latency can hide slow requests.

Important percentiles:

- p50: Median.
- p90: 90% of requests are faster than this.
- p95: 95% are faster than this.
- p99: 99% are faster than this.

Example:

```text
p50 = 80 ms
p95 = 400 ms
p99 = 2 s
```

Most requests are fast, but a significant tail is slow.

---

# 12. Prometheus and Node Exporter

Node Exporter exposes Linux host metrics for Prometheus.

Typical metrics include:

```promql
node_cpu_seconds_total
node_memory_MemAvailable_bytes
node_memory_MemTotal_bytes
node_memory_SwapFree_bytes
node_filesystem_avail_bytes
node_filesystem_size_bytes
node_disk_read_time_seconds_total
node_disk_write_time_seconds_total
node_network_receive_drop_total
node_network_transmit_drop_total
node_load1
node_load5
node_load15
```

## 12.1 CPU usage percentage

```promql
100 *
(1 - avg by (instance) (
  rate(node_cpu_seconds_total{mode="idle"}[5m])
))
```

## 12.2 CPU by mode

```promql
sum by (instance, mode) (
  rate(node_cpu_seconds_total[5m])
)
```

## 12.3 Memory utilization

```promql
100 *
(
  1 -
  node_memory_MemAvailable_bytes
  /
  node_memory_MemTotal_bytes
)
```

## 12.4 Swap usage

```promql
100 *
(
  1 -
  node_memory_SwapFree_bytes
  /
  node_memory_SwapTotal_bytes
)
```

Protect against systems with no swap:

```promql
100 *
(
  1 -
  node_memory_SwapFree_bytes
  /
  clamp_min(node_memory_SwapTotal_bytes, 1)
)
```

## 12.5 Filesystem utilization

```promql
100 *
(
  1 -
  node_filesystem_avail_bytes{fstype!~"tmpfs|overlay"}
  /
  node_filesystem_size_bytes{fstype!~"tmpfs|overlay"}
)
```

## 12.6 Load per CPU

```promql
node_load1
/
count by (instance) (
  node_cpu_seconds_total{mode="idle"}
)
```

## 12.7 Network drops

```promql
rate(node_network_receive_drop_total[5m])
```

```promql
rate(node_network_transmit_drop_total[5m])
```

## 12.8 Disk read/write throughput

```promql
rate(node_disk_read_bytes_total[5m])
```

```promql
rate(node_disk_written_bytes_total[5m])
```

## 12.9 Saturation alerts

A useful alert should include:

- Instance.
- Resource.
- Current value.
- Duration.
- Runbook link.
- Impact.
- Suggested first checks.

Avoid alerting on every short-lived CPU spike. Use a duration such as:

```yaml
for: 10m
```

The correct duration depends on the workload.

---

# 13. Performance Troubleshooting with Grafana

A useful dashboard should contain:

## Host overview

- CPU utilization.
- Load average.
- Memory utilization.
- Swap activity.
- Disk utilization.
- Disk latency.
- Network throughput.
- Network drops.
- Filesystem usage.

## Application overview

- Request rate.
- Error rate.
- p50/p95/p99 latency.
- Active requests.
- Worker count.
- Queue depth.
- Database latency.
- Cache hit ratio.

## Correlation workflow

1. Select the incident time range.
2. Compare application latency with request rate.
3. Compare latency with CPU, memory, disk, and network.
4. Check whether all instances behave similarly.
5. Identify the first metric that changed.
6. Correlate with deployment and infrastructure events.

---

# 14. Performance and CI/CD

A deployment can cause performance degradation even when the service is technically “up.”

Common causes:

- New code has an inefficient query.
- Worker count changed.
- JVM options changed.
- NGINX buffering changed.
- Log level changed to `DEBUG`.
- A dependency introduced excessive CPU use.
- A new process consumes memory.
- A health check is too expensive.
- Cache warm-up was not considered.

## 14.1 Compare before and after deployment

Collect:

```bash
date
git rev-parse HEAD
systemctl status myapp --no-pager
ps -ef | grep myapp
free -h
uptime
```

Compare:

- CPU.
- RSS.
- Request latency.
- Error rate.
- Restart count.
- Open connections.
- Disk writes.
- Log volume.

## 14.2 Safe rollback decision

Rollback is reasonable when:

- The regression began immediately after deployment.
- The previous version is known to be healthy.
- The rollback procedure is tested.
- The change is causing material impact.
- Rollback risk is lower than continued operation.

Do not rollback blindly if the problem is caused by infrastructure, database schema, or an unrelated dependency.

---

# 15. Performance and Ansible

Ansible can collect evidence from multiple hosts.

Example:

```bash
ansible all -i inventory -m shell -a 'uptime && free -h && df -h'
```

Collect top CPU processes:

```bash
ansible all -i inventory -m shell \
  -a "ps -eo pid,%cpu,%mem,stat,cmd --sort=-%cpu | head -15"
```

Collect kernel messages:

```bash
ansible all -i inventory -m shell \
  -a "journalctl -k -n 50 --no-pager"
```

For production, prefer:

- Read-only commands.
- Controlled timeouts.
- Limited output.
- Secure handling of logs.
- No broad destructive remediation.
- A dedicated diagnostic role or playbook.

---

# 16. Production Troubleshooting Scenarios

## Scenario 1: CPU is 100% after deployment

### Evidence

```bash
uptime
nproc
top
ps -eo pid,%cpu,cmd --sort=-%cpu | head
pidstat -u 5 5
journalctl -u myapp --since "30 minutes ago"
```

### Likely causes

- Busy loop.
- Increased traffic.
- Excessive worker count.
- Garbage collection.
- Inefficient code.
- Unexpected retry loop.

### Response

1. Confirm the process.
2. Check application traffic and errors.
3. Compare deployment version.
4. Scale out or reduce load if necessary.
5. Roll back if the regression is confirmed.
6. Verify CPU and latency recovery.

---

## Scenario 2: Load average is high but CPU is mostly idle

### Evidence

```bash
uptime
mpstat 5 5
vmstat 5 5
iostat -xz 5 5
ps -eo pid,stat,cmd | awk '$2 ~ /D/'
```

### Likely causes

- Disk I/O wait.
- Network filesystem delay.
- Blocked processes.
- Storage failure or contention.

### Response

Investigate I/O latency and blocked tasks instead of killing random CPU processes.

---

## Scenario 3: Application is killed unexpectedly

### Evidence

```bash
journalctl -k -g 'oom|out of memory|killed process'
free -h
swapon --show
ps -eo pid,%mem,rss,cmd --sort=-%mem | head
```

### Likely causes

- Memory leak.
- Traffic spike.
- Too many workers.
- Undersized instance.
- Container or system memory limit.

### Response

- Confirm OOM evidence.
- Identify the largest consumer.
- Check memory trend.
- Apply temporary capacity mitigation.
- Fix allocation or leak.
- Add memory alerts.

---

## Scenario 4: Disk is full, but `du` does not explain usage

### Evidence

```bash
df -h
sudo du -xhd1 / | sort -h
sudo lsof +L1
```

### Likely cause

Deleted files are still open by a process.

### Response

Reload or restart the responsible service through the approved procedure, then fix log rotation or file-handling behavior.

---

## Scenario 5: API latency increased, but CPU is normal

### Evidence

- HTTP p95/p99 latency.
- Database latency.
- External API timing.
- Connection pool usage.
- DNS timing.
- NGINX access logs.
- Application logs.

Commands:

```bash
ss -s
ss -tan
dig api.example.com
curl -w '\nDNS: %{time_namelookup}\nConnect: %{time_connect}\nTLS: %{time_appconnect}\nTTFB: %{time_starttransfer}\nTotal: %{time_total}\n' \
  -o /dev/null -s https://api.example.com/health
```

### Likely causes

- Slow database.
- External dependency.
- Connection pool exhaustion.
- Network retransmissions.
- Application lock contention.

---

## Scenario 6: EC2 application slows after sustained traffic

Check:

- CPU utilization.
- CPU credit balance.
- Network allowance.
- EBS throughput and IOPS.
- Instance limits.
- Application worker saturation.

Do not assume that increasing application workers will solve an instance-level limit.

---

## Scenario 7: Jenkins build agent becomes slow

Check:

```bash
uptime
free -h
df -h
df -ih
ps -ef | grep -E 'java|jenkins'
iostat -xz 5 5
```

Common causes:

- Multiple concurrent builds.
- Workspace accumulation.
- Large artifact extraction.
- Docker or package cache growth.
- Java heap pressure.
- Disk I/O saturation.
- Log flooding.

Mitigations:

- Control executor count.
- Clean workspaces safely.
- Archive only required artifacts.
- Monitor disk and memory.
- Separate heavy builds across agents.
- Avoid running destructive cleanup during active builds.

---

# 17. A Practical 10-Minute Investigation Checklist

## Minute 1: Define impact

- What service?
- Which hosts?
- Which users?
- What changed?

## Minutes 2–3: Check host health

```bash
uptime
nproc
free -h
df -h
systemctl --failed
```

## Minutes 4–5: Check CPU and memory

```bash
top
vmstat 5 5
ps -eo pid,%cpu,%mem,stat,cmd --sort=-%cpu | head -20
```

## Minutes 6–7: Check disk and network

```bash
iostat -xz 5 3
ss -s
ip -s link
```

## Minutes 8–9: Correlate

- Logs.
- Deployment history.
- Prometheus.
- Grafana.
- Load balancer metrics.
- Database metrics.

## Minute 10: Decide

- Mitigate.
- Roll back.
- Scale.
- Escalate.
- Continue collecting evidence.

---

# 18. Common Mistakes

## Mistake 1: Treating load average as CPU percentage

Load average includes runnable and uninterruptible tasks.

## Mistake 2: Treating low free memory as failure

Use `available`, swap activity, and memory pressure.

## Mistake 3: Restarting services before collecting evidence

A restart may erase clues and hide the root cause.

## Mistake 4: Killing the top CPU process immediately

The process may be performing legitimate work or handling a traffic spike.

## Mistake 5: Running `drop_caches` as a fix

This is not a standard performance remedy.

## Mistake 6: Looking only at host metrics

Application, database, dependency, and load balancer metrics are also required.

## Mistake 7: Alerting on every short spike

Use duration, baselines, and service impact.

## Mistake 8: Ignoring cloud limits

Check CPU credits, EBS limits, network limits, and instance sizing.

---

# 19. Command Reference

## CPU

```bash
uptime
top
mpstat -P ALL 5 5
vmstat 5 10
pidstat -u 5 5
ps -eo pid,%cpu,cmd --sort=-%cpu | head
```

## Memory

```bash
free -h
vmstat 5 10
ps -eo pid,%mem,rss,cmd --sort=-%mem | head
swapon --show
journalctl -k -g 'oom|out of memory'
```

## Disk

```bash
df -h
df -ih
lsblk
findmnt
iostat -xz 5 5
pidstat -d 5 5
sudo iotop -oPa
sudo lsof +L1
```

## Network

```bash
ss -tulpen
ss -s
ip -s link
ip route
nstat -az
sar -n DEV 5 5
tracepath example.com
nc -vz example.com 443
```

## Processes

```bash
ps -ef
pstree -ap
pgrep -af java
systemctl show myapp -p MainPID
sudo lsof -p PID
cat /proc/PID/limits
```

---

# 20. Interview Questions and Answers

## Q1. What is the difference between load average and CPU utilization?

CPU utilization shows how much CPU time is being used. Load average represents runnable tasks plus tasks in uninterruptible sleep. High load with low CPU may indicate I/O wait.

## Q2. A server has load average 12 on a 16-vCPU machine. Is it overloaded?

Not necessarily. Load must be interpreted relative to CPU count and I/O wait. A load of 12 may be acceptable on 16 CPUs, but application latency and saturation metrics are still required.

## Q3. Why can memory usage be high while the system is healthy?

Linux uses free memory for cache. The `available` metric is more useful than `free` alone.

## Q4. How do you identify an OOM kill?

```bash
journalctl -k -g 'oom|out of memory|killed process'
```

Then correlate with memory usage and process RSS.

## Q5. What does high `%wa` indicate?

High `%wa` indicates CPU time spent waiting for I/O. Investigate disk latency, queue depth, blocked processes, and storage limits.

## Q6. What does high `%st` mean on a VM?

CPU steal time means the VM's requested CPU time was taken by the hypervisor for other workloads.

## Q7. How do you identify the process consuming the most CPU?

```bash
ps -eo pid,%cpu,cmd --sort=-%cpu | head
```

Or use `top` and press `Shift+P`.

## Q8. How do you identify a disk bottleneck?

Use:

```bash
iostat -xz 5 5
pidstat -d 5 5
```

Look at `await`, queue size, throughput, and device utilization.

## Q9. Why can `df` show more usage than `du`?

Deleted files may still be open by running processes.

```bash
sudo lsof +L1
```

## Q10. How do you troubleshoot high API latency when CPU is normal?

Check request rate, error rate, database latency, external dependencies, connection pools, DNS, TCP retransmissions, and application logs.

## Q11. What are the four golden signals?

Latency, traffic, errors, and saturation.

## Q12. Why are p95 and p99 useful?

They expose tail latency that averages can hide.

## Q13. What is the difference between `RSS` and `VSZ`?

RSS is resident physical memory. VSZ is virtual address space and may include mapped but unused memory.

## Q14. What is a zombie process?

A zombie is a terminated child process whose parent has not collected its exit status. It does not consume normal CPU or memory like a running process.

## Q15. Why should you not restart a service immediately during an incident?

Restarting may temporarily hide the issue and destroy useful evidence. Collect enough data first unless immediate mitigation is necessary.

## Q16. How do you troubleshoot a slow Jenkins agent?

Check CPU, memory, disk capacity, inode usage, I/O latency, concurrent executors, workspace size, Java memory, and build logs.

## Q17. How do Prometheus and node_exporter help?

Node Exporter exposes host metrics. Prometheus stores and queries them. Grafana visualizes them and alerting rules identify sustained saturation.

## Q18. What is CPU throttling or CPU credit exhaustion?

It is a restriction on available CPU capacity due to resource limits or exhausted burst credits. It can cause latency even when the application expects more CPU.

## Q19. What is the difference between throughput and latency?

Throughput is work completed per unit time. Latency is the time required for one operation. A system can have high throughput and poor tail latency.

## Q20. What is the correct performance fix?

The correct fix is based on evidence: optimize code, tune configuration, remove contention, increase capacity, scale out, repair storage/network, or roll back a regression.

---

# 21. Interview Scenario: Explain Your Investigation

A strong answer follows this format:

```text
First, I define the impact and check whether the issue is isolated or global.
Then I compare the current metrics with the normal baseline.
I check CPU, load, memory, swap, disk latency, network errors, and process-level usage.
Next, I correlate host metrics with application latency, errors, logs, database metrics, and recent deployments.
I apply the smallest safe mitigation, verify recovery, and document the root cause.
```

This demonstrates operational maturity because it shows:

- Structured diagnosis.
- Evidence-based decisions.
- Awareness of application and infrastructure layers.
- Safe production behavior.
- Verification after remediation.

---

# 22. Final DevOps Principles

1. **Measure before changing.**
2. **Use baselines, not arbitrary thresholds.**
3. **Load average is not CPU utilization.**
4. **Memory used is not automatically memory pressure.**
5. **Disk capacity and disk latency are different.**
6. **Host health does not guarantee application health.**
7. **Correlate metrics, logs, deployments, and dependencies.**
8. **Cloud instances have provider-specific limits.**
9. **Prefer reversible mitigations.**
10. **Always verify recovery and document the root cause.**
