# Linux L2 / Senior DevOps Engineer Interview Questions & Answers

This guide is designed for **L2 / Senior Linux / DevOps Engineer interviews**.  
It goes beyond basic definitions and focuses on:

- What the concept means
- Why it is used
- Important commands
- Troubleshooting approach
- Real production examples
- Interview follow-up questions

---

## Table of Contents

1. What is Linux?
2. Linux vs Unix
3. Linux filesystem hierarchy
4. What is a shell?
5. How do you find files?
6. How does `grep` work?
7. Soft link vs hard link
8. How do you manage services?
9. Shell scripting vs programming languages
10. What is SSH?
11. How do you check system resource usage?
12. What is a package manager?
13. How does `chmod` work?
14. What is the Linux kernel?
15. Archive and compression
16. Explain Linux boot process
17. What are processes and threads?
18. How do you troubleshoot a high CPU issue?
19. How do you troubleshoot high memory usage?
20. What is load average?
21. How do you troubleshoot disk space issues?
22. What is an inode?
23. What is the difference between `df` and `du`?
24. What are file descriptors?
25. What is `/proc`?
26. What is `/sys`?
27. Explain Linux permissions and ownership
28. What is `sudo`?
29. Explain `umask`
30. How do you schedule jobs with cron?
31. Cron vs systemd timers
32. What are environment variables?
33. What is PATH?
34. How do you troubleshoot a command not found?
35. How do you troubleshoot a service that will not start?
36. How do you troubleshoot a port connectivity problem?
37. Explain `ss` and network troubleshooting
38. Explain DNS troubleshooting in Linux
39. What are signals and `kill`?
40. `kill`, `pkill`, and `killall`
41. What is a zombie process?
42. What is an orphan process?
43. Explain `ps` and process investigation
44. What is `nice` and `renice`?
45. Explain `systemctl`
46. Explain `/etc/fstab`
47. What is mounting?
48. Explain LVM
49. How do you extend an LVM filesystem?
50. How do you troubleshoot a read-only filesystem?
51. What is swap?
52. How do you troubleshoot disk I/O?
53. What are logs and `journalctl`?
54. Explain log rotation
55. How do you find large files?
56. How do you identify deleted files still consuming disk space?
57. What is SELinux?
58. What is AppArmor?
59. SELinux vs AppArmor
60. How do you troubleshoot permission denied?
61. How do you troubleshoot SSH login failure?
62. SSH key-based authentication
63. SSH hardening
64. What is `/etc/hosts`?
65. Explain `/etc/resolv.conf`
66. TCP vs UDP
67. Common Linux networking commands
68. How do you identify which process owns a port?
69. How do you test connectivity at different layers?
70. Explain localhost, 0.0.0.0 and 127.0.0.1
71. What is a default gateway?
72. What is a routing table?
73. What is a firewall?
74. `iptables` vs `nftables`
75. What is `systemd`?
76. Explain targets/runlevels
77. Explain `/etc/passwd`, `/etc/shadow`, `/etc/group`
78. User and group management
79. Explain sticky bit, SUID and SGID
80. What happens when you execute a Linux command?
81. Bash exit codes
82. Pipes and redirection
83. `find -exec` and xargs
84. `awk` vs `sed`
85. How do you troubleshoot a production Linux server systematically?
86. Senior L2 production troubleshooting scenario
87. Quick command cheat sheet
88. Interview answer framework

---

# 1. What is Linux?

Linux is an **open-source Unix-like operating system kernel**. In common usage, "Linux" often refers to a complete operating system distribution built around the Linux kernel, GNU utilities, libraries, package managers, and other software.

Examples:

- Ubuntu
- Debian
- Red Hat Enterprise Linux (RHEL)
- Rocky Linux
- AlmaLinux
- Fedora
- Amazon Linux

### Key responsibilities of the kernel

The kernel manages:

- CPU scheduling
- Memory management
- Processes
- Devices
- Filesystems
- Networking
- System calls
- Security mechanisms

### Senior interview answer

> Linux is an open-source Unix-like operating system based on the Linux kernel. The kernel manages hardware resources such as CPU, memory, devices and networking, while user-space utilities, libraries and applications provide the complete operating-system environment.

---

# 2. Differentiate between Linux and Unix

| Linux | Unix |
|---|---|
| Open-source kernel | Traditionally proprietary/commercial Unix systems |
| Developed initially by Linus Torvalds | Unix originated at AT&T Bell Labs |
| Many distributions | Vendor-specific implementations |
| Ubuntu, RHEL, Debian | AIX, Solaris, HP-UX |
| Large community ecosystem | Traditionally vendor-supported |
| Common in cloud/DevOps | Common in legacy enterprise environments |

Linux was designed as a Unix-like system but is not simply a copy of traditional commercial Unix.

### Interview point

A good answer should avoid saying:

> "Linux is Unix."

Better:

> "Linux is Unix-like; it follows many Unix design principles but has its own kernel and implementation."

---

# 3. Explain the Linux filesystem hierarchy

Linux uses a **single hierarchical filesystem tree** beginning at `/`.

Important directories:

| Directory | Purpose |
|---|---|
| `/` | Root of filesystem |
| `/bin` | Essential user commands |
| `/sbin` | Essential system/admin commands |
| `/boot` | Bootloader and kernel files |
| `/dev` | Device files |
| `/etc` | System configuration |
| `/home` | User home directories |
| `/root` | Root user's home |
| `/var` | Variable data such as logs |
| `/tmp` | Temporary files |
| `/usr` | User applications/libraries |
| `/opt` | Optional third-party software |
| `/proc` | Process/kernel runtime information |
| `/sys` | Kernel/device information |
| `/run` | Runtime state |
| `/mnt` | Temporary mount point |
| `/media` | Removable media |

### Important production examples

Logs:

```bash
/var/log/
```

Service configuration:

```bash
/etc/systemd/system/
```

SSH configuration:

```bash
/etc/ssh/sshd_config
```

User data:

```bash
/home/mukesh/
```

---

# 4. What is a shell?

A shell is a **command interpreter** that provides an interface between the user and the operating system.

Common shells:

- Bash
- Zsh
- Dash
- Ksh
- Fish

Check your shell:

```bash
echo $SHELL
```

Check available shells:

```bash
cat /etc/shells
```

Current process shell can also be investigated with:

```bash
ps -p $$ -o comm=
```

### Shell responsibilities

A shell:

1. Reads commands.
2. Expands variables.
3. Performs wildcard expansion.
4. Handles pipes and redirection.
5. Starts processes.
6. Returns exit status.

Example:

```bash
cat /var/log/syslog | grep ERROR
```

The shell creates the pipeline between the commands.

---

# 5. How do you find files in Linux?

The most important command is `find`.

Example:

```bash
find /var/log -name "*.log"
```

Find by type:

```bash
find /var/log -type f
```

Find directories:

```bash
find /opt -type d
```

Find files larger than 1 GB:

```bash
find / -type f -size +1G 2>/dev/null
```

Find files modified in the last 24 hours:

```bash
find /var/log -type f -mtime -1
```

Find files by owner:

```bash
find /home -user mukesh
```

### `locate`

```bash
locate nginx.conf
```

`locate` is generally faster because it searches an indexed database, but its database may not contain newly created files until updated.

### Senior point

Use `find` when you need real-time filesystem traversal and complex conditions.

---

# 6. Explain the use of grep

`grep` searches text for matching patterns.

Basic:

```bash
grep "ERROR" application.log
```

Case-insensitive:

```bash
grep -i "error" application.log
```

Show line numbers:

```bash
grep -n "ERROR" application.log
```

Recursive search:

```bash
grep -r "database" /etc/myapp/
```

Invert match:

```bash
grep -v "INFO" application.log
```

Count matches:

```bash
grep -c "ERROR" application.log
```

Extended regular expressions:

```bash
grep -E "ERROR|WARN|CRITICAL" application.log
```

### Production example

```bash
grep -i "connection refused" /var/log/myapp/app.log
```

Then correlate with service status:

```bash
systemctl status myapp
```

---

# 7. Soft link vs hard link

## Soft/symbolic link

A symbolic link points to another pathname.

```bash
ln -s /opt/app/config.yaml /etc/app-config.yaml
```

Check:

```bash
ls -l /etc/app-config.yaml
```

If the original target is deleted, the symbolic link becomes broken.

## Hard link

```bash
ln original.txt hardlink.txt
```

Both names reference the same inode.

### Differences

| Soft Link | Hard Link |
|---|---|
| Points to pathname | Points to inode |
| Different inode | Same inode |
| Can cross filesystems | Normally cannot cross filesystems |
| Can link directories in some environments, but generally avoided | Cannot normally link directories |
| Can become dangling | Remains valid while another hard link exists |

Check inode:

```bash
ls -li original.txt hardlink.txt
```

### Senior interview point

Deleting a filename does not necessarily delete the underlying data. The inode/data remains while at least one hard link references it.

---

# 8. How do you manage services in Linux?

Modern Linux distributions commonly use `systemd`.

Start:

```bash
sudo systemctl start nginx
```

Stop:

```bash
sudo systemctl stop nginx
```

Restart:

```bash
sudo systemctl restart nginx
```

Reload:

```bash
sudo systemctl reload nginx
```

Enable at boot:

```bash
sudo systemctl enable nginx
```

Enable and start:

```bash
sudo systemctl enable --now nginx
```

Check status:

```bash
systemctl status nginx
```

Check whether running:

```bash
systemctl is-active nginx
```

Check logs:

```bash
journalctl -u nginx
```

### Senior troubleshooting sequence

```bash
systemctl status nginx
journalctl -u nginx -n 100 --no-pager
ss -lntp
```

Then inspect:

- configuration
- dependencies
- permissions
- ports
- certificates
- environment variables
- disk space
- SELinux/AppArmor
- resource limits

---

# 9. Shell scripting vs programming languages

Shell scripting is optimized for **command orchestration and operating-system automation**.

Example:

```bash
#!/bin/bash

systemctl restart nginx
systemctl is-active nginx
```

Programming languages such as Python, Go and Java are better suited for:

- complex business logic
- large applications
- structured data processing
- APIs
- concurrency
- testing
- maintainable software systems

### Interview answer

> I use shell scripting primarily for Linux automation, deployment tasks, file operations and command orchestration. For complex application logic or reusable tooling, I prefer Python, Go or another general-purpose language.

---

# 10. What is SSH and how is it used?

SSH stands for **Secure Shell**. It provides encrypted remote access to systems.

Basic:

```bash
ssh user@server
```

Specify key:

```bash
ssh -i mykey.pem ubuntu@server
```

Specify port:

```bash
ssh -p 2222 user@server
```

Copy files:

```bash
scp file.txt user@server:/tmp/
```

SSH keys:

```bash
ssh-keygen
```

Public key:

```bash
~/.ssh/id_ed25519.pub
```

Server authorized keys:

```bash
~/.ssh/authorized_keys
```

### Troubleshoot SSH

Run verbose mode:

```bash
ssh -vvv user@server
```

Check server:

```bash
systemctl status ssh
```

Check listening port:

```bash
ss -lntp | grep :22
```

Check firewall/security group/network ACL where applicable.

---

# 11. How do you check system resource usage?

### CPU

```bash
top
```

or:

```bash
htop
```

### Memory

```bash
free -h
```

### Disk

```bash
df -h
```

### Disk usage by directories

```bash
du -sh /var/*
```

### Load

```bash
uptime
```

### Processes

```bash
ps aux
```

### I/O

```bash
iostat
```

### Memory statistics

```bash
vmstat 1
```

### Senior approach

Do not immediately assume CPU is the problem.

Check:

```text
uptime
top
free -h
df -h
iostat
vmstat
ps aux
```

Then determine whether the bottleneck is:

- CPU
- memory
- disk I/O
- filesystem
- network
- process saturation

---

# 12. What is a package manager?

A package manager installs, upgrades, removes and manages software packages and dependencies.

Debian/Ubuntu:

```bash
apt update
apt install nginx
apt remove nginx
apt upgrade
```

RHEL/Fedora:

```bash
dnf install nginx
dnf update
dnf remove nginx
```

Older RHEL systems may use:

```bash
yum
```

### Why package managers matter

They manage:

- dependencies
- package versions
- repositories
- upgrades
- package metadata
- installation/removal

Check installed package:

```bash
dpkg -l | grep nginx
```

or:

```bash
rpm -qa | grep nginx
```

---

# 13. Explain chmod

`chmod` changes file permissions.

Example:

```bash
chmod 755 script.sh
```

Meaning:

```text
Owner  = rwx
Group  = r-x
Others = r-x
```

Numeric values:

```text
r = 4
w = 2
x = 1
```

Therefore:

```text
7 = rwx
6 = rw-
5 = r-x
4 = r--
0 = ---
```

