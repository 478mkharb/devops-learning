# Docker Interview Preparation — Topic 11: Docker Networking Troubleshooting

> **Purpose:** Learn how to systematically diagnose Docker networking failures instead of guessing.
>
> **Scope:** This topic focuses on practical connectivity troubleshooting: DNS, ports, interfaces, routes, network membership, published ports, container reachability, and host connectivity.

---

## 241. How do you troubleshoot a Docker container that cannot reach another container?

### Short Interview Answer

I troubleshoot from the bottom up:

```text
1. Is both container running?
2. Are they on the same Docker network?
3. Does DNS resolve the target name?
4. Is the target application listening?
5. Is the correct container port being used?
6. Are network/firewall rules blocking traffic?
```

### Example

Check containers:

```bash
docker ps
```

Inspect network:

```bash
docker network inspect app-net
```

Test DNS from the application container:

```bash
docker exec app getent hosts db
```

Test connectivity:

```bash
docker exec app nc -vz db 5432
```

if `nc` is available.

### Troubleshooting Flow

```text
Container running?
      ↓ yes
Same network?
      ↓ yes
DNS resolves?
      ↓ yes
Service listening?
      ↓ yes
Correct port?
      ↓ yes
Firewall/routing?
```

### Common Interview Trap

Do not immediately blame Docker DNS.

First verify basic network membership and whether the target process is actually listening.

### Interview Point

**Troubleshoot container-to-container connectivity layer by layer.**

---

## 242. How do you check which Docker networks a container is connected to?

### Short Interview Answer

Use:

```bash
docker inspect <container>
```

or inspect the network itself:

```bash
docker network inspect <network>
```

### Example

```bash
docker inspect app
```

Look at the container's network configuration.

Or:

```bash
docker network inspect app-net
```

to see connected containers.

### Why It Matters

A very common failure is:

```text
app → backend
```

but:

```text
app ∉ backend-network
backend ∈ backend-network
```

Therefore the expected connectivity does not exist.

### Common Interview Trap

A container having a Docker network interface does not mean it is connected to every Docker network.

### Interview Point

**First verify network membership before troubleshooting higher layers.**

---

## 243. How do you check a container's IP address?

### Short Interview Answer

Use `docker inspect`.

### Example

```bash
docker inspect \
  -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' \
  app
```

Or inspect the complete configuration:

```bash
docker inspect app
```

### Important Caveat

The IP is generally dynamic.

A recreated container may receive a different IP.

### Better Application Practice

Use:

```text
db:5432
```

rather than:

```text
172.20.0.5:5432
```

when Docker DNS/service naming is available.

### Interview Point

**Inspect IPs for troubleshooting; don't normally hard-code them into application configuration.**

---

## 244. How do you test DNS resolution from inside a container?

### Short Interview Answer

Use tools such as:

```bash
getent hosts <name>
```

or:

```bash
nslookup <name>
```

or:

```bash
dig <name>
```

depending on which utilities are installed.

### Example

```bash
docker exec app getent hosts db
```

Expected conceptual result:

```text
db → container IP
```

### If DNS Fails

Check:

```bash
docker network inspect app-net
```

and:

```bash
docker inspect app
```

Then verify that:

- both containers are attached to the appropriate network
- the name is correct
- the application is using the expected DNS configuration
- the network configuration is healthy

### Common Interview Trap

`ping` is not a pure DNS test.

A failed ping can occur because ICMP is unavailable/blocked even when DNS resolution works.

### Interview Point

**Separate name resolution from actual application connectivity.**

---

## 245. What if DNS resolves but the application is still unreachable?

### Short Interview Answer

Then DNS is probably not the primary problem. Check whether the target application is listening on the expected interface and port, then test TCP connectivity and inspect application/container configuration.

### Example

Suppose:

```text
db
```

resolves successfully:

```bash
getent hosts db
```

but:

```bash
nc -vz db 5432
```

fails.

Now check the database container:

```bash
docker exec db ss -lntp
```

if `ss` is available.

### Important Check

The service must listen on an interface reachable from the container network.

For example:

```text
127.0.0.1:5432
```

