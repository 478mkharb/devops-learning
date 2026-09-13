# systemd and Managing Production Services for DevOps

## Scope

This guide covers `systemd` from a DevOps Engineer's perspective.

The focus is on:

- Running applications reliably
- Service startup and restart behavior
- Deployment automation
- Logs and troubleshooting
- Environment files
- Dependencies
- Resource limits
- Security controls
- Health validation
- Production incident scenarios

---

# 1. What is systemd?

`systemd` is the service manager used by most modern Linux distributions, including Ubuntu.

It manages:

- System services
- Application processes
- Startup ordering
- Service dependencies
- Timers
- Mounts
- Sockets
- Resource controls
- Service logs through journald

Check whether systemd is PID 1:

```bash
ps -p 1 -o pid,comm,args
```

Check systemd version:

```bash
systemd --version
```

## DevOps relevance

systemd is commonly used to manage:

- Java Spring Boot applications
- Python Flask or FastAPI services
- Go APIs
- NGINX
- Prometheus
- Node Exporter
- Custom workers
- Ansible-deployed applications
- EC2-hosted services

---

# 2. What is a systemd unit?

A unit is a resource managed by systemd.

Common unit types:

| Unit | Purpose |
|---|---|
| `.service` | Long-running or one-shot service |
| `.timer` | Scheduled task |
| `.socket` | Socket activation |
| `.target` | Grouping and synchronization |
| `.mount` | Filesystem mount |
| `.path` | Watches filesystem paths |
| `.slice` | Resource-management group |

List units:

```bash
systemctl list-units
```

List service units:

```bash
systemctl list-units --type=service
```

List installed service files:

```bash
systemctl list-unit-files --type=service
```

---

# 3. Service states

Check service status:

```bash
systemctl status nginx
```

Common states:

- `active (running)`: Service is running
- `active (exited)`: Command completed successfully; common for one-shot services
- `inactive`: Not running
- `failed`: Startup or execution failed
- `activating`: Starting
- `deactivating`: Stopping
- `masked`: Explicitly prevented from starting

Check only the active state:

```bash
systemctl is-active nginx
```

Check whether enabled at boot:

```bash
systemctl is-enabled nginx
```

Check whether it failed:

```bash
systemctl is-failed nginx
```

---

# 4. Basic service commands

```bash
sudo systemctl start myapp
sudo systemctl stop myapp
sudo systemctl restart myapp
sudo systemctl reload myapp
sudo systemctl status myapp
```

Enable at boot:

```bash
sudo systemctl enable myapp
```

Enable and start immediately:

```bash
sudo systemctl enable --now myapp
```

Disable from boot:

```bash
sudo systemctl disable myapp
```

Stop and disable:

```bash
sudo systemctl disable --now myapp
```

## Important distinction

- `start`: Starts the service now.
- `enable`: Configures startup during boot.
- `restart`: Stops and starts the service.
- `reload`: Asks the application to reload configuration without a full restart, if supported.

`enable` does not normally start a service immediately unless `--now` is used.

---

# 5. Where should custom service files be stored?

For locally created services, use:

```text
/etc/systemd/system/
```

Example:

```text
/etc/systemd/system/myapp.service
```

Distribution-provided units are commonly located under:

```text
/usr/lib/systemd/system/
```

or:

```text
/lib/systemd/system/
```

Inspect the exact unit path:

```bash
systemctl show -p FragmentPath myapp
```

View the complete unit:

```bash
systemctl cat myapp
```

## DevOps recommendation

Keep custom application unit files in your configuration repository and deploy them through Ansible or another configuration-management system.

---

# 6. Basic production service unit

Example for a Java application:

```ini
[Unit]
Description=OT Microservices Salary API
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=appuser
Group=appuser
WorkingDirectory=/opt/otms/salary-api
ExecStart=/usr/bin/java -jar /opt/otms/salary-api/salary-api.jar
Restart=on-failure
RestartSec=5
EnvironmentFile=-/etc/otms/salary-api.env

[Install]
WantedBy=multi-user.target
```

Create the file:

```bash
sudo nano /etc/systemd/system/salary-api.service
```

Reload systemd:

```bash
sudo systemctl daemon-reload
```

Start and enable:

```bash
sudo systemctl enable --now salary-api
```

Verify:

```bash
systemctl status salary-api
```

---

# 7. What does `daemon-reload` do?

When a unit file changes, systemd must reread the unit definitions.

Run:

```bash
sudo systemctl daemon-reload
```

This does not restart the application.

Typical deployment sequence:

```bash
sudo cp myapp.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl restart myapp
```

## Interview point

Use `daemon-reload` after changing:

- Unit files
- Drop-in configuration
- Dependencies
- Environment-related unit settings

Do not run it after every ordinary application code deployment unless the unit configuration changed.

---

# 8. `ExecStart`, `ExecStop` and `ExecReload`

Example:

```ini
[Service]
ExecStart=/opt/myapp/bin/start
ExecStop=/opt/myapp/bin/stop
ExecReload=/opt/myapp/bin/reload
```

Important points:

- `ExecStart` defines the main start command.
- `ExecStop` defines custom stop behavior.
- `ExecReload` defines how configuration is reloaded.
- Commands should use absolute paths.
- Do not rely on interactive shell aliases.
- Avoid shell-specific syntax unless explicitly using a shell.

If shell features are required:

```ini
ExecStart=/bin/bash -c '/opt/myapp/start.sh --environment production'
```

Use shell wrappers only when necessary because they add another process layer and can complicate signal handling.

---

# 9. Service types

Common `Type=` values:

| Type | Use |
|---|---|
| `simple` | Main process starts immediately |
| `exec` | systemd waits until the executable is successfully invoked |
| `forking` | Program forks into the background |
| `oneshot` | Command performs a task and exits |
| `notify` | Application signals readiness to systemd |
| `idle` | Delays execution until other jobs finish |

For most modern foreground applications, prefer:

```ini
Type=exec
```

or:

```ini
Type=simple
```

Avoid `Type=forking` unless the application is designed to daemonize.

---

# 10. Why should applications run in the foreground?

A foreground process is easier for systemd to supervise.

Preferred:

```ini
ExecStart=/usr/bin/java -jar /opt/myapp/app.jar
```

Avoid:

```ini
ExecStart=/opt/myapp/start.sh
```

where `start.sh` contains:

```bash
java -jar app.jar &
```

The backgrounded process may escape systemd's direct supervision.

## DevOps principle

Let the service manager own the main application process. Do not double-daemonize applications.

---

# 11. Restart policies

Common options:

```ini
Restart=no
Restart=on-failure
Restart=on-abnormal
Restart=always
```

Example:

```ini
Restart=on-failure
RestartSec=5
```

Meaning:

- Restart after a failure
- Wait five seconds before restarting

Useful controls:

```ini
StartLimitIntervalSec=60
StartLimitBurst=5
```

These prevent an endlessly failing service from restarting without limit.

Inspect restart configuration:

```bash
systemctl show myapp -p Restart -p RestartUSec
```

## Important

Automatic restart does not fix the underlying problem. Always inspect the first failure in the journal.

---

# 12. Restart versus reload

## Restart

```bash
sudo systemctl restart nginx
```

The process is stopped and started again.

Potential impact:

- Active connections may be interrupted
- In-memory state may be lost
- Startup dependencies are reinitialized

## Reload

```bash
sudo systemctl reload nginx
```

The process remains running and rereads configuration if it supports reload.

Check whether reload is supported:

```bash
systemctl show nginx -p CanReload
```

For NGINX:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

Always validate configuration before reload.

---

# 13. Service dependencies

Example:

```ini
[Unit]
After=network-online.target postgresql.service
Requires=postgresql.service
```

Important directives:

| Directive | Meaning |
|---|---|
| `After=` | Ordering only |
| `Before=` | Reverse ordering |
| `Requires=` | Strong dependency relationship |
| `Wants=` | Weaker dependency |
| `PartOf=` | Couples lifecycle operations |
| `BindsTo=` | Stronger relationship tied to unit existence |

