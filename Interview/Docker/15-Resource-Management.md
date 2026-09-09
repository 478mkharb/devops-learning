# Docker Interview Preparation — Topic 15: Docker Resource Management

> **Interview focus:** Understand how Docker controls CPU, memory, PIDs, block I/O, and related resources; how limits differ from reservations; what OOM behavior means; and how to diagnose resource-pressure problems in production.

---

## Q358. Why is Docker resource management required?

### Short Interview Answer

Containers share the host kernel and physical resources, so one container can potentially consume excessive CPU, memory, processes, or I/O and affect other workloads.

Docker resource controls provide isolation at the **resource-usage** level.

Without limits:

```text
Container A ──┐
Container B ──┼── Host CPU / Memory / PIDs / I/O
Container C ──┘
```

If Container A consumes excessive resources:

```text
Container A
     ↓
resource exhaustion
     ↓
other workloads affected
```

### Common resources

- CPU
- Memory
- PIDs
- Block I/O
- Network indirectly through host/network controls
- Storage capacity through filesystem/storage design

### Important distinction

Linux namespaces provide **isolation of visibility and resources such as processes and networks**.

Linux cgroups provide **resource accounting and control**.

### Interview Point

> Containers isolate applications, but resource limits prevent one workload from consuming an unfair or dangerous amount of shared host capacity.

---

# CPU Management

## Q359. How do you limit CPU usage for a Docker container?

### Short Interview Answer

Docker provides CPU controls such as:

```bash
--cpus
--cpu-quota
--cpu-period
--cpu-shares
--cpuset-cpus
```

The most straightforward modern option is often:

```bash
docker run --cpus="1.5" myapp
```

This limits the container to approximately 1.5 CPUs worth of CPU time over time.

### Example

```bash
docker run -d \
  --name cpu-app \
  --cpus="1.0" \
  myapp
```

Conceptually:

```text
Host
CPU 0 ───────────────┐
CPU 1 ───────────────┤
CPU 2 ───────────────┤
                     │
                 Container
                 --cpus=1
```

The container can use approximately one CPU's worth of compute capacity, subject to host scheduling and contention.

### Interview Point

> `--cpus` provides a convenient CPU quota-style limit for a container.

---

## Q360. What is the difference between `--cpus` and `--cpuset-cpus`?

### Short Interview Answer

`--cpus` limits **how much CPU time** the container can consume, while `--cpuset-cpus` restricts **which CPU cores** the container can run on.

### `--cpus`

```bash
docker run --cpus="1.5" myapp
```

Means approximately:

```text
CPU capacity limit = 1.5 CPUs
```

### `--cpuset-cpus`

```bash
docker run \
  --cpuset-cpus="0,2" \
  myapp
```

Restricts execution to CPUs:

```text
CPU 0
CPU 2
```

### Conceptual difference

```text
--cpus
   ↓
How much CPU?

--cpuset-cpus
   ↓
Which CPUs?
```

They can also be combined.

### Interview Point

> CPU quota controls CPU consumption; CPU affinity controls where the workload can execute.

---

## Q361. What are CPU shares in Docker?

### Short Interview Answer

CPU shares provide a **relative CPU weight** used when CPU resources are contended. They are not the same as a hard CPU-capacity limit.

Example:

```bash
docker run -d \
  --cpu-shares=1024 \
  --name app1 \
  myapp
```

Another container:

```bash
docker run -d \
  --cpu-shares=512 \
  --name app2 \
  myapp
```

Under CPU contention, `app1` has a higher relative weight than `app2`.

### Important distinction

CPU shares do not mean:

```text
1024 = one CPU
512  = half CPU
```

They are relative weights.

### Interview Point

> CPU shares express relative priority under contention, not a fixed CPU limit.

---

## Q362. What is CPU quota and CPU period?

### Short Interview Answer

CPU quota and period can be used to define how much CPU time a container may consume during a scheduling period.

Example:

```bash
docker run \
  --cpu-period=100000 \
  --cpu-quota=50000 \
  myapp
```

