# Processes, Jobs and Application Lifecycle for DevOps

## Scope

This guide explains Linux processes from a **DevOps Engineer's point of view**.

The focus is on:

- Running and supervising applications
- Jenkins and CI/CD execution
- systemd services
- Application restarts and graceful shutdowns
- CPU and memory troubleshooting
- Zombie, orphan and stuck processes
- Process-level networking and file inspection
- Production deployment and rollback scenarios

---

# 1. What is a process?

A process is a running instance of a program.

Examples in a DevOps environment:

- `nginx`
- Java Spring Boot application
- Python Flask application
- Go API
- Jenkins agent process
- Ansible process
- Terraform process
- Prometheus or Grafana process

Every process normally has:

- A Process ID, or PID
- A parent process ID, or PPID
- A user identity
- An environment
- Open file descriptors
- A current working directory
- A process state
- Resource usage such as CPU and memory

Check a process:

```bash
ps -ef | grep nginx
```

Better:

```bash
pgrep -a nginx
```

---

# 2. What are PID and PPID?

- **PID**: ID of the current process
- **PPID**: ID of the parent process

Display them:

```bash
ps -eo pid,ppid,user,stat,cmd
```

Example:

```text
PID   PPID  USER   STAT CMD
1     0     root   Ss   /sbin/init
900   1     root   Ssl  /usr/sbin/sshd
1500  900   ubuntu S    bash
1600  1500  ubuntu S    java -jar app.jar
```

## DevOps relevance

Understanding PPID helps identify:

- Which service started an application
- Whether a process was launched by Jenkins
- Whether a process became orphaned
- Which supervisor should restart it

---

# 3. Why is PID 1 important?

On a normal system, PID 1 is the first userspace process.

On Ubuntu, it is usually `systemd`.

PID 1 is responsible for important system-level process management, including:

- Starting services
- Reaping orphaned processes
- Coordinating shutdown
- Managing service dependencies

Check it:

```bash
ps -p 1 -o pid,comm,args
```

In containers, PID 1 may be the application itself or a container init process. This matters because PID 1 has special signal-handling and child-reaping responsibilities.

---

# 4. Process states

Display process state:

```bash
ps -eo pid,stat,cmd
```

Common states:

| State | Meaning | DevOps relevance |
|---|---|---|
| `R` | Running or runnable | Consuming or waiting for CPU |
| `S` | Interruptible sleep | Normal for many idle applications |
| `D` | Uninterruptible sleep | Often blocked on I/O |
| `T` | Stopped or traced | Debugging or job-control issue |
| `Z` | Zombie | Exited child not yet reaped |
| `I` | Idle kernel thread | Usually kernel-related |

## Important interview point

A process in `D` state may not respond immediately even to:

```bash
kill -9 <PID>
```

The process is often waiting inside the kernel for an operation such as disk or network I/O to complete.

---

# 5. How do you view running processes?

## `ps`

Snapshot of processes:

```bash
ps aux
```

Process tree:

```bash
ps -ef --forest
```

Useful custom output:

```bash
ps -eo pid,ppid,stat,user,%cpu,%mem,etime,cmd --sort=-%cpu
```

Sort by memory:

```bash
ps -eo pid,ppid,user,%mem,%cpu,cmd --sort=-%mem
```

## `top`

```bash
top
```

Useful interactive keys:

- `P`: Sort by CPU
- `M`: Sort by memory
- `1`: Show individual CPU cores
- `k`: Send a signal to a process
- `c`: Show full command line
- `H`: Show threads
- `q`: Quit

## `htop`

```bash
htop
```

`htop` is easier to navigate but may not be installed by default.

---

# 6. How do you find a process by name?

```bash
pgrep -a java
pgrep -af "java -jar"
```

Find the PID of a service:

```bash
pidof nginx
```

Find matching processes:

```bash
ps -ef | grep '[n]ginx'
```

The bracket pattern avoids matching the `grep` command itself.

---

# 7. What is the difference between `kill`, `pkill` and `killall`?

## Kill by PID

```bash
kill <PID>
```

Default signal is usually `SIGTERM`.