## Important interview point

`After=` does not itself start or require another service. It only controls ordering when both units are involved.

---

# 14. Network readiness is not application readiness

This is not enough:

```ini
After=network-online.target
```

The network being available does not guarantee:

- DNS works
- Database is ready
- Redis is ready
- Elasticsearch is ready
- Remote API is reachable
- Application dependencies are healthy

Use application-level readiness checks and retry logic.

For example:

```bash
curl --fail http://127.0.0.1:8080/health
```

A robust application should retry temporary dependency failures rather than failing permanently during boot.

---

# 15. Environment files

Example unit:

```ini
[Service]
EnvironmentFile=/etc/otms/myapp.env
```

Example environment file:

```bash
APP_ENV=production
PORT=8080
LOG_LEVEL=INFO
```

Use optional file syntax when appropriate:

```ini
EnvironmentFile=-/etc/otms/myapp.env
```

Inspect configured environment:

```bash
systemctl show myapp -p Environment
```

## Security warning

Do not place secrets directly in world-readable unit files.

Protect environment files:

```bash
sudo chown root:root /etc/otms/myapp.env
sudo chmod 600 /etc/otms/myapp.env
```

For cloud workloads, prefer:

- IAM roles
- AWS Systems Manager Parameter Store
- AWS Secrets Manager
- Vault
- Restricted secret files

---

# 16. WorkingDirectory and file paths

Example:

```ini
[Service]
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/python3 /opt/myapp/app.py
```

Use absolute paths for:

- Executables
- Scripts
- Configuration files
- Log paths
- Working directories

A service may fail even though the command works manually because:

- The working directory differs
- `PATH` differs
- Environment variables are missing
- The service runs as another user
- File permissions differ
- Relative paths resolve differently

Always reproduce the service context when troubleshooting.

---

# 17. Running as a dedicated service account

Example:

```ini
[Service]
User=appuser
Group=appuser
```

Benefits:

- Limits filesystem access
- Reduces blast radius
- Avoids running applications as root
- Improves auditability
- Supports least privilege

Create a system account:

```bash
sudo useradd --system --home /opt/myapp --shell /usr/sbin/nologin appuser
```

Ensure ownership:

```bash
sudo chown -R appuser:appuser /opt/myapp
```

Do not grant `sudo` access to the application account unless there is a specific, controlled requirement.

---

# 18. Service security hardening

Useful options:

```ini
[Service]
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=full
ProtectHome=true
UMask=027
```

Other controls may include:

```ini
ReadWritePaths=/var/lib/myapp
CapabilityBoundingSet=
RestrictAddressFamilies=AF_INET AF_INET6 AF_UNIX
```

Apply hardening carefully. Overly restrictive settings can break applications that need:

- Write access
- Network access
- Specific capabilities
- Home-directory access
- Temporary files

Test in staging before production rollout.

---

# 19. Resource limits

Example:

```ini
[Service]
LimitNOFILE=65535
TasksMax=4096
MemoryMax=2G
CPUQuota=200%
```

Meaning:

- `LimitNOFILE`: Maximum open file descriptors
- `TasksMax`: Maximum tasks/threads in the service
- `MemoryMax`: Memory limit
- `CPUQuota=200%`: Up to two CPU cores worth of CPU time

Inspect limits:

```bash
systemctl show myapp | grep -E 'LimitNOFILE|TasksMax|MemoryMax|CPUQuota'
```

Resource limits should be based on observed workload and capacity planning.

---

# 20. Logging with journald

View logs:

```bash
journalctl -u myapp
```

Follow logs:

```bash
journalctl -u myapp -f
```

Recent logs:

```bash
journalctl -u myapp -n 100 --no-pager
```

Logs since boot:

```bash
journalctl -u myapp -b
```

Logs within a time range:

```bash
journalctl -u myapp --since "30 minutes ago"
```

Show only errors:

```bash
journalctl -u myapp -p err
```

Show kernel logs:

```bash
journalctl -k
```

## DevOps practice

Use structured application logs and forward them to a centralized logging platform when required.

---