Conceptually:

```text
Period = 100000 microseconds
Quota  = 50000 microseconds

50% of one CPU
```

The convenient equivalent for many use cases is:

```bash
--cpus="0.5"
```

### Interview Point

> CPU quota defines allowed CPU time within a period; `--cpus` provides a simpler interface for expressing the limit.

---

## Q363. What happens if a CPU-limited container tries to use more CPU?

### Short Interview Answer

The container is throttled rather than simply being allowed to consume unlimited CPU.

For example:

```bash
docker run --cpus="0.5" cpu-intensive-app
```

If the application continuously wants more CPU than its limit permits:

```text
Application wants 2 CPUs
        ↓
Docker/cgroup limit
        ↓
approximately 0.5 CPU capacity available
        ↓
CPU throttling
```

### Important distinction

CPU limits generally cause **throttling**, not an OOM-style container termination.

### Interview Point

> CPU pressure from a CPU limit normally manifests as throttling and reduced performance rather than memory-style OOM termination.

---

# Memory Management

## Q364. How do you limit memory for a Docker container?

### Short Interview Answer

Use:

```bash
--memory
```

or:

```bash
-m
```

Example:

```bash
docker run -d \
  --memory="512m" \
  myapp
```

This limits the container's memory usage according to the configured cgroup memory controls.

### Example

```bash
docker run \
  --memory=512m \
  --memory-swap=512m \
  myapp
```

The exact interaction between memory and swap depends on the host/kernel configuration and Docker's configured limits.

### Interview Point

> Memory limits protect the host and neighboring workloads from unbounded container memory consumption.

---

## Q365. What happens when a container reaches its memory limit?

### Short Interview Answer

The kernel's cgroup memory controller prevents the container from exceeding its configured memory boundary. If memory cannot be reclaimed and allocation cannot continue, processes in the cgroup can be killed by the OOM mechanism.

Typical symptoms include:

```text
OOMKilled
exit code 137
```

### Important distinction

```text
exit 137
=
128 + 9
=
SIGKILL
```

This is **consistent with** an OOM kill but does not prove OOM by itself.

### Better diagnosis

Check:

```bash
docker inspect <container>
```

Look for:

```text
OOMKilled
```

Also inspect:

```bash
docker events
docker stats
journalctl
dmesg
```

depending on the environment and permissions.

### Interview Point

> A memory limit can result in OOM killing; verify the actual OOM state instead of assuming every exit 137 is an OOM.

---

## Q366. What is the difference between `--memory` and `--memory-swap`?

### Short Interview Answer

`--memory` controls the memory limit, while `--memory-swap` controls the combined memory+swap limit where swap accounting is supported and enabled.

Example:

```bash
docker run \
  --memory=512m \
  --memory-swap=1g \
  myapp
```

Conceptually:

```text
Memory limit       = 512 MB
Memory + swap max  = 1 GB
```

So the container may have up to approximately:

```text
512 MB RAM
+
512 MB swap
```

subject to system configuration.

### Important caveat

Swap behavior depends on the host operating system/kernel and Docker configuration.

### Interview Trap

Do not automatically interpret:

```text
--memory-swap=1g
```

as:

```text
1 GB RAM + 1 GB swap
```

It represents the configured memory+swap limit in Docker's Linux memory-control model.

### Interview Point

> `--memory` and `--memory-swap` are related but represent different boundaries.

---

## Q367. What is `--memory-reservation`?

### Short Interview Answer

`--memory-reservation` provides a **soft memory limit** that can be used as a lower-priority/reclaim-related target under memory pressure, whereas `--memory` is the hard memory limit.

Example:

```bash
docker run \
  --memory=1g \
  --memory-reservation=512m \
  myapp
```

Conceptually:

```text
Reservation = 512 MB
Hard limit  = 1 GB
```

### Interview Point

> Reservation and limit are different: a reservation is not the same as a hard upper bound.

---

## Q368. What is `--oom-kill-disable`?

### Short Interview Answer

