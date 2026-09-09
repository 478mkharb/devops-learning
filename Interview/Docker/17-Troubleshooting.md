# Docker Interview Preparation — Topic 17: Docker Troubleshooting

> **Interview focus:** Diagnose Docker problems systematically instead of guessing.  
> The goal is to move from **container state → logs → configuration → resources → network/storage → host/daemon**.

---

# 438. What is your general Docker troubleshooting methodology?

## Short Interview Answer

I troubleshoot Docker from the outside in:

```text
Container state
      ↓
Logs / exit code
      ↓
Inspect configuration
      ↓
Process / resource usage
      ↓
Network / storage
      ↓
Docker daemon / host
      ↓
Application itself
```

I first identify **what is failing**, then collect evidence before changing anything.

## Detailed Explanation

A good troubleshooting sequence is:

1. Check whether the container exists and its state.
2. Check `docker logs`.
3. Check exit code and health status.
4. Use `docker inspect` for configuration and state.
5. Check processes and resources.
6. If connectivity is involved, inspect network membership, DNS, routes and ports.
7. If data is involved, inspect mounts and disk usage.
8. If Docker itself is failing, check the daemon and host.
9. Reproduce the problem with the smallest useful test.
10. Make the smallest corrective change and verify.

### Useful commands

```bash
docker ps -a
docker logs <container>
docker inspect <container>
docker stats
docker top <container>
docker events
docker system df
docker network inspect <network>
docker volume inspect <volume>
```

## Common Interview Trap

Do not immediately restart the container. A restart can hide the original failure and destroy useful diagnostic context.

## Interview Point

**Troubleshooting means collecting evidence first, not blindly restarting containers.**

---

# 439. A container will not start. How do you troubleshoot it?

## Short Interview Answer

I check whether the failure happens during image creation, container creation, or process startup:

```bash
docker ps -a
docker inspect <container>
docker logs <container>
```

Then I check image availability, command/entrypoint, mounts, ports, permissions and resource constraints.

## Detailed Explanation

Typical causes include:

- invalid image/reference
- missing image
- invalid command or entrypoint
- missing executable
- invalid volume mount
- permission problems
- port conflict
- invalid environment configuration
- incompatible architecture
- resource constraints

For a newly created container:

```bash
docker run --name test <image>
```

If it fails immediately:

```bash
docker ps -a
docker logs test
docker inspect test
```

For example:

```text
exec /app/start.sh: permission denied
```

The image may contain the script, but it is not executable.

Possible diagnosis:

```bash
docker run --rm --entrypoint /bin/sh <image>
ls -l /app/start.sh
```

## Common Interview Trap

"Container won't start" is not enough information. First determine whether Docker itself rejects the container or the container's main process exits immediately.

## Interview Point

**Separate Docker-level errors from application process failures.**

---

# 440. A container starts and immediately exits. Why?

## Short Interview Answer

A container normally lives as long as its **main process** runs. If PID 1 exits, the container stops.

I check:

```bash
docker ps -a
docker logs <container>
docker inspect <container>
```

especially the exit code and command.

## Example

```bash
docker run --name test alpine
```

The default process may finish immediately, so the container exits.

Interactive shell:

```bash
docker run -it alpine sh
```

Long-running application:

```bash
docker run myapp
```

The application process must remain active.

## Common Causes

- application crashed
- wrong `CMD`
- wrong `ENTRYPOINT`
- missing configuration
- missing dependency
- application intentionally completed
- shell script exited
- signal caused shutdown

## Common Interview Trap

A stopped container is not necessarily "broken." A batch job can correctly exit with code `0`.

## Interview Point

**Container lifecycle follows the main process lifecycle.**

---

# 441. A container keeps restarting. How do you troubleshoot it?

## Short Interview Answer

I check logs, exit code, health status, restart policy and resource limits:

```bash
docker ps
docker logs --tail 100 <container>
docker inspect <container>
docker stats
```

Then I determine why the application exits.

## Detailed Explanation

Typical loop:

```text
start
  ↓
application fails
  ↓
process exits
  ↓
restart policy starts it again
  ↓
application fails
  ↓
repeat
```

Check restart policy:

```bash
docker inspect <container> \
  --format '{{json .HostConfig.RestartPolicy}}'
```

Check state:

```bash
docker inspect <container> \
  --format '{{json .State}}'
```

If logs show:

```text
connection refused to database
```

the restart policy is not the root cause. The database/configuration/dependency needs investigation.

## Common Interview Trap

Do not "fix" a restart loop merely by disabling the restart policy. That changes the symptom, not necessarily the cause.

## Interview Point

**Restart policy explains why it keeps restarting; logs explain why it keeps failing.**

---

# 442. `docker pull` fails. What do you check?

## Short Interview Answer

I check:

1. image name and tag
2. registry availability
3. authentication
4. authorization
5. network connectivity
6. architecture/platform compatibility
7. registry policy or rate limits

Example:

```bash
docker pull nginx:1.27
```

## Diagnostic Thinking

```text
Correct image reference?
        ↓
Registry reachable?
        ↓
Authenticated?
        ↓
Authorized?
        ↓
Requested tag exists?
        ↓
Platform supported?
```

## Common Interview Trap

`pull access denied` can mean the repository is private, the credentials are wrong, or the repository/tag does not exist.

## Interview Point

**Separate name/reference problems from authentication, authorization and connectivity problems.**

---

# 443. The image cannot be found. What could be wrong?

## Short Interview Answer

I verify the complete image reference:

```text
[registry/]repository[:tag|@digest]
```

For example:

```bash
docker pull nginx:latest
```

If the intended image is private:

```bash
docker login <registry>
docker pull <registry>/<repository>:<tag>
```

## Common Causes

- typo in repository name
- wrong registry
- wrong tag
- private repository without permission
- image was never pushed
- image was deleted
- production references a tag that does not exist

## Common Interview Trap

Do not assume `latest` exists. A repository can have no `latest` tag.

## Interview Point

**Validate the exact image reference rather than assuming the tag exists.**

---

# 444. Docker says the container name is already in use. What do you do?

## Short Interview Answer

Container names must be unique on a Docker host.

Find the existing container:

```bash
docker ps -a --filter name=myapp
```

Then either reuse it:

```bash
docker start myapp
```

or remove/rename it:

```bash
docker rename myapp myapp-old
```

```bash
docker rm myapp
```

## Common Interview Trap

Deleting a container is destructive to its writable layer. Check its volumes before removal if data matters.

## Interview Point

**Container names identify container objects and cannot be duplicated on the same Docker host.**

---

# 445. Docker reports "port is already allocated." What does it mean?

## Short Interview Answer

Another process or container is already using the requested **host port**.

Check containers:

```bash
docker ps --format 'table {{.Names}}\t{{.Ports}}'
```

Check host listeners:

```bash
sudo ss -lntp
```

Then either stop the conflicting service or choose another host port.

Example:

```bash
docker run -p 8081:80 nginx
```

## Important Distinction

Two containers can both listen on:

```text
container port 80
```

but they cannot normally both publish:

```text
host port 8080
```

on the same host IP/protocol combination.

## Interview Point

**Port conflicts concern the host-side published port.**

---

# 446. The application is running, but I cannot reach it. How do you troubleshoot?

## Short Interview Answer

I verify the complete path:

```text
Client
  ↓
Host published port
  ↓
Docker port mapping
  ↓
Container network
  ↓
Application listening port
  ↓
Application
```

Commands:

```bash
docker ps
docker port <container>
docker inspect <container>
docker logs <container>
```

Inside the container:

```bash
ss -lntp
```

Then test locally:

```bash
curl http://127.0.0.1:<port>
```

and from the appropriate source:

```bash
curl http://<host>:<published-port>
```

## Common Causes

- application listening only on `127.0.0.1`
- wrong container port
- wrong `-p` mapping
- host firewall
- cloud security group
- application not actually running
- wrong network
- reverse proxy configuration

## Interview Point

**"Container is running" does not mean "application is reachable."**

---

# 447. Docker logs are empty. Why?

## Short Interview Answer

`docker logs` reads the container's configured logging stream. It does not automatically read arbitrary application log files.

If the application writes logs to:

```text
/var/log/myapp/app.log
```

instead of stdout/stderr, `docker logs` may show nothing.

## Better Container Pattern

Prefer application logs to:

```text
stdout
stderr
```

Then:

```bash
docker logs <container>
```

works naturally with Docker's logging system.

## Other Possibilities

- container has not produced output
- logging driver behaves differently
- application daemonizes itself
- logs are written to files
- output was redirected

## Common Interview Trap

Do not say "`docker logs` shows all application logs." It shows logs captured through the container's logging mechanism.

## Interview Point

**Containerized applications should generally emit operational logs to stdout/stderr.**

---

# 448. What is the difference between Docker logs and application log files?

| Docker logs | Application log file |
|---|---|
| Reads container logging stream | Reads a file |
| Commonly stdout/stderr | `/var/log/...` or application directory |
| Docker logging driver can process it | Application/file system manages it |
| Easy for container platforms to collect | Requires file management/rotation/mounting |

Example:

```bash
docker logs api
```

versus:

```bash
docker exec api cat /app/logs/app.log
```

## Interview Point

**Docker logs are not a filesystem log browser.**

---

# 449. Why does `docker exec` fail?

## Short Interview Answer

`docker exec` requires a **running container** and a valid executable inside it.

```bash
docker exec -it <container> sh
```

If the container is stopped:

```text
Cannot exec into a stopped container
```