Symbolic:

```bash
chmod u+x script.sh
chmod g+w file.txt
chmod o-r file.txt
```

Recursive:

```bash
chmod -R 755 /opt/app
```

### Senior warning

Avoid blindly doing:

```bash
chmod -R 777 /opt/app
```

This can create serious security problems.

---

# 14. What is the kernel?

The kernel is the core component responsible for managing system resources and providing the interface between applications and hardware.

It handles:

- CPU scheduling
- memory
- processes
- networking
- devices
- filesystems
- system calls

Check kernel:

```bash
uname -r
```

More information:

```bash
uname -a
```

Kernel messages:

```bash
dmesg
```

On systemd systems:

```bash
journalctl -k
```

---

# 15. How do you archive and compress files?

`tar` is commonly used for archiving.

Create archive:

```bash
tar -cvf backup.tar /etc/myapp
```

Extract:

```bash
tar -xvf backup.tar
```

Gzip:

```bash
tar -czvf backup.tar.gz /etc/myapp
```

Extract:

```bash
tar -xzvf backup.tar.gz
```

Bzip2:

```bash
tar -cjvf backup.tar.bz2 /etc/myapp
```

Xz:

```bash
tar -cJvf backup.tar.xz /etc/myapp
```

List archive:

```bash
tar -tvf backup.tar
```

### Important distinction

`tar` primarily **archives** files.

`gzip`, `bzip2` and `xz` **compress** data.

A command such as:

```bash
tar -czf backup.tar.gz directory/
```

combines both operations.

---

# 16. Explain the Linux boot process

A simplified boot sequence is:

```text
BIOS/UEFI
   |
   v
Bootloader (GRUB)
   |
   v
Linux Kernel
   |
   v
initramfs
   |
   v
systemd (PID 1)
   |
   v
Targets/services
```

### Troubleshooting boot

Useful commands:

```bash
systemctl --failed
journalctl -b
journalctl -b -p err
dmesg
```

Check boot time:

```bash
systemd-analyze
```

Detailed blame:

```bash
systemd-analyze blame
```

---

# 17. What are processes and threads?

A **process** is an executing instance of a program with its own process resources.

A thread is an execution unit within a process.

Inspect:

```bash
ps -ef
```

Process tree:

```bash
pstree
```

Threads:

```bash
ps -eLf
```

PID:

```bash
pgrep nginx
```

### Senior point

Processes provide stronger isolation. Threads within the same process generally share the process address space and resources.

---

# 18. How do you troubleshoot high CPU usage?

Start with:

```bash
uptime
top
```

Identify CPU-consuming processes:

```bash
ps -eo pid,ppid,cmd,%cpu --sort=-%cpu | head
```

Then investigate the process.

Check whether CPU is:

- user CPU
- system CPU
- I/O wait
- steal time

Using:

```bash
top
```

Check per-CPU behavior:

```bash
mpstat -P ALL 1
```

### L2 approach

1. Confirm the symptom.
2. Identify the process.
3. Determine whether CPU usage is sustained.
4. Check recent deployments/config changes.
5. Inspect application logs.
6. Check threads if required.
7. Determine whether the process is expected to be busy.
8. Restart only when justified.

---

# 19. How do you troubleshoot high memory usage?

Start:

```bash
free -h
```

Then:

```bash
ps aux --sort=-%mem | head
```

Check memory continuously:

```bash
vmstat 1
```

Inspect a process:

```bash
pmap -x <PID>
```

### Important distinction

Linux uses otherwise-unused RAM for filesystem cache. Therefore:

> High memory usage does not automatically mean a memory problem.

Pay attention to:

- `available` memory
- swap activity
- OOM events
- process RSS
- memory growth over time

Check OOM events:

```bash
journalctl -k | grep -i oom
```

---

# 20. What is load average?

Load average represents the average number of tasks that are runnable or waiting for certain uninterruptible operations, commonly I/O.

Check:

```bash
uptime
```

Example:

```text
load average: 2.00, 1.50, 1.00
```

These correspond approximately to:

```text
1 minute
5 minutes
15 minutes
```

### Important interview point

A load average of `4` means different things on a 4-core system versus a 16-core system.

Check CPU count:

```bash
nproc
```

A high load can result from CPU pressure **or I/O-related waiting**.

Therefore, never diagnose load using load average alone.

---

# 21. How do you troubleshoot disk space issues?

Check filesystem usage:

```bash
df -h
```

Then find large directories:

```bash
du -xhd1 /var | sort -h
```

Find large files:

```bash
find /var -xdev -type f -size +1G -ls
```

Also check inode usage:

```bash
df -i
```

### Common causes

- large application logs
- old backups
- container/image data
- core dumps
- temporary files
- deleted-but-open files

---

# 22. What is an inode?

An inode stores filesystem metadata about a file, such as:

- ownership
- permissions
- timestamps
- file size
- file type
- pointers to data blocks

The filename is stored separately in directory structures.

Check inode:

```bash
ls -li file.txt
```

Filesystem inode usage:

```bash
df -i
```

### Important scenario

A filesystem can show:

```text
Disk space: 20% used
Inodes: 100% used
```

You may be unable to create new files even though free disk space exists.

This commonly happens when there are huge numbers of small files.

---

# 23. Difference between df and du

## `df`

Shows filesystem-level usage.

```bash
df -h
```

It answers:

> How full is the filesystem?

## `du`

Shows directory/file-level usage.

```bash
du -sh /var/log
```

It answers:

> Which files/directories are consuming space?

### Troubleshooting example

If:

```bash
df -h
```

shows 90% usage, use:

```bash
du -xhd1 /
```

to locate the consuming directory.

If `df` and `du` appear inconsistent, investigate deleted-but-open files.

---

# 24. What are file descriptors?

A file descriptor (FD) is a numeric handle used by a process to access an open file, socket, pipe, device, etc.

Standard descriptors:

```text
0 = stdin
1 = stdout
2 = stderr
```

Check process FDs:

```bash
ls -l /proc/<PID>/fd
```

Check system limits:

```bash
ulimit -n
```

Check process limits:

```bash
cat /proc/<PID>/limits
```

### Production problem

If an application reaches its open-file limit, you may see:

```text
Too many open files
```

Then investigate:

- FD limit
- application connection leaks
- number of sockets
- log files
- service configuration

---

# 25. What is /proc?

`/proc` is a virtual filesystem exposing kernel and process information.

Examples:

```bash
cat /proc/cpuinfo
cat /proc/meminfo
cat /proc/uptime
```