## Kill by process name

```bash
pkill -TERM -f "java -jar app.jar"
```

## Kill all matching process names

```bash
killall nginx
```

Use name-based commands carefully. A broad pattern may terminate unrelated processes.

Prefer a service manager such as systemd for production applications.

---

# 8. Important Linux signals

| Signal | Number | Purpose |
|---|---:|---|
| `SIGTERM` | 15 | Request graceful termination |
| `SIGKILL` | 9 | Force termination; cannot be caught or ignored |
| `SIGINT` | 2 | Interrupt, commonly Ctrl+C |
| `SIGHUP` | 1 | Hangup; often used by daemons to reload configuration |
| `SIGSTOP` | 19 | Stop process; cannot be caught |
| `SIGCONT` | 18 | Continue stopped process |

Examples:

```bash
kill -TERM 1234
kill -HUP 1234
kill -9 1234
```

## Production rule

Use this order when possible:

1. Ask the application to shut down gracefully.
2. Wait for active requests or jobs to finish.
3. Send `SIGTERM`.
4. Use `SIGKILL` only when the process does not exit and the impact is understood.

`SIGKILL` prevents application cleanup such as:

- Closing database connections
- Flushing buffers
- Completing transactions
- Removing temporary files

---

# 9. What is graceful shutdown?

Graceful shutdown means stopping an application without abruptly interrupting active work.

A production application may need to:

- Stop accepting new requests
- Finish active requests
- Stop background workers
- Close database connections
- Flush logs
- Commit or safely abandon work
- Exit with a meaningful status

For a systemd service:

```bash
sudo systemctl stop myapp
```

For a process:

```bash
kill -TERM <PID>
```

A well-designed application should handle `SIGTERM`.

## DevOps interview point

Graceful shutdown is essential during:

- Rolling deployments
- Auto Scaling instance termination
- Kubernetes pod termination
- Blue-green deployments
- Load balancer deregistration

---

# 10. Foreground and background processes

Run in the foreground:

```bash
./app
```

Run in the background:

```bash
./app &
```

Show current shell jobs:

```bash
jobs -l
```

Move a job to the foreground:

```bash
fg %1
```

Resume a stopped job in the background:

```bash
bg %1
```

Suspend a foreground process:

```text
Ctrl+Z
```

Terminate a foreground process:

```text
Ctrl+C
```

## Why this matters in DevOps

A command started in a terminal may stop when:

- The SSH session disconnects
- The shell exits
- The terminal closes
- The CI job finishes

For production applications, use systemd, a container runtime, or another process supervisor instead of relying on shell backgrounding.

---

# 11. `nohup`, `disown` and `setsid`

## `nohup`

```bash
nohup ./app > app.log 2>&1 &
```

This protects the process from the normal hangup signal.

## `disown`

```bash
./app &
disown
```

This removes the job from the shell's job table.

## `setsid`

```bash
setsid ./app > app.log 2>&1 &
```

Starts the command in a new session.

## Important limitation

These commands are useful for temporary troubleshooting or one-off tasks, but they do not provide:

- Automatic restart
- Health checks
- Dependency ordering
- Structured status
- Centralized logs
- Resource limits
- Deployment lifecycle management

For long-running services, prefer systemd.

---

# 12. Process vs daemon vs service

| Term | Meaning |
|---|---|
| Process | Running instance of a program |
| Daemon | Background process designed to provide a service |
| Service | A managed unit of functionality, often controlled by systemd |
| Worker | Process that performs background tasks |
| Supervisor | Component that starts, stops and monitors processes |

Examples:

- `nginx` worker process
- Java API process
- Celery worker
- Jenkins agent
- systemd service unit

A service can manage one process or multiple processes.

---

# 13. Why is systemd better than `./app &`?

A systemd unit can provide:

- Automatic startup after boot
- Restart on failure
- Dependency ordering
- Standardized logs
- Environment configuration
- Resource limits
- Security hardening
- Explicit stop and restart behavior
- Status and health visibility

Example:

```bash
sudo systemctl status myapp
sudo systemctl restart myapp
sudo journalctl -u myapp -f
```

