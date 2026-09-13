# Linux Fundamentals for DevOps

## Scope

This document covers Linux fundamentals from a **DevOps Engineer's perspective**. The focus is on application deployment, CI/CD, automation, troubleshooting, cloud servers, containers, monitoring, and production operations.

---

## 1. What is Linux?

Linux is an open-source operating-system kernel. A complete Linux operating system normally includes the kernel, user-space utilities, libraries, package manager, services, and administrative tools.

From a DevOps perspective, Linux is important because:

- Most cloud workloads run on Linux.
- Jenkins, Ansible, Terraform runners, Kubernetes nodes, and monitoring agents commonly run on Linux.
- Application deployment and troubleshooting depend heavily on Linux commands.
- Linux provides process, networking, filesystem, permission, and service-management primitives.

---

## 2. Explain the Linux architecture

The major layers are:

```text
Users / Applications
        |
Shells and User-Space Utilities
        |
System Libraries
        |
System Calls
        |
Linux Kernel
        |
Hardware / Virtual Hardware
```

### Important kernel responsibilities

- Process scheduling
- Memory management
- Filesystem access
- Networking
- Device management
- Security enforcement
- System calls

A DevOps engineer normally works in user space but troubleshoots problems caused by kernel, resource, filesystem, or networking behavior.

---

## 3. What is a distribution?

A Linux distribution packages the Linux kernel with user-space software and a package-management system.

Examples:

| Distribution | Package manager | Common use |
|---|---|---|
| Ubuntu | `apt` / `dpkg` | Cloud, application servers, development |
| Debian | `apt` / `dpkg` | Stable servers |
| RHEL | `dnf` / `rpm` | Enterprise systems |
| Rocky Linux | `dnf` / `rpm` | RHEL-compatible servers |
| Amazon Linux | `dnf` / `yum` | AWS workloads |
| Alpine | `apk` | Small container images |

Do not assume that commands or paths are identical across distributions.

---

## 4. What is the kernel version?

The kernel version identifies the running Linux kernel.

```bash
uname -r
```

More complete information:

```bash
uname -a
```

Useful commands:

```bash
cat /etc/os-release
hostnamectl
```

DevOps use cases:

- Confirming the operating system before automation
- Checking kernel compatibility
- Troubleshooting driver or filesystem issues
- Verifying the base image used by an EC2 instance
- Recording environment details during incident analysis

---

## 5. What is a shell?

A shell is a command interpreter that accepts commands and executes programs.

Common shells:

- `bash`
- `sh`
- `zsh`
- `fish`

Check the current shell:

```bash
echo "$SHELL"
```

Check the shell of the current process:

```bash
ps -p $$ -o pid,ppid,comm,args
```

The `SHELL` variable indicates the user's configured login shell, while the process command shows the shell currently executing.

---

## 6. What is Bash?

Bash means **Bourne Again Shell**. It is widely used for:

- Deployment scripts
- CI/CD pipeline steps
- Server bootstrap scripts
- Health checks
- Log processing
- Backup and maintenance automation

Example:

```bash
#!/usr/bin/env bash

set -Eeuo pipefail

echo "Starting deployment"
```

### Meaning of the options

- `-e`: exit when a command fails
- `-u`: treat unset variables as errors
- `-E`: preserve error traps in functions and subshells
- `-o pipefail`: fail a pipeline if any command in it fails

Use these carefully. Some scripts need explicit handling for expected non-zero exit codes.

---

## 7. What is the difference between a command, program, and process?

### Command

A command is what you type into a shell:

```bash
ls -l
```

### Program

A program is executable code stored on disk.

Examples:

```text
/usr/bin/ls
/usr/bin/python3
/usr/bin/java
```

### Process

A process is a running instance of a program.

Example:

```bash
python3 app.py
```

The Python interpreter becomes a process.

Check processes:

```bash
ps aux
```

DevOps relevance:

- A deployment may succeed but the application process may immediately exit.
- A service may be running but listening on the wrong port.
- Multiple processes may consume CPU or memory.
- A zombie or orphan process may indicate application lifecycle problems.

---

## 8. What is a PID?

PID means **Process ID**. Linux assigns a numeric PID to each running process.

