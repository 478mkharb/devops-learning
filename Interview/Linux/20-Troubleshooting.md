# Real-World DevOps Troubleshooting Scenarios

A practical, interview-oriented troubleshooting handbook for DevOps engineers working with Linux, Jenkins, Ansible, Terraform, Packer, AWS EC2, NGINX, application services, databases, observability, containers, and Kubernetes.

> **Operating principle:** Do not guess. Collect evidence, reduce the search space, make the smallest safe change, and validate the result.

---

## 1. The DevOps Incident Troubleshooting Method

A reliable troubleshooting workflow is:

1. **Detect** — confirm the alert or user-reported symptom.
2. **Scope** — identify affected services, hosts, users, regions, and time window.
3. **Stabilize** — stop the incident from getting worse.
4. **Collect evidence** — logs, metrics, process state, network state, deployment history.
5. **Form hypotheses** — list likely causes and rank them.
6. **Test hypotheses** — run focused commands; change one variable at a time.
7. **Remediate** — apply the safest fix.
8. **Validate** — test from the user’s perspective and from the infrastructure layer.
9. **Prevent recurrence** — add monitoring, automation, tests, documentation, or capacity.

### Never start with random restarts

A restart may hide the original evidence. Before restarting, collect:

```bash
date
hostname
uptime
systemctl status <service> --no-pager
journalctl -u <service> -n 100 --no-pager
ps auxww
ss -lntup
df -h
free -h
```

If the service is actively causing damage, stabilize first, but record what you can.

---

## 2. First Five Minutes of an Incident

### Minute 0–1: Confirm

- Is the problem real?
- Is it one user, one host, one service, or the whole platform?
- Did it begin after a deployment, configuration change, AMI update, or network change?

Useful checks:

```bash
date
hostname
uptime
curl -I http://127.0.0.1
curl -I http://<service-host>:<port>
```

### Minute 1–3: Scope

Check:

- HTTP status codes
- Error rate
- Latency
- CPU and memory
- Disk space and inode usage
- Recent deployments
- Service and dependency health
- DNS resolution
- Security group, route, and subnet changes

### Minute 3–5: Stabilize

Possible actions:

- Roll back a known-bad deployment.
- Remove an unhealthy instance from service.
- Stop a runaway process.
- Increase capacity if safe.
- Disable a failing scheduled job.
- Route traffic to a healthy environment.
- Temporarily reduce expensive traffic.

Record every action and its time.

---

## 3. First Fifteen Minutes Checklist

```text
[ ] Confirm symptom and impact
[ ] Identify start time
[ ] Identify recent changes
[ ] Determine blast radius
[ ] Check service status
[ ] Check logs around the start time
[ ] Check CPU, RAM, disk, and inodes
[ ] Check listening ports
[ ] Check DNS and connectivity
[ ] Check dependency health
[ ] Stabilize or rollback if necessary
[ ] Validate recovery
[ ] Preserve evidence
[ ] Create follow-up prevention tasks
```

---

## 4. Evidence Collection and Command Discipline

### Capture system context

```bash
date -Is
hostnamectl
uname -a
cat /etc/os-release
uptime
who
last -n 10
```

### Capture service context

```bash
systemctl status <service> --no-pager
systemctl cat <service>
systemctl show <service> \
  -p ActiveState -p SubState -p MainPID \
  -p User -p Group -p ExecStart -p Environment
journalctl -u <service> --since "30 minutes ago" --no-pager
```

### Capture network context

```bash
ip -br addr
ip route
ss -lntup
getent hosts <hostname>
resolvectl status
curl -v http://<host>:<port>/health
```

### Capture resource context

```bash
free -h
vmstat 1 5
df -hT
df -ih
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
```

### Good troubleshooting habits

- Use timestamps.
- Save command output.
- Prefer read-only commands first.
- Do not expose secrets in logs or screenshots.
- Avoid `chmod -R 777`.
- Avoid deleting logs before collecting them.
- Avoid changing multiple unrelated settings together.
- Make reversible changes whenever possible.

---

# Part I — Linux and Host Troubleshooting

## 5. Scenario: CPU Usage Is 100%

### Symptom

The application is slow, requests time out, or load average is high.

### Likely causes

- Infinite loop or CPU-heavy request.
- Too many worker processes.
- Compression, encryption, or PDF generation.
- Runaway shell script.
- High system CPU caused by I/O, interrupts, or kernel work.
- A noisy neighbor on a shared VM.

### Commands

```bash
uptime
top
ps -eo pid,ppid,user,stat,pcpu,pmem,etime,cmd --sort=-pcpu | head -20
pidstat -u -p ALL 1 5
vmstat 1 5
mpstat -P ALL 1 5
```

### Interpretation

- High `%us`: application/user-space CPU.
- High `%sy`: kernel/system CPU.
- High `%wa`: processes waiting for I/O.
- High load with low CPU may indicate blocked I/O.
- One process at 100% on a single-core VM can saturate the host.