It controls whether Docker disables the kernel OOM killer for processes in the container's cgroup.

Example:

```bash
docker run \
  --memory=512m \
  --oom-kill-disable \
  myapp
```

### Why dangerous?

If a workload reaches its memory boundary and OOM killing is disabled, the system can experience severe memory pressure rather than safely killing the offending process.

This option should therefore be used only with a clear understanding of the workload and host behavior.

### Interview Trap

Do not present this as a general solution to OOM problems.

### Interview Point

> Disabling OOM killing can increase host-level risk; first fix the application's memory behavior or choose an appropriate memory limit.

---

## Q369. What is `docker stats`?

### Short Interview Answer

`docker stats` displays live resource-usage information for running containers.

Example:

```bash
docker stats
```

Typical metrics include:

```text
CPU %
Memory usage / limit
Memory %
Network I/O
Block I/O
PIDs
```

### Example

```text
CONTAINER ID   CPU %   MEM USAGE / LIMIT   MEM %
abc123         72%     420MiB / 512MiB     82%
```

### Why useful?

It provides a quick operational view when investigating:

- CPU spikes.
- Memory growth.
- Network activity.
- Block I/O.
- Process-count growth.

### Interview Point

> `docker stats` shows runtime resource usage; it does not replace host-level monitoring.

---

## Q370. How do you identify whether a container is consuming too much memory?

### Short Interview Answer

Start with:

```bash
docker stats
```

Then compare:

```text
memory usage
vs
memory limit
```

Inspect container state:

```bash
docker inspect <container>
```

Check for:

```text
OOMKilled
```

Inside the container, if available:

```bash
free -h
ps aux --sort=-%mem
```

Also investigate the host:

```bash
free -h
vmstat
```

and kernel logs where appropriate:

```bash
dmesg
```

### Important nuance

Container memory metrics can differ from what a process appears to report inside the container because cgroup accounting and kernel memory behavior matter.

### Interview Point

> Diagnose memory pressure from both the container's cgroup metrics and the host's overall memory state.

---

# Process Limits

## Q371. What is a PID limit in Docker?

### Short Interview Answer

A PID limit restricts the number of processes that can exist in a container's cgroup.

Example:

```bash
docker run \
  --pids-limit=100 \
  myapp
```

This helps protect the host against runaway process creation.

### Scenario

A buggy process repeatedly forks:

```text
process
 ├── child
 ├── child
 ├── child
 └── ...
```

Without a suitable limit, it can exhaust available process resources.

With:

```bash
--pids-limit=100
```

the cgroup can restrict process creation.

### Interview Point

> PID limits protect against process exhaustion, including fork-bomb-like behavior.

---

## Q372. What is a fork bomb, and how can Docker resource controls help?

### Short Interview Answer

A fork bomb is a process that continuously creates more processes until system process resources are exhausted.

Conceptually:

```text
Process
 ↓
2 processes
 ↓
4
 ↓
8
 ↓
16
 ↓
...
```

A container-level PID limit can constrain the blast radius:

```bash
docker run \
  --pids-limit=100 \
  myapp
```

### Important point

A PID limit does not make malicious or buggy code safe. It limits one resource-exhaustion path.

### Interview Point

> Resource limits are defense-in-depth controls, not a substitute for application and container security.

---

# CPU and Memory Reservations

## Q373. What is the difference between a resource limit and a resource reservation?

### Short Interview Answer

A **limit** defines an upper boundary for resource consumption, while a **reservation** expresses a softer resource requirement/target used under contention.

For memory:

```bash
--memory=1g
--memory-reservation=512m
```

Conceptually:

```text
0
│
├── normal target / reservation
│       512 MB
│
├── hard limit
│       1 GB
│
└── above limit → constrained / OOM behavior
```

### Important nuance

Docker resource semantics differ by resource type and host/kernel implementation. Do not assume CPU and memory reservations behave identically.

### Interview Point

> A limit is a boundary; a reservation is not simply another name for a limit.

---

# Block I/O