```bash
echo $$
```

This prints the current shell's PID.

Find the PID of an application:

```bash
pgrep -af nginx
pgrep -af java
pgrep -af gunicorn
```

Inspect a process:

```bash
ps -fp <PID>
```

The process with PID `1` is the first user-space process started by the kernel. On most modern Ubuntu systems, it is `systemd`.

---

## 9. What is PPID?

PPID means **Parent Process ID**.

```bash
ps -o pid,ppid,comm,args -p <PID>
```

Parent-child relationships matter when troubleshooting:

- Shell scripts launching applications
- Jenkins agents launching build processes
- Systemd launching services
- Orphaned processes
- Process termination behavior

Example:

```text
systemd
 └── gunicorn
      ├── worker
      └── worker
```

---

## 10. What is the difference between foreground and background processes?

A foreground process occupies the current terminal:

```bash
python3 app.py
```

A background process runs without blocking the shell:

```bash
python3 app.py &
```

Useful job commands:

```bash
jobs
fg
bg
```

Do not use `&` as a production service-management solution. For production, use `systemd`, a container runtime, or an orchestrator.

---

## 11. What is a daemon?

A daemon is a background service that performs work without direct user interaction.

Examples:

- `sshd`
- `cron`
- `systemd-journald`
- `nginx`
- `node_exporter`
- `amazon-ssm-agent`

Check services:

```bash
systemctl list-units --type=service
```

DevOps use case:

A deployed application should normally be managed as a service so that it can:

- Start automatically
- Restart after failure
- Write logs consistently
- Run under a dedicated user
- Have resource and dependency settings

---

## 12. What is the difference between a service and a process?

A process is a running execution unit.

A service is a managed application or background function. A service may contain one or more processes.

Example:

```bash
systemctl status nginx
```

This shows service-level information, including the main process PID.

A service can be:

- Active and running
- Inactive
- Failed
- Activating
- Deactivating

A process can exist without being managed by systemd, for example:

```bash
./app &
```

---

## 13. What is the Linux filesystem hierarchy?

Important directories:

| Directory | Purpose | DevOps relevance |
|---|---|---|
| `/` | Filesystem root | Base of all paths |
| `/etc` | Configuration | Service and application configuration |
| `/var` | Variable data | Logs, caches, queues |
| `/var/log` | Logs | Troubleshooting and monitoring |
| `/home` | User home directories | Source code and user files |
| `/opt` | Optional application software | Manually installed tools |
| `/usr/bin` | User commands | Executables |
| `/usr/lib` | Libraries | Runtime dependencies |
| `/tmp` | Temporary files | Build and temporary data |
| `/run` | Runtime state | PID files, sockets |
| `/proc` | Process/kernel view | Runtime diagnostics |
| `/sys` | Kernel/device view | Hardware and kernel information |
| `/dev` | Device files | Disks and devices |

---

## 14. Why should application code not always be placed in `/var/www`?

`/var/www` is common for web content, but it is not mandatory.

A DevOps deployment should choose a consistent structure such as:

```text
/opt/myapp/
├── releases/
│   ├── 2026-09-13_1000/
│   └── 2026-09-13_1100/
├── current -> releases/2026-09-13_1100
├── shared/
│   ├── logs/
│   └── uploads/
└── config/
```

Advantages:

- Clear separation of releases
- Easy rollback using symlinks
- Application files separated from persistent data
- Predictable paths for Ansible and Jenkins
- Reduced risk of overwriting active files

---

## 15. What is an absolute path?

An absolute path starts from `/`.

```bash
/etc/nginx/nginx.conf
/home/ubuntu/app
```

It does not depend on the current directory.

Use absolute paths in automation when possible:

```bash
/usr/bin/python3 /opt/myapp/app.py
```

This avoids failures caused by Jenkins, cron, systemd, or Ansible using a different working directory.

---

## 16. What is a relative path?

A relative path is interpreted from the current working directory.

```bash
./app.py
../config/app.yaml
logs/app.log
```

Check the current directory:

```bash
pwd
```

In CI/CD, relative paths can fail because the workspace may differ between:

- Local shell
- Jenkins controller
- Jenkins agent
- Ansible remote host
- Systemd service
- Cron job

---

## 17. What is the difference between `PATH` and `PWD`?

`PATH` is the list of directories searched for executable commands.

```bash
echo "$PATH"
```

`PWD` is the current working directory.

```bash
echo "$PWD"
```

Find the executable being used:

```bash
command -v python3
command -v terraform
command -v ansible
```

This is useful when multiple versions are installed.

---

## 18. Why does a command work manually but fail in Jenkins?

Common reasons:

1. Different user
2. Different `PATH`
3. Different working directory
4. Different environment variables
5. Different Python virtual environment
6. Missing permissions
7. Non-interactive shell behavior
8. Missing SSH keys or cloud credentials
9. Different Java or Maven version
10. Different filesystem mounts

Debug inside the pipeline:

```bash
whoami
pwd
id
env | sort
echo "$PATH"
command -v java
java -version
```

Avoid printing secrets. Use Jenkins credentials bindings rather than exposing tokens in logs.

---

## 19. What are environment variables?

Environment variables are key-value values inherited by child processes.

```bash
export APP_ENV=production
echo "$APP_ENV"
```

Run a command with a temporary variable:

```bash
APP_ENV=staging ./deploy.sh
```

Common DevOps variables:

```text
PATH
HOME
USER
SHELL
JAVA_HOME
AWS_REGION
KUBECONFIG
TF_VAR_environment
```

Do not store secrets directly in shell history, source code, or pipeline logs.

---

## 20. What is the difference between shell variables and environment variables?

Shell variable:

```bash
APP_ENV=production
```

Environment variable:

```bash
export APP_ENV=production
```

Only exported variables are inherited by child processes.

Example:

```bash
APP_ENV=production
bash -c 'echo "$APP_ENV"'
```

The child normally receives no value.

```bash
export APP_ENV=production
bash -c 'echo "$APP_ENV"'
```

The child receives `production`.

---

## 21. What is `sudo`?

`sudo` allows an authorized user to execute a command with another user's privileges, commonly root.

```bash
sudo systemctl restart nginx
```

Check current identity:

```bash
id
```

Use least privilege. A deployment account should not automatically receive unrestricted root access unless required.

For Ansible, avoid embedding sudo passwords in playbooks. Prefer:

- SSH keys
- Proper `sudoers` configuration
- Ansible Vault for necessary secrets
- Dedicated deployment users

---

## 22. What is root?

`root` is the superuser with UID `0`.

```bash
id root
```

Root can modify almost every part of the system. Running applications as root increases the impact of application compromise.

Better production practice:

- Create a dedicated service account
- Set ownership only on required directories
- Use `NoNewPrivileges` where appropriate
- Restrict sudo access
- Avoid world-writable application directories

---

## 23. What is a terminal, TTY, and pseudo-terminal?

A terminal provides an interface for interacting with a shell.

A TTY is a terminal device. SSH commonly provides a pseudo-terminal.

Check terminal information:

```bash
tty
```

Check whether a command has a terminal:

```bash
test -t 1 && echo "interactive" || echo "non-interactive"
```

This matters because commands may behave differently in:

- SSH sessions
- Jenkins jobs
- Cron
- Systemd
- Docker exec sessions
- Ansible tasks

---

## 24. What is the difference between login and non-login shells?

A login shell reads login-related startup files. An interactive non-login shell reads different files.

Typical Bash files:

```text
/etc/profile
~/.bash_profile
~/.bash_login
~/.profile
~/.bashrc
```

A common DevOps issue is installing a tool and adding it to `~/.bashrc`, then discovering that Jenkins or systemd cannot find it.

For services, define required environment variables explicitly instead of depending on a user's interactive shell.

---

## 25. What is `cron` and how is it different from systemd timers?

`cron` schedules commands using time-based expressions.

```bash
crontab -e
```

Example:

```cron
*/5 * * * * /usr/local/bin/health-check.sh
```

Systemd timers provide stronger integration with services, logging, dependencies, and status inspection.

Use:

- `cron` for simple scheduled tasks
- systemd timers for service-oriented production workloads
- Jenkins or an orchestrator for deployment workflows