If the image is minimal and has no shell:

```text
exec: "bash": executable file not found
```

Try:

```bash
docker exec -it <container> sh
```

if `sh` exists.

## Common Interview Trap

Do not assume every Linux-based image contains Bash. Minimal images may not.

## Interview Point

**`exec` starts a new process inside a running container; it does not start stopped containers.**

---

# 450. Why does `docker exec -it container bash` fail in a minimal image?

## Short Interview Answer

Because the image may not contain Bash.

For example, an Alpine-based image commonly has:

```bash
sh
```

rather than:

```bash
bash
```

Try:

```bash
docker exec -it container sh
```

If even `sh` is unavailable, use another debugging method such as inspecting logs, configuration, filesystem contents through available tooling, or a separate debug container where appropriate.

## Interview Point

**Minimal images reduce attack surface but can make interactive debugging harder.**

---

# 451. A container reports "permission denied." How do you troubleshoot it?

## Short Interview Answer

I identify **what operation** is denied and then check:

- UID/GID
- file permissions
- ownership
- mount permissions
- executable bit
- security controls
- user configured by `USER`
- host filesystem permissions for bind mounts

Example:

```bash
docker exec <container> id
docker exec <container> ls -l /app/start.sh
```

Check:

```bash
docker inspect <container>
```

for the configured user.

## Common Example

A non-root container tries to write to a bind-mounted host directory owned by another UID:

```text
Permission denied
```

The correct solution is usually to align ownership/permissions rather than simply running everything as root.

## Interview Point

**Permission debugging starts with identity and ownership, not `chmod 777`.**

---

# 452. Volume data appears to be missing. What do you check?

## Short Interview Answer

I verify that I am using the **same volume**, mounted at the expected path, and that the application is actually writing there.

```bash
docker volume ls
docker volume inspect <volume>
docker inspect <container>
```

Check mounts:

```bash
docker inspect <container> \
  --format '{{json .Mounts}}'
```

## Common Causes

- different volume name
- anonymous volume
- wrong mount destination
- application writes somewhere else
- bind mount points to a different host directory
- permissions prevent writes
- volume was removed

## Interview Point

**"The volume exists" is not enough; verify the actual container-to-volume mount.**

---

# 453. How do you troubleshoot an incorrect volume mount?

## Short Interview Answer

Check:

```bash
docker inspect <container>
docker volume inspect <volume>
```

Confirm:

```text
Source
Destination
ReadOnly
Type
```

Example expected:

```text
volume: db-data
container path: /var/lib/postgresql/data
```

If a bind mount is used, verify the host path:

```bash
ls -ld /host/path
```

## Common Interview Trap

A correct volume mounted to the wrong destination can make it look like data disappeared.

## Interview Point

**Always verify both sides of the mount: source and destination.**

---

# 454. Docker host disk is full. What do you check?

## Short Interview Answer

First identify whether the problem is the filesystem, Docker's stored objects, or application data.

```bash
df -h
df -i
docker system df
```

Then inspect Docker storage and large directories.

On Linux:

```bash
sudo du -xh /var/lib/docker | sort -h | tail
```

Use carefully because Docker manages files under its data root.

## Common Causes

- unused images
- stopped containers
- unused volumes
- build cache
- container logs
- application data
- deleted files still held open by processes

## Interview Point

**Disk-full troubleshooting must distinguish byte exhaustion (`df -h`) from inode exhaustion (`df -i`).**

---

# 455. How do you check Docker storage usage?

## Short Interview Answer

Use:

```bash
docker system df
```

For detailed information:

```bash
docker system df -v
```

This helps identify space consumed by images, containers, local volumes and build cache.

## Interview Point

**Start with `docker system df` before blindly pruning resources.**

---

# 456. What are dangling images?

## Short Interview Answer

A dangling image is an image reference with no useful repository/tag reference, commonly shown as:

```text
<none>    <none>
```

They can be left behind after builds or image replacement.

Check:

```bash
docker image ls --filter dangling=true
```

Remove dangling images:

```bash
docker image prune
```

## Common Interview Trap

Dangling does not mean every untagged-looking artifact is automatically safe to delete. Understand what is referenced before cleanup in production systems.

## Interview Point

**Dangling images are untagged image artifacts, often resulting from image rebuilds.**

---

# 457. What is the difference between dangling and unused images?

## Short Interview Answer

**Dangling** images are untagged and generally not associated with a repository/tag.

**Unused** images are not currently referenced by any containers and can include tagged images.

This distinction matters when deciding how aggressive cleanup should be.

## Commands

```bash
docker image prune
```

typically targets dangling images.

```bash
docker image prune -a
```

is more aggressive and removes images not used by existing containers.

## Interview Point

**`prune -a` is broader than ordinary image pruning.**

---

# 458. How do you safely use Docker prune?