## Q374. How can Docker control block I/O?

### Short Interview Answer

Docker can expose block-I/O-related controls such as:

```text
--blkio-weight
--device-read-bps
--device-write-bps
--device-read-iops
--device-write-iops
```

These controls depend on the host kernel, storage subsystem, and Docker support.

### Example

```bash
docker run \
  --device-read-bps /dev/sda:10mb \
  myapp
```

This can constrain read bandwidth for the specified block device where supported.

### Why useful?

A storage-heavy container can otherwise create contention:

```text
Database container
       ↓
heavy disk I/O
       ↓
other workloads become slow
```

### Interview Point

> Block-I/O controls can limit storage pressure, but their effectiveness depends on the host storage stack and kernel support.

---

## Q375. What is `--blkio-weight`?

### Short Interview Answer

It provides a relative weight for block I/O scheduling among competing workloads where supported.

Conceptually:

```text
Container A → weight 800
Container B → weight 200
```

Under contention, A receives a larger relative share.

It is analogous in concept to CPU relative weighting but applies to block I/O.

### Interview Point

> I/O weight is relative priority, not a fixed MB/s limit.

---

# Device Access

## Q376. What does `--device` do?

### Short Interview Answer

`--device` exposes a host device to a container, subject to Docker's device and security controls.

Example:

```bash
docker run \
  --device=/dev/snd \
  myapp
```

This can be required for workloads that need access to hardware devices.

### Security concern

Giving containers device access can significantly increase their capabilities.

Do not expose host devices unnecessarily.

### Interview Point

> Device access is powerful and should follow least privilege.

---

# Resource Visibility and Isolation

## Q377. Does a container see the host's total memory?

### Short Interview Answer

The answer depends on the container runtime, cgroup configuration, kernel, and application behavior. Modern Linux/container environments can expose cgroup-aware resource limits to applications, but tools inside a container may not always present resource information exactly as a host administrator sees it.

For troubleshooting, prefer cgroup-aware/container-aware information and Docker metrics.

Useful commands include:

```bash
docker stats
docker inspect
```

and, on Linux systems:

```bash
cat /sys/fs/cgroup/...
```

with the exact paths depending on cgroup version and configuration.

### Interview Point

> Do not assume traditional host-oriented tools inside a container always describe the container's effective resource limits correctly.

---

## Q378. What is the difference between resource isolation and resource limiting?

### Short Interview Answer

**Resource isolation** separates how workloads see or account for resources, while **resource limiting** actively constrains how much of a resource they can consume.

For example:

```text
cgroups
   ├── accounting
   ├── limits
   └── control

namespaces
   ├── PID isolation
   ├── network isolation
   └── filesystem/mount isolation
```

### Interview Point

> Namespaces primarily provide isolation of system views; cgroups provide resource accounting and control.

---

# OOM and Troubleshooting

## Q379. How would you troubleshoot a container that keeps getting OOMKilled?

### Strong Interview Answer

I would first confirm that the container is actually being killed by the OOM mechanism, then determine whether the limit is too low or the application is unexpectedly consuming memory.

### Step 1 — Inspect state

```bash
docker inspect <container>
```

Check:

```text
OOMKilled
ExitCode
```

### Step 2 — Check memory usage

```bash
docker stats <container>
```

Look for:

```text
MEM USAGE / LIMIT
```

### Step 3 — Check application behavior

Inside the container:

```bash
ps aux --sort=-%mem
```

If available:

```bash
free -h
```

### Step 4 — Check host/kernel evidence

```bash
dmesg
```

or:

```bash
journalctl
```

depending on the environment.

### Step 5 — Determine the cause

Possible causes:

```text
Memory leak
Large workload
Incorrect memory limit
Unexpected traffic
Unbounded cache
Too many processes
JVM/heap configuration
Native memory usage
```

### Step 6 — Fix the correct layer

Possible actions:

- Fix application memory leak.
- Reduce memory usage.
- Configure application heap/cache correctly.
- Increase container memory limit when justified.
- Scale horizontally.
- Add monitoring and alerting.