---

## 26. What is an exit status?

Every command returns an exit status.

- `0` usually means success
- Non-zero usually means failure or a special condition

Check it:

```bash
echo $?
```

Example:

```bash
grep -q "ready" app.log
echo $?
```

In CI/CD, exit codes determine whether a build or deployment step succeeds.

Be careful:

```bash
grep pattern file || true
```

This intentionally ignores the failure, but should be used only when failure is expected and documented.

---

## 27. What is the difference between `&&`, `||`, and `;`?

```bash
command1 && command2
```

Runs `command2` only if `command1` succeeds.

```bash
command1 || command2
```

Runs `command2` if `command1` fails.

```bash
command1 ; command2
```

Runs `command2` regardless of the result of `command1`.

Deployment example:

```bash
./build.sh && ./tests.sh && ./deploy.sh
```

This prevents deployment when the build or tests fail.

---

## 28. What is a pipe?

A pipe sends standard output from one command to standard input of another.

```bash
ps aux | grep nginx
```

More reliable process filtering:

```bash
pgrep -af nginx
```

Example:

```bash
journalctl -u nginx --no-pager | grep -i error
```

With `pipefail`, a pipeline can correctly report failure from an earlier command.

---

## 29. What are standard input, output, and error?

Linux processes normally use:

| Descriptor | Number | Meaning |
|---|---:|---|
| stdin | 0 | Standard input |
| stdout | 1 | Standard output |
| stderr | 2 | Standard error |

Redirect output:

```bash
command > output.log
```

Redirect errors:

```bash
command 2> error.log
```

Redirect both:

```bash
command > output.log 2>&1
```

Append:

```bash
command >> output.log 2>&1
```

In production, prefer structured logging and centralized collection instead of uncontrolled local log files.

---

## 30. What is a symlink?

A symbolic link points to another path.

```bash
ln -s /opt/myapp/releases/v2 /opt/myapp/current
```

Inspect it:

```bash
ls -l /opt/myapp/current
readlink -f /opt/myapp/current
```

Symlinks are useful for zero- or low-downtime deployments:

```text
current -> release-v1
current -> release-v2
```

A rollback can point `current` back to the previous release, provided compatibility and service restart requirements are handled.

---

## 31. What is the difference between hard links and symbolic links?

### Hard link

- Points to the same inode
- Normally cannot cross filesystems
- Usually cannot link directories
- Remains valid if the original filename is deleted

```bash
ln file1 file2
```

### Symbolic link

- Stores a path
- Can cross filesystems
- Can point to directories
- Breaks if the target path disappears

```bash
ln -s file1 link1
```

DevOps deployments generally use symbolic links for release switching.

---

## 32. What is an inode?

An inode stores filesystem metadata such as:

- File type
- Owner
- Permissions
- Size
- Timestamps
- Link count
- Data block references

Check inode usage:

```bash
df -i
```

A filesystem can have free disk space but no free inodes. This can prevent new files from being created.

Typical cause:

- Millions of tiny files
- Poor temporary-file cleanup
- Large build caches
- Application-generated session files

---

## 33. What is a mount point?

A mount point is a directory where a filesystem becomes accessible.

```bash
findmnt
lsblk -f
df -hT
```

Example:

```text
/dev/xvdf1 mounted on /data
```

DevOps relevance:

- Application data may be on a separate EBS volume.
- Logs may use a dedicated filesystem.
- A missing mount can cause an application to write to the root disk unexpectedly.
- A mount failure can make a service start with an empty directory.

---

## 34. What is swap?

Swap is disk space used as an extension of virtual memory.

Check:

```bash
free -h
swapon --show
```

Swap is slower than RAM. High swap activity can cause latency.

In cloud environments, investigate:

- Instance memory size
- Application memory leaks
- JVM heap configuration
- Container memory limits
- Kernel memory pressure

Do not assume that adding swap fixes the underlying memory problem.

---

## 35. What is load average?

Load average represents the average number of tasks waiting for CPU or uninterruptible execution, commonly related to I/O.

```bash
uptime
w
top
```

A load average of `4` means different things on a 1-vCPU and a 16-vCPU machine.