is different from:

```text
0.0.0.0:5432
```

The first is loopback-only within that network namespace.

### Common Interview Trap

> "If DNS works, networking is working."

No.

DNS is only one layer.

### Interview Point

**DNS success proves name resolution, not application reachability.**

---

## 246. How do you verify whether an application is listening on the expected port?

### Short Interview Answer

Inspect the listening sockets inside the container.

### Example

```bash
docker exec app ss -lntp
```

or:

```bash
docker exec app netstat -lntp
```

if available.

### Look For

If the application should listen on:

```text
8080
```

verify something like:

```text
0.0.0.0:8080
```

or the container's appropriate interface/address.

### Problem Example

Application listens on:

```text
127.0.0.1:8080
```

Another container tries:

```text
app:8080
```

and cannot connect.

### Why?

The service is bound only to its own loopback interface.

### Interview Point

**Check the actual listening address and port, not just the Dockerfile's `EXPOSE`.**

---

## 247. How do you distinguish a DNS problem from a TCP connectivity problem?

### Short Interview Answer

Test them independently.

### DNS Test

```bash
getent hosts db
```

If this returns an IP:

```text
DNS/name resolution works
```

### TCP Test

```bash
nc -vz db 5432
```

If this succeeds:

```text
TCP connection works
```

### Application Test

For HTTP:

```bash
curl -v http://web:8080/health
```

### Layered Model

```text
DNS
 ↓
TCP
 ↓
HTTP/application
```

A failure at a higher layer does not automatically mean a lower layer is broken.

### Common Interview Trap

Do not use only:

```bash
ping
```

to diagnose an HTTP or database problem.

### Interview Point

**Test DNS, then TCP, then the application protocol.**

---

## 248. Why might `ping` fail even though the application works?

### Short Interview Answer

`ping` uses ICMP, while the application may use TCP or UDP. ICMP can be unavailable or blocked even when application traffic works.

### Example

```bash
ping db
```

fails.

But:

```bash
nc -vz db 5432
```

succeeds.

This means:

```text
ICMP path/test → failed
TCP 5432       → working
```

### Why It Matters

A failed ping does not prove that all network connectivity is broken.

### Common Interview Trap

> "If ping fails, Docker networking is broken."

False.

### Interview Point

**Use a test matching the protocol you actually need.**

---

## 249. How do you troubleshoot `connection refused`?

### Short Interview Answer

`connection refused` usually means the destination was reachable at the network level, but no process accepted the connection on that address/port, or the host actively rejected it.

### Check

```bash
docker exec target ss -lntp
```

Then verify:

```text
correct port?
correct bind address?
application running?
container healthy?
```

### Example

Client:

```bash
curl http://web:8080
```

returns:

```text
connection refused
```

Possible cause:

```text
web container running
BUT
application not listening on 8080
```

### Common Interview Trap

Do not immediately interpret "connection refused" as a DNS failure.

DNS and TCP connection establishment are different stages.

### Interview Point

**Refused usually means you reached the destination but nothing accepted that connection on the requested endpoint.**

---

## 250. How do you troubleshoot `connection timed out`?

### Short Interview Answer

A timeout often indicates that the connection attempt is not receiving the expected response, potentially because of routing, firewall rules, security controls, incorrect addressing, or an unavailable path.

### Check

```text
Is destination correct?
Is container/network reachable?
Is route present?
Are firewall rules blocking traffic?
Is service listening?
Is traffic crossing a host/network boundary?
```

### Useful Commands

```bash
docker inspect
docker network inspect
ip route
ss -lntp
```

and appropriate connectivity tests.

### Important Distinction

```text
connection refused
    ↓
destination reachable, active rejection/no listener likely

connection timed out
    ↓
no timely response
```

These are not identical failures.

### Common Interview Trap

A timeout does not automatically mean the application is down.

### Interview Point

**Timeout = investigate the path and filtering before assuming the process is dead.**

---

## 251. What is the difference between `connection refused` and `connection timed out`?

### Short Interview Answer

They indicate different failure patterns.