### Interview Point

> Increasing the memory limit may hide the symptom; always determine why the application needs the memory.

---

## Q380. A container is slow but its CPU usage is only 20%. What would you investigate?

### Strong Interview Answer

Low CPU usage does not prove the application has sufficient resources.

I would investigate:

```text
Memory pressure
Disk I/O
Network latency
CPU throttling
Application locks
External dependencies
Database latency
Thread/process limits
```

### Check container metrics

```bash
docker stats
```

### Check host

```bash
top
free -h
iostat
vmstat
```

where available.

### Check CPU throttling

Inspect cgroup metrics or Docker/runtime metrics appropriate to the host.

If the container has a restrictive CPU quota, it may experience throttling even if instantaneous CPU percentage appears misleading.

### Check application

```text
request latency
database latency
thread pools
connection pools
logs
```

### Interview Point

> "CPU is low" only eliminates one obvious symptom; it does not prove the container is healthy.

---

## Q381. A container uses 100% CPU. Is that automatically a problem?

### Short Interview Answer

No.

100% CPU can be expected if:

- The container is intentionally CPU-bound.
- It has a one-CPU limit and is fully utilizing it.
- The workload is processing normally.

It becomes a problem when it causes:

- Latency.
- CPU throttling.
- Host contention.
- Unexpected cost.
- Resource starvation for other workloads.

### Example

```bash
docker run --cpus=1 myapp
```

The application showing approximately:

```text
100% CPU
```

can mean it is fully using its allowed one CPU.

### Interview Point

> CPU percentage must be interpreted relative to the container's configured limit and workload expectations.

---

## Q382. What is CPU throttling?

### Short Interview Answer

CPU throttling occurs when a cgroup reaches its configured CPU quota and the scheduler prevents the workload from consuming additional CPU during the relevant period.

Conceptually:

```text
Application wants CPU
       ↓
CPU quota reached
       ↓
cgroup throttling
       ↓
wait
       ↓
next available period
```

### Symptoms

- Increased application latency.
- Lower throughput.
- CPU usage appearing capped.
- High throttled-time metrics.

### Important distinction

Throttling is not the same as CPU saturation on the whole host.

A container may be throttled because **its own configured limit is too low**, even when the host still has idle CPU.

### Interview Point

> A container can be CPU-throttled even when the host is not CPU-saturated.

---

# Runtime Monitoring

## Q383. What is the difference between `docker stats` and host monitoring?

### Short Interview Answer

`docker stats` provides container-level runtime metrics from Docker, while host monitoring provides broader system-level visibility.

### Docker

```bash
docker stats
```

Useful for:

```text
Container CPU
Container memory
Network I/O
Block I/O
PIDs
```

### Host monitoring

Tools such as:

```bash
top
vmstat
iostat
free
```

show host-level behavior.

### Production monitoring

A production platform may use:

```text
Prometheus
Node Exporter
cAdvisor/runtime metrics
Cloud monitoring
Application metrics
```

### Interview Point

> Container-level metrics tell you which workload is behaving badly; host-level metrics tell you whether the entire node is under pressure.

---

## Q384. How would you choose CPU and memory limits for a production container?

### Strong Interview Answer

I would not choose arbitrary values.

I would:

1. Measure normal resource usage.
2. Load-test realistic traffic.
3. Observe peak usage.
4. Understand startup spikes.
5. Understand JVM/runtime overhead where applicable.
6. Set a justified limit with operational headroom.
7. Monitor throttling and OOM events.
8. Re-tune based on production behavior.

### Example

If testing shows:

```text
Normal memory: 300 MB
Peak:          420 MB
```

I might initially choose a limit above the observed peak with reasonable headroom, rather than blindly setting:

```text
--memory=420m
```

### Interview Point

> Resource limits should be evidence-based and validated under realistic workload.

---

# Resource Management Scenarios

## Q385. Why can two containers on the same host have very different CPU usage?

### Short Interview Answer