# 21. Standard output and error

For many services, applications can write logs to stdout and stderr.

Possible settings:

```ini
[Service]
StandardOutput=journal
StandardError=journal
```

View logs:

```bash
journalctl -u myapp -f
```

Avoid mixing multiple logging strategies without a clear plan.

For example, decide whether the application writes to:

- journald
- files collected by an agent
- stdout captured by a runtime
- A centralized logging service

---

# 22. Journal retention and disk usage

Check journal disk usage:

```bash
journalctl --disk-usage
```

Vacuum old logs:

```bash
sudo journalctl --vacuum-time=14d
```

Vacuum by size:

```bash
sudo journalctl --vacuum-size=1G
```

Check persistent journal configuration:

```bash
cat /etc/systemd/journald.conf
```

Do not delete logs blindly during an incident. Preserve relevant evidence before cleanup.

---

# 23. Troubleshooting a failed service

Start with:

```bash
systemctl status myapp
journalctl -u myapp -b --no-pager
```

Then inspect:

```bash
systemctl show myapp -p ExecMainCode -p ExecMainStatus -p MainPID
```

Check:

```bash
ls -l /opt/myapp
sudo -u appuser /usr/bin/java -version
sudo ss -ltnp
```

Typical causes:

- Incorrect executable path
- Missing environment file
- Permission denied
- Port already in use
- Missing runtime
- Invalid configuration
- Wrong working directory
- Database unavailable
- SELinux/AppArmor denial
- Service account cannot read files

---

# 24. Troubleshooting a service stuck in restart loop

Commands:

```bash
systemctl status myapp
journalctl -u myapp --since "10 minutes ago"
systemctl show myapp -p NRestarts
systemctl show myapp -p ExecMainStatus
```

Look for:

- Immediate process exit
- Configuration syntax errors
- Missing secrets
- Dependency failures
- Invalid command-line arguments
- Wrong Java/Python version
- Permission problems
- OOM kills

Temporarily stop the loop if necessary:

```bash
sudo systemctl stop myapp
```

Fix the root cause, then start it again.

---

# 25. Service start limits

If a service fails too many times, systemd may refuse further starts.

Check:

```bash
systemctl status myapp
```

Reset the failed state:

```bash
sudo systemctl reset-failed myapp
```

Then start:

```bash
sudo systemctl start myapp
```

`reset-failed` does not fix the application. It only clears the recorded failed state and start-limit counters.

---

# 26. Drop-in configuration

Instead of editing a vendor unit directly, create a drop-in:

```bash
sudo systemctl edit myapp
```

Example:

```ini
[Service]
Environment="LOG_LEVEL=DEBUG"
RestartSec=10
```

Inspect merged configuration:

```bash
systemctl cat myapp
```

After changes:

```bash
sudo systemctl daemon-reload
sudo systemctl restart myapp
```

Drop-ins are useful for environment-specific overrides without modifying the original unit file.

---

# 27. Deployment pattern with Ansible

A common deployment sequence:

```yaml
- name: Deploy application artifact
  copy:
    src: app.jar
    dest: /opt/myapp/app.jar
    owner: appuser
    group: appuser
    mode: "0750"

- name: Install systemd unit
  template:
    src: myapp.service.j2
    dest: /etc/systemd/system/myapp.service
    mode: "0644"
  notify:
    - Reload systemd
    - Restart application

- name: Ensure service is enabled and running
  systemd:
    name: myapp
    enabled: true
    state: started
```

Handlers:

```yaml
handlers:
  - name: Reload systemd
    systemd:
      daemon_reload: true

  - name: Restart application
    systemd:
      name: myapp
      state: restarted
```

## Important improvement

Use handlers so the service restarts only when the artifact or unit configuration changes.

---

# 28. Deployment pattern with Jenkins

A Jenkins pipeline may execute:

```bash
scp app.jar ubuntu@server:/opt/myapp/app.jar
ssh ubuntu@server 'sudo systemctl restart myapp'
ssh ubuntu@server 'sudo systemctl is-active --quiet myapp'
ssh ubuntu@server 'curl --fail http://127.0.0.1:8080/health'
```