| Error | Typical Interpretation |
|---|---|
| Connection refused | Destination reachable but connection rejected/no listener |
| Connection timed out | No timely response; path/filtering/availability issue possible |

### Example

```text
app → db:5432
```

Refused:

```text
db reachable
5432 not accepting connection
```

Timeout:

```text
traffic may not be reaching db
or response may be blocked
```

### Important Caveat

These are diagnostic clues, not absolute proofs of one specific root cause.

### Interview Point

**Use the error type to narrow the investigation, not to skip investigation.**

---

## 252. How do you troubleshoot a published Docker port that cannot be reached from the host?

### Short Interview Answer

Check the container's state, application listening port, port-publishing configuration, host binding, and host firewall/network configuration.

### Step 1

```bash
docker ps
```

Verify the container is running.

### Step 2

```bash
docker port web
```

Check the published mapping.

Example:

```text
80/tcp → 0.0.0.0:8080
```

### Step 3

Check the application:

```bash
docker exec web ss -lntp
```

### Step 4

Test from host:

```bash
curl -v http://127.0.0.1:8080
```

### Step 5

Inspect:

```bash
docker inspect web
```

### Step 6

Check host firewall and routing if necessary.

### Troubleshooting Model

```text
Host:8080
   ↓
Docker port publication
   ↓
Container:80
   ↓
Application listening on 80?
```

### Common Interview Trap

Do not assume that:

```dockerfile
EXPOSE 80
```

means:

```text
host:80 is reachable
```

### Interview Point

**Trace the published-port path from host socket to application listener.**

---

## 253. What does `docker port` do?

### Short Interview Answer

`docker port` displays the published port mappings for a container.

### Example

```bash
docker port web
```

Possible output:

```text
80/tcp -> 0.0.0.0:8080
```

### Why Useful?

It quickly answers:

```text
Which host port maps to which container port?
```

### Common Interview Trap

`docker port` shows published mappings; it does not prove the application is actually listening and healthy.

### Interview Point

**Port mapping exists ≠ application is healthy.**

---

## 254. How do you troubleshoot container-to-host connectivity?

### Short Interview Answer

First determine what "host" means and how the container is expected to reach it, then inspect routing, host addresses, platform-specific gateway behavior, firewall rules, and the target service's listening interface.

### Important Concept

Inside a normal Linux bridge-networked container:

```text
127.0.0.1
```

means the container itself.

It does not mean the host.

### Possible Approaches

Depending on the platform/configuration, the host can be reached through:

```text
host-gateway mapping
Docker-provided host name where supported/configured
host network mode
explicit host/gateway address
```

### Example

A common Linux Docker pattern is:

```bash
docker run \
  --add-host host.docker.internal:host-gateway \
  myapp
```

Then the application can use:

```text
host.docker.internal
```

where this mapping is configured and supported.

### Common Interview Trap

Do not claim one universal host address works identically on Linux, Docker Desktop, Windows, and macOS.

### Interview Point

**Host reachability is platform/configuration dependent; do not confuse host localhost with container localhost.**

---

## 255. How do you troubleshoot container-to-internet connectivity?

### Short Interview Answer

Check the container's network configuration, default route, DNS resolution, host connectivity, NAT/firewall rules, and any proxy requirements.

### Step 1 — DNS

```bash
docker exec app getent hosts example.com
```

If DNS fails, investigate DNS first.

### Step 2 — Route

```bash
docker exec app ip route
```

Look for a default route.

### Step 3 — Application-Level Test

```bash
docker exec app curl -v https://example.com
```

if `curl` is installed.

### Step 4 — Host

Verify the Docker host itself can reach the internet.

### Step 5 — NAT/Firewall

On Linux, inspect host networking/firewall configuration if required.

### Conceptual Path

```text
Container
   ↓
container interface
   ↓
Docker bridge
   ↓
host routing/NAT
   ↓
external network
```

### Common Interview Trap

Do not troubleshoot DNS only when the actual problem is missing routing or blocked egress.

### Interview Point

**Separate DNS, routing, NAT, firewall, and application-protocol failures.**

---

## 256. How do you troubleshoot DNS failures inside Docker containers?

### Short Interview Answer