Because their workloads and resource controls can differ.

For example:

```text
Container A
--cpus=2

Container B
--cpus=0.5
```

Even if both are CPU-bound, their allowed CPU capacity differs.

CPU shares, quotas, CPU affinity, application behavior, and host contention can also affect observed usage.

### Interview Point

> Container CPU usage is determined by workload plus cgroup scheduling controls plus host contention.

---

## Q386. A container has a 1 GB memory limit but the application reports less than 1 GB. Why could it still be OOMKilled?

### Strong Interview Answer

The application's own memory reporting may not include every form of memory accounted by the container's cgroup.

Potential contributors include:

- Native allocations.
- Runtime overhead.
- Other processes in the container.
- File/cache memory depending on accounting and workload.
- Memory used outside the application's own reporting mechanism.

Therefore:

```text
application-reported memory
```

is not necessarily identical to:

```text
container cgroup memory usage
```

### Troubleshooting

Compare:

```bash
docker stats
```

with application metrics and cgroup information.

### Interview Point

> Diagnose OOM from the container/cgroup perspective, not only from the application's heap metric.

---

## Q387. Can a container consume all host CPU if no CPU limit is configured?

### Short Interview Answer

Yes. A CPU-intensive container without a CPU quota can consume available CPU capacity, subject to normal host scheduling and competition from other workloads.

This does not mean it bypasses the Linux scheduler; it means there may be no container-level CPU cap preventing it from using available CPU.

### Example

```bash
docker run -d \
  --name cpu-hog \
  cpu-intensive-image
```

If the application continuously consumes CPU:

```text
CPU-hog
   ↓
uses available CPU
   ↓
other workloads may receive less CPU
```

### Interview Point

> Resource limits are especially important on shared hosts.

---

## Q388. Can a container consume all host memory if no memory limit is configured?

### Short Interview Answer

A container without an appropriate memory limit can consume substantial host memory and contribute to system-wide memory pressure. The Linux kernel ultimately manages host memory, but without container-level controls the workload has a larger opportunity to consume available memory.

### Possible result

```text
Container
   ↓
large allocation
   ↓
host memory pressure
   ↓
reclaim / swap if available
   ↓
host OOM conditions possible
```

### Interview Point

> Memory limits are a host-protection mechanism, not merely an application setting.

---

## Q389. What happens when multiple containers have CPU limits whose total exceeds host capacity?

### Short Interview Answer

The configured limits represent maximum potential consumption, not guaranteed dedicated physical CPUs.

For example, on a host with:

```text
4 CPUs
```

you could configure:

```text
Container A → 2 CPUs
Container B → 2 CPUs
Container C → 2 CPUs
```

Total configured potential:

```text
6 CPUs
```

If all three are CPU-bound simultaneously, the host scheduler must share the available 4 CPUs.

### Important distinction

```text
CPU limit
≠
dedicated physical CPU
```

### Interview Point

> CPU limits define maximum scheduling capacity, not guaranteed dedicated hardware.

---

## Q390. What happens when memory limits across containers exceed physical host memory?

### Short Interview Answer

Unlike CPU overcommit, memory is constrained by actual available memory plus swap and kernel behavior. If multiple containers collectively demand more memory than the host can safely provide, the host can experience memory pressure and OOM conditions.

Example:

```text
Host RAM = 8 GB

Container A limit = 4 GB
Container B limit = 4 GB
Container C limit = 4 GB
```

The limits sum to:

```text
12 GB
```

If all containers actually consume their full limits simultaneously, the host cannot satisfy all demands from 8 GB RAM alone.

### Interview Point

> Memory limits protect individual containers but do not magically create physical memory.

---

# Practical Commands

## Q391. How do you inspect a container's configured resource limits?

### Short Interview Answer

Use:

```bash
docker inspect <container>
```

and look at the container's host configuration/resource-related settings.

You can also use:

```bash
docker stats <container>
```

for live usage.

### Useful commands

```bash
docker inspect app
docker stats app
docker info
```