Example unit:

```ini
[Unit]
Description=My Application
After=network-online.target
Wants=network-online.target

[Service]
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/java -jar /opt/myapp/app.jar
Restart=on-failure
RestartSec=5
EnvironmentFile=/etc/myapp/myapp.env

[Install]
WantedBy=multi-user.target
```

Apply it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now myapp
```

---

# 14. How do you inspect a systemd service's process?

```bash
systemctl status myapp
systemctl show myapp -p MainPID
systemctl show myapp -p ExecMainStatus
systemctl show myapp -p SubState
```

Find all processes in the service cgroup:

```bash
systemctl status myapp
systemctl show myapp -p ControlGroup
```

Inspect the process tree:

```bash
pstree -ap "$(systemctl show -p MainPID --value myapp)"
```

This is more reliable than searching only by process name.

---

# 15. What is an exit code?

A process returns an exit status when it finishes.

Convention:

- `0`: Success
- Non-zero: Failure or special condition

Check the last command's exit code:

```bash
echo $?
```

Example:

```bash
systemctl show myapp -p ExecMainStatus
```

In Bash:

```bash
./deploy.sh
status=$?

if [ "$status" -ne 0 ]; then
  echo "Deployment failed"
  exit "$status"
fi
```

## CI/CD relevance

Jenkins, GitHub Actions and other CI systems use exit codes to determine whether a stage succeeded or failed.

A script that prints an error but exits `0` may incorrectly mark a deployment as successful.

---

# 16. What are orphan processes?

An orphan process is a process whose original parent has exited.

The process is adopted by another process, normally PID 1 or a subreaper.

Find PPID:

```bash
ps -eo pid,ppid,stat,cmd
```

Orphan processes are not automatically bad. They become a problem when they:

- Continue running after a deployment
- Keep ports open
- Consume memory
- Duplicate workers
- Escape CI job cleanup

A proper supervisor should own the application lifecycle.

---

# 17. What are zombie processes?

A zombie is a process that has exited but whose parent has not yet collected its exit status.

Find zombies:

```bash
ps -eo pid,ppid,stat,cmd | awk '$3 ~ /Z/'
```

Or:

```bash
ps aux | awk '$8 ~ /^Z/'
```

A zombie does not consume normal CPU or memory like a running process, but it consumes a process table entry.

## How to troubleshoot

1. Identify the parent:

```bash
ps -o pid,ppid,stat,cmd -p <ZOMBIE_PID>
```

2. Inspect the parent:

```bash
ps -fp <PPID>
```

3. Fix the parent application so it calls `wait()` or correctly reaps children.

4. Restart the parent if safe.

Killing a zombie itself usually does not work because it has already exited.

---

# 18. CPU priority: `nice` and `renice`

Start with lower CPU scheduling priority:

```bash
nice -n 10 ./batch-job.sh
```

Change priority of an existing process:

```bash
renice 10 -p <PID>
```

Lower nice values generally mean higher priority. Increasing priority may require elevated privileges.

## DevOps use cases

- Run backup jobs with lower priority
- Prevent batch processing from affecting APIs
- Reduce impact of large compression jobs
- Separate interactive workloads from background jobs

Do not use priority changes as a substitute for proper capacity planning.

---

# 19. CPU affinity

CPU affinity controls which CPU cores a process can use.

Check affinity:

```bash
taskset -cp <PID>
```

Start on selected CPUs:

```bash
taskset -c 0,1 ./app
```

CPU affinity can be useful for specialized workloads, but it can also reduce scheduler flexibility. Use it only when measurements justify it.

---

# 20. Inspect a process through `/proc`

Linux exposes process information under:

```text
/proc/<PID>/
```

Useful files:

| Path | Information |
|---|---|
| `/proc/<PID>/cmdline` | Command line |
| `/proc/<PID>/environ` | Environment |
| `/proc/<PID>/cwd` | Current working directory |
| `/proc/<PID>/exe` | Executable |
| `/proc/<PID>/fd/` | Open file descriptors |
| `/proc/<PID>/limits` | Resource limits |
| `/proc/<PID>/status` | Process status |
| `/proc/<PID>/smaps` | Detailed memory mappings |
| `/proc/<PID>/net/` | Network-related information |

Examples:

```bash
readlink -f /proc/<PID>/exe
readlink -f /proc/<PID>/cwd
tr '\0' '\n' < /proc/<PID>/environ
cat /proc/<PID>/limits
ls -l /proc/<PID>/fd
```

Access to environment and file information may require the same user or root privileges.

---

# 21. How do you find files opened by a process?

Use `lsof`:

```bash
sudo lsof -p <PID>
```

Find which process has a file open:

```bash
sudo lsof /var/log/myapp/app.log
```

Find which process is using a port:

```bash
sudo lsof -i :8080
```

## DevOps scenarios

- Log file cannot be deleted or rotated
- Port is already in use
- Application still uses an old deployment directory
- Disk space is consumed by deleted-but-open files

Find deleted files still held open:

```bash
sudo lsof +L1
```

The disk space may not be released until the process closes the file.

---

# 22. How do you map a listening port to a process?

```bash
sudo ss -ltnp
```

For a specific port:

```bash
sudo ss -ltnp 'sport = :8080'
```

Alternative:

```bash
sudo lsof -nP -iTCP:8080 -sTCP:LISTEN
```

Example troubleshooting flow:

```bash
curl -v http://127.0.0.1:8080/health
sudo ss -ltnp 'sport = :8080'
ps -fp <PID>
sudo lsof -p <PID>
```

A process listening on a port does not automatically mean the application is healthy. The socket may accept connections while the application is failing internally.

---

# 23. Process environment and configuration

Inspect environment variables:

```bash
tr '\0' '\n' < /proc/<PID>/environ
```

For systemd:

```bash
systemctl show myapp -p Environment
```

Be careful: environment variables may contain secrets.

Do not expose values such as:

- Database passwords
- Cloud credentials
- API tokens
- SMTP passwords
- Private keys

Prefer secure configuration mechanisms such as:

- Restricted environment files
- IAM roles
- AWS Systems Manager Parameter Store
- AWS Secrets Manager
- Vault
- Kubernetes Secrets with appropriate controls

---

# 24. Resource limits

View shell limits:

```bash
ulimit -a
```

View process limits:

```bash
cat /proc/<PID>/limits
```

Common limits:

- Maximum open files
- Maximum processes
- Maximum locked memory
- Maximum core dump size
- Maximum stack size

For systemd:

```ini
[Service]
LimitNOFILE=65535
TasksMax=4096
```

After changing a unit:

```bash
sudo systemctl daemon-reload
sudo systemctl restart myapp
```

A common production failure is:

```text
Too many open files
```

This may indicate either:

- An insufficient file descriptor limit
- A file descriptor leak
- Too many connections
- Missing connection cleanup

Do not increase limits blindly; investigate usage first.

---

# 25. Application lifecycle during deployment

A typical deployment lifecycle is:

```text
Build
  |