### Safe response

1. Identify the process.
2. Check its logs and recent deployment.
3. Reduce traffic or worker count if needed.
4. Restart only after evidence collection.
5. Fix the code, query, loop, or configuration.

---

## 6. Scenario: Memory Is Exhausted

### Symptom

Processes are killed, SSH becomes unreliable, or the kernel reports OOM events.

### Commands

```bash
free -h
cat /proc/meminfo | head -20
ps aux --sort=-%mem | head -20
dmesg -T | grep -iE 'oom|out of memory|killed process'
journalctl -k -b --no-pager | grep -iE 'oom|out of memory'
vmstat 1 5
```

### Likely causes

- Memory leak.
- Too many application workers.
- Large JVM heap.
- Elasticsearch or ScyllaDB memory pressure.
- Large batch or PDF operation.
- Insufficient instance size.
- No swap where swap would provide protection.

### Important distinction

Linux page cache is reclaimable. Do not assume all memory shown as used is unavailable.

### Fix options

- Reduce worker count.
- Tune JVM heap or service memory limits.
- Fix the leak.
- Add capacity.
- Use bounded queues and pagination.
- Configure appropriate swap where operationally acceptable.
- Add memory alerts before OOM occurs.

---

## 7. Scenario: Disk Is Full

### Symptom

Deployments fail, applications cannot write, databases stop, or systemd services fail.

### Commands

```bash
df -hT
df -ih
sudo du -xhd1 / | sort -h
sudo du -xhd1 /var | sort -h
sudo lsof +L1
journalctl --disk-usage
```

### Common causes

- Application logs.
- Journald retention.
- Old artifacts and workspaces.
- Core dumps.
- Temporary files.
- Deleted files still held open by a process.
- Database or Elasticsearch data growth.

### Important distinction

A deleted file may still consume disk space if a process keeps it open. Find it with:

```bash
sudo lsof +L1
```

Restart the owning process only after assessing impact.

### Prevention

- Configure `logrotate`.
- Set journald retention.
- Clean Jenkins workspaces and artifacts.
- Monitor both disk percentage and absolute free space.
- Monitor inode usage.
- Set database and index retention policies.

---

## 8. Scenario: Inodes Are Exhausted

### Symptom

`No space left on device` appears even though `df -h` shows free capacity.

### Commands

```bash
df -ih
sudo find /var -xdev -type f | wc -l
sudo find /tmp -xdev -type f | wc -l
```

### Causes

- Millions of small files.
- Temporary files.
- Jenkins workspaces.
- Application-generated fragments.
- Cache directories.

### Fix

Identify the directory with the most files, clean safely, and add retention controls.

---

## 9. Scenario: A Port Is Already in Use

### Symptom

An application fails to start with “Address already in use”.

### Commands

```bash
sudo ss -lntup
sudo lsof -nP -iTCP:<port> -sTCP:LISTEN
sudo fuser -v <port>/tcp
```

### Troubleshooting sequence

1. Identify the owning PID.
2. Check whether it is the expected service.
3. Inspect the process command line.
4. Check systemd and deployment scripts.
5. Stop the stale process gracefully.
6. Correct duplicate startup logic.

```bash
ps -fp <PID>
systemctl status <service>
systemctl list-units --type=service | grep -i <name>
```

---

## 10. Scenario: systemd Service Fails to Start

### Commands

```bash
systemctl status <service> --no-pager
journalctl -u <service> -b --no-pager
systemctl cat <service>
systemd-analyze verify /etc/systemd/system/<service>.service
```

### Common causes

- Wrong `ExecStart` path.
- Wrong user or group.
- Missing environment file.
- Virtual environment not used.
- Working directory missing.
- Port conflict.
- Permission denied.
- Dependency unavailable.
- Syntax error in unit file.

### After changing the unit file

```bash
sudo systemctl daemon-reload
sudo systemctl restart <service>
sudo systemctl status <service> --no-pager
```

### Production validation

```bash
systemctl is-enabled <service>
systemctl is-active <service>
systemctl show <service> -p Restart -p RestartUSec
```

---

## 11. Scenario: Permission Denied

### Diagnose the complete path

```bash
namei -l /path/to/file
ls -ld /path /path/to
ls -l /path/to/file
id <service-user>
getfacl /path/to/file
```

Check:

- File owner and group.
- Execute permission on every parent directory.
- ACLs.
- SELinux or AppArmor.
- Read-only mount.
- Service user identity.
- File created by a different deployment user.

### Better fix

Use ownership and group design:

```bash
sudo chown -R appuser:appgroup /opt/myapp
sudo chmod -R u=rwX,g=rX,o= /opt/myapp
```

Do not solve application permissions with `777`.

---

# Part II — Networking and AWS Troubleshooting

## 12. Scenario: DNS Resolution Fails

### Commands

```bash
getent hosts example.com
resolvectl status
resolvectl query example.com
dig example.com
cat /etc/resolv.conf
```