Depending on Docker version and configuration, `docker inspect` exposes resource-related configuration under the container's host configuration.

### Interview Point

> Use `inspect` for configuration and `stats` for live usage.

---

## Q392. How do you see which containers are consuming resources?

### Command

```bash
docker stats
```

For a single container:

```bash
docker stats app
```

For a one-time snapshot:

```bash
docker stats --no-stream
```

### Useful output

```text
CPU %
MEM USAGE / LIMIT
MEM %
NET I/O
BLOCK I/O
PIDS
```

### Interview Point

> `docker stats --no-stream` is especially useful in troubleshooting scripts because it returns a snapshot.

---

## Q393. How would you identify a CPU-hungry process inside a container?

### First:

```bash
docker stats <container>
```

Then enter the container:

```bash
docker exec -it <container> sh
```

Use available tools:

```bash
ps aux
top
```

or:

```bash
ps aux --sort=-%cpu
```

If the image is minimal and lacks diagnostic tools, inspect from the host where appropriate.

### Interview Point

> Do not assume every production image contains debugging utilities; minimal images often do not.

---

## Q394. How would you identify a memory-hungry process inside a container?

### Example

```bash
docker exec <container> \
  ps aux --sort=-%mem
```

Or interactively:

```bash
docker exec -it <container> sh
ps aux
```

Then correlate process-level information with:

```bash
docker stats
```

### Important nuance

Process RSS/heap reporting and cgroup memory accounting are not necessarily identical.

### Interview Point

> Correlate process-level metrics with container-level cgroup metrics.

---

# Resource Management Design

## Q395. What are good resource-management practices for production Docker containers?

### Strong Interview Answer

I would:

- Set appropriate CPU limits where required.
- Set appropriate memory limits.
- Monitor CPU throttling.
- Monitor OOM events.
- Consider PID limits.
- Avoid unlimited process creation.
- Measure before selecting limits.
- Leave host capacity for system services.
- Avoid overcommitting memory blindly.
- Load-test before production.
- Alert on sustained resource pressure.
- Right-size containers based on actual workload.

### Example baseline

```text
Application
   │
   ├── CPU limit
   ├── Memory limit
   ├── PID limit
   ├── monitoring
   └── alerting
```

### Interview Point

> Resource management is a combination of limits, measurement, capacity planning, and monitoring.

---

# Quick Revision

| Topic | Key Point |
|---|---|
| Resource management | Prevents one workload from exhausting shared resources |
| cgroups | Resource accounting and control |
| `--cpus` | CPU capacity limit |
| `--cpuset-cpus` | CPU affinity |
| CPU shares | Relative CPU weight under contention |
| CPU quota | CPU time allowed per period |
| CPU throttling | Workload constrained by CPU quota |
| `--memory` | Memory limit |
| `--memory-swap` | Combined memory+swap boundary |
| Memory reservation | Softer memory target/constraint |
| OOMKilled | Container/process affected by OOM mechanism |
| Exit 137 | SIGKILL; OOM is common but not proof |
| `--oom-kill-disable` | Disables OOM killing for cgroup; potentially dangerous |
| `--pids-limit` | Limits processes/PIDs |
| Fork bomb | Runaway process creation |
| Block I/O | Storage I/O controls |
| `--blkio-weight` | Relative I/O weight |
| `--device` | Exposes host device |
| `docker stats` | Live container resource usage |
| `docker inspect` | Container configuration/state |
| CPU limit | Not dedicated physical CPU |
| Memory limit | Does not create physical memory |
| Overcommit CPU | Possible; scheduler shares capacity |
| Overcommit memory | Can cause host memory pressure/OOM |
| Resource limit selection | Measure and load-test first |

---

# High-Value Interview Traps

### Trap 1 — "`--cpus=1` gives the container one dedicated CPU"

**Incorrect.**

It limits CPU consumption to approximately one CPU's worth of capacity; it does not reserve a dedicated physical core.

---

### Trap 2 — "`--cpu-shares=1024` means 1024 CPU units"