Package artifact
  |
Transfer artifact
  |
Stop or drain old version
  |
Start new version
  |
Health check
  |
Register with load balancer
  |
Monitor
  |
Rollback if required
```

The process-management part must answer:

- Who starts the application?
- Who stops it?
- How is the PID tracked?
- What happens if startup fails?
- What happens if the process crashes?
- How are active requests drained?
- How is the old version rolled back?
- How are logs and metrics preserved?

A production deployment should not depend on manually finding a PID and running `kill -9`.

---

# 26. Safe restart pattern

A safer restart process is:

```bash
sudo systemctl restart myapp
sudo systemctl is-active --quiet myapp
curl --fail --silent http://127.0.0.1:8080/health
```

More explicit:

```bash
sudo systemctl stop myapp

# Optional: wait for graceful shutdown
sleep 3

sudo systemctl start myapp
sudo systemctl is-active --quiet myapp
curl --fail http://127.0.0.1:8080/health
```

In production, use:

- Readiness checks
- Load balancer draining
- Deployment locks
- Timeouts
- Rollback logic
- Versioned artifacts

---

# 27. Scenario: Java process is consuming high memory

## Symptoms

```bash
top
ps -eo pid,%mem,%cpu,rss,vsz,cmd --sort=-%mem | head
```

## Investigation

```bash
PID=<java_pid>