Check the container's resolver configuration, Docker network configuration, DNS resolution itself, host DNS connectivity, and any custom DNS settings.

### Commands

```bash
docker exec app cat /etc/resolv.conf
```

Then:

```bash
docker exec app getent hosts example.com
```

Inspect the network:

```bash
docker network inspect app-net
```

### Questions to Ask

```text
Can the container resolve internal names?
Can it resolve public names?
Does the host resolve the same name?
Is custom DNS configured?
Is the container attached to the expected network?
```

### Important Distinction

These are different:

```text
db
```

Docker service/container name resolution on an appropriate Docker network.

versus:

```text
example.com
```

external DNS resolution.

### Common Interview Trap

A container resolving `db` does not prove external DNS resolution is working.

### Interview Point

**Test internal Docker DNS and external DNS separately.**

---

## 257. How do you troubleshoot a container that has no network interface?

### Short Interview Answer

Inspect its network mode and network configuration.

### Example

```bash
docker inspect app
```

Check whether it uses:

```text
none
```

or another network configuration.

Inside the container:

```bash
docker exec app ip addr
```

if the tool exists.

### Possible Cause

The container may intentionally use:

```bash
--network none
```

### Another Possibility

The container may be using:

```text
host
```

network mode, where the normal isolated container interface model does not apply.

### Common Interview Trap

Do not assume that every container must have an interface called exactly `eth0`.

### Interview Point

**Check network mode before assuming the interface is broken.**

---

## 258. How do you troubleshoot a container connected to the wrong network?

### Short Interview Answer

Inspect network membership, disconnect it from the incorrect network if appropriate, and connect it to the correct network.

### Commands

```bash
docker inspect app
```

Then:

```bash
docker network ls
docker network inspect backend
```

Connect:

```bash
docker network connect backend app
```

Disconnect:

```bash
docker network disconnect wrong-net app
```

### Example

```text
app
 ├── wrong-net
 └── frontend
```

but it should be:

```text
app
 ├── frontend
 └── backend
```

### Important Point

A container can belong to multiple networks, so simply adding the correct network may not be sufficient if the incorrect network creates unwanted connectivity.

### Interview Point

**Fix both missing connectivity and unintended connectivity.**

---

## 259. How do you troubleshoot intermittent container connectivity?

### Short Interview Answer

Look for container recreation, changing IP addresses, service restarts, DNS behavior, resource exhaustion, network saturation, connection limits, and external dependencies.

### Important Question

Is the application using:

```text
hard-coded container IP
```

instead of:

```text
service/container name
```

If containers are recreated:

```text
old IP
  ↓
container removed
  ↓
new container
  ↓
new IP
```

Hard-coded addressing can break.

### Other Checks

```bash
docker events
docker inspect
docker stats
docker logs
docker network inspect
```

### Common Interview Trap

Do not assume intermittent connectivity is always a Docker networking bug.

Application timeouts, resource pressure, DNS behavior, dependency failures, and external network problems can produce similar symptoms.

### Interview Point

**For intermittent failures, correlate network symptoms with container lifecycle and application events.**

---

## 260. How do you troubleshoot a container that works by IP but not by name?

### Short Interview Answer

That strongly suggests a name-resolution problem rather than basic IP connectivity.

### Test

```bash
docker exec app getent hosts db
```

If:

```text
db → no result
```

but:

```bash
docker exec app nc -vz 172.x.x.x 5432
```

works, investigate DNS/network membership.

### Check

```bash
docker network inspect app-net
```

Verify:

```text
app connected
db connected
```

### Common Causes

- containers are not on the same appropriate network
- incorrect name
- DNS configuration issue
- container recreated/network topology changed

### Common Interview Trap

Do not "fix" this by permanently hard-coding the IP.

### Interview Point

**IP works + name fails = investigate name resolution first.**

---

## 261. How do you troubleshoot a container that works internally but not from outside the host?

### Short Interview Answer

Check whether the service is published, which host address it is bound to, the host firewall, upstream network/security controls, and whether the application is listening correctly.

### Example

Container works:

```bash
docker exec web curl http://127.0.0.1:80
```