**Incorrect.**

CPU shares are relative weights.

---

### Trap 3 — "100% CPU always means a problem"

**Incorrect.**

Interpret it relative to the configured limit and workload.

---

### Trap 4 — "Exit code 137 always means OOM"

**Incorrect.**

137 means the process was killed with SIGKILL. OOM is a common cause, but verify `OOMKilled` and system evidence.

---

### Trap 5 — "Increasing the memory limit fixes an OOM problem"

Not necessarily.

The application may have:

```text
memory leak
unbounded cache
incorrect heap
unexpected workload
```

---

### Trap 6 — "CPU throttling means the host CPU is exhausted"

**Incorrect.**

A container can be throttled because its own cgroup CPU limit is too low even while the host has idle CPU.

---

### Trap 7 — "Memory limits can exceed host memory without consequences"

**Incorrect.**

Configured limits can collectively exceed physical memory, but if workloads actually consume those limits simultaneously, the host can experience memory pressure and OOM.

---

### Trap 8 — "PID limits are only for security"

Not only security.

They also protect against accidental process explosions and resource exhaustion.

---

### Trap 9 — "Docker stats is enough for production monitoring"

**Incorrect.**

It is useful for troubleshooting, but production observability normally requires host-level and application-level metrics as well.

---

### Trap 10 — "A container with no CPU limit can use unlimited CPU beyond the host"

**Incorrect.**

It cannot create CPU capacity; it can consume available host CPU subject to the host scheduler.

---

# High-Value Interview Follow-Up Questions

1. What are Linux cgroups?
2. How does Docker use cgroups?
3. How do you limit CPU?
4. `--cpus` vs `--cpuset-cpus`?
5. CPU shares vs CPU limits?
6. What is CPU throttling?
7. Can a CPU-limited container be throttled while the host has idle CPU?
8. How do you limit memory?
9. What happens when a container reaches its memory limit?
10. What does exit code 137 mean?
11. How do you prove a container was OOMKilled?
12. `--memory` vs `--memory-swap`?
13. What is memory reservation?
14. What is `--oom-kill-disable`?
15. What is a PID limit?
16. Why use `--pids-limit`?
17. How can a fork bomb affect a host?
18. How do you inspect live container resource usage?
19. `docker stats` vs `docker inspect`?
20. Why can application memory metrics differ from container memory metrics?
21. How do you choose production CPU/memory limits?
22. Can CPU limits across containers exceed host CPU capacity?
23. What happens if memory limits collectively exceed host RAM?
24. How do you troubleshoot high CPU?
25. How do you troubleshoot OOMKilled containers?
26. What are Docker block-I/O controls?
27. What does `--blkio-weight` do?
28. What does `--device` do?
29. Why can a container be slow with low CPU?
30. What production resource-management practices would you recommend?

---

# Final Interview Answer

If asked **"How does Docker manage container resources?"**, a strong answer is:

> "Docker uses Linux cgroups to account for and control resources such as CPU, memory, PIDs, and block I/O. For CPU, I can use options such as `--cpus` for a capacity limit, `--cpuset-cpus` for CPU affinity, and CPU shares for relative priority under contention. For memory, I can set `--memory` and configure memory+swap behavior where supported. If a container exceeds its memory boundary and the system cannot satisfy allocations, OOM killing can occur. I would verify that using `docker inspect`, `docker stats`, and host or kernel evidence rather than assuming every exit code 137 is an OOM. I also consider PID limits to prevent process exhaustion. In production, I would choose limits based on load testing and observed usage, leave capacity for host services, monitor throttling and OOM events, and continuously right-size the workloads."

---

# One-Line Memory Map

```text
cgroups      = resource control/accounting
--cpus       = how much CPU
--cpuset     = which CPUs
shares       = relative CPU priority
--memory     = memory ceiling
swap         = memory + swap boundary
pids         = process-count protection
throttling   = CPU quota reached
OOMKilled    = memory allocation could not continue
stats        = live usage
inspect      = configured state
```