ps -p "$PID" -o pid,ppid,%cpu,%mem,rss,vsz,etime,cmd
cat /proc/"$PID"/status
cat /proc/"$PID"/limits
sudo lsof -p "$PID" | wc -l
```

Check kernel OOM events:

```bash
sudo journalctl -k | grep -i -E 'out of memory|killed process'
```

Check JVM details if available:

```bash
jcmd <PID> VM.flags
jcmd <PID> GC.heap_info
jcmd <PID> Thread.print
```

Possible causes:

- Heap too large
- Native memory growth
- Memory leak
- Too many threads
- Excessive buffers
- File descriptor or connection leak
- Host memory pressure

Do not immediately kill the process. First collect evidence and determine whether the application is serving traffic.

---

# 28. Scenario: Process is running but application is unavailable

Check:

```bash
systemctl status myapp
sudo ss -ltnp
curl -v http://127.0.0.1:8080/health
sudo journalctl -u myapp -n 100 --no-pager
```

Possible causes:

- Listening only on `127.0.0.1` when external access is required
- Wrong port
- Application deadlock
- Dependency failure
- Database connection exhaustion
- Firewall or security group issue
- Reverse proxy misconfiguration
- Health endpoint failure

A running PID is not a health check.

---

# 29. Scenario: Service repeatedly restarts

Inspect:

```bash
systemctl status myapp
journalctl -u myapp -b --no-pager
systemctl show myapp -p Restart -p RestartUSec
```

Check the exit status:

```bash
systemctl show myapp -p ExecMainStatus -p ExecMainCode
```

Common causes:

- Invalid configuration
- Missing environment variable
- Wrong file permissions
- Missing executable
- Port already in use
- Incorrect Java/Python runtime
- Dependency unavailable
- Application exits immediately
- OOM termination

Do not simply increase `RestartSec`. Find the first failure in the logs.

---

# 30. Scenario: Port is already in use after deployment

```bash
sudo ss -ltnp 'sport = :8080'
sudo lsof -nP -iTCP:8080 -sTCP:LISTEN
ps -fp <PID>
```

Determine whether:

- The previous version is still running
- A duplicate process started
- Another service owns the port
- A stale process escaped the CI job

Preferred fix:

```bash
sudo systemctl stop myapp
sudo systemctl start myapp
```

Avoid broad commands such as:

```bash
pkill -f java
```

They may terminate unrelated Java applications.

---

# 31. Scenario: Zombie processes are increasing

Investigate:

```bash
ps -eo pid,ppid,stat,cmd | awk '$3 ~ /Z/'
```

Group by parent:

```bash
ps -eo ppid=,stat= | awk '$2 ~ /Z/ {count[$1]++} END {for (p in count) print p,count[p]}'
```

Then inspect the parent process.

The real fix is normally in the parent application or worker framework. Restarting the host may clear the symptom, but it does not fix the defect.

---

# 32. Scenario: Jenkins job leaves application processes behind

Common causes:

- Application started directly in the build shell
- Process detached from the job
- No systemd ownership
- Incorrect cleanup
- Deployment script does not stop the previous version
- Child processes survive after the shell exits

Better design:

1. Build artifact in Jenkins.
2. Copy artifact to the target host.
3. Let systemd own the application.
4. Use `systemctl restart myapp`.
5. Validate the service.
6. Keep deployment and runtime lifecycle separate.

This avoids making Jenkins the permanent process supervisor.

---

# 33. Scenario: Process was killed by the OOM killer

Check:

```bash
sudo journalctl -k | grep -i -E 'oom|out of memory|killed process'
dmesg -T | grep -i -E 'oom|out of memory|killed process'
```

Check memory:

```bash
free -h
vmstat 1 5
ps -eo pid,%mem,rss,cmd --sort=-%mem | head
```

Possible remediation:

- Fix memory leak
- Tune application heap
- Reduce concurrency
- Add memory limits
- Increase instance size
- Add swap only when appropriate
- Improve alerting
- Separate workloads

Do not assume the process with the highest current memory usage was necessarily the process killed earlier; correlate timestamps from kernel logs.

---

# 34. Scenario: Graceful deployment behind a load balancer

A safer sequence:

1. Mark the instance or target as draining.
2. Stop sending new requests.
3. Wait for active requests to finish.
4. Send `SIGTERM` or stop the systemd service.
5. Deploy the new artifact.
6. Start the service.
7. Run local health checks.
8. Wait for load balancer health checks.
9. Re-enable traffic.
10. Monitor errors, latency and saturation.

This is more reliable than restarting an application while it is still receiving traffic.

---

# 35. Useful command reference

| Requirement | Command |
|---|---|
| List processes | `ps aux` |
| Process tree | `ps -ef --forest` |
| Find process | `pgrep -af pattern` |
| Interactive view | `top` |
| Better interactive view | `htop` |
| Process details | `ps -fp PID` |
| Send graceful signal | `kill -TERM PID` |
| Force kill | `kill -KILL PID` |
| Jobs | `jobs -l` |
| Foreground job | `fg %1` |
| Background job | `bg %1` |
| Open files | `lsof -p PID` |
| Port owner | `ss -ltnp` |
| Process executable | `readlink -f /proc/PID/exe` |
| Process environment | `tr '\0' '\n' < /proc/PID/environ` |
| Process limits | `cat /proc/PID/limits` |
| Service status | `systemctl status service` |
| Service logs | `journalctl -u service` |
| Main service PID | `systemctl show service -p MainPID` |
| Kernel OOM logs | `journalctl -k` |
| CPU priority | `nice`, `renice` |
| CPU affinity | `taskset` |

---

# 36. Interview questions to practice

1. What is the difference between a process, daemon and service?
2. What are PID and PPID?
3. Why is PID 1 important?
4. Explain Linux process states.
5. Why might `kill -9` fail to terminate a process in `D` state?
6. What is the difference between `SIGTERM` and `SIGKILL`?
7. What happens when you run `./app &` over SSH?
8. Why is `nohup` not a replacement for systemd?
9. What is the difference between an orphan and a zombie process?
10. How do you identify a process consuming the most memory?
11. How do you find which process owns port 8080?
12. How do you inspect a process's environment?
13. How do you find deleted files still held open?
14. What does an exit code of `0` mean?
15. Why might a Jenkins job leave child processes behind?
16. How do you troubleshoot a service that keeps restarting?
17. How do you distinguish a running process from a healthy application?
18. What is graceful shutdown and why is it important?
19. How do systemd restart policies work?
20. How would you deploy a new version without abruptly dropping traffic?
21. How do you investigate an OOM-killed process?
22. What are `nice` and `renice` used for?
23. What are resource limits and why does `LimitNOFILE` matter?
24. Why should Jenkins build the artifact but systemd own the runtime?
25. What evidence would you collect before killing a production process?

---

# 37. Interview checklist

You should be able to explain and demonstrate:

- [ ] PID, PPID and process trees
- [ ] Process states
- [ ] `ps`, `top`, `pgrep`, `pkill`
- [ ] Linux signals
- [ ] Graceful versus forceful termination
- [ ] Shell jobs and background processes
- [ ] Limitations of `nohup`
- [ ] systemd process ownership
- [ ] Exit codes in CI/CD
- [ ] Orphan and zombie processes
- [ ] `/proc` inspection
- [ ] `lsof` and `ss`
- [ ] Resource limits
- [ ] OOM investigation
- [ ] Jenkins child-process cleanup
- [ ] Safe application restart
- [ ] Load balancer draining
- [ ] Rollback and health checks

---

# Key DevOps principle

**A production application should have a clear owner for its lifecycle.**

Jenkins should build and deploy the artifact. A process supervisor such as systemd, a container runtime or Kubernetes should own the running application.

If nobody clearly owns the process, you will eventually see:

- Duplicate application instances
- Orphaned workers
- Port conflicts
- Uncontrolled restarts
- Missing logs
- Broken deployments
- Difficult rollbacks