For production:

- Prefer SSM or Ansible where available
- Use versioned artifact directories
- Avoid copying directly over a running file when unsafe
- Validate before switching traffic
- Capture logs on failure
- Have a rollback procedure

---

# 29. Atomic release directories

Instead of deploying directly into one directory:

```text
/opt/myapp/
  current -> releases/2026-09-13-001
  releases/
    2026-09-13-001/
    2026-09-12-003/
```

Deployment approach:

1. Copy artifact to a new release directory.
2. Validate files and configuration.
3. Update the `current` symlink.
4. Restart or reload the service.
5. Run health checks.
6. Roll back the symlink if needed.

Example:

```bash
ln -sfn /opt/myapp/releases/2026-09-13-001 /opt/myapp/current
sudo systemctl restart myapp
```

Make sure the systemd unit points to:

```ini
WorkingDirectory=/opt/myapp/current
```

---

# 30. Graceful shutdown settings

Useful settings:

```ini
[Service]
TimeoutStopSec=30
KillSignal=SIGTERM
SendSIGKILL=yes
```

Meaning:

- `TimeoutStopSec`: How long systemd waits for shutdown
- `KillSignal`: Signal sent during stop
- `SendSIGKILL`: Whether to force kill after timeout

Use application-aware shutdown behavior. A very long timeout can delay recovery; a very short timeout can interrupt active work.

---

# 31. Health checks after restart

Example:

```bash
sudo systemctl restart myapp

for i in {1..30}; do
  if curl --fail --silent http://127.0.0.1:8080/health; then
    echo "Application is healthy"
    exit 0
  fi
  sleep 2
done

echo "Application failed health check"
sudo journalctl -u myapp -n 100 --no-pager
exit 1
```

A strong health check should verify more than the TCP port. Depending on the service, it may check:

- Application process
- Database connectivity
- Required dependency availability
- Internal readiness
- Queue connectivity
- Critical configuration

Avoid making a liveness check depend on every external system unless that is intentional.

---

# 32. Scenario: `systemctl restart` succeeds but the app is not ready

A successful restart command only means systemd completed the requested operation. It does not guarantee application readiness.

Validate:

```bash
systemctl is-active myapp
curl --fail http://127.0.0.1:8080/health
journalctl -u myapp -n 50 --no-pager
```

Possible reasons:

- Application starts asynchronously
- Database migrations are running
- Port opens before readiness
- Background initialization failed
- Health endpoint is failing
- Dependency connection retries are ongoing

Always include a readiness check in CI/CD.

---

# 33. Scenario: Unit works manually but fails through systemd

Compare:

- User
- Working directory
- `PATH`
- Environment variables
- File permissions
- Shell
- Current directory
- Available credentials
- Runtime version

Test as the service user:

```bash
sudo -u appuser /usr/bin/java -jar /opt/myapp/app.jar
```

Inspect the unit:

```bash
systemctl cat myapp
```

Check logs:

```bash
journalctl -u myapp -b
```

The most common cause is that the interactive shell has environment variables or PATH entries that systemd does not have.

---

# 34. Scenario: Service cannot bind to its port

```bash
sudo ss -ltnp 'sport = :8080'
sudo lsof -nP -iTCP:8080 -sTCP:LISTEN
```

Then check:

- Existing application instance
- Duplicate deployment
- Incorrect service stop behavior
- Another application using the port
- IPv4 versus IPv6 binding
- Configuration mismatch

Avoid killing all processes matching a generic name.

---

# 35. Scenario: Service was killed by OOM

Inspect kernel logs:

```bash
journalctl -k | grep -i -E 'oom|out of memory|killed process'
```

Inspect service limits:

```bash
systemctl show myapp -p MemoryMax
```

Inspect host memory:

```bash
free -h
vmstat 1 5
```

Possible actions:

- Fix memory leak
- Tune JVM heap or worker count
- Set an appropriate memory limit
- Increase instance capacity
- Reduce concurrency
- Improve memory alerts
- Separate workloads