Process information:

```bash
cat /proc/1234/status
```

Command line:

```bash
cat /proc/1234/cmdline
```

Open file descriptors:

```bash
ls -l /proc/1234/fd
```

`/proc` is extremely useful during production troubleshooting.

---

# 26. What is /sys?

`/sys` is a virtual filesystem exposing information and controls related to devices, drivers, kernel subsystems and hardware.

Examples:

```bash
ls /sys/class/net
```

Network interfaces:

```bash
ls /sys/class/net/
```

It is commonly used by system tools and administrators for hardware/kernel inspection.

---

# 27. Explain Linux permissions and ownership

Every file has:

- owner
- group
- permissions

Example:

```bash
-rwxr-x---
```

Breakdown:

```text
-       file type
rwx     owner
r-x     group
---     others
```

Check ownership:

```bash
ls -l file
```

Change owner:

```bash
sudo chown user:group file
```

Change group:

```bash
sudo chgrp group file
```

Change permissions:

```bash
chmod 640 file
```

---

# 28. What is sudo?

`sudo` allows an authorized user to execute commands with another user's privileges, commonly root.

Example:

```bash
sudo systemctl restart nginx
```

Configuration:

```bash
/etc/sudoers
```

Safely edit:

```bash
sudo visudo
```

### Senior security principle

Prefer least privilege rather than giving users unrestricted root access.

A user should receive only the permissions necessary to perform their job.

---

# 29. Explain umask

`umask` controls which permission bits are removed from newly created files/directories.

Check:

```bash
umask
```

Example:

```text
0022
```

Conceptually, applications start with default permissions and the umask removes permissions.

Typical defaults:

```text
Files:       666
Directories: 777
```

With a common umask of `022`:

```text
Files:       644
Directories: 755
```

Actual creation behavior also depends on the application.

---

# 30. How do you schedule jobs with cron?

Edit current user's crontab:

```bash
crontab -e
```

List:

```bash
crontab -l
```

Example:

```cron
0 2 * * * /opt/scripts/backup.sh
```

Meaning:

```text
minute = 0
hour   = 2
day    = every day
month  = every month
weekday= every day
```

Check cron service:

```bash
systemctl status cron
```

On RHEL-like systems:

```bash
systemctl status crond
```

### Production recommendation

Use absolute paths and redirect logs:

```cron
0 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1
```

---

# 31. Cron vs systemd timers

Cron is simple and widely supported.

Systemd timers provide tighter integration with systemd.

Advantages of timers can include:

- service dependency handling
- centralized journal logging
- calendar/monotonic scheduling
- better integration with systemd units

For modern systemd-based environments, timers are often preferred for service-oriented scheduled jobs.

---

# 32. What are environment variables?

Environment variables are key-value data inherited by processes.

Example:

```bash
export APP_ENV=production
```

Check:

```bash
echo $APP_ENV
```

All environment:

```bash
env
```

Common variables:

```text
PATH
HOME
USER
SHELL
LANG
```

For systemd services, environment variables can be defined using unit configuration, environment files, or drop-ins.

---

# 33. What is PATH?

`PATH` tells the shell where to search for executable commands.

Check:

```bash
echo $PATH
```

Find command location:

```bash
which nginx
```

Better for shell command lookup:

```bash
command -v nginx
```

Example:

If `/usr/local/bin` is in PATH, typing:

```bash
mycommand
```

causes the shell to search directories in PATH for the executable.

---

# 34. How do you troubleshoot "command not found"?

Check:

```bash
command -v command_name
```

Check PATH:

```bash
echo $PATH
```

Check whether package is installed.

For Ubuntu:

```bash
dpkg -l | grep package
```

For RHEL:

```bash
rpm -q package
```

If installed but unavailable:

- executable may not be in PATH
- shell environment may differ
- command may be installed under a different name
- permissions may prevent execution

---

# 35. How do you troubleshoot a service that will not start?

Use a structured approach.

### Step 1

```bash
systemctl status myservice
```

### Step 2

```bash
journalctl -u myservice -n 100 --no-pager
```

### Step 3

Check configuration.

For example:

```bash
nginx -t
```

### Step 4

Check port:

```bash
ss -lntp
```

### Step 5

Check dependencies:

```bash
systemctl list-dependencies myservice
```

### Step 6

Check resources:

```bash
df -h
free -h
```

### Step 7

Check permissions/SELinux/AppArmor.

### Step 8

Start again after fixing the root cause.

Do not repeatedly restart without understanding the failure.

---

# 36. How do you troubleshoot a port connectivity problem?

Break the problem into layers.

### Is service listening?

```bash
ss -lntp
```

### Is the local service healthy?

```bash
systemctl status myservice
```

### Test locally

```bash
curl http://127.0.0.1:8080
```

### Test remote TCP connectivity

```bash
nc -vz server 8080
```

Then inspect:

- local firewall
- remote firewall
- cloud security group
- network ACL
- route
- load balancer
- service binding address

### Critical distinction

A service listening on:

```text
127.0.0.1:8080
```

is not reachable through the server's external IP.

A service listening on:

```text
0.0.0.0:8080
```

can listen on all IPv4 interfaces, subject to firewall/network controls.

---

# 37. Explain ss

`ss` displays socket statistics.

Listening TCP ports:

```bash
ss -lnt
```

Listening TCP with processes:

```bash
sudo ss -lntp
```

UDP:

```bash
ss -lun
```

Established connections:

```bash
ss -tn state established
```

Find port:

```bash
ss -lntp | grep :8080
```

### Why `ss`?

It is the modern replacement commonly used instead of older `netstat` workflows.

---

# 38. Explain DNS troubleshooting

First inspect resolver configuration:

```bash
cat /etc/resolv.conf
```

Test DNS:

```bash
getent hosts example.com
```

Using DNS utilities if installed:

```bash
dig example.com
```

or:

```bash
nslookup example.com
```

Test specific DNS server:

```bash
dig @8.8.8.8 example.com
```

### Troubleshooting sequence

```text
Application
   |
   v
Resolver configuration
   |
   v
DNS server
   |
   v
Network connectivity
```

Check whether the issue is:

- DNS resolution
- TCP connectivity
- firewall
- application configuration

---

# 39. What are signals?

Signals are asynchronous notifications delivered to processes.

Common signals:

```text
SIGTERM = 15
SIGKILL = 9
SIGHUP  = 1
SIGINT  = 2
```

Terminate gracefully:

```bash
kill -15 <PID>
```