## Short Interview Answer

First inspect what is consuming space:

```bash
docker system df
docker system df -v
```

Then choose the narrowest cleanup command needed.

Examples:

```bash
docker container prune
docker image prune
docker network prune
docker volume prune
docker builder prune
```

A broad cleanup can be:

```bash
docker system prune
```

but I would use it deliberately, especially on shared or production hosts.

## Common Interview Trap

Do not casually run:

```bash
docker system prune -a --volumes
```

on a production host. Volumes may contain persistent data.

## Interview Point

**Clean up selectively and understand the lifecycle of every resource before deletion.**

---

# 459. A container has very high CPU usage. How do you investigate?

## Short Interview Answer

Start with:

```bash
docker stats
```

Then inspect processes:

```bash
docker top <container>
```

and investigate the application.

Check configured CPU limits:

```bash
docker inspect <container>
```

## Diagnostic Flow

```text
High container CPU
      ↓
Which process?
      ↓
Application behavior?
      ↓
CPU limit/throttling?
      ↓
Host contention?
      ↓
Expected load or abnormal behavior?
```

## Interview Point

**High CPU is a symptom. Identify the process and workload before changing CPU limits.**

---

# 460. A container is OOMKilled. What do you check?

## Short Interview Answer

I verify the state and memory configuration:

```bash
docker inspect <container>
docker stats
```

Check:

```text
OOMKilled
ExitCode
Memory limit
```

For example:

```bash
docker inspect <container> \
  --format 'OOMKilled={{.State.OOMKilled}} ExitCode={{.State.ExitCode}}'
```

Then determine whether:

- the application has a memory leak
- the configured limit is too low
- workload increased
- multiple processes consume memory
- host memory pressure exists

## Common Interview Trap

Exit code `137` alone does **not prove** OOM. It means the process was killed by `SIGKILL`; OOM is one common reason.

## Interview Point

**Confirm `OOMKilled` rather than inferring OOM from exit code alone.**

---

# 461. The container is slow but CPU usage is low. What could be wrong?

## Short Interview Answer

Low CPU does not mean the application is healthy.

Possible bottlenecks include:

- disk I/O
- network I/O
- database latency
- external API latency
- memory pressure
- CPU throttling
- lock/contention
- application-level waits

Useful commands:

```bash
docker stats
docker top <container>
```

Then inspect application metrics and dependencies.

## Interview Point

**Performance troubleshooting is broader than CPU troubleshooting.**

---

# 462. A container is marked unhealthy. Does Docker automatically restart it?

## Short Interview Answer

**No.** An `unhealthy` healthcheck result and a stopped container are different states.

A healthcheck reports application health:

```text
healthy
unhealthy
starting
```

A restart policy controls container restart behavior after the main process exits.

## Example

```text
Main process still running
        ↓
Healthcheck fails
        ↓
Container = unhealthy
        ↓
No automatic restart solely because of healthcheck
```

An orchestrator or external automation may choose to replace/restart an unhealthy container.

## Interview Point

**Health status is not the same as process lifecycle.**

---

# 463. DNS resolution fails inside a container. How do you troubleshoot it?

## Short Interview Answer

First verify network membership, then DNS configuration and resolution:

```bash
docker inspect <container>
```

Inside the container:

```bash
cat /etc/resolv.conf
getent hosts example.com
```

Also test connectivity to the expected DNS infrastructure where appropriate.

## Common Causes

- broken Docker DNS
- wrong network
- custom DNS configuration
- host/network connectivity issue
- DNS server unavailable
- VPN/network interaction
- application-specific resolver behavior

## Interview Point

**Test name resolution separately from TCP connectivity.**

---

# 464. The container cannot reach another container. What do you check?

## Short Interview Answer

I verify both containers are attached to a common Docker network.

```bash
docker network inspect <network>
```

Then test by service/container name:

```bash
getent hosts <container-name>
```

and TCP:

```bash
nc -vz <container-name> <port>
```

## Common Causes

- containers are on different networks
- wrong service/container name
- application not listening
- application listening only on localhost
- wrong port
- network policy/firewall behavior
- application startup dependency

## Interview Point

**Same host does not automatically mean same Docker network.**

---

# 465. What is the difference between connection refused and connection timed out?

## Short Interview Answer

### Connection refused

The connection attempt reached the destination path sufficiently for a refusal, commonly because nothing is listening on that port or the connection is actively rejected.

### Connection timed out

No response arrived within the expected time. Possible causes include routing problems, filtering, firewall/security rules, unreachable destination or a silently dropped packet.

## Diagnostic Commands

```bash
nc -vz host port
curl -v http://host:port
```

## Interview Point

**Refused often points toward a listener/application problem; timeout often points toward reachability or filtering, but neither message alone proves the root cause.**

---

# 466. DNS works, but the application is unreachable. What next?