Compare load with CPU count:

```bash
nproc
```

Interpretation requires checking:

- CPU utilization
- I/O wait
- Runnable processes
- Disk latency
- Memory pressure

---

## 36. What is the difference between CPU usage and load average?

CPU usage tells how busy CPUs are.

Load average includes tasks waiting for CPU and certain uninterruptible states, often I/O-related.

Example:

- CPU usage may be low.
- Load average may be high.
- The reason may be blocked disk I/O rather than CPU saturation.

Useful commands:

```bash
top
vmstat 1
iostat -xz 1
```

---

## 37. What is a zombie process?

A zombie is a terminated child process whose parent has not yet collected its exit status.

Find zombies:

```bash
ps -eo stat,pid,ppid,comm | awk '$1 ~ /^Z/'
```

A zombie does not consume normal CPU, but many zombies can consume process-table entries.

The correct fix is usually to investigate the parent process and its child-reaping logic. Killing a zombie itself does not work because it has already exited.

---

## 38. What is an orphan process?

An orphan is a process whose original parent has exited.

The process is re-parented to another process, usually PID 1 or a subreaper.

Orphans are not automatically bad. However, unexpected orphan processes may indicate:

- Incorrect application shutdown
- Broken process supervision
- Poor shell scripting
- Incorrect use of background processes

Use systemd or a proper process manager for long-running applications.

---

## 39. What is a file descriptor?

A file descriptor is a number representing an open file, socket, pipe, or similar resource.

Check limits:

```bash
ulimit -n
```

Inspect open descriptors:

```bash
lsof -p <PID>
```

Check system-wide usage:

```bash
cat /proc/sys/fs/file-nr
```

Applications may fail with errors such as:

```text
Too many open files
```

Possible causes:

- File descriptors not closed
- Too many network connections
- Incorrect connection pooling
- Excessive log files
- Low service limits

---

## 40. What is a service account?

A service account is a dedicated operating-system user used to run an application or agent.

Example:

```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin myapp
```

Benefits:

- Limits permissions
- Separates application ownership
- Improves auditing
- Reduces impact of compromise
- Avoids running services as root

---

## 41. What is idempotency in Linux automation?

An operation is idempotent when running it multiple times produces the same intended state.

Bad example:

```bash
echo "export APP_ENV=prod" >> ~/.bashrc
```

Repeated execution adds duplicate lines.

Better approach:

```bash
grep -qxF 'export APP_ENV=prod' ~/.bashrc || \
  echo 'export APP_ENV=prod' >> ~/.bashrc
```

Configuration-management tools such as Ansible are designed around desired state and idempotency.

---

## 42. Why should DevOps engineers avoid using `kill -9` immediately?

`kill -9` sends `SIGKILL`, which cannot be caught or handled.

It prevents an application from:

- Closing files cleanly
- Flushing buffers
- Finishing transactions
- Removing temporary files
- Releasing application-level locks

Preferred sequence:

```bash
kill -TERM <PID>
sleep 5
kill -KILL <PID>
```

Use `SIGKILL` only when graceful termination fails or the process is irrecoverably stuck.

---

## 43. What is graceful shutdown?

Graceful shutdown allows an application to stop accepting new work and finish or safely terminate existing work.

For services:

```bash
systemctl stop myapp
```

For a process:

```bash
kill -TERM <PID>
```

Graceful shutdown is important for:

- APIs
- Message consumers
- Database clients
- Kubernetes workloads
- Rolling deployments
- Load-balanced services

A deployment that kills processes abruptly may cause failed requests or duplicate message processing.

---

## 44. What Linux information should you collect first during an incident?

A practical initial collection:

```bash
date
hostname
uptime
whoami
id
uname -a
cat /etc/os-release
df -hT
df -i
free -h
uptime
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
ss -tulpn
systemctl --failed
journalctl -p err -b --no-pager
```

Also collect:

```bash
ip addr
ip route
resolvectl status
```

Do not change the system before collecting enough evidence unless there is an urgent availability or security requirement.

---

## 45. Scenario: The Jenkins deployment says “permission denied”

Check:

```bash
whoami
id
namei -l /opt/myapp/current
ls -ld /opt /opt/myapp
ls -l /opt/myapp/current
```

Possible causes:

- Jenkins user lacks directory execute permission
- File is owned by another user
- Service account cannot read configuration
- Parent directory is inaccessible
- SELinux/AppArmor policy blocks access
- Filesystem is mounted read-only

Fix the ownership and permissions according to the required access, not by blindly running:

```bash
chmod -R 777 /opt/myapp
```

---

## 46. Scenario: The application works over SSH but fails as a systemd service

Likely causes:

- Different `PATH`
- Missing environment variables
- Incorrect `WorkingDirectory`
- Relative paths
- Wrong user
- Missing permissions
- Different virtual environment
- Service starts before a dependency is ready

Inspect:

```bash
systemctl status myapp
journalctl -u myapp -b --no-pager
systemctl cat myapp
```

Use explicit values in the unit:

```ini
[Service]
User=myapp
WorkingDirectory=/opt/myapp/current
EnvironmentFile=/etc/myapp/myapp.env
ExecStart=/opt/myapp/venv/bin/gunicorn app:app
Restart=on-failure
```

---

## 47. Scenario: A server has free disk space, but the application cannot create files

Check inode usage:

```bash
df -h
df -i
```

Find directories with many files:

```bash
find /var -xdev -type f | cut -d/ -f1-4 | sort | uniq -c | sort -nr | head
```

Also check deleted files still held open:

```bash
lsof +L1
```

A deleted log file may still consume disk space until the process closes it.

---

## 48. Scenario: Port 8080 is not reachable

Check whether the application listens:

```bash
ss -ltnp | grep ':8080'
```

Test locally:

```bash
curl -v http://127.0.0.1:8080/health
```

Check:

```bash
ip addr
ip route
sudo ufw status
sudo nft list ruleset
```

For cloud systems also check:

- Security group
- Network ACL
- Route table
- Load balancer target health
- Application bind address
- Reverse-proxy configuration

An application listening on `127.0.0.1:8080` is not reachable through the server's external IP. It may need to bind to the appropriate interface.

---

## 49. Scenario: A deployment filled `/tmp`

Investigate:

```bash
df -h /tmp
sudo du -xhd1 /tmp | sort -h
find /tmp -xdev -type f -printf '%s %p\n' | sort -nr | head
```

Check which processes hold deleted files:

```bash
sudo lsof +L1
```

Prevention:

- Configure application cleanup
- Use systemd `RuntimeDirectory` or `TemporaryDirectory`
- Configure log rotation
- Set retention policies
- Monitor filesystem usage
- Avoid storing permanent application data in `/tmp`

---

## 50. DevOps command quick reference

```bash
# Identity
whoami
id
hostnamectl

# OS and kernel
cat /etc/os-release
uname -r

# Filesystem
pwd
ls -lah
find .
df -hT
df -i
du -sh *

# Processes
ps aux
pgrep -af java
top
kill -TERM <PID>

# Services
systemctl status <service>
systemctl restart <service>
systemctl enable <service>
journalctl -u <service> -f

# Networking
ip addr
ip route
ss -ltnp
curl -v http://127.0.0.1:8080
dig example.com

# Logs
journalctl -b
tail -f /var/log/syslog
grep -i error app.log

# Environment
env
printenv PATH
command -v terraform

# Storage and memory
free -h
lsblk
mount
findmnt
```

---

## Interview checklist

You should be able to explain and demonstrate:

- Linux architecture
- Kernel versus distribution
- Shell versus process
- PID and PPID
- Foreground/background execution
- Service versus process
- Filesystem hierarchy
- Absolute versus relative paths
- Environment variables
- `sudo` and service accounts
- Exit codes
- Pipes and redirection
- Symlinks
- Inodes
- Mount points
- Swap and load average
- Zombies and orphans
- File descriptors
- Graceful shutdown
- Idempotent automation
- Initial incident triage
- Jenkins versus SSH environment differences
- Port and permission troubleshooting

---

## Key DevOps principle

> Do not merely ask whether a Linux command works. Ask whether it is repeatable, observable, secure, recoverable, and suitable for automation.