Force kill:

```bash
kill -9 <PID>
```

### Important interview point

`SIGTERM` gives the process an opportunity to clean up.

`SIGKILL` cannot be caught or handled by the process.

Therefore:

> Prefer SIGTERM first; use SIGKILL only when necessary.

---

# 40. kill vs pkill vs killall

### kill

Targets a PID:

```bash
kill 1234
```

### pkill

Can target by process name/pattern:

```bash
pkill nginx
```

### killall

Targets processes by exact command name on typical Linux implementations:

```bash
killall nginx
```

Use carefully in production because multiple processes may match.

---

# 41. What is a zombie process?

A zombie is a process that has finished execution but whose parent has not yet collected its exit status.

Find zombies:

```bash
ps -eo pid,ppid,state,cmd | awk '$3=="Z"'
```

A zombie is already dead; sending `kill -9` to the zombie itself does not solve the problem.

Investigate its parent process.

```bash
ps -p <PPID> -o pid,cmd
```

The parent normally needs to reap the child.

---

# 42. What is an orphan process?

An orphan is a process whose original parent has exited.

The process is re-parented to an appropriate system process, traditionally PID 1 or another designated subreaper.

Check:

```bash
ps -eo pid,ppid,cmd
```

Orphans are not inherently problematic. Zombies and resource leaks are the bigger concern.

---

# 43. Explain ps

`ps` provides a snapshot of processes.

Common:

```bash
ps aux
```

Full process hierarchy:

```bash
ps -ef
```

Find a process:

```bash
ps -ef | grep nginx
```

Better:

```bash
pgrep -a nginx
```

Custom fields:

```bash
ps -eo pid,ppid,user,%cpu,%mem,stat,cmd
```

Sort by CPU:

```bash
ps -eo pid,cmd,%cpu --sort=-%cpu | head
```

---

# 44. What are nice and renice?

Linux process scheduling has a **niceness** value.

Check:

```bash
ps -eo pid,ni,cmd
```

Start with a niceness:

```bash
nice -n 10 command
```

Change existing process:

```bash
renice 10 -p <PID>
```

Higher niceness generally means lower scheduling priority relative to less-nice processes.

Use carefully; changing priority can affect application behavior.

---

# 45. Explain systemctl

`systemctl` is the primary management command for systemd.

Useful commands:

```bash
systemctl status nginx
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl reload nginx
systemctl enable nginx
systemctl disable nginx
systemctl enable --now nginx
```

List services:

```bash
systemctl list-units --type=service
```

Failed units:

```bash
systemctl --failed
```

Inspect unit:

```bash
systemctl cat nginx
```

Show dependencies:

```bash
systemctl list-dependencies nginx
```

---

# 46. Explain /etc/fstab

`/etc/fstab` defines filesystems that should be mounted automatically.

Typical fields:

```text
device  mountpoint  filesystem  options  dump  fsck
```

Example:

```text
/dev/xvdf1 /data ext4 defaults,nofail 0 2
```

After changing fstab, test:

```bash
sudo mount -a
```

### Senior warning

A malformed `/etc/fstab` can cause boot problems.

Always validate carefully before rebooting.

Using UUIDs is often preferable:

```bash
blkid
```

---

# 47. What is mounting?

Mounting attaches a filesystem to a directory in the Linux filesystem tree.

Example:

```bash
sudo mount /dev/xvdf1 /data
```

Check:

```bash
mount
```

or:

```bash
findmnt
```

Unmount:

```bash
sudo umount /data
```

If busy:

```bash
sudo lsof +D /data
```

or:

```bash
sudo fuser -vm /data
```

Do not blindly use force options in production without understanding the impact.

---

# 48. Explain LVM

LVM means **Logical Volume Manager**.

Basic architecture:

```text
Physical Disk
     |
     v
Physical Volume (PV)
     |
     v
Volume Group (VG)
     |
     v
Logical Volume (LV)
     |
     v
Filesystem
```

Useful commands:

```bash
pvs
vgs
lvs
```

Create PV:

```bash
pvcreate /dev/xvdf
```

Create VG:

```bash
vgcreate vgdata /dev/xvdf
```

Create LV:

```bash
lvcreate -L 10G -n lvapp vgdata
```

Format:

```bash
mkfs.ext4 /dev/vgdata/lvapp
```

---

# 49. How do you extend an LVM filesystem?

Check:

```bash
df -h
lvs
vgs
```

Extend LV:

```bash
lvextend -L +5G /dev/vgdata/lvapp
```

For ext4:

```bash
resize2fs /dev/vgdata/lvapp
```

For XFS:

```bash
xfs_growfs /mountpoint
```

Many modern systems can combine extension and filesystem resize:

```bash
lvextend -r -L +5G /dev/vgdata/lvapp
```

Always verify filesystem type first.

---

# 50. How do you troubleshoot a read-only filesystem?

Check:

```bash
findmnt
mount
dmesg | tail -100
```

Look for filesystem or storage errors:

```bash
journalctl -k
```

Possible causes:

- filesystem corruption
- underlying disk/storage errors
- kernel protection after I/O failure
- mount options
- administrative remount

Do not simply remount read-write without identifying why it became read-only.

---

# 51. What is swap?

Swap is disk-backed space used when memory pressure occurs.

Check:

```bash
free -h
swapon --show
```

Memory statistics:

```bash
vmstat 1
```

Important:

> Swap is not a replacement for adequate RAM.

Heavy swapping can cause severe performance degradation.

---

# 52. How do you troubleshoot disk I/O?

Useful commands:

```bash
iostat -xz 1
```

and:

```bash
vmstat 1
```

Also:

```bash
top
```

Look for:

- high I/O wait
- high disk utilization
- latency
- throughput saturation
- queue depth
- application-specific I/O

Identify processes:

```bash
iotop
```

if installed and permitted.

Then correlate with:

- database workload
- logs
- backups
- large file operations
- storage performance limits

---

# 53. Explain logs and journalctl

System/application logs are essential for troubleshooting.

Systemd journal:

```bash
journalctl
```

Service logs:

```bash
journalctl -u nginx
```

Recent logs:

```bash
journalctl -u nginx -n 100
```

Follow:

```bash
journalctl -u nginx -f
```

Since boot:

```bash
journalctl -b
```

Errors:

```bash
journalctl -p err
```

Kernel:

```bash
journalctl -k
```

---

# 54. Explain log rotation

Log rotation prevents logs from consuming the entire filesystem.

Common tool:

```bash
logrotate
```

Configuration:

```bash
/etc/logrotate.conf
/etc/logrotate.d/
```

Typical rotation operations include:

- rotate old logs
- compress old logs
- keep a configured number of files
- create new logs
- optionally signal/reload applications

Check configuration:

```bash
cat /etc/logrotate.conf
ls /etc/logrotate.d/
```

---

# 55. How do you find large files?

Example:

```bash
find /var -type f -size +500M -ls
```

Largest files:

```bash
find /var -type f -printf '%s %p\n' 2>/dev/null | sort -nr | head
```

Largest directories:

```bash
du -xhd1 /var | sort -h
```

For production systems, use `-x` where appropriate to avoid crossing filesystem boundaries.

---

# 56. How do you identify deleted files still consuming disk?

This is a classic L2 troubleshooting question.

Check:

```bash
lsof +L1
```

or:

```bash
lsof | grep deleted
```

A process can keep a deleted file open.

The directory entry is gone, but the file's storage remains allocated until the process closes the file descriptor.

### Typical scenario

```text
df -h      -> filesystem 95% full
du -sh *   -> usage looks much lower
```

This mismatch can indicate deleted-but-open files.

---

# 57. What is SELinux?

SELinux is a Linux security mechanism implementing **mandatory access control (MAC)**.

It can restrict what processes are allowed to access even when normal Unix permissions appear correct.

Check:

```bash
getenforce
```

Modes:

```text
Enforcing
Permissive
Disabled
```

Check contexts:

```bash
ls -Z
```

Audit messages:

```bash
ausearch -m AVC -ts recent
```

### Senior troubleshooting point

If:

```text
chmod
chown
```

look correct but access is still denied, investigate SELinux.

Do not immediately disable SELinux as a troubleshooting shortcut.

---

# 58. What is AppArmor?

AppArmor is another Linux security framework implementing mandatory access controls using application profiles.

Check:

```bash
aa-status
```

Profiles define what applications are allowed to access.

Ubuntu commonly uses AppArmor extensively.

---

# 59. SELinux vs AppArmor

| SELinux | AppArmor |
|---|---|
| Label/context based | Path/profile oriented |
| Common in RHEL family | Common in Ubuntu |
| Fine-grained MAC | Fine-grained MAC |
| Uses security contexts | Uses application profiles |

Both provide additional controls beyond standard discretionary Unix permissions.

---

# 60. How do you troubleshoot "Permission denied"?

Use this order:

### 1. Check permissions

```bash
ls -l file
```

### 2. Check ownership

```bash
stat file
```

### 3. Check parent directories

```bash
namei -l /path/to/file
```

This is extremely useful because the user needs execute/search permission on directories along the path.

### 4. Check ACLs

```bash
getfacl file
```

### 5. Check SELinux

```bash
ls -Z file
getenforce
```

### 6. Check AppArmor

```bash
aa-status
```

### 7. Check service identity

```bash
ps -ef | grep application
```

The application may be running as a different user than expected.

---

# 61. How do you troubleshoot SSH login failure?

First identify the failure type:

- timeout
- connection refused
- authentication failure
- permission denied
- key rejection

### Client

```bash
ssh -vvv user@server
```

### Server

```bash
systemctl status ssh
```

Check port:

```bash
ss -lntp | grep :22
```

Check logs:

```bash
journalctl -u ssh
```

or distribution-specific authentication logs.

Check:

```bash
~/.ssh/authorized_keys
```

Permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

Also investigate firewall/security groups, routing and account restrictions.

---

# 62. Explain SSH key-based authentication

The client has:

```text
Private key
Public key
```

The public key is placed on the server in:

```bash
~/.ssh/authorized_keys
```

The private key should remain secret.

Generate:

```bash
ssh-keygen -t ed25519
```

Copy key:

```bash
ssh-copy-id user@server
```

### Security principle

Never share private keys.

---

# 63. SSH hardening

Common hardening practices:

- disable direct root login where appropriate
- prefer key authentication
- restrict users/groups
- use strong cryptographic algorithms according to organizational policy
- monitor authentication logs
- minimize exposed SSH access
- use firewall/security-group restrictions
- keep OpenSSH patched

Configuration:

```bash
/etc/ssh/sshd_config
```

Validate before restart/reload:

```bash
sshd -t
```

Then:

```bash
systemctl reload ssh
```

---

# 64. What is /etc/hosts?

`/etc/hosts` provides local static hostname-to-IP mappings.

Example:

```text
10.10.10.20 app01
10.10.10.30 db01
```

Test:

```bash
getent hosts app01
```

It can be useful for local overrides and temporary troubleshooting, but large environments should normally use proper DNS/service discovery.

---

# 65. Explain /etc/resolv.conf

`/etc/resolv.conf` contains resolver configuration, such as DNS nameservers and search domains.

Example:

```text
nameserver 10.0.0.2
search example.internal
```

On modern distributions, it may be managed by:

- systemd-resolved
- NetworkManager
- DHCP
- cloud-init
- other network configuration systems

Therefore, manually editing it may not persist.

---

# 66. TCP vs UDP

### TCP

- connection-oriented
- reliable delivery
- ordered data
- retransmission
- congestion control

Used commonly for:

- HTTP/HTTPS
- SSH
- databases

### UDP

- connectionless
- lower protocol overhead
- no built-in reliable delivery

Used commonly for:

- DNS in many cases
- DHCP
- streaming/real-time applications
- telemetry protocols depending on implementation

---

# 67. Common Linux networking commands

Interfaces:

```bash
ip addr
```

Routes:

```bash
ip route
```

Neighbors:

```bash
ip neigh
```

Connectivity:

```bash
ping host
```

DNS:

```bash
dig host
```

TCP port:

```bash
nc -vz host 443
```

Sockets:

```bash
ss -lntp
```

HTTP:

```bash
curl -v http://host:8080
```

Packet capture:

```bash
sudo tcpdump -i eth0 port 443
```

---

# 68. How do you identify which process owns a port?

Use:

```bash
sudo ss -lntp | grep :8080
```

Or:

```bash
sudo lsof -i :8080
```

Or:

```bash
sudo fuser -v 8080/tcp
```

This gives you the process/PID.

Then:

```bash
ps -p <PID> -f
```

---

# 69. How do you test connectivity at different layers?

A good L2 engineer does not simply run `ping`.

### Layer 3

```bash
ping host
```

### DNS

```bash
getent hosts host
```

### TCP

```bash
nc -vz host 443
```