### Distinguish failures

- Name does not resolve: DNS problem.
- Name resolves but connection fails: routing, firewall, service, or TLS problem.
- Internal name fails but public name works: private DNS or resolver configuration.
- Old IP appears: caching or TTL issue.

### Check application runtime

A shell resolving a name does not guarantee that the application uses the same resolver configuration.

---

## 13. Scenario: Host Is Reachable but Port Is Not

### Test in layers

```bash
ping <host>
nc -vz <host> <port>
curl -v http://<host>:<port>
```

`ping` may be blocked and is not proof of application availability.

### Check server side

```bash
ss -lntup
sudo ufw status verbose
sudo nft list ruleset
```

### AWS checks

- Security group inbound rule.
- Security group outbound rule.
- Network ACL inbound and outbound rules.
- Route table.
- Subnet association.
- Service bind address.
- Instance firewall.
- Load balancer target health.

---

## 14. Scenario: EC2 in a Private Subnet Cannot Reach the Internet

### Symptoms

- `apt update` fails.
- SSM Agent is offline.
- Package downloads time out.
- External APIs or SMTP endpoints cannot be reached.

### Required path

```text
EC2 private subnet
        |
        v
Route table: 0.0.0.0/0 -> NAT Gateway
        |
        v
NAT Gateway in public subnet
        |
        v
Internet Gateway
        |
        v
Internet
```

### Check

- NAT Gateway is available.
- NAT Gateway is in a public subnet.
- Public subnet route table points to Internet Gateway.
- Private subnet route table points to NAT Gateway.
- Network ACL permits ephemeral return traffic.
- Security group permits outbound traffic.
- DNS support and hostnames are enabled.
- NAT route is in the correct Availability Zone where relevant.

---

## 15. Scenario: SSM Agent Is Offline

### Checklist

1. SSM Agent is installed and running.
2. Instance has an IAM instance profile.
3. Instance role includes appropriate Systems Manager permissions.
4. Instance can reach SSM endpoints.
5. DNS works.
6. System clock is correct.
7. Agent logs show the actual error.

```bash
sudo systemctl status amazon-ssm-agent --no-pager
sudo journalctl -u amazon-ssm-agent -n 100 --no-pager
curl -I https://ssm.<region>.amazonaws.com
date -Is
```

### Common causes

- Missing or incorrect instance profile.
- Private subnet without NAT or VPC endpoints.
- Security group or NACL egress restriction.
- Wrong region.
- Agent stopped.
- Credentials unavailable.
- DNS failure.

### Important distinction

An EC2 instance can be running while SSM is offline. EC2 state and SSM management state are different.

---

## 16. Scenario: SSH Through a Bastion Times Out

### Check the path

```text
Client -> Bastion -> Private EC2
```

### Verify

- Bastion is reachable.
- Bastion security group allows SSH from the client.
- Private EC2 security group allows SSH from the bastion security group, not necessarily the client IP.
- Network ACLs allow traffic and return ephemeral ports.
- Private route table has the correct local route.
- SSH daemon is running on the target.
- Correct private IP and key are used.

```bash
ssh -vvv -J ubuntu@<bastion> ubuntu@<private-ip>
```

### Better alternative

For managed EC2 fleets, prefer SSM Session Manager when available. It avoids exposing SSH and reduces bastion dependency.

---

## 17. Scenario: TLS or HTTPS Fails

### Commands

```bash
curl -vk https://example.com
openssl s_client -connect example.com:443 -servername example.com
date -Is
```

### Likely causes

- Certificate expired.
- Wrong hostname/SNI.
- Incomplete certificate chain.
- TLS version mismatch.
- Load balancer listener misconfiguration.
- Backend speaks HTTP while proxy expects HTTPS.
- System clock incorrect.

---

# Part III — Jenkins and CI/CD Troubleshooting

## 18. Scenario: Jenkins Job Cannot Write to Workspace

### Check

```bash
df -h
df -ih
ls -ld "$WORKSPACE"
id
ps -ef | grep -i jenkins
```

### Causes

- Workspace owned by another user.
- Disk full.
- Previous build left root-owned files.
- Concurrent builds use the same directory.
- Workspace path is missing.
- Agent disconnected.

### Fix

Use Jenkins-managed ownership and cleanup. Avoid manually changing permissions without understanding the agent user.

---

## 19. Scenario: Jenkins Agent Has the Wrong Java Version

### Symptom

Maven, plugins, or the application fails because the repository requires a different Java version.

### Commands

```bash
java -version
which java
readlink -f "$(which java)"
mvn -version
echo "$JAVA_HOME"
```

### Diagnose

- Controller Java version.
- Agent Java version.
- Maven tool configuration.
- `JAVA_HOME`.
- Project compiler target.
- Jenkins tool installations.

### Prevention

- Pin tool versions.
- Declare required Java version in the build.
- Use separate agents for incompatible toolchains.
- Print tool versions at the start of every pipeline.