## Short Interview Answer

If:

```bash
getent hosts service
```

works, DNS is probably functioning.

Next test TCP:

```bash
nc -vz service 8080
```

Then application protocol:

```bash
curl -v http://service:8080
```

Finally verify the application listener:

```bash
ss -lntp
```

## Diagnostic Chain

```text
DNS
 ↓
TCP
 ↓
Application protocol
 ↓
Application behavior
```

## Interview Point

**Successful DNS resolution proves naming works, not that the application is healthy.**

---

# 467. A container can reach an internal service but cannot reach the Internet. Why?

## Short Interview Answer

I check the container's default route, Docker network gateway, host forwarding/NAT and external network controls.

Inside the container:

```bash
ip route
```

Check Docker network:

```bash
docker network inspect <network>
```

Then distinguish:

```text
DNS failure
vs
routing failure
vs
NAT/firewall failure
```

Test:

```bash
getent hosts example.com
curl -v https://example.com
```

## Common Causes

- missing/default route issue
- host firewall
- NAT/forwarding issue
- upstream firewall
- DNS works but HTTPS blocked
- corporate/VPN network restrictions

## Interview Point

**Internal connectivity and Internet egress are separate paths.**

---

# 468. The host cannot reach the container. How do you troubleshoot?

## Short Interview Answer

First determine whether the application is published.

```bash
docker ps
docker port <container>
```

If using:

```bash
-p 8080:80
```

test:

```bash
curl http://127.0.0.1:8080
```

Then check:

- application listener
- port mapping
- host firewall
- Docker networking
- application bind address

## Common Interview Trap

A container port being `EXPOSE`d does not make it reachable from the host.

## Interview Point

**Verify the host-side published port rather than only the container port.**

---

# 469. The container cannot reach the host. What do you check?

## Short Interview Answer

I first identify the Docker network and host gateway behavior rather than assuming that container `localhost` means host localhost.

Inside the container:

```bash
ip route
```

On a Docker bridge network, the gateway is often associated with the host-side bridge interface.

The exact method depends on platform/network mode.

## Important Rule

Inside a container:

```text
127.0.0.1
```

means the **container itself**, not the host.

## Interview Point

**Container localhost and host localhost are different network namespaces.**

---

# 470. Docker daemon is not running. How do you troubleshoot it?

## Short Interview Answer

On a systemd-based Linux host:

```bash
sudo systemctl status docker
sudo systemctl restart docker
sudo journalctl -u docker --no-pager
```

Also check:

```bash
docker info
```

if the client can communicate with the daemon.

## Possible Causes

- daemon crash
- configuration error
- storage problem
- dependency failure
- permission/system issue
- disk full
- daemon startup configuration problem

## Common Interview Trap

Do not immediately reinstall Docker. Check daemon logs first.

## Interview Point

**`journalctl -u docker` is a key Linux diagnostic source for Docker daemon failures.**

---

# 471. Where do you check Docker daemon logs on Linux?

## Short Interview Answer

For a systemd-managed Docker daemon:

```bash
sudo journalctl -u docker
```

Useful variants:

```bash
sudo journalctl -u docker -n 100
sudo journalctl -u docker -f
```

You can also check service status:

```bash
sudo systemctl status docker
```

## Interview Point

**Container logs and daemon logs answer different questions.**

---

# 472. Docker socket permission is denied. What does it mean?

## Short Interview Answer

The Docker CLI is trying to access the Docker daemon socket, commonly:

```text
/var/run/docker.sock
```

and the current user lacks permission.

Check:

```bash
ls -l /var/run/docker.sock
groups
```

On many Linux installations, authorized users are granted access through the `docker` group.

## Security Warning

Access to the Docker daemon socket is highly privileged. Giving a user Docker access should be treated as a significant privilege decision.

## Common Interview Trap

Adding users to the `docker` group is not equivalent to granting harmless application access.

## Interview Point

**Docker socket access can effectively provide host-level control.**

---

# 473. How do you troubleshoot rootless Docker?

## Short Interview Answer

I first confirm that Docker is actually running in rootless mode, then inspect the user-level service, environment and rootless prerequisites.

Useful commands may include:

```bash
docker info
systemctl --user status docker
journalctl --user -u docker
```

The exact troubleshooting path depends on the rootless setup.

## Important Difference

Rootless Docker changes daemon/container privileges and networking/storage behavior compared with a traditional rootful daemon.

## Interview Point

**Do not apply rootful Docker troubleshooting assumptions blindly to rootless Docker.**

---

# 474. A Docker build fails. How do you troubleshoot it?

## Short Interview Answer

I identify the exact failing build step and inspect:

- Dockerfile instruction
- build context
- base image
- package/network access
- permissions
- build arguments
- secrets
- architecture/platform
- cache behavior

Run with detailed output:

```bash
docker build --progress=plain -t myapp .
```

If necessary:

```bash
docker build --no-cache --progress=plain -t myapp .
```

## Interview Point

**Find the first failing build instruction; later errors are often consequences.**

---

# 475. Why is a Docker build context unexpectedly large?

## Short Interview Answer

The directory supplied as the build context may contain unnecessary files.

Check:

```bash
du -sh .
```

and review `.dockerignore`.

Example:

```dockerignore
.git
node_modules
*.log
tmp
coverage
.env
```

## Important Point

`.dockerignore` reduces what is sent into the build context. It is also important for preventing accidental inclusion of sensitive files, but it should not be treated as the primary secret-management mechanism.

## Interview Point

**A small build context improves build performance and reduces accidental input to the build.**

---

# 476. Docker build cache is not working. Why?

## Short Interview Answer

I look for the first instruction where the cache stops matching.

Common causes:

- frequently changing files copied too early
- changed build arguments
- changed base image
- changed package manifests
- intentionally disabled cache
- build context changed
- different builder/environment

Example optimization:

```dockerfile
COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build
```

instead of copying the entire source tree before dependency installation.

## Interview Point

**Cache optimization is primarily about arranging stable inputs before volatile inputs.**

---

# 477. BuildKit or buildx fails. How do you approach it?

## Short Interview Answer

I first determine whether the problem is:

- Dockerfile/build definition
- builder configuration
- platform emulation
- registry access
- cache
- BuildKit itself

Check:

```bash
docker buildx ls
docker buildx inspect
docker buildx version
```

For a specific builder:

```bash
docker buildx inspect <builder>
```

## Interview Point

**Separate a Dockerfile failure from a builder/platform failure.**

---

# 478. The image architecture does not match the host. What happens?

## Short Interview Answer

An image built for one architecture may not execute natively on another.

For example:

```text
linux/amd64
linux/arm64
```

Check the image/platform:

```bash
docker image inspect <image>
```

For multi-platform images, Docker can select an appropriate platform variant.

For explicit selection:

```bash
docker run --platform linux/amd64 <image>
```

Emulation may be required and can have performance implications.

## Interview Point

**"Image exists" does not guarantee "image is executable on this CPU architecture."**

---

# 479. What does `exec format error` usually indicate?

## Short Interview Answer

A common cause is an executable built for the wrong CPU architecture.

Example:

```text
amd64 binary
```

running on:

```text
arm64 host
```

can result in:

```text
exec format error
```

Other malformed executable/entrypoint cases are possible, but architecture mismatch is a key interview answer.

## Troubleshooting

Check:

```bash
uname -m
docker image inspect <image>
```

and the image manifest/platform.

## Interview Point

**Architecture mismatch is a classic "works on one machine, fails on another" container problem.**

---

# 480. How do you debug a minimal container that has no shell?

## Short Interview Answer

Do not assume that adding Bash to production is the best fix.

Use external evidence first:

```bash
docker logs <container>
docker inspect <container>
docker stats
docker top <container>
```

For networking, use a separate debug container attached to the same network where appropriate.

For filesystem/configuration problems, inspect the image/build artifacts or use a purpose-built debug image.

## Why?

Minimal images intentionally omit unnecessary tools to reduce size and attack surface.

## Interview Point

**Production image minimalism and debugging convenience are separate concerns.**

---

# 481. When do you use `docker inspect`, `docker logs`, `docker stats`, and `docker top`?

| Command | Primary purpose |
|---|---|
| `docker inspect` | Configuration + metadata + state |
| `docker logs` | Container logging stream |
| `docker stats` | Live resource usage |
| `docker top` | Processes running inside container |

### Example

```bash
docker inspect api
docker logs api
docker stats api
docker top api
```

## Interview Memory Map

```text
inspect → What is configured?
logs    → What did it say?
stats   → What resources is it using?
top     → What processes are running?
```

---

# 482. What is your production Docker troubleshooting workflow?

## Short Interview Answer

I follow a controlled sequence:

```text
1. Define symptom
2. Check container state
3. Check logs and exit code
4. Inspect configuration
5. Check health
6. Check processes/resources
7. Check network
8. Check storage
9. Check Docker daemon/host
10. Check application/dependencies
11. Make minimal change
12. Verify recovery
13. Record root cause
```

## Why This Works

It avoids jumping randomly between application, Docker and infrastructure layers.

## Interview Point

**A structured troubleshooting process is more valuable than memorizing isolated commands.**

---

# 483. Scenario: Application is unreachable from users, but the container is running. What do you do?

## Interview Answer

I would trace the request path:

```text
User
 ↓
DNS
 ↓
Load balancer / host
 ↓
Published port
 ↓
Docker network
 ↓
Container port
 ↓
Application listener
```

Then test each layer.

### Inside container

