# Important Linux Kernel Parameters for DevOps

## Table of Contents

1. [Introduction](#1-introduction)
2. [What is a Linux Kernel Parameter?](#2-what-is-a-linux-kernel-parameter)
3. [What is sysctl?](#3-what-is-sysctl)
4. [Where sysctl Parameters Come From](#4-where-sysctl-parameters-come-from)
5. [Read, Change, and Verify Parameters](#5-read-change-and-verify-parameters)
6. [Temporary vs Persistent Changes](#6-temporary-vs-persistent-changes)
7. [Recommended sysctl Configuration](#7-recommended-sysctl-configuration)
8. [Memory and Virtual Memory Parameters](#8-memory-and-virtual-memory-parameters)
9. [File Descriptor Parameters](#9-file-descriptor-parameters)
10. [Network Queue and Socket Parameters](#10-network-queue-and-socket-parameters)
11. [TCP Parameters](#11-tcp-parameters)
12. [IP Forwarding Parameters](#12-ip-forwarding-parameters)
13. [Reverse Path Filtering](#13-reverse-path-filtering)
14. [Kernel Security Parameters](#14-kernel-security-parameters)
15. [Useful Parameters for High-Traffic Servers](#15-useful-parameters-for-high-traffic-servers)
16. [DevOps Workloads and Typical Examples](#16-devops-workloads-and-typical-examples)
17. [Troubleshooting Workflow](#17-troubleshooting-workflow)
18. [Important Cautions](#18-important-cautions)
19. [sysctl vs ulimit](#19-sysctl-vs-ulimit)
20. [Frequently Asked Interview Questions](#20-frequently-asked-interview-questions)
21. [One-Line Interview Answers](#21-one-line-interview-answers)

---

# 1. Introduction

Linux exposes many **kernel tunables** that control behavior related to:

* Memory management
* Networking
* File descriptors
* Connection queues
* TCP behavior
* Packet forwarding
* Security
* Virtual memory

For DevOps engineers, these parameters are useful when operating:

* Web servers
* Reverse proxies
* Load balancers
* Databases
* Elasticsearch
* Redis
* Kubernetes nodes
* High-concurrency APIs
* Container hosts
* Monitoring systems

The `sysctl` utility provides a standard interface for reading and changing many parameters exposed through:

```text
/proc/sys/
```

Linux kernel documentation defines `sysctl` parameters under areas such as:

```text
/proc/sys/vm/
 /proc/sys/fs/
 /proc/sys/net/
 /proc/sys/kernel/
```

---

# 2. What is a Linux Kernel Parameter?

A **kernel parameter** is a tunable value that changes how the Linux kernel behaves.

Example:

```text
net.core.somaxconn
```

This controls the maximum socket listen backlog exposed through the `listen()` API. Current Linux kernel documentation lists a default of 4096 on current kernels and notes that this was 128 before Linux 5.4. citeturn976649search0

Another example:

```text
vm.swappiness
```

controls how aggressively the kernel considers swapping anonymous memory relative to filesystem-backed pages.

Parameters are exposed using dotted names:

```text
net.ipv4.ip_forward
vm.swappiness
fs.file-max
kernel.pid_max
```

---

# 3. What is sysctl?

`sysctl` is a command-line interface for reading and changing kernel parameters at runtime.

It works with values exposed under:

```text
/proc/sys/
```

For example:

```bash
sysctl net.ipv4.ip_forward
```

Example output:

```text
net.ipv4.ip_forward = 0
```

Change it temporarily:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

The `sysctl` utility supports both reading and writing kernel parameters, and `sysctl -p` can load settings from a configuration file. citeturn976649search6turn976649search7

---

# 4. Where sysctl Parameters Come From

The naming hierarchy generally mirrors `/proc/sys`.

For example:

```text
/proc/sys/net/ipv4/ip_forward
```

becomes:

```text
net.ipv4.ip_forward
```

Another example:

```text
/proc/sys/vm/swappiness
```

becomes:

```text
vm.swappiness
```

Common namespaces:

```text
kernel.*
fs.*
vm.*
net.*
user.*
```

Not every available kernel setting is a sysctl setting. Many kernel features have other configuration mechanisms.

---

# 5. Read, Change, and Verify Parameters

## Read One Parameter

```bash
sysctl net.ipv4.ip_forward
```

or:

```bash
cat /proc/sys/net/ipv4/ip_forward
```

---

## Read a Parameter Without Its Name

```bash
sysctl -n net.ipv4.ip_forward
```

Example:

```text
0
```

---

## List Available Parameters

```bash
sysctl -a
```

You can filter:

```bash
sysctl -a | grep ipv4
```

or:

```bash
sysctl -a | grep tcp
```

The `sysctl -a` command displays currently available parameters; deprecated/verboten parameters are excluded unless explicitly requested by the relevant option. citeturn976649search6

---

## Change a Parameter Temporarily

```bash
sudo sysctl -w vm.swappiness=10
```

Format:

```bash
sudo sysctl -w parameter=value
```

Example:

```bash
sudo sysctl -w net.core.somaxconn=4096
```

---

## Verify the Change

```bash
sysctl vm.swappiness
```

or:

```bash
cat /proc/sys/vm/swappiness
```

---

# 6. Temporary vs Persistent Changes

This is one of the most important concepts.

## Temporary Change

```bash
sudo sysctl -w vm.swappiness=10
```

The running kernel changes immediately.

However, the setting is not automatically guaranteed to survive a reboot unless it is written to persistent configuration.

---

## Persistent Change

Create a configuration file under:

```text
/etc/sysctl.d/
```

Example:

```bash
sudo nano /etc/sysctl.d/99-devops-tuning.conf
```

Add:

```ini
vm.swappiness = 10
net.core.somaxconn = 4096
```

Then load it:

```bash
sudo sysctl --system
```

`systemd-sysctl` reads `sysctl.d` configuration during boot, while `sysctl --system` loads system configuration files. citeturn976649search3turn976649search6

---

## `/etc/sysctl.conf` vs `/etc/sysctl.d/`

Both can be used with the `procps` tooling, but for modern Linux systems it is generally cleaner to put administrator-managed settings in a dedicated file under:

```text
/etc/sysctl.d/
```

For example:

```text
/etc/sysctl.d/99-devops-tuning.conf
```

The sysctl configuration lookup order includes `/etc/sysctl.d/`, `/run/sysctl.d/`, `/usr/local/lib/sysctl.d/`, `/usr/lib/sysctl.d/`, `/lib/sysctl.d/`, followed by `/etc/sysctl.conf` for `sysctl --system`; systemd's boot-time handling is documented separately in `sysctl.d(5)`. citeturn976649search1turn976649search3

---

# 7. Recommended sysctl Configuration

A practical DevOps approach is:

```text
1. Measure current behavior
2. Read current sysctl value
3. Understand what the parameter changes
4. Test temporarily
5. Measure again
6. Make persistent only after validation
7. Document why it was changed
8. Keep rollback values
```

Example:

```bash
sysctl vm.swappiness

sudo sysctl -w vm.swappiness=10

sysctl vm.swappiness
```

Then persist:

```ini
# /etc/sysctl.d/99-devops-tuning.conf
vm.swappiness = 10
```

Load:

```bash
sudo sysctl --system
```

---

# 8. Memory and Virtual Memory Parameters

## 8.1 `vm.swappiness`

```text
vm.swappiness
```

Controls the kernel's tendency to use swap.

Read:

```bash
sysctl vm.swappiness
```

Set temporarily:

```bash
sudo sysctl -w vm.swappiness=10
```

Persistent:

```ini
vm.swappiness = 10
```

### Why DevOps Engineers Care

For memory-sensitive workloads such as:

* Databases
* Elasticsearch
* High-throughput application servers
* Kubernetes nodes

excessive swapping can increase latency.

### Important

There is no universally correct value such as `10`, `1`, or `0` for every server. Tune based on workload, available RAM, swap configuration, and observed behavior.

---

## 8.2 `vm.max_map_count`

```text
vm.max_map_count
```

Controls the maximum number of memory-map areas a process may have.

Memory-map areas can be created by:

* `malloc`
* `mmap`
* `mprotect`
* `madvise`
* Loading shared libraries

The current kernel documentation lists a default of `65530`. citeturn976649search8

Read:

```bash
sysctl vm.max_map_count
```

Set temporarily:

```bash
sudo sysctl -w vm.max_map_count=262144
```

Persist:

```ini
vm.max_map_count = 262144
```

### DevOps Use Case

This parameter is commonly encountered when running applications that create a large number of memory mappings, including Elasticsearch-based workloads.

Do not change it blindly; use the value required by the application documentation or the observed workload.

---

## 8.3 `vm.overcommit_memory`

```text
vm.overcommit_memory
```

Controls Linux memory overcommit policy.

Common values:

```text
0 → Heuristic overcommit
1 → Always overcommit
2 → Strict overcommit
```

Read:

```bash
sysctl vm.overcommit_memory
```

Example:

```bash
sudo sysctl -w vm.overcommit_memory=1
```

### DevOps Use Case

This parameter is often discussed for Redis deployments and other memory-sensitive applications.

Always understand the application's memory behavior before changing it.

---

## 8.4 `vm.dirty_ratio`

```text
vm.dirty_ratio
```

Defines the percentage of total writable memory that can contain dirty pages before the process generating the writes may be forced to participate in writeback.

Check:

```bash
sysctl vm.dirty_ratio
```

Example:

```bash
sudo sysctl -w vm.dirty_ratio=20
```

---

## 8.5 `vm.dirty_background_ratio`

```text
vm.dirty_background_ratio
```

Controls when background writeback starts based on the amount of dirty memory.

Check:

```bash
sysctl vm.dirty_background_ratio
```

Example:

```bash
sudo sysctl -w vm.dirty_background_ratio=10
```

### DevOps Use Case

Useful when tuning write-heavy workloads, but these values should be validated against storage latency and application behavior.

---

# 9. File Descriptor Parameters

Large-scale services can run out of file descriptors.

A file descriptor can represent:

* Open file
* TCP socket
* UNIX socket
* Pipe
* Device

---

## 9.1 `fs.file-max`

```text
fs.file-max
```

Controls the system-wide limit on allocated file handles.

Read:

```bash
sysctl fs.file-max
```

Set:

```bash
sudo sysctl -w fs.file-max=2097152
```

Persistent:

```ini
fs.file-max = 2097152
```

### Why It Matters

High-concurrency systems may have thousands or millions of simultaneously open files/sockets.

Examples:

```text
Nginx
Elasticsearch
Databases
API Servers
Kubernetes Nodes
```

---

## Important: `fs.file-max` Is Not the Same as `ulimit -n`

`fs.file-max` is a **system-wide kernel limit**.

```text
fs.file-max
   ↓
System-wide file handles
```

A process also has per-process limits:

```text
ulimit -n
```

So increasing only `fs.file-max` may not solve a "too many open files" problem.

See [sysctl vs ulimit](#19-sysctl-vs-ulimit).

---

# 10. Network Queue and Socket Parameters

These parameters become important for busy web servers, API servers, proxies, and load balancers.

---

## 10.1 `net.core.somaxconn`

```text
net.core.somaxconn
```

Controls the maximum socket listen backlog exposed through `listen()`.

Check:

```bash
sysctl net.core.somaxconn
```

Example:

```bash
sudo sysctl -w net.core.somaxconn=4096
```

Current Linux kernel documentation lists `4096` as the default and notes that older kernels used `128`. citeturn976649search0

### Use Case

High-concurrency services such as:

```text
Nginx
HAProxy
Application Servers
API Gateways
```

may need an appropriate listen backlog.

Important:

> Raising `somaxconn` does not automatically mean an application will use that entire value. The application's own `listen()` backlog and other kernel limits also matter.

---

## 10.2 `net.ipv4.tcp_max_syn_backlog`

```text
net.ipv4.tcp_max_syn_backlog
```

Controls the maximum number of remembered connection requests that have not yet received the final ACK.

Read:

```bash
sysctl net.ipv4.tcp_max_syn_backlog
```

Example:

```bash
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=8192
```

### Use Case

Useful to understand when a server receives large numbers of new TCP connection attempts.

It is related to:

```text
SYN backlog
      +
listen backlog
      +
Application backlog
```

Do not tune it without looking at actual connection pressure.

---

## 10.3 `net.core.netdev_max_backlog`

```text
net.core.netdev_max_backlog
```

Controls the maximum number of packets queued on the input side when the interface receives packets faster than the kernel can process them.

Read:

```bash
sysctl net.core.netdev_max_backlog
```

Example:

```bash
sudo sysctl -w net.core.netdev_max_backlog=5000
```

### Use Case

Relevant for:

* High packet rates
* Network appliances
* Proxies
* Load balancers
* High-throughput hosts

---

# 11. TCP Parameters

---

## 11.1 `net.ipv4.ip_local_port_range`

```text
net.ipv4.ip_local_port_range
```

Defines the range of local ephemeral ports used for automatic port allocation.

Read:

```bash
sysctl net.ipv4.ip_local_port_range
```

Example output:

```text
32768 60999
```

Set:

```bash
sudo sysctl -w net.ipv4.ip_local_port_range="10240 65535"
```

Persistent:

```ini
net.ipv4.ip_local_port_range = 10240 65535
```

### DevOps Use Case

Important for systems making very large numbers of outbound connections.

Examples:

```text
API Gateway
Reverse Proxy
Microservices
NAT Hosts
Load Generators
```

A small ephemeral port range can contribute to outbound connection exhaustion.

---

## 11.2 `net.ipv4.ip_local_reserved_ports`

```text
net.ipv4.ip_local_reserved_ports
```

Reserves ports so automatic port allocation will not use them.

Example:

```bash
sudo sysctl -w net.ipv4.ip_local_reserved_ports="30000-30010,8080"
```

The Linux kernel documentation notes that this setting is independent of `ip_local_port_range`; both are considered when selecting automatically allocated ports. citeturn976649search0

This is useful when specific ports must remain available for known services.

---

## 11.3 `net.ipv4.tcp_fin_timeout`

```text
net.ipv4.tcp_fin_timeout
```

Controls how long orphaned TCP sockets remain in the `FIN-WAIT-2` state before being closed.

Read:

```bash
sysctl net.ipv4.tcp_fin_timeout
```

Example:

```bash
sudo sysctl -w net.ipv4.tcp_fin_timeout=30
```

### Caution

Do not treat this as a general "make TCP faster" knob. Changing TCP timers can affect connection behavior and should be driven by observed workload symptoms.

---

## 11.4 `net.ipv4.tcp_tw_reuse`

```text
net.ipv4.tcp_tw_reuse
```

Controls reuse of `TIME_WAIT` sockets for certain outbound connections.

The Linux kernel exposes this as a TCP sysctl. citeturn976649search5

Check:

```bash
sysctl net.ipv4.tcp_tw_reuse
```

### Caution

Do not enable TCP TIME_WAIT reuse simply because you see many `TIME_WAIT` sockets.

First investigate:

```bash
ss -s
ss -tan state time-wait
```

and determine whether ephemeral-port exhaustion or connection churn is actually a problem.

---

## 11.5 TCP Keepalive Parameters

Useful parameters include:

```text
net.ipv4.tcp_keepalive_time
net.ipv4.tcp_keepalive_intvl
net.ipv4.tcp_keepalive_probes
```

Read:

```bash
sysctl net.ipv4.tcp_keepalive_time
sysctl net.ipv4.tcp_keepalive_intvl
sysctl net.ipv4.tcp_keepalive_probes
```

Conceptually:

```text
No traffic
   │
   ▼
keepalive_time
   │
   ▼
Probe
   │
   ▼
keepalive_intvl
   │
   ▼
Additional probes
   │
   ▼
Connection considered dead
```

### DevOps Use Case

Useful when long-lived TCP connections must detect broken peers.

Examples:

* Proxies
* Databases
* Long-running API connections
* Service-to-service communication

---

## 11.6 `net.ipv4.tcp_syncookies`

```text
net.ipv4.tcp_syncookies
```

Controls TCP SYN cookie behavior.

Check:

```bash
sysctl net.ipv4.tcp_syncookies
```

Linux supports SYN cookies as a mechanism for handling certain SYN-flood conditions.

Do not treat this as a substitute for a complete DDoS protection strategy.

---

# 12. IP Forwarding Parameters

## 12.1 `net.ipv4.ip_forward`

```text
net.ipv4.ip_forward
```

Controls IPv4 packet forwarding.

Check:

```bash
sysctl net.ipv4.ip_forward
```

Enable temporarily:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

Persist:

```ini
net.ipv4.ip_forward = 1
```

### DevOps Use Cases

Important for:

* Routers
* NAT gateways
* Containers
* Kubernetes nodes
* Network appliances

Example:

```text
Client
  │
  ▼
Linux Router
  │
  ├── eth0
  └── eth1
       │
       ▼
   Destination
```

Without forwarding enabled, the host normally will not route IPv4 packets between interfaces as a router.

---

## 12.2 IPv6 Forwarding

The IPv6 equivalent is:

```text
net.ipv6.conf.all.forwarding
```

Check:

```bash
sysctl net.ipv6.conf.all.forwarding
```

Enable:

```bash
sudo sysctl -w net.ipv6.conf.all.forwarding=1
```

---

# 13. Reverse Path Filtering

## `net.ipv4.conf.all.rp_filter`

Reverse Path Filtering helps protect against packets arriving with source addresses that do not have a valid reverse route according to the configured mode.

Check:

```bash
sysctl net.ipv4.conf.all.rp_filter
```

Common mode values are:

```text
0 → Disabled
1 → Strict mode
2 → Loose mode
```

### DevOps Use Case

This matters in environments with:

* Multiple network interfaces
* Asymmetric routing
* Policy routing
* VPNs
* Load-balancer nodes
* Complex cloud networking

### Important

Disabling or loosening reverse-path filtering can change network security behavior. Do not change it merely to "fix networking" without understanding the routing design.

---

# 14. Kernel Security Parameters

Several `kernel.*` parameters are security-oriented.

These should be changed carefully and based on the host's security policy.

## 14.1 `kernel.randomize_va_space`

Controls userspace virtual-address layout randomization behavior.

Read:

```bash
sysctl kernel.randomize_va_space
```

Typical values:

```text
0 → Disabled
1 → Conservative randomization
2 → More complete randomization
```

A common hardened configuration keeps ASLR enabled.

---

## 14.2 `kernel.pid_max`

Controls the maximum PID value the kernel can allocate.

Check:

```bash
sysctl kernel.pid_max
```

This can matter on systems creating very large numbers of processes.

However:

> Do not increase `pid_max` simply because a server has many processes. Investigate process-management behavior and other relevant limits first.

---

# 15. Useful Parameters for High-Traffic Servers

A useful mental checklist for high-concurrency systems is:

```text
             High-Traffic Linux Host
                      │
     ┌────────────────┼─────────────────┐
     ▼                ▼                 ▼
File Descriptors    Network          Ephemeral Ports
     │                │                 │
fs.file-max      somaxconn         ip_local_port_range
                 backlog           reserved ports
```

Then consider:

```text
Memory
  ├── vm.swappiness
  ├── vm.max_map_count
  └── dirty-page controls

TCP
  ├── tcp_max_syn_backlog
  ├── tcp_keepalive_*
  └── tcp_fin_timeout
```

Do not use one "tuning profile" for every server.

A database server, Nginx proxy, Kubernetes node, and Elasticsearch node have different bottlenecks.

---

# 16. DevOps Workloads and Typical Examples

## Nginx / Reverse Proxy

Potentially relevant:

```text
net.core.somaxconn
net.ipv4.tcp_max_syn_backlog
net.core.netdev_max_backlog
fs.file-max
net.ipv4.ip_local_port_range
```

Goal:

```text
Handle many concurrent connections
        +
Avoid queue / descriptor exhaustion
```

---

## Elasticsearch

A common parameter to know is:

```text
vm.max_map_count
```

Check:

```bash
sysctl vm.max_map_count
```

If the Elasticsearch deployment documentation requires a higher value:

```bash
sudo sysctl -w vm.max_map_count=262144
```

Then persist it in `/etc/sysctl.d/`.

Do not assume that this one setting is sufficient for Elasticsearch production tuning.

---

## Redis

Parameters that may appear in Redis deployment guidance include:

```text
vm.overcommit_memory
```

and memory-related host settings such as:

```text
vm.swappiness
```

The correct values depend on the Redis architecture and workload.

---

## Kubernetes Node

Common kernel/sysctl considerations include:

```text
net.ipv4.ip_forward
net.ipv6.conf.all.forwarding
net.ipv4.conf.all.rp_filter
fs.file-max
net.core.somaxconn
```

Container and Kubernetes networking can also require additional kernel modules, bridge/netfilter configuration, and distribution-specific settings that are not all controlled by sysctl.

---

## NAT / Router / Bastion Host

Useful concepts include:

```text
net.ipv4.ip_forward
net.ipv6.conf.all.forwarding
net.ipv4.ip_local_port_range
net.ipv4.ip_local_reserved_ports
```

A typical routing host may look like:

```text
Private Network
      │
      ▼
Linux NAT / Router
      │
      ▼
Internet / External Network
```

---

# 17. Troubleshooting Workflow

Do not start by randomly changing sysctl values.

Use this workflow.

## Step 1: Identify the symptom

Examples:

```text
Too many open files
Connection refused
Connection timeout
Ephemeral port exhaustion
Packet drops
Excessive swapping
Application requires more memory maps
```

## Step 2: Inspect Current Values

```bash
sysctl -a | grep <keyword>
```

Examples:

```bash
sysctl -a | grep somaxconn
sysctl -a | grep file-max
sysctl -a | grep ip_local
sysctl -a | grep swappiness
```

## Step 3: Inspect Runtime Behavior

Network:

```bash
ss -s
```

Sockets:

```bash
ss -tan
```

TIME_WAIT:

```bash
ss -tan state time-wait
```

Memory:

```bash
free -h
vmstat 1
```

Files:

```bash
cat /proc/sys/fs/file-nr
```

CPU / Load:

```bash
uptime
top
```

## Step 4: Change One Parameter

Example:

```bash
sudo sysctl -w net.core.somaxconn=4096
```

## Step 5: Measure Again

```bash
ss -s
vmstat 1
```

## Step 6: Persist Only After Validation

Create:

```text
/etc/sysctl.d/99-devops-tuning.conf
```

Then:

```bash
sudo sysctl --system
```

## Step 7: Document the Reason

Example:

```ini
# Required for application listener backlog under expected connection rate.
net.core.somaxconn = 4096
```

---

# 18. Important Cautions

## 1. Do Not Blindly Copy Tuning Guides

A value that helps one workload can hurt another.

For example:

```text
Database
≠
Nginx
≠
Redis
≠
Kubernetes
≠
Elasticsearch
```

---

## 2. Test Before Persisting

Use:

```bash
sudo sysctl -w ...
```

first.

Then observe the system.

Only afterward put the value into persistent configuration.

---

## 3. Keep the Previous Value

Before changing:

```bash
sysctl net.core.somaxconn
```

Record it.

Then change:

```bash
sudo sysctl -w net.core.somaxconn=4096
```

Rollback:

```bash
sudo sysctl -w net.core.somaxconn=<old-value>
```

---

## 4. `sysctl` Does Not Solve Every Limit

Some limits are controlled by:

* systemd unit settings
* PAM limits
* `ulimit`
* cgroups
* Container runtime settings
* Application configuration
* Network configuration
* Kernel modules

Example:

```text
Too many open files
        │
        ├── fs.file-max
        ├── process limits
        ├── systemd LimitNOFILE
        └── application behavior
```

---

## 5. Some Parameters Depend on Kernel Version

Parameter availability and defaults can vary by kernel version and distribution.

Always check:

```bash
uname -r
```

and:

```bash
sysctl <parameter>
```

The upstream kernel documentation should be treated as the authoritative reference for the running kernel family.

---

# 19. sysctl vs ulimit

These are often confused.

## sysctl

Primarily used for **kernel-wide or namespace-wide tunables** exposed under `/proc/sys`.

Example:

```bash
sysctl fs.file-max
```

## ulimit

Used for resource limits applied to a shell/process context.

Example:

```bash
ulimit -n
```

This commonly shows the maximum number of open file descriptors available to the current shell/process context.

### Comparison

| Feature | sysctl | ulimit |
|---|---|---|
| Primary scope | Kernel tunables | Process/resource limits |
| Typical interface | `/proc/sys` | Shell/PAM/systemd limits |
| Example | `net.core.somaxconn` | `nofile` |
| Example command | `sysctl -w ...` | `ulimit -n ...` |

For systemd services, also inspect:

```bash
systemctl show <service> | grep LimitNOFILE
```

---

# 20. Frequently Asked Interview Questions

| Question | Answer |
|---|---|
| **What is sysctl?** | A command-line interface for reading and changing Linux kernel parameters at runtime. |
| **Where are sysctl parameters exposed?** | Under `/proc/sys/`. |
| **How do you read a sysctl parameter?** | `sysctl parameter.name` |
| **How do you change a parameter temporarily?** | `sudo sysctl -w parameter.name=value` |
| **How do you persist a sysctl change?** | Put the setting in a file under `/etc/sysctl.d/` and apply it with `sysctl --system`. |
| **What is `vm.swappiness`?** | A virtual-memory tuning parameter controlling the kernel's swapping tendency. |
| **What is `vm.max_map_count`?** | The maximum number of memory-map areas a process may have. |
| **Why is `vm.max_map_count` important for Elasticsearch?** | Some Elasticsearch workloads require a higher maximum number of memory mappings. |
| **What is `fs.file-max`?** | A system-wide limit related to the number of allocated file handles. |
| **Is `fs.file-max` the same as `ulimit -n`?** | No. `fs.file-max` is system-wide; `ulimit -n` is a per-process/resource-limit setting. |
| **What is `net.core.somaxconn`?** | The kernel limit on the socket listen backlog exposed through `listen()`. |
| **What is `tcp_max_syn_backlog`?** | The maximum queue of remembered connection requests that have not completed the TCP handshake. |
| **What is `net.ipv4.ip_local_port_range`?** | The range of local ephemeral ports used for automatic port allocation. |
| **Why tune ephemeral ports?** | High-volume outbound connections can exhaust the available local port range. |
| **What is `net.ipv4.ip_forward`?** | Controls whether the Linux host forwards IPv4 packets between interfaces. |
| **What is `rp_filter`?** | Reverse-path filtering behavior for IPv4 source-route validation. |
| **What is `tcp_tw_reuse`?** | A TCP setting controlling reuse of certain `TIME_WAIT` sockets for outbound connections. |
| **Should `tcp_tw_reuse` always be enabled?** | No. First determine whether connection churn or ephemeral-port exhaustion is actually a problem. |
| **What is `sysctl --system`?** | Loads system sysctl configuration files. |
| **What is `sysctl -p`?** | Loads settings from `/etc/sysctl.conf` or a specified configuration file. |
| **Why use `/etc/sysctl.d/`?** | It provides a clean way to store persistent administrator configuration in dedicated files. |
| **Should every Linux server have the same sysctl tuning?** | No. Values should match workload, kernel version, network design, and observed bottlenecks. |
| **Can sysctl changes require a reboot?** | Many sysctl changes apply immediately, but some settings are not available until a related kernel module/interface exists and some parameters are not sysctls at all. |
| **How do you troubleshoot a sysctl-related performance issue?** | Measure the current value and runtime symptoms, change one parameter, measure again, validate the effect, and only then persist it. |

---

# 21. One-Line Interview Answers

| Parameter / Concept | One-Line Answer |
|---|---|
| **sysctl** | Runtime interface for Linux kernel tunables. |
| **`/proc/sys`** | Procfs hierarchy exposing many kernel tunables. |
| **`vm.swappiness`** | Controls the kernel's tendency to use swap. |
| **`vm.max_map_count`** | Maximum number of memory-map areas allowed for a process. |
| **`vm.overcommit_memory`** | Controls Linux memory overcommit policy. |
| **`vm.dirty_ratio`** | Threshold related to the amount of dirty writable memory before writeback pressure is applied. |
| **`vm.dirty_background_ratio`** | Threshold used to begin background writeback based on dirty memory. |
| **`fs.file-max`** | System-wide limit related to allocated file handles. |
| **`net.core.somaxconn`** | Maximum socket listen backlog exposed through `listen()`. |
| **`net.core.netdev_max_backlog`** | Maximum number of packets queued on the input side when packets arrive faster than processing. |
| **`tcp_max_syn_backlog`** | Maximum queue of incomplete TCP connection requests. |
| **`ip_local_port_range`** | Range of ephemeral ports used for automatic local port allocation. |
| **`ip_local_reserved_ports`** | Ports excluded from automatic port allocation. |
| **`tcp_fin_timeout`** | Timeout associated with orphaned TCP `FIN-WAIT-2` sockets. |
| **`tcp_tw_reuse`** | Controls reuse of certain `TIME_WAIT` sockets for outbound connections. |
| **`tcp_keepalive_*`** | Controls TCP keepalive timing and probe behavior. |
| **`tcp_syncookies`** | Controls TCP SYN cookie behavior for SYN-flood protection. |
| **`ip_forward`** | Enables/disables IPv4 packet forwarding. |
| **`rp_filter`** | Controls IPv4 reverse-path filtering behavior. |
| **Persistent sysctl** | Store settings in `/etc/sysctl.d/*.conf` and apply them with `sysctl --system`. |
| **Temporary sysctl** | Change the running kernel value using `sysctl -w`. |
| **`fs.file-max` vs `ulimit -n`** | System-wide file-handle limit versus per-process file-descriptor limit. |

---

# Practical DevOps Cheat Sheet

## Inspect

```bash
uname -r
sysctl vm.swappiness
sysctl vm.max_map_count
sysctl fs.file-max
sysctl net.core.somaxconn
sysctl net.core.netdev_max_backlog
sysctl net.ipv4.tcp_max_syn_backlog
sysctl net.ipv4.ip_local_port_range
sysctl net.ipv4.ip_forward
sysctl net.ipv4.conf.all.rp_filter
```

## Temporary tuning

```bash
sudo sysctl -w vm.swappiness=10
sudo sysctl -w vm.max_map_count=262144
sudo sysctl -w net.core.somaxconn=4096
sudo sysctl -w net.ipv4.ip_forward=1
```

## Persistent tuning

```bash
sudo nano /etc/sysctl.d/99-devops-tuning.conf
```

Example:

```ini
# DevOps host tuning
vm.swappiness = 10
vm.max_map_count = 262144
net.core.somaxconn = 4096
```

Apply:

```bash
sudo sysctl --system
```

Verify:

```bash
sysctl vm.swappiness
sysctl vm.max_map_count
sysctl net.core.somaxconn
```

## Network troubleshooting

```bash
ss -s
ss -tan
ss -tan state time-wait
```

## Memory troubleshooting

```bash
free -h
vmstat 1
```

## Kernel version

```bash
uname -r
```

---

# Final Mental Model

```text
                    Linux Kernel
                         │
            ┌────────────┼─────────────┐
            ▼            ▼             ▼
          Memory      Network       Files
            │            │             │
            ▼            ▼             ▼
      vm.* parameters  net.*       fs.*
            │            │             │
            └────────────┼─────────────┘
                         ▼
                      sysctl
                         │
          ┌──────────────┴──────────────┐
          ▼                             ▼
   Temporary Change              Persistent Change
   sysctl -w                     /etc/sysctl.d/*.conf
          │                             │
          ▼                             ▼
     Running Kernel               sysctl --system
```

The most important DevOps rule is:

> **Do not tune Linux kernel parameters because a value appears in a tuning guide. Identify the workload bottleneck, inspect the current kernel behavior, change one parameter at a time, measure the result, and only then make the change persistent.**