---

## 20. Scenario: Maven Build Fails with Missing JaCoCo or Coverage Data

### Check

```bash
mvn -version
mvn clean test
find . -maxdepth 4 -type f | grep -Ei 'jacoco|surefire|coverage'
```

### Likely causes

- Plugin not bound to the correct lifecycle phase.
- Tests did not run.
- Wrong module path.
- Java version mismatch.
- Coverage file generated in a different directory.
- Jenkins workspace was cleaned too early.
- SonarQube expects a path that does not exist.

### Prevention

- Run tests before coverage analysis.
- Verify expected files before SonarQube.
- Fail clearly when tests are skipped.
- Keep build, test, coverage, and scan stages logically ordered.

---

## 21. Scenario: Jenkins Pipeline Fails During GitLeaks or SonarQube

### GitLeaks checks

- Is the repository checked out correctly?
- Is the tool installed?
- Is the scan running against the intended commit?
- Are test fixtures or generated files producing false positives?
- Is the secret actually exposed?

### SonarQube checks

```bash
curl -I http://<sonarqube-host>:9000
```

Verify:

- Scanner configuration.
- Server URL.
- Authentication token.
- Project key.
- Source and test paths.
- Java compatibility.
- Quality gate behavior.

Never print tokens in pipeline logs.

---

## 22. Scenario: Pipeline Hangs

### Investigate

- Waiting for manual approval.
- Interactive command awaiting input.
- Child process left running.
- Deadlocked test.
- Network call without timeout.
- Lock held by another build.
- Jenkins agent disconnected.

```bash
ps -ef --forest
ss -ntp
```

### Prevention

- Use command timeouts.
- Avoid interactive commands.
- Use explicit approval stages.
- Clean child processes.
- Set pipeline and stage timeouts.
- Make external calls retryable and bounded.

---

# Part IV — Ansible Troubleshooting

## 23. Scenario: Ansible Cannot Connect

### SSH

```bash
ansible-inventory --graph
ansible all -m ping -vvv
ssh -vvv ubuntu@<host>
```

Check:

- Inventory host and group.
- Correct user.
- Key permissions.
- Security groups.
- Route and bastion path.
- Python interpreter on target.

### SSM-based execution

Do not assume an SSM-managed host is reachable by SSH. Verify the selected connection plugin and its prerequisites.

---

## 24. Scenario: Ansible Fails at `become`

### Symptom

`sudo: a password is required`.

### Causes

- User is not allowed passwordless sudo.
- Wrong become user.
- Playbook is executed with an unexpected user.
- Sudo policy differs between hosts.

### Commands

```bash
whoami
id
sudo -l
```

### Fix

Use an approved privilege-escalation design. Do not place sudo passwords in plaintext playbooks or source control.

---

## 25. Scenario: Ansible YAML or Variable Error

### Diagnose

```bash
ansible-playbook --syntax-check site.yml
ansible-playbook site.yml --check --diff
ansible-playbook site.yml -vvv
```

Check:

- Indentation.
- Quoting.
- Variable names.
- Group variables.
- Host variables.
- Jinja2 expressions.
- Boolean values.
- List versus dictionary structure.

---

## 26. Scenario: Ansible Task Is Not Idempotent

### Symptom

Every run reports `changed`.

### Causes

- Shell command without a proper state check.
- Timestamp or random output.
- Unconditional file replacement.
- Service restarted every run.
- Command lacks `creates`, `removes`, or a check mode strategy.

### Better approach

Prefer modules such as:

- `package`
- `file`
- `template`
- `copy`
- `service`
- `user`
- `mount`

Use handlers for service restarts after configuration changes.

---

# Part V — Terraform and Packer Troubleshooting

## 27. Scenario: Terraform `init` Fails

### Commands

```bash
terraform version
terraform init -reconfigure
terraform providers
```

### Common causes

- Backend bucket unavailable.
- Incorrect AWS credentials or region.
- Provider version conflict.
- Network or proxy issue.
- State lock or backend permission issue.
- Incorrect backend key.

### Backend permissions commonly required

- Read/write state object.
- List bucket where needed.
- Locking mechanism permissions if configured.
- KMS permissions if state is encrypted with a customer-managed key.

---

## 28. Scenario: Terraform State Is Locked

### First rule

Do not force-unlock immediately.

### Investigate

- Is another pipeline running?
- Did a previous job crash?
- Is the lock owner identifiable?
- Is the state backend healthy?

Only remove a stale lock after confirming no legitimate Terraform operation is active.

### Prevention

- One state owner per environment.
- Serialize Terraform execution.
- Use remote state.
- Use pipeline locks.
- Keep plan and apply workflow controlled.

---

## 29. Scenario: Terraform Plan Shows Unexpected Replacement

### Investigate

```bash
terraform plan
terraform state list
terraform state show <resource>
terraform show
```

### Likely causes