### TLS/HTTP

```bash
curl -vk https://host
```

### Routing

```bash
ip route
```

### Packet-level

```bash
tcpdump
```

### Important

Ping failure does **not** automatically mean the application is unreachable. ICMP may be blocked while TCP/HTTPS works.

---

# 70. Explain localhost, 0.0.0.0 and 127.0.0.1

`127.0.0.1` is the IPv4 loopback address.

`localhost` normally resolves to the local host, commonly including loopback addresses.

`0.0.0.0` has special meanings depending on context.

For a server listening socket:

```text
0.0.0.0:8080
```

means listening on all IPv4 interfaces.

Example:

```bash
ss -lnt
```

If an application listens only on:

```text
127.0.0.1:8080
```

external clients generally cannot connect to it.

---

# 71. What is a default gateway?

The default gateway is the router used when the destination does not match a more specific route in the routing table.

Check:

```bash
ip route
```

Example:

```text
default via 10.0.1.1 dev eth0
```

This means traffic without a more specific route is sent through `10.0.1.1`.

---

# 72. What is a routing table?

The routing table determines where packets should be sent.

View:

```bash
ip route
```

Example concepts:

```text
10.0.0.0/16 dev eth0
default via 10.0.1.1 dev eth0
```

Troubleshoot route selection:

```bash
ip route get 8.8.8.8
```

This is a very useful production command.

---

# 73. What is a firewall?

A firewall controls network traffic based on rules.

It can filter based on:

- source IP
- destination IP
- protocol
- port
- interface
- connection state

Linux environments may use:

- nftables
- iptables
- firewalld
- UFW

Cloud environments may additionally have:

- security groups
- network ACLs
- load balancer controls

---

# 74. iptables vs nftables

`iptables` is the traditional Linux packet-filtering interface.

`nftables` is the newer packet-filtering framework.

Modern distributions increasingly use nftables underneath or directly.

Check:

```bash
sudo nft list ruleset
```

On systems using firewalld:

```bash
firewall-cmd --list-all
```

On Ubuntu with UFW:

```bash
sudo ufw status
```

Do not assume the same firewall management mechanism on every distribution.

---

# 75. What is systemd?

systemd is a system and service manager used by many modern Linux distributions.

It typically runs as:

```text
PID 1
```

Responsibilities include:

- service management
- boot orchestration
- dependency management
- logging integration
- socket activation
- timers
- resource controls
- mount management

Useful:

```bash
systemctl
journalctl
```

---

# 76. Explain targets/runlevels

Older SysV init systems used runlevels.

Systemd uses targets.

Examples:

```bash
systemctl get-default
```

Common targets:

```text
multi-user.target
graphical.target
rescue.target
```

Set default target:

```bash
sudo systemctl set-default multi-user.target
```

Targets group related units and define desired system states.

---

# 77. Explain passwd, shadow and group

### `/etc/passwd`

Contains user account information such as:

- username
- UID
- GID
- home directory
- login shell

```bash
cat /etc/passwd
```

### `/etc/shadow`

Contains password hashes and password-aging information.

```bash
sudo cat /etc/shadow
```

### `/etc/group`

Contains group information.

```bash
cat /etc/group
```

Passwords should not be stored in plaintext.

---

# 78. User and group management

Create user:

```bash
sudo useradd -m mukesh
```

Set password:

```bash
sudo passwd mukesh
```

Create group:

```bash
sudo groupadd developers
```

Add user:

```bash
sudo usermod -aG developers mukesh
```

Check groups:

```bash
id mukesh
groups mukesh
```

### Important

Use:

```bash
usermod -aG group user
```

rather than accidentally replacing the user's supplementary groups.

---

# 79. Explain sticky bit, SUID and SGID

## Sticky bit

Commonly used on shared directories such as `/tmp`.

```bash
ls -ld /tmp
```

It prevents users from deleting files owned by other users in a sticky directory, subject to root/privileged rules.

## SUID

Executable runs with the file owner's effective privileges.

Example:

```bash
find / -perm -4000 -type f 2>/dev/null
```

## SGID

For files, executable group privileges can be inherited during execution.

For directories, new files/subdirectories can inherit the directory's group.

Find SGID:

```bash
find / -perm -2000 -type f 2>/dev/null
```

These permissions should be audited carefully because they can have security implications.

---

# 80. What happens when you execute a Linux command?

Consider:

```bash
ls -l /tmp
```

The shell:

1. Reads the command.
2. Performs parsing.
3. Performs expansions.
4. Searches PATH for the executable if required.
5. Creates a process.
6. The process loads the executable.
7. The kernel provides required resources.
8. The command executes.
9. The process exits.
10. The shell receives the exit status.

For external commands, process creation commonly involves mechanisms such as `fork()`/`clone()` and `execve()` depending on implementation.

---

# 81. Explain Bash exit codes

A successful command normally returns:

```text
0
```

Non-zero generally indicates failure.

Check previous command:

```bash
echo $?
```

Example:

```bash
ls /tmp
echo $?
```

In scripts:

```bash
if command; then
    echo "Success"
else
    echo "Failure"
fi
```

Useful script practice:

```bash
set -euo pipefail
```

But understand the semantics before applying it blindly to complex scripts.

---

# 82. Explain pipes and redirection

Pipe:

```bash
command1 | command2
```

Example:

```bash
ps aux | grep nginx
```

Standard output:

```bash
command > file
```

Append:

```bash
command >> file
```

Standard error:

```bash
command 2> error.log
```

Both stdout and stderr:

```bash
command > output.log 2>&1
```

Modern shorthand:

```bash
command &> output.log
```

Input:

```bash
command < input.txt
```

---

# 83. find -exec and xargs

Example:

```bash
find /tmp -type f -name "*.log" -exec rm {} \;
```

`find -exec` executes a command for matching files.

Using batch execution:

```bash
find /tmp -type f -name "*.log" -exec rm {} +
```

`xargs` builds command arguments from input:

```bash
find /tmp -type f -name "*.log" -print0 | xargs -0 rm
```

`-print0` and `-0` safely handle filenames containing spaces/newlines.

### Senior point

Avoid unsafe constructs such as:

```bash
for f in $(find ...)
```

because whitespace and special characters can break filename handling.

---

# 84. awk vs sed

### sed

Best for stream editing/substitution.

```bash
sed 's/old/new/g' file
```

### awk

Best for structured text processing and column-based operations.

```bash
awk '{print $1, $3}' file
```

Example:

```bash
df -h | awk 'NR>1 {print $1, $5}'
```

### Interview answer

> I use `sed` primarily for transformations and substitutions, while `awk` is useful for field-based parsing, filtering and calculations.

---

# 85. How do you troubleshoot a production Linux server systematically?

Do not start changing things immediately.

Use this sequence:

## 1. Define the symptom

Examples:

- server slow
- application unavailable
- SSH unavailable
- disk full
- high CPU
- high memory
- service failed

## 2. Check recent changes

Look for:

- deployments
- configuration changes
- package updates
- certificates
- infrastructure changes

## 3. Check system health

```bash
uptime
top
free -h
df -h
df -i
```

## 4. Check service

```bash
systemctl status service
journalctl -u service
```

## 5. Check network

```bash
ip addr
ip route
ss -lntp
```

## 6. Check logs

```bash
journalctl
```

and application logs.

## 7. Check security

- permissions
- SELinux
- AppArmor
- firewall

## 8. Check dependencies

Database, DNS, filesystem, network, certificates, external services.

## 9. Apply the smallest safe fix

Avoid unnecessary changes.

## 10. Verify

Confirm:

- service health
- application response
- logs
- resource utilization

## 11. Document root cause

A senior engineer should distinguish:

```text
Symptom
Cause
Fix
Prevention
```

---

# 86. Senior L2 production troubleshooting scenario

### Scenario

An application suddenly returns HTTP 502.

### Step 1: Check application/service

```bash
systemctl status app
journalctl -u app -n 100
```

### Step 2: Check listening port

```bash
ss -lntp
```

Suppose application should listen on `8080`, but nothing is listening.

### Step 3: Check application logs

```bash
journalctl -u app
```

Suppose:

```text
Address already in use
```

### Step 4: Find port owner

```bash
sudo ss -lntp | grep :8080
```

Suppose an old process owns it.

### Step 5: Inspect process

```bash
ps -p <PID> -f
```

Determine why it exists.

### Step 6: Correct the root cause

Do not simply kill it without understanding why it is running.

### Step 7: Restart application

```bash
sudo systemctl restart app
```

### Step 8: Verify

```bash
systemctl status app
ss -lntp | grep :8080
curl -v http://127.0.0.1:8080/health
```

### Step 9: Test end-to-end

Check reverse proxy/load balancer and client request.

### Senior answer

> I would first isolate whether the 502 is generated by the proxy or backend. Then I would verify backend service health, listening socket, logs, process ownership and dependencies. I would fix the underlying cause rather than simply restarting the application, then validate the service locally and end-to-end.

---

# 87. Quick Linux L2 Command Cheat Sheet

## System

```bash
uname -a
hostnamectl
uptime
date
```

## CPU

```bash
top
ps aux --sort=-%cpu | head
nproc
mpstat -P ALL 1
```

## Memory

```bash
free -h
vmstat 1
ps aux --sort=-%mem | head
```

## Disk

```bash
df -h
df -i
du -sh *
find / -type f -size +1G 2>/dev/null
```

## Processes

```bash
ps -ef
pgrep -a nginx
pstree
kill -15 PID
```

## Services

```bash
systemctl status nginx
systemctl restart nginx
systemctl --failed
journalctl -u nginx
```

## Network

```bash
ip addr
ip route
ss -lntp
ping host
nc -vz host 443
curl -v URL
```

## DNS

```bash
getent hosts example.com
dig example.com
```

## Files

```bash
find /path -name "*.log"
grep -rin "error" /var/log
ls -li file
stat file
```

## Permissions

```bash
ls -l
namei -l /path/file
getfacl file
chmod
chown
```

## Storage

```bash
lsblk
blkid
findmnt
pvs
vgs
lvs
```

## Logs

```bash
journalctl
journalctl -b
journalctl -p err
journalctl -u service
```

---

# 88. Interview Answer Framework

For L2/Senior interviews, avoid giving only a definition.

Use this structure:

### 1. Definition

Explain what it is.

### 2. Purpose

Explain why it exists.

### 3. Commands

Give 2–4 practical commands.

### 4. Production example

Explain where you used it.

### 5. Troubleshooting

Explain how you would investigate failure.

### Example

Question:

> How do you troubleshoot high disk usage?

Strong answer:

> First I verify filesystem utilization using `df -h` and inode utilization using `df -i`. If the filesystem is full, I use `du` and `find` to identify the largest directories and files. I also check for deleted-but-open files using `lsof +L1`, because those files can consume disk space while being invisible in normal directory listings. Then I inspect logs, backups or application-generated files and apply the smallest safe cleanup. Finally I verify filesystem utilization and investigate log rotation or retention so the issue does not recur.

This is much stronger than:

> "I use `df -h` to check disk."

---

# Senior L2 Interview Topics You Should Be Able To Explain

Before an interview, make sure you can troubleshoot these without memorizing commands mechanically:

- CPU saturation
- Memory pressure
- OOM killer
- Load average
- Disk full
- Inode exhaustion
- Disk I/O latency
- Deleted-open files
- File descriptor exhaustion
- Service startup failures
- systemd dependencies
- SSH authentication
- SSH connectivity
- DNS failures
- Routing problems
- Port/listener issues
- Firewall problems
- SELinux/AppArmor denials
- Filesystem mounting
- `/etc/fstab`
- LVM
- swap
- process states
- zombie processes
- signals
- permissions
- ACLs
- SUID/SGID/sticky bit
- cron
- systemd timers
- logs/journalctl
- log rotation
- shell scripting
- `grep`, `awk`, `sed`
- `find` and `xargs`
- TCP vs UDP
- basic packet analysis
- Linux boot process

---

# Final Interview Tip

For an L2 engineer, interviewers usually care less about whether you remember every command and more about whether you can **isolate a failure logically**.

A strong troubleshooting thought process is:

```text
SYMPTOM
   |
   v
SCOPE
   |
   v
RECENT CHANGES
   |
   v
SYSTEM HEALTH
   |
   v
PROCESS / SERVICE
   |
   v
NETWORK / STORAGE / DEPENDENCY
   |
   v
LOGS
   |
   v
ROOT CAUSE
   |
   v
SAFE FIX
   |
   v
VERIFICATION
   |
   v
PREVENTION
```

When answering an interview question, connect the command to the reason you are running it.

For example:

> "I use `ss -lntp` not just to check ports, but to verify whether the expected process is actually listening on the expected interface and port."

That demonstrates **L2 troubleshooting maturity** rather than simple command memorization.