Do not simply increase `MemoryMax` without understanding host capacity.

---

# 36. Scenario: Service must be stopped before maintenance

```bash
sudo systemctl stop myapp
sudo systemctl is-active myapp
```

Check remaining processes:

```bash
pgrep -af myapp
pstree -ap "$(systemctl show -p MainPID --value myapp)"
```

If child processes remain, inspect the service configuration and process tree. The service may be launching children incorrectly or using a wrapper that does not forward signals.

---

# 37. Timers as a cron alternative

A systemd timer can schedule a service.

Example timer:

```ini
[Unit]
Description=Run cleanup job daily

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

Check timers:

```bash
systemctl list-timers
```

`Persistent=true` allows a missed run to be triggered after the system becomes available again.

Use timers for tasks such as:

- Cleanup
- Reports
- Certificate checks
- Backup scripts
- Maintenance jobs

---

# 38. Useful systemd commands

| Requirement | Command |
|---|---|
| Service status | `systemctl status myapp` |
| Start service | `systemctl start myapp` |
| Stop service | `systemctl stop myapp` |
| Restart service | `systemctl restart myapp` |
| Reload service | `systemctl reload myapp` |
| Enable at boot | `systemctl enable myapp` |
| Enable and start | `systemctl enable --now myapp` |
| Disable service | `systemctl disable myapp` |
| Reload unit definitions | `systemctl daemon-reload` |
| Show unit file | `systemctl cat myapp` |
| Show unit path | `systemctl show -p FragmentPath myapp` |
| Show main PID | `systemctl show -p MainPID myapp` |
| Show exit status | `systemctl show -p ExecMainStatus myapp` |
| Follow logs | `journalctl -u myapp -f` |
| Logs since boot | `journalctl -u myapp -b` |
| Reset failed state | `systemctl reset-failed myapp` |
| List timers | `systemctl list-timers` |

---

# 39. Interview questions

1. What is systemd?
2. What is a systemd unit?
3. What is the difference between `start` and `enable`?
4. What does `daemon-reload` do?
5. What is the difference between restart and reload?
6. Explain `After=`, `Requires=` and `Wants=`.
7. Why does `After=network-online.target` not guarantee application readiness?
8. Why should applications run in the foreground under systemd?
9. What is the purpose of `Restart=on-failure`?
10. How do you troubleshoot a service in a restart loop?
11. How do you inspect the main PID of a service?
12. How do you view service logs?
13. How do you run a service as a non-root user?
14. How do you configure environment variables?
15. What are systemd drop-ins?
16. What is the difference between `Type=simple`, `Type=exec` and `Type=forking`?
17. How do you configure resource limits?
18. How do you reset a failed service?
19. How do you deploy a systemd unit using Ansible?
20. Why should Jenkins not permanently supervise production processes?
21. How do you validate application health after restart?
22. What is an atomic release directory?
23. How do you investigate a port conflict?
24. How do you investigate an OOM-killed service?
25. How would you implement graceful shutdown?

---

# 40. Interview checklist

- [ ] Explain systemd and unit types
- [ ] Use `start`, `stop`, `restart`, `reload`, `enable`
- [ ] Explain `daemon-reload`
- [ ] Create a production service unit
- [ ] Run applications as dedicated users
- [ ] Configure environment files
- [ ] Configure restart policies
- [ ] Understand dependencies and ordering
- [ ] Read journald logs
- [ ] Use drop-in overrides
- [ ] Configure resource limits
- [ ] Troubleshoot restart loops
- [ ] Validate health after deployment
- [ ] Deploy units through Ansible
- [ ] Use versioned release directories
- [ ] Explain graceful shutdown
- [ ] Distinguish service status from application readiness

---

# Key DevOps principle

**systemd should own the runtime lifecycle of long-running applications.**

CI/CD tools such as Jenkins should build, test and deploy artifacts. The service manager should handle:

- Startup
- Shutdown
- Restart
- Logging
- Resource controls
- Boot integration
- Failure recovery

A deployment is not successful merely because `systemctl restart` returned successfully. The application must also pass its health checks.