- Immutable argument changed.
- Resource moved in configuration.
- Incorrect import.
- Drift.
- Provider behavior change.
- Changed `for_each` or `count` key.
- AMI or subnet changed.

### Safe process

1. Read the plan carefully.
2. Identify the exact replacement trigger.
3. Compare with the real AWS resource.
4. Check whether the replacement is intended.
5. Use `moved` blocks or import where appropriate.
6. Never approve destructive changes blindly.

---

## 30. Scenario: Terraform Cannot Find AWS Resources

### Check

- Active AWS identity:

```bash
aws sts get-caller-identity
```

- Region:

```bash
aws configure get region
```

- Provider configuration.
- Account ID.
- Resource tags.
- Data source filters.
- Resource state.

A resource existing in one region or account does not mean Terraform can discover it in another.

---

## 31. Scenario: Packer Build Fails Before Launch

### Typical causes

- Missing `subnet_id`.
- Missing `security_group_id`.
- Wrong source AMI.
- Invalid instance profile.
- Unsupported region.
- Missing variables.
- Incorrect builder configuration.

### Validate

```bash
packer fmt -check .
packer validate .
packer inspect .
```

### During the build

Check:

- Temporary instance state.
- IAM instance profile.
- SSM connectivity.
- Provisioner logs.
- Package installation.
- Service enablement.
- AMI cleanup behavior.

### AMI validation

After creation:

- Launch a test instance.
- Verify expected services.
- Verify ports and health endpoints.
- Verify logs.
- Verify SSM.
- Verify no secrets or temporary credentials are baked into the image.

---

# Part VI — Application and Reverse Proxy Troubleshooting

## 32. Scenario: NGINX Returns 502 Bad Gateway

### Meaning

NGINX can receive the request but cannot successfully communicate with the upstream application.

### Commands

```bash
sudo nginx -t
sudo systemctl status nginx --no-pager
sudo tail -n 100 /var/log/nginx/error.log
ss -lntup
curl -v http://127.0.0.1:<upstream-port>/health
```

### Causes

- Backend process is down.
- Wrong upstream port.
- Backend listens only on another address.
- Firewall blocks local or remote traffic.
- Backend crashes on request.
- Unix socket permissions.
- Timeout or response size issue.

### Fix sequence

1. Test backend directly.
2. Check backend service logs.
3. Verify NGINX upstream configuration.
4. Validate configuration.
5. Reload NGINX.
6. Test through NGINX.

---

## 33. Scenario: NGINX Returns 403 for Frontend Files

### Check

```bash
namei -l /var/www/html
ls -la /var/www/html
sudo nginx -T
```

### Causes

- Missing execute permission on parent directories.
- Wrong file owner.
- Directory index missing.
- `try_files` misconfiguration.
- SELinux/AppArmor denial.
- NGINX user cannot read files.

For a React single-page application, routing often needs a fallback such as `index.html`.

---

## 34. Scenario: Frontend Shows Old JavaScript After Deployment

### Causes

- Browser cache.
- CDN cache.
- NGINX cache.
- Old files not removed.
- Service worker cache.
- Deployment copied files into the wrong directory.
- HTML references old asset names.

### Diagnose

```bash
curl -I https://<frontend-host>
curl -s https://<frontend-host> | head
find /var/www/html -maxdepth 2 -type f | head
```

### Prevention

- Use hashed asset filenames.
- Set suitable cache headers.
- Deploy atomically.
- Remove stale files safely.
- Version service workers carefully.

---

## 35. Scenario: Spring Boot Service Does Not Start

### Commands

```bash
java -version
systemctl status salary-api --no-pager
journalctl -u salary-api -n 150 --no-pager
ss -lntup
```

### Common causes

- Wrong Java version.
- Port conflict.
- Missing environment variable.
- Invalid application properties.
- Database unavailable.
- Migration failure.
- JAR path incorrect.
- Insufficient memory.

### Dependency isolation

Test the database independently before blaming the application:

```bash
nc -vz <db-host> <db-port>
```

---

## 36. Scenario: Flask/Gunicorn Service Fails

### Check

```bash
which python
python --version
which gunicorn
gunicorn --version
systemctl cat attendance-api
journalctl -u attendance-api -n 100 --no-pager
```

### Causes

- Wrong virtual environment.
- Missing package.
- Wrong WSGI module.
- Import error.
- Working directory incorrect.
- Service user cannot read code or configuration.
- Binding only to `127.0.0.1` when remote access is expected.
- Redis or PostgreSQL unavailable.

---

## 37. Scenario: Go Binary Fails on EC2

### Commands

```bash
file ./employee-api
ldd ./employee-api
uname -m
ls -l ./employee-api
```

### Causes

- Binary built for the wrong architecture.
- Missing execute permission.
- Dynamic library missing.
- Wrong configuration path.
- Port conflict.
- ScyllaDB endpoint unavailable.
- Binary copied without its required configuration.

Build for the target architecture and validate the binary on a clean host.