Host works:

```bash
curl http://127.0.0.1:8080
```

External client fails:

```text
remote-client → host:8080 → timeout
```

Now investigate:

```text
host binding
firewall
security group/network ACL if cloud-hosted
routing
upstream load balancer
```

### Important Distinction

A Docker networking problem and a cloud/network perimeter problem may look identical from the application client.

### Interview Point

**Trace the request from external client → host → Docker publication → container → application.**

---

## 262. How do you troubleshoot a Docker network where the subnet conflicts with another network?

### Short Interview Answer

Inspect the network subnet and compare it with the host, VPN, LAN, or other Docker networks. Use a non-overlapping subnet if a conflict exists.

### Inspect

```bash
docker network inspect app-net
```

Look for:

```text
Subnet
Gateway
```

### Example

Docker network:

```text
172.20.0.0/16
```

Corporate VPN:

```text
172.20.0.0/16
```

Now routing becomes ambiguous.

### Better Design

Choose non-overlapping ranges, for example:

```text
Docker network → 172.30.0.0/16
VPN            → 172.20.0.0/16
```

### Common Interview Trap

Subnet conflicts are not solved by changing the container's individual IP manually.

The network addressing design needs to be corrected.

### Interview Point

**Docker subnets must not unnecessarily overlap with networks the host needs to reach.**

---

## 263. What commands are most useful for Docker networking troubleshooting?

### Short Interview Answer

I use Docker-level inspection commands plus Linux networking tools.

### Docker Commands

```bash
docker network ls
docker network inspect <network>
docker inspect <container>
docker port <container>
docker exec <container> ...
docker logs <container>
```

### Linux Commands

```bash
ip addr
ip route
ss -lntp
ip neigh
```

and, when available/useful:

```bash
tcpdump
```

### DNS Tools

```bash
getent hosts
nslookup
dig
```

### Connectivity Tools

```bash
curl
nc
```

### Layered Approach

```text
Docker topology
      ↓
Linux interface/route
      ↓
DNS
      ↓
TCP
      ↓
Application protocol
```

### Common Interview Trap

Do not run random commands without knowing what hypothesis each command tests.

### Interview Point

**Every troubleshooting command should answer a specific networking question.**

---

## 264. How would you troubleshoot a Docker networking issue in an interview?

### Short Interview Answer

I would avoid guessing and demonstrate a layered troubleshooting process.

### Example Scenario

> "The application container cannot connect to PostgreSQL."

### Step 1 — Containers

```bash
docker ps -a
```

Are both containers running?

### Step 2 — Network

```bash
docker network inspect app-net
```

Are both containers attached?

### Step 3 — DNS

```bash
docker exec app getent hosts db
```

Does `db` resolve?

### Step 4 — TCP

```bash
docker exec app nc -vz db 5432
```

Can TCP connect?

### Step 5 — Database Listener

```bash
docker exec db ss -lntp
```

Is PostgreSQL actually listening?

### Step 6 — Logs

```bash
docker logs db
```

Is PostgreSQL reporting startup/auth/configuration errors?

### Step 7 — Configuration

Check:

```text
hostname
port
credentials
network
application configuration
```

### Step 8 — Host/Firewall

If traffic crosses the host or external network boundary, inspect:

```text
host firewall
routing
cloud security controls
load balancer
```

### Interview Point

**Explain your hypothesis at each step instead of listing commands without reasoning.**

---

# Quick Revision

| Symptom | First Investigation |
|---|---|
| Container cannot reach container | Network membership |
| Name doesn't resolve | DNS/network attachment |
| Name resolves but connection fails | Listener/TCP |
| `connection refused` | Service/listener/port |
| `connection timed out` | Routing/firewall/path |
| Host cannot reach published port | `docker port`, listener, host firewall |
| Container cannot reach internet | DNS → route → NAT/firewall |
| IP works but name fails | DNS |
| Container has no expected interface | Network mode |
| Wrong network | `inspect`, `network connect/disconnect` |
| Intermittent connectivity | Lifecycle/IP/DNS/resource correlation |
| External access fails | Host publication + perimeter/network |
| Subnet conflict | Inspect Docker subnet and routing |
| Need topology | `docker network inspect` |
| Need container config | `docker inspect` |
| Need listening ports | `ss -lntp` |
| Need routes | `ip route` |
| Need DNS test | `getent hosts` |
| Need HTTP test | `curl -v` |
| Need TCP test | `nc -vz` |
| Need packet-level proof | `tcpdump` |