```bash
ss -lntp
curl http://127.0.0.1:<app-port>
```

### Docker host

```bash
docker port <container>
curl http://127.0.0.1:<published-port>
```

### Container networking

```bash
docker inspect <container>
docker network inspect <network>
```

### External path

Check firewall/security group/load balancer configuration as applicable.

## Common Root Causes

- application bound to `127.0.0.1` inside container
- wrong port mapping
- missing published port
- host firewall
- cloud firewall/security group
- wrong load balancer target port
- unhealthy application

## Interview Point

**Trace the packet path layer by layer instead of assuming Docker is the problem.**

---

# 484. Scenario: Container is repeatedly OOMKilled. How do you troubleshoot it?

## Interview Answer

First confirm:

```bash
docker inspect <container> \
  --format 'OOMKilled={{.State.OOMKilled}} ExitCode={{.State.ExitCode}}'
```

Then inspect usage:

```bash
docker stats <container>
```

I compare actual memory usage with the configured limit.

Then determine whether the cause is:

```text
Memory leak?
Large workload?
Too-low container limit?
Unexpected traffic?
Multiple processes?
Host memory pressure?
```

I would fix the underlying cause and then validate the memory limit with realistic load.

## Common Interview Trap

Simply increasing memory indefinitely may hide an application leak.

---

# 485. Scenario: Docker host disk is full. What do you do?

## Interview Answer

First:

```bash
df -h
df -i
docker system df -v
```

Then identify the consumer:

```text
Docker images?
Containers?
Volumes?
Build cache?
Logs?
Application data?
Other host directories?
```

Use targeted cleanup:

```bash
docker image prune
docker container prune
docker builder prune
```

For volumes:

```bash
docker volume prune
```

only after verifying that unused volumes do not contain required data.

## Interview Point

**Do not start with `docker system prune -a --volumes` on production.**

---

# 486. Scenario: Production suddenly cannot pull an image. What do you check?

## Interview Answer

I verify:

1. exact image reference
2. tag/digest existence
3. registry reachability
4. authentication
5. authorization
6. registry rate/policy limits
7. TLS/certificate issues
8. platform compatibility

Commands:

```bash
docker login <registry>
docker pull <registry>/<repo>:<tag>
```

If the deployment references a digest, verify that the digest exists and is accessible.

## Interview Point

**A failed pull is not automatically a network problem.**

---

# 487. Scenario: Container starts and immediately exits in production, but works locally. What do you check?

## Interview Answer

I compare the runtime environments.

```text
Image
Command/entrypoint
Environment variables
Secrets/configuration
Mounted files
User/permissions
Architecture
Network dependencies
Resource limits
```

Commands:

```bash
docker logs <container>
docker inspect <container>
```

I compare the effective production configuration against the local working configuration.

## Common Root Causes

- missing environment variable
- missing secret
- wrong database endpoint
- permission difference
- architecture mismatch
- missing mounted configuration
- different startup command

## Interview Point

**"Works locally" means the application worked under a different environment; compare the environments systematically.**

---

# 488. What do you check when an application works locally but fails in production?

## Short Interview Answer

I compare the complete runtime contract:

```text
Same image?
Same architecture?
Same environment?
Same secrets/config?
Same ports?
Same DNS?
Same dependencies?
Same user/permissions?
Same resource constraints?
Same network path?
```

I avoid changing multiple things at once.

## Interview Point

**Containerization standardizes the image, but production behavior still depends on runtime configuration and infrastructure.**

---

# 489. Give me a practical Docker troubleshooting checklist.

## Checklist

### Container

```bash
docker ps -a
docker inspect <container>
docker logs <container>
```

### Process

```bash
docker top <container>
```

### Resources

```bash
docker stats
```

### Network

```bash
docker network ls
docker network inspect <network>
docker port <container>
```

Inside container:

```bash
ip addr
ip route
cat /etc/resolv.conf
getent hosts <name>
ss -lntp
```

### Storage

```bash
docker system df -v
docker volume ls
docker volume inspect <volume>
df -h
df -i
```

### Daemon

```bash
sudo systemctl status docker
sudo journalctl -u docker -n 100
```

### Build

```bash
docker build --progress=plain .
docker buildx ls
```

## Interview Point

**Use the checklist according to the symptom; do not execute every command blindly.**

---

# 490. Give a strong interview answer for a Docker troubleshooting scenario.

## Final Interview Answer

> "I first define the exact symptom and identify whether the problem is at the container, application, network, storage, resource, Docker daemon, or host layer. I check `docker ps -a`, then `docker logs` and the container's exit code. Next I use `docker inspect` to verify the effective configuration, mounts, networking and restart policy. If the issue is performance or instability, I check `docker stats` and `docker top`. For connectivity issues, I verify network membership, DNS, routes, listeners and published ports, testing DNS, TCP and application protocol separately. For storage issues, I inspect mounts and Docker/host disk usage. If Docker itself appears unhealthy, I check the daemon status and `journalctl -u docker`. Finally, I compare the production runtime configuration with the known-good environment, make the smallest corrective change, verify recovery, and document the root cause."