---

# Part VII — Database and Data Pipeline Troubleshooting

## 38. Scenario: PostgreSQL Connection Refused

### Commands

```bash
sudo systemctl status postgresql --no-pager
ss -lntup | grep 5432
sudo -u postgres psql -c '\l'
```

### Causes

- PostgreSQL stopped.
- Listening only on localhost.
- Wrong port.
- Firewall.
- Wrong hostname.
- Connection pool exhausted.
- Database server overloaded.

### Authentication versus connectivity

- `Connection refused`: listener or network path issue.
- `timeout`: routing/firewall/security group/NACL issue.
- `password authentication failed`: credentials or authentication configuration.
- `database does not exist`: wrong database name or provisioning failure.

---

## 39. Scenario: Liquibase Migration Fails

### Check

- Database connectivity.
- Database user permissions.
- Changelog path.
- ChangeSet checksum.
- Existing schema state.
- Lock table.

### Safe process

1. Read the exact migration error.
2. Check `DATABASECHANGELOGLOCK`.
3. Confirm whether a migration is running.
4. Never manually delete migration history without understanding the consequences.
5. Back up before destructive schema changes.

---

## 40. Scenario: Redis Connection Refused

### Commands

```bash
sudo systemctl status redis-server --no-pager
ss -lntup | grep 6379
redis-cli ping
```

### Causes

- Redis stopped.
- Wrong host or port.
- Protected mode.
- Bind address.
- Firewall.
- Authentication requirement.
- Memory pressure or eviction behavior.

### Prevention

- Monitor memory and evictions.
- Define TTLs.
- Avoid storing unbounded data.
- Protect Redis from public exposure.
- Use health checks.

---

## 41. Scenario: ScyllaDB/CQL Connection Fails

### Check

```bash
sudo systemctl status scylla-server --no-pager
ss -lntup | grep 9042
cqlsh <host> 9042
```

### Causes

- Service stopped.
- Wrong listen or broadcast address.
- Port blocked.
- Node not ready.
- Wrong keyspace/table.
- Authentication or authorization failure.
- Disk or memory pressure.

### Application-level checks

- Confirm keyspace exists.
- Confirm table schema.
- Confirm consistency level.
- Confirm the application points to the intended cluster.
- Check whether writes succeed but reads use a different source.

---

## 42. Scenario: Elasticsearch Index Is Missing or Wrong

### Commands

```bash
curl -s http://<es-host>:9200/_cluster/health?pretty
curl -s http://<es-host>:9200/_cat/indices?v
curl -s http://<es-host>:9200/_cat/nodes?v
```

### Common causes

- Application configured with the wrong index name.
- Index was never created.
- Alias points elsewhere.
- Authentication failure.
- Disk watermark blocks writes.
- Mapping rejects a document.
- Cluster is red or unavailable.

### Prevention

- Centralize index names in configuration.
- Monitor cluster health.
- Monitor disk watermarks.
- Version mappings and templates.
- Validate index existence during deployment.

---

## 43. Scenario: Notification Worker Does Not Send Salary Emails

### Troubleshooting chain

```text
Salary API writes source data
        |
        v
Data mirrored/indexed in Elasticsearch
        |
        v
Worker queries expected index
        |
        v
Pending record is selected
        |
        v
PDF is generated
        |
        v
SMTP connection succeeds
        |
        v
Email is sent
        |
        v
Record status is updated
```

### Check each boundary

1. Does the salary record exist in the source database?
2. Is the ES document present?
3. Is the worker querying the correct index?
4. Does the query match the pending status?
5. Can the worker write the PDF?
6. Can the host reach the SMTP server?
7. Is the sender authenticated?
8. Is the status update successful?
9. Is the same record being retried repeatedly?

### Important design rule

Use the primary database as the source of truth. Treat Elasticsearch as a searchable mirror or index unless the architecture explicitly defines otherwise.

---

# Part VIII — Monitoring and Observability

## 44. Scenario: Prometheus Target Is Down

### Check

```bash
curl -s http://<prometheus-host>:9090/api/v1/targets
curl -s http://<target-host>:<exporter-port>/metrics | head
```

### Causes

- Exporter stopped.
- Wrong target address.
- Security group or firewall.
- DNS failure.
- Wrong scrape path.
- TLS or authentication mismatch.
- Prometheus configuration error.

### Validate configuration

```bash
promtool check config prometheus.yml
promtool check rules rules.yml
```

---

## 45. Scenario: Node Exporter Is Up but Grafana Shows No Data

### Check

- Target is `UP` in Prometheus.
- Query returns data in Prometheus.
- Dashboard uses the correct data source.
- Dashboard variables are populated.
- Label names match.
- Time range is correct.
- Recording rules exist if referenced.

### Useful PromQL

```promql
up
node_cpu_seconds_total
node_memory_MemAvailable_bytes
node_filesystem_avail_bytes
rate(node_cpu_seconds_total[5m])
```