---

# High-Value Interview Traps

### Trap 1 — "Ping failing means Docker networking is broken."

**Wrong.**

Ping uses ICMP. Your application may use TCP or UDP.

---

### Trap 2 — "DNS resolving means the service is reachable."

**Wrong.**

DNS only proves name resolution.

```text
DNS
 ↓
TCP
 ↓
Application
```

Each layer can fail independently.

---

### Trap 3 — "Connection refused means the network is completely down."

**Wrong.**

It often indicates that the destination was reachable but no service accepted the requested connection.

---

### Trap 4 — "Connection timeout means the application is definitely down."

**Wrong.**

Routing, firewalling, security controls, or other path issues can produce timeouts.

---

### Trap 5 — "Use the container IP to fix DNS."

**Wrong.**

Container IPs can change. Fix the name-resolution/network topology problem.

---

### Trap 6 — "If the container is running, the application is listening."

**Wrong.**

A container can be running while the application process has failed, is listening on another port, or is bound only to loopback.

---

### Trap 7 — "`EXPOSE` proves the port is reachable."

**Wrong.**

`EXPOSE` is not a connectivity test and does not publish a host port.

---

### Trap 8 — "If the application works inside the container, external access must work."

**Wrong.**

The failure may be in:

```text
port publishing
host firewall
routing
cloud security controls
load balancer
```

---

### Trap 9 — "All networking problems are Docker problems."

**Wrong.**

The failure may be:

```text
application
container
Docker network
Linux host
firewall
cloud network
external dependency
```

---

# Interview Follow-Up Questions

After answering Docker networking troubleshooting, an interviewer may ask:

1. How does Docker implement port publishing on Linux?
2. What iptables/nftables rules are involved?
3. How does Docker's embedded DNS work?
4. How does a veth pair move packets between namespaces?
5. How does NAT allow containers to access the internet?
6. How would you use `tcpdump` to prove where packets stop?
7. How do you troubleshoot Docker networking on an EC2 instance?
8. What happens if Docker subnet overlaps with a corporate VPN?
9. How do you restrict container-to-container traffic?
10. How would you troubleshoot a Docker Compose application where one service cannot reach another?
11. How do Docker network problems differ from Kubernetes network problems?
12. How would you diagnose a port that works locally but not from another host?

---

# Final Interview Answer

> **"For Docker networking troubleshooting, I use a layered approach rather than guessing. First I verify that the containers are running and attached to the expected Docker network. Then I test DNS resolution using tools such as `getent hosts`, followed by TCP connectivity using `nc` or the application's protocol using `curl`. If DNS works but the connection fails, I check whether the target process is actually listening on the expected address and port using `ss`. For host or external connectivity, I then trace port publishing, host routing, firewall/security controls, and any cloud network components. I also distinguish connection refused from timeout because they provide different diagnostic clues. The key is to identify exactly which layer—DNS, TCP, Docker networking, Linux networking, firewall, or application—is failing."**

---

# One-Line Memory Map

```text
DOCKER NETWORK TROUBLESHOOTING

CONTAINER
   ↓
NETWORK MEMBERSHIP
   ↓
DNS
   ↓
IP ROUTE
   ↓
TCP PORT
   ↓
APPLICATION
   ↓
HOST FIREWALL / CLOUD NETWORK
   ↓
EXTERNAL CLIENT
```

## Three Golden Tests

```bash
# 1. Can I resolve the name?
getent hosts db

# 2. Can I establish TCP?
nc -vz db 5432

# 3. Does the application protocol work?
curl -v http://web:8080/health
```

```text
Name fails
   → DNS/network membership

Name works, TCP fails
   → listener/port/routing/firewall

TCP works, application fails
   → application protocol/configuration
```