## Why This Is Strong

It demonstrates:

- structured thinking
- Linux knowledge
- Docker knowledge
- networking knowledge
- storage knowledge
- resource troubleshooting
- production discipline

## High-Value Interview Trap

Avoid answers such as:

> "I restart Docker and see if it works."

A restart may be appropriate as a recovery action, but it is **not a troubleshooting methodology**.

---

# Quick Revision

| Area | First commands/checks |
|---|---|
| Container state | `docker ps -a` |
| Logs | `docker logs` |
| Configuration | `docker inspect` |
| Processes | `docker top` |
| Resources | `docker stats` |
| Ports | `docker port` |
| Networks | `docker network inspect` |
| DNS | `getent hosts` |
| TCP | `nc -vz` |
| Application | `curl -v` |
| Listener | `ss -lntp` |
| Storage | `docker system df -v` |
| Volumes | `docker volume inspect` |
| Host disk | `df -h`, `df -i` |
| Daemon | `systemctl status docker` |
| Daemon logs | `journalctl -u docker` |
| Build debugging | `--progress=plain` |
| Builder | `docker buildx ls` |

---

# High-Value Interview Traps

### 1. Running container ≠ healthy application

```text
Container running
       ≠
Application healthy
```

### 2. Exit 137 ≠ automatically OOM

```text
137 = 128 + SIGKILL(9)
```

OOM is a common cause, but verify:

```text
OOMKilled=true
```

### 3. `EXPOSE` ≠ published port

```text
EXPOSE 8080
```

does not publish the port.

Publishing requires:

```bash
docker run -p 8080:8080 image
```

### 4. DNS success ≠ application success

```text
DNS → TCP → HTTP/application
```

Test each layer separately.

### 5. `docker logs` ≠ every log file

It primarily exposes the container's configured logging stream.

### 6. `localhost` inside container ≠ host

```text
container localhost
        ≠
host localhost
```

### 7. `docker restart` ≠ recreate

Restart:

```text
same container
same ID
same writable layer
```

Recreate:

```text
new container
new ID
new writable layer
```

### 8. `docker system prune` needs care

Especially:

```bash
docker system prune -a --volumes
```

can remove resources you still need.

### 9. Minimal image ≠ easy debugging

Minimal images intentionally omit tools such as Bash, curl, ping, etc.

### 10. Restart loop ≠ restart-policy problem

The restart policy may only be exposing an underlying application failure.

---

# Interview Follow-Up Questions

An interviewer can extend Docker troubleshooting with:

1. What is the first command you run when a container fails?
2. How do you distinguish an application crash from a Docker problem?
3. Why can a running container still be unreachable?
4. How would you diagnose `connection refused`?
5. How would you diagnose a timeout?
6. How do you verify whether DNS is working?
7. How do you find which process is consuming CPU?
8. How do you confirm OOMKilled?
9. How do you investigate Docker disk usage?
10. How do you troubleshoot a port conflict?
11. How do you troubleshoot a container with no shell?
12. How do you troubleshoot a Docker daemon failure?
13. What is the difference between container logs and daemon logs?
14. What would you compare between local and production?
15. What would you do if the container keeps restarting?
16. What would you check if `docker exec` fails?
17. How would you investigate a permission denied error?
18. How would you safely clean disk space?
19. How would you troubleshoot `exec format error`?
20. How would you debug an application that is healthy internally but unreachable externally?

---

# Final Interview Answer

If asked:

> **"How do you troubleshoot Docker issues in production?"**

Answer:

> "I troubleshoot from the symptom toward the underlying layer. I start with container state, logs and exit code, then inspect configuration and health status. For performance issues I check processes and resource usage. For connectivity issues I trace DNS, routing, TCP connectivity, application listeners and published ports separately. For storage issues I verify mounts and disk usage. If the Docker service itself is involved, I check daemon status and daemon logs. I also compare the production runtime configuration with the known-good environment. I make the smallest corrective change possible, verify the result, and document the root cause."

---

# One-Line Memory Map

```text
Symptom
  ↓
State
  ↓
Logs
  ↓
Exit Code
  ↓
Inspect
  ↓
Process / Resources
  ↓
Network
  ↓
Storage
  ↓
Daemon / Host
  ↓
Application / Dependencies
  ↓
Fix → Verify → Root Cause
```

---

# Topic 17 Complete

**Questions covered: Q438–Q490**

**Core skill:**  
> **Don't guess. Isolate the failing layer, collect evidence, test one hypothesis at a time, fix the root cause, and verify recovery.**