A Grafana dashboard cannot display data that Prometheus has not scraped or that the query does not select.

---

## 46. Scenario: Blackbox Exporter Probe Fails

### Check

```bash
curl -v "http://<blackbox-host>:9115/probe?module=http_2xx&target=https://example.com"
```

### Causes

- Incorrect module name.
- Invalid YAML.
- Target DNS failure.
- TLS certificate issue.
- Network access restriction.
- Probe timeout.
- Unsupported configuration field for the installed exporter version.

### Prevention

- Validate exporter configuration.
- Keep configuration compatible with the installed version.
- Test probes manually.
- Alert on probe failure and probe latency.

---

## 47. Scenario: Alert Is Not Firing

### Check

- Expression returns a series.
- Rule file is loaded.
- Rule evaluation is healthy.
- `for` duration has elapsed.
- Labels and alert routing match.
- Alertmanager is reachable.
- Notification receiver is configured.

Do not test alerts only by reading the rule text; evaluate the expression in Prometheus.

---

# Part IX — Docker, Kubernetes, and Cloud-Native Scenarios

## 48. Scenario: Kubernetes Pod Is `CrashLoopBackOff`

### Commands

```bash
kubectl get pods -A
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
kubectl get events -n <namespace> --sort-by=.lastTimestamp
```

### Causes

- Application exits.
- Bad environment variable.
- Missing secret or config map.
- Dependency unavailable.
- Wrong command or arguments.
- Failed health check.
- Permission issue.
- OOM kill.

Use `--previous` to inspect the previous crashed container.

---

## 49. Scenario: Kubernetes Pod Is `ImagePullBackOff`

### Check

```bash
kubectl describe pod <pod> -n <namespace>
```

### Causes

- Image name or tag typo.
- Registry unavailable.
- Missing image pull secret.
- Private registry authentication failure.
- Unsupported architecture.
- Image was deleted.

---

## 50. Scenario: Kubernetes Pod Is `Pending`

### Check

```bash
kubectl describe pod <pod> -n <namespace>
kubectl get nodes
kubectl describe node <node>
```

### Causes

- Insufficient CPU or memory.
- Taints without tolerations.
- Node selector or affinity mismatch.
- PVC cannot bind.
- Resource quota.
- No suitable node.

---

## 51. Scenario: Kubernetes Service Has No Endpoints

### Check

```bash
kubectl get svc <service> -n <namespace> -o yaml
kubectl get endpoints <service> -n <namespace>
kubectl get endpointslices -n <namespace>
kubectl get pods --show-labels -n <namespace>
```

### Likely cause

The Service selector does not match the Pod labels, or Pods are not Ready.

---

## 52. Scenario: Kubernetes Readiness or Liveness Probe Fails

### Distinction

- **Readiness** controls whether traffic is sent to the Pod.
- **Liveness** determines whether the container should be restarted.
- **Startup probe** protects slow-starting applications from premature liveness failures.

### Check

- Correct port.
- Correct path.
- Bind address.
- Startup duration.
- Dependency behavior.
- Probe timeout and failure threshold.

Do not use liveness probes to test every external dependency; that can cause restart storms.

---

## 53. Scenario: Container Is `OOMKilled`

### Check

```bash
kubectl describe pod <pod>
kubectl top pod <pod>
kubectl top node
```

### Causes

- Container memory limit too low.
- Memory leak.
- JVM heap not sized for the container.
- Large batch operation.
- Too many workers.

Tune requests, limits, and application memory together.

---

# Part X — Security Troubleshooting

## 54. Scenario: Application Cannot Read a File After Security Hardening

Check:

```bash
getenforce
ls -Z /path/to/file
ausearch -m avc -ts recent
aa-status
```

### Possible causes

- SELinux context mismatch.
- AppArmor profile denial.
- File permissions.
- Service running under a different user.
- Read-only filesystem.

Do not disable SELinux or AppArmor as the first fix. Identify and correct the policy or context.

---

## 55. Scenario: Secret or Credential Is Exposed in a Pipeline

### Immediate actions

1. Revoke or rotate the credential.
2. Stop further exposure.
3. Remove it from logs and artifacts where possible.
4. Determine whether it was accessed.
5. Fix the pipeline and storage design.
6. Audit other repositories and logs.

### Prevention

- Use Jenkins credentials or a secrets manager.
- Mask sensitive values.
- Never echo tokens.
- Avoid secrets in AMIs, Git, Terraform variables, or shell history.
- Use least privilege.
- Rotate credentials regularly.

---

# Part XI — Incident Runbooks

## 56. HTTP 5xx Runbook

```text
Check external response
        |
        v
Is NGINX/load balancer healthy?
        |
        +-- No -> inspect proxy/LB logs and listener
        |
        v
Is backend process running?
        |
        +-- No -> inspect systemd and application logs
        |
        v
Can backend be reached directly?
        |
        +-- No -> port, bind address, firewall, dependency
        |
        v
Does backend return an application error?
        |
        +-- Yes -> inspect code, database, cache, configuration
        |
        v
Validate through the full request path
```

## 57. EC2 Unreachable Runbook

```text
Is instance running?
        |
        v
Is SSM online?
        |
        +-- Yes -> use Session Manager and inspect host
        |
        +-- No -> inspect IAM profile, agent, DNS, NAT/VPC endpoints
        |
        v
Check route tables
        |
        v
Check security groups
        |
        v
Check NACLs
        |
        v
Check host firewall and service state
```

## 58. Database Failure Runbook

1. Confirm listener.
2. Confirm network path.
3. Confirm authentication.
4. Confirm database exists.
5. Check disk, memory, and connection count.
6. Check locks and long-running queries.
7. Check recent migrations.
8. Check application pool configuration.
9. Validate reads and writes.
10. Document data integrity impact.

---

# Part XII — OT-Microservices Example Troubleshooting Map

## 59. Reference Flow

```text
Browser
  |
  v
NGINX + React frontend
  |
  +--> employee-api (Go) ------> ScyllaDB
  |
  +--> attendance-api (Flask) -> PostgreSQL
  |                              |
  |                              +--> Redis cache
  |
  +--> salary-api (Spring Boot) -> ScyllaDB
                                  |
                                  v
                           Elasticsearch mirror
                                  |
                                  v
                           Notification worker
                                  |
                                  +--> PDF generation
                                  |
                                  +--> SMTP email
```

## 60. Symptom-to-Component Table

| Symptom | First place to check | Next dependency |
|---|---|---|
| Frontend page does not load | NGINX and browser network tab | Static files, DNS, TLS |
| API returns 502 | NGINX error log | Backend service and port |
| Employee create fails | Go service logs | ScyllaDB and CQL |
| Attendance save fails | Flask/Gunicorn logs | PostgreSQL and migrations |
| Slow attendance requests | Flask logs and metrics | Redis/PostgreSQL |
| Salary list is empty | Spring Boot logs | ScyllaDB query and schema |
| Salary appears in DB but not worker | Worker logs | ES index and query |
| PDF not generated | Worker logs and filesystem | ReportLab and permissions |
| Email not sent | SMTP logs and connectivity | Credentials, TLS, provider |
| Metrics missing | Prometheus targets | Exporter and network |
| EC2 inaccessible | SSM and route tables | IAM, NAT, SG, NACL |

---

# Part XIII — Interview Questions

## 61. Core Questions

1. What is your first action during a production incident?
2. How do you distinguish CPU saturation from I/O wait?
3. How do you investigate `No space left on device`?
4. How do you find a process listening on a port?
5. Why can a deleted file still consume disk space?
6. How do you troubleshoot a systemd service?
7. What is the difference between connection refused and connection timeout?
8. How do security groups differ from network ACLs?
9. Why can an EC2 instance be running but SSM offline?
10. How do you troubleshoot a bastion connection?
11. How do you identify a Jenkins workspace permission problem?
12. How do you handle incompatible Java versions?
13. Why might a Maven coverage file be missing?
14. How do you debug an Ansible `become` failure?
15. Why should Terraform state not be edited manually?
16. What causes Terraform to replace a resource?
17. What should be checked when Packer variables are missing?
18. How do you troubleshoot NGINX 502?
19. How do you distinguish PostgreSQL authentication failure from network failure?
20. Why should Elasticsearch not automatically be treated as the source of truth?
21. What is the difference between Kubernetes readiness and liveness?
22. How do you debug `CrashLoopBackOff`?
23. What causes `ImagePullBackOff`?
24. How do you troubleshoot a Service with no endpoints?
25. How do you investigate a Prometheus target that is down?
26. How do you respond to an exposed credential?
27. Why is disabling SELinux usually a poor first fix?
28. What evidence should be preserved after an incident?
29. How do you validate that a fix really worked?
30. What preventive action would you add after a recurring incident?

---

# Part XIV — Final Troubleshooting Checklist

Before closing an incident:

```text
[ ] User-visible symptom is resolved
[ ] Error rate and latency returned to normal
[ ] All affected services are healthy
[ ] Dependencies are healthy
[ ] No hidden retry storm remains
[ ] Disk, memory, and CPU are stable
[ ] Monitoring confirms recovery
[ ] Logs contain no continuing errors
[ ] Temporary mitigation is documented
[ ] Root cause or current leading hypothesis is recorded
[ ] Follow-up actions have owners and deadlines
[ ] Secrets and sensitive evidence are protected
[ ] Runbook is updated
```

## Final Rule

A strong DevOps engineer does not merely restart the service.

A strong DevOps engineer can explain:

- **What failed**
- **Why it failed**
- **How the failure was detected**
- **How the blast radius was limited**
- **Which evidence proved the cause**
- **Why the fix was safe**
- **How recovery was validated**
- **What will prevent recurrence**
