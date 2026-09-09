# Docker Interview Preparation — Topic 10: Docker Networking Fundamentals

> **Purpose:** Build a precise mental model of Docker networking, including network namespaces, bridges, container-to-container communication, port publishing, DNS, and the major Docker network drivers.
>
> **Scope:** This topic covers networking fundamentals. Detailed networking troubleshooting, packet flow, connectivity diagnosis, and advanced failure scenarios are covered separately.

---

## 211. Why does a Docker container need a network namespace?

### Short Interview Answer

A network namespace gives a container an isolated network stack, including its own interfaces, routes, firewall rules, and network namespace-local ports.

### Conceptual Model

```text
Host Network Namespace
        │
        ├── eth0
        ├── routes
        └── ports
             │
             │ isolation
             ↓
Container Network Namespace
        ├── eth0
        ├── routes
        └── ports
```

This allows two containers to have their own network environments rather than directly sharing the host's network namespace.

### Important Point

A container's:

```bash
localhost
```

normally refers to the container's own network namespace, not the Docker host.

### Common Interview Trap

A network namespace is not the same thing as a Docker network.

```text
Network namespace → Linux isolation mechanism
Docker network    → Docker-managed networking configuration
```

### Interview Point

**Network namespace provides isolation; Docker networking connects isolated namespaces according to the selected network configuration.**

---

## 212. What are Docker network drivers?

### Short Interview Answer

Docker network drivers determine how Docker provides networking for containers.

Common drivers include:

```text
bridge
host
none
overlay
macvlan
ipvlan
```

The exact availability and behavior depends on the Docker environment.

### Typical Use

```text
bridge  → common single-host container networking
host    → container shares host network namespace
none    → no normal container networking
overlay → multi-host networking, commonly with Swarm
macvlan → container gets its own MAC address on the network
ipvlan  → IP-based networking with different L2/L3 behavior
```

### Common Interview Trap

Do not say:

> "Bridge is the only Docker network."

Docker supports multiple network drivers for different requirements.

### Interview Point

**The network driver determines the networking model used by a Docker network.**

---

## 213. What is Docker bridge networking?

### Short Interview Answer

Bridge networking connects containers on the same Docker host through a Linux bridge and virtual interfaces.

### Conceptual Flow

```text
Container A
   │
veth pair
   │
   ↓
Docker bridge
   │
veth pair
   │
   ↓
Container B
```

A typical Docker installation has a default bridge network.

### Example

```bash
docker network ls
```

You may see:

```text
bridge
host
none
```

Create a user-defined bridge:

```bash
docker network create app-net
```

Run containers on it:

```bash
docker run -d --name web --network app-net nginx
docker run -d --name app --network app-net myapp
```

### Why It Matters

Containers connected to the same appropriate bridge network can communicate using Docker's networking facilities.

### Interview Point

**Bridge networking provides container connectivity on a single Docker host.**

---

## 214. What is the difference between the default bridge and a user-defined bridge?

### Short Interview Answer

Both provide bridge networking, but user-defined bridge networks provide better isolation and built-in container-name DNS resolution.

### Default Bridge

Containers attached to the default:

```text
bridge
```

network have historically had more limited automatic name-resolution behavior than containers on user-defined bridge networks.

### User-Defined Bridge

Create:

```bash
docker network create app-net
```

Run:

```bash
docker run -d --name db --network app-net postgres
docker run -d --name app --network app-net myapp
```

The application can generally resolve:

```text
db
```

through Docker's embedded DNS.

### Why Prefer User-Defined Networks?

They provide:

- better isolation
- automatic DNS-based service discovery
- explicit network membership
- easier application topology management

### Common Interview Trap

Do not claim the default bridge and user-defined bridge are identical.

### Interview Point

**For multi-container applications, user-defined bridge networks are generally preferable to relying on the default bridge.**

---

## 215. How do containers communicate with each other on the same Docker network?

### Short Interview Answer

Containers attached to the same user-defined network can communicate using the network's virtual connectivity and Docker's embedded DNS for container/service-name resolution.

### Example

Create network:

```bash
docker network create app-net
```

Database:

```bash
docker run -d \
  --name db \
  --network app-net \
  postgres
```

Application:

```bash
docker run -d \
  --name app \
  --network app-net \
  myapp
```

The application can use:

```text
db:5432
```

instead of hard-coding the database container's IP.

### Flow

```text
app
 │
 │ DNS lookup: db
 ↓
Docker embedded DNS
 │
 ↓
db container IP
 │
 ↓
database
```

### Important Point

Both containers must share a network that provides the required connectivity.

### Common Interview Trap

Do not use the host's `localhost` to reach another container.

### Interview Point

**Same Docker network + container/service name = stable application-level connectivity.**

---

## 216. How does Docker provide DNS for containers?

### Short Interview Answer

Docker provides an embedded DNS service for containers on user-defined networks. Container/service names can resolve to the appropriate container IP addresses.

### Example

```bash
docker network create app-net

docker run -d --name redis --network app-net redis
docker run -it --rm --network app-net busybox
```

From the second container, a name lookup for:

```text
redis
```

can resolve to the Redis container's address.

### Why Useful?

Container IPs can change when containers are recreated.

Names provide a stable logical identity:

```text
redis
```

rather than:

```text
172.x.x.x
```

### Common Interview Trap

Do not say Docker DNS permanently maps a name to one fixed IP.

If a container is recreated, its IP may change while the name remains the application-level reference.

### Interview Point

**Docker DNS provides dynamic service/name resolution rather than requiring hard-coded container IPs.**

---

## 217. What is a Docker bridge?

### Short Interview Answer

A Docker bridge is a Linux software bridge used to connect container network interfaces on the Docker host.

### Conceptual Model

```text
             Linux Host
                 │
          ┌──────┴──────┐
          │ Docker      │
          │ bridge      │
          └──┬──────┬───┘
             │      │
           veth     veth
             │      │
          Container Container
```

Each container has a network interface in its network namespace, typically connected through a virtual Ethernet pair to the host-side networking infrastructure.

### Why It Matters

The bridge provides Layer-2-style connectivity between attached interfaces on the host.

Docker can additionally configure routing/NAT/firewall rules as required.

### Interview Point

**The bridge is the host-side virtual switching component connecting container interfaces.**

---

## 218. What is a veth pair?

### Short Interview Answer

A veth pair is a pair of interconnected virtual Ethernet interfaces. One end can exist in a container's network namespace and the other on the host.

### Conceptual Flow

```text
Container namespace
    eth0
      │
      │ veth pair
      │
Host namespace
    vethXXXX
      │
      ↓
Docker bridge
```

Packets entering one end appear at the other end.

### Why Docker Uses It

It provides a virtual connection between:

```text
container network namespace
```

and:

```text
host networking infrastructure
```

### Common Interview Trap

A veth pair is not itself a Docker network driver.

It is a Linux networking mechanism used to connect network namespaces.

### Interview Point

**veth pair = virtual cable between network namespaces.**

---

## 219. What is `docker0`?

### Short Interview Answer

`docker0` is the traditional Linux bridge interface created by Docker Engine for the default bridge network on many Linux installations.

### Example

On a Linux Docker host:

```bash
ip link show docker0
```

may show the Docker bridge.

### Conceptual Flow

```text
Container
   ↓
veth
   ↓
docker0
   ↓
host networking
```

### Important Caveat

Do not assume every Docker networking configuration uses `docker0`.

User-defined networks create their own bridge interfaces, and other drivers use different mechanisms.

### Common Interview Trap

> "Every Docker container always connects to docker0."

False.

A container can be attached to user-defined bridge, host, none, overlay, and other network configurations.

### Interview Point

**`docker0` is associated with Docker's default bridge networking on Linux, not with all Docker networks.**

---

## 220. What is port publishing in Docker?

### Short Interview Answer

Port publishing maps a port on the Docker host to a port in the container so traffic arriving through the published host port can reach the container service.

### Example

```bash
docker run -d -p 8080:80 nginx
```

Meaning:

```text
Host port 8080
       ↓
Container port 80
```

Then:

```text
http://host:8080
```

can reach the NGINX service listening on container port 80, subject to host/network configuration.

### Important Distinction

```text
EXPOSE 80
```

does not publish the port.

```bash
-p 8080:80
```

publishes it.

### Common Interview Trap

> "Container port 80 is automatically reachable from the host."

Not necessarily.

### Interview Point

**`-p` publishes; `EXPOSE` documents.**

---

## 221. What does `-p 8080:80` mean?

### Short Interview Answer

It maps host port `8080` to container port `80`.

### Example

```bash
docker run -d -p 8080:80 nginx
```

Conceptually:

```text
Client
  │
  │ :8080
  ↓
Docker host
  │
  │ port mapping
  ↓
Container :80
  │
  ↓
NGINX
```

### Syntax

```text
-p HOST_PORT:CONTAINER_PORT
```

You can specify an IP as well:

```bash
docker run -p 127.0.0.1:8080:80 nginx
```

This restricts the published host binding to the specified host address.

### Common Interview Trap

The order matters.

```text
-p 8080:80
```

does not mean:

```text
container 8080 → host 80
```

It means:

```text
host 8080 → container 80
```

### Interview Point

**Remember: HOST:CONTAINER.**

---

## 222. What is the difference between `EXPOSE` and `-p`?

### Short Interview Answer

`EXPOSE` documents the port an application expects to use; `-p` publishes a container port onto the host.

### Dockerfile

```dockerfile
EXPOSE 8080
```

This does not make the application externally reachable by itself.

### Runtime

```bash
docker run -p 8080:8080 myapp
```

publishes the port.

### Conceptual Model

```text
EXPOSE
   ↓
documentation/metadata

-p
   ↓
actual host port publishing
```

### Common Interview Trap

`EXPOSE` does not configure host firewall rules or create host-port publishing by itself.

### Interview Point

**EXPOSE = declaration; `-p` = publication.**

---

## 223. What happens if two containers publish the same host port?

### Short Interview Answer

Two containers generally cannot simultaneously bind the same host IP and host port combination.

### Example

First:

```bash
docker run -d --name web1 -p 8080:80 nginx
```

Then:

```bash
docker run -d --name web2 -p 8080:80 nginx
```

The second publication will normally fail because:

```text
host:8080
```

is already allocated for the relevant binding.

### Solution

Use a different host port:

```bash
docker run -d --name web2 -p 8081:80 nginx
```

Now:

```text
host:8080 → web1:80
host:8081 → web2:80
```

### Important Point

Both containers can listen on the same **container port** because they have separate network namespaces.

The conflict is the host-side published port/binding.

### Interview Point

**Container ports can be identical; conflicting host bindings cannot coexist on the same host address/port.**

---

## 224. Can containers communicate without publishing ports?

### Short Interview Answer

Yes. Containers on the same appropriate Docker network can communicate directly without publishing their ports to the host.

### Example

```bash
docker network create app-net

docker run -d \
  --name db \
  --network app-net \
  postgres
```

The database does not need:

```bash
-p 5432:5432
```

for another container on `app-net` to connect to:

```text
db:5432
```

### Why?

Publishing is primarily for making a container service reachable through a host interface.

Container-to-container communication can occur through the Docker network directly.

### Common Interview Trap

> "Every container port must be published for another container to use it."

False.

### Interview Point

**Publish ports for host/external access; use Docker networking for internal container communication.**

---

## 225. What does `--network host` do?

### Short Interview Answer

On supported platforms, host networking makes the container use the host's network namespace instead of getting a separate container network namespace.

### Example

```bash
docker run --network host nginx
```

### Conceptual Difference

Normal:

```text
Host namespace
     │
     └── isolated container namespace
             └── eth0
```

Host networking:

```text
Host network namespace
          ↑
          │
       container
```

### Consequence

The container does not have the usual isolated container IP/network stack.

The application binds directly to host networking resources.

### Common Interview Trap

With host networking, `-p` port publishing is generally unnecessary and does not provide the normal bridge-network port-mapping behavior.

### Interview Point

**Host networking trades network isolation for direct access to the host network namespace.**

---

## 226. What is the `none` network driver?

### Short Interview Answer

`none` gives the container no normal Docker-managed network connectivity.

### Example

```bash
docker run --network none alpine
```

The container still has its loopback interface, but normal external/container networking is not provided.

### Why Useful?

It can be useful when an application does not need networking or when network isolation is desired.

### Conceptual Model

```text
Container
   │
   └── loopback

No normal Docker network
```

### Common Interview Trap

`none` does not mean the container process itself disappears from the host.

It only concerns its networking configuration.

### Interview Point

**`none` provides network isolation with only minimal loopback networking.**

---

## 227. What is Docker host networking useful for?

### Short Interview Answer

Host networking can be useful for workloads that need direct access to the host's network stack or where avoiding the additional network namespace/NAT path is beneficial.

### Potential Advantages

- direct host network access
- no traditional port publishing
- potentially lower networking overhead in appropriate workloads

### Trade-Offs

- reduced network isolation
- host-port conflicts
- less portable assumptions
- platform-specific behavior

### Common Interview Trap

Do not describe host networking as "always faster."

Performance depends on workload and platform, and the security/isolation trade-off is often more important.

### Interview Point

**Host networking is a deliberate isolation trade-off, not a default optimization.**

---

## 228. What is an overlay network?

### Short Interview Answer

An overlay network provides virtual networking across multiple Docker hosts, allowing containers on different hosts to communicate as though they are connected to a common logical network.

### Conceptual Model

```text
Docker Host A                  Docker Host B
┌──────────────┐              ┌──────────────┐
│ Container A  │              │ Container B  │
└──────┬───────┘              └──────┬───────┘
       │                             │
       └──── overlay network ────────┘
```

The underlying physical/host networks carry the traffic while the overlay provides logical container connectivity.

### Common Use

Docker Swarm uses overlay networks for multi-host container networking.

### Important Caveat

Do not assume ordinary single-host Docker Compose bridge networks automatically become overlay networks.

### Interview Point

**Overlay extends container networking across multiple Docker hosts.**

---

## 229. What is the difference between bridge and overlay networking?

### Short Interview Answer

Bridge networking is primarily for containers on one Docker host; overlay networking is designed to connect containers/services across multiple Docker hosts.

### Comparison

| Feature | Bridge | Overlay |
|---|---|---|
| Typical scope | Single host | Multiple hosts |
| Common use | Local containers | Swarm/multi-host |
| Network isolation | Host-level | Cluster/service-level |
| Virtual connectivity | Linux bridge | Overlay mechanism |

### Example

Bridge:

```text
Host
 ├── container A
 └── container B
```

Overlay:

```text
Host A                    Host B
 └── container A  ←────→  container B
```

### Common Interview Trap

Do not claim overlay is simply "a faster bridge."

It solves a different topology problem.

### Interview Point

**Bridge = single-host; overlay = multi-host logical network.**

---

## 230. What is the host network namespace vs container network namespace?

### Short Interview Answer

The host network namespace contains the host's network interfaces, routes, sockets, and networking state. A normal container gets a separate network namespace with its own network interfaces, routes, and port space.

### Normal Container

```text
Host namespace
    │
    └── container namespace
          ├── eth0
          ├── routes
          └── ports
```

### Host Network Mode

```text
Host namespace
      ↑
      │
 container
```

### Why It Matters

With isolated namespaces:

```text
container localhost ≠ host localhost
```

With host networking:

```text
container uses host network namespace
```

### Interview Point

**Network namespace separation is the basis of normal container network isolation.**

---

## 231. What does `localhost` mean inside a container?

### Short Interview Answer

`localhost` refers to the loopback interface of the container's own network namespace.

### Example

If:

```text
Container A
```

runs an application on:

```text
127.0.0.1:8080
```

then another container:

```text
Container B
```

cannot reach it using:

```text
127.0.0.1:8080
```

because B's `localhost` is B itself.

### Correct Pattern

If both are on:

```text
app-net
```

B can use the service/container name:

```text
A:8080
```

### Common Interview Trap

> "`localhost` means the Docker host."

Usually false for a normally network-isolated container.

### Interview Point

**Inside a normal container, `localhost` means that container's network namespace.**

---

## 232. Why can't one container reach another container using `localhost`?

### Short Interview Answer

Because each normal container has its own network namespace and therefore its own loopback interface.

### Example

```text
Container A:
127.0.0.1 → A

Container B:
127.0.0.1 → B
```

Therefore:

```text
B → 127.0.0.1:8080
```

tries to reach B, not A.

### Correct Approach

Create a shared network:

```bash
docker network create app-net
```

Attach both containers and connect using:

```text
service-name:port
```

### Interview Point

**`localhost` is namespace-local.**

---

## 233. How can a container access the internet?

### Short Interview Answer

On typical Docker bridge networking, the container sends traffic through its virtual interface and Docker/host networking performs the necessary routing/NAT so outbound traffic can reach external networks, assuming the host and network configuration permit it.

### Conceptual Flow

```text
Container
   ↓
veth
   ↓
Docker bridge
   ↓
Host routing/NAT
   ↓
Host network
   ↓
Internet
```

### Important Point

Internet access depends on the host's network connectivity and firewall/NAT configuration.

### Common Interview Trap

Docker does not create an independent physical internet connection for every container.

### Interview Point

**Container internet access normally uses the host's networking path plus Docker-managed networking rules.**

---

## 234. How does a container receive an IP address?

### Short Interview Answer

Docker assigns an IP address from the address pool associated with the Docker network when the container is connected to that network.

### Example

```bash
docker network create app-net
docker run -d --name web --network app-net nginx
```

Inspect:

```bash
docker inspect web
```

The network configuration includes the container's assigned IP.

### Important Caveat

The exact address is dynamic and can change when a container is recreated.

### Better Application Pattern

Use:

```text
web
```

through Docker DNS rather than storing:

```text
172.x.x.x
```

in application configuration.

### Interview Point

**Docker networks manage container IP allocation; application design should avoid depending on ephemeral container IPs.**

---

## 235. Can one container belong to multiple Docker networks?

### Short Interview Answer

Yes. A container can be connected to multiple Docker networks.

### Example

```bash
docker network create frontend
docker network create backend

docker run -d --name app --network frontend myapp

docker network connect backend app
```

Now:

```text
app
 ├── frontend network
 └── backend network
```

### Why Useful?

It can isolate traffic paths.

For example:

```text
frontend
   │
   ↓
application
   │
   ↓
backend
```

The database may only be connected to:

```text
backend
```

while the public-facing component is connected to:

```text
frontend
```

### Security Benefit

Network segmentation can reduce unnecessary connectivity.

### Common Interview Trap

Connecting a container to multiple networks does not automatically mean every container on those networks can communicate with every other network.

The container has interfaces/connectivity on each attached network, but network membership still determines which peers share that network.

### Interview Point

**Multiple networks enable controlled network segmentation and multi-tier connectivity.**

---

## 236. What is `docker network connect`?

### Short Interview Answer

It connects an existing container to an existing Docker network.

### Example

```bash
docker network connect backend app
```

Now `app` is attached to `backend`.

### Reverse Operation

```bash
docker network disconnect backend app
```

removes that network attachment.

### Why Useful?

It allows network topology to be changed without recreating the container in appropriate situations.

### Common Interview Trap

`docker network connect` does not create a new network.

Create one with:

```bash
docker network create backend
```

### Interview Point

**`network connect` changes an existing container's network membership.**

---

## 237. How do you inspect a Docker network?

### Short Interview Answer

Use:

```bash
docker network inspect <network>
```

### Example

```bash
docker network inspect app-net
```

This can show:

- network driver
- subnet/gateway information
- connected containers
- network configuration
- IPAM details

### Why Useful?

It helps answer:

```text
Which containers are connected?
What subnet is being used?
What gateway exists?
Which driver is this network using?
```

### Common Interview Trap

Do not confuse:

```bash
docker network inspect
```

with:

```bash
docker inspect <container>
```

The first focuses on the network; the second focuses on the object being inspected.

### Interview Point

**Use network inspect to understand Docker's network topology.**

---

## 238. What is Docker IPAM?

### Short Interview Answer

IPAM stands for IP Address Management. Docker IPAM manages IP address allocation for Docker networks, including subnets, gateways, and container addresses.

### Example

Create a network:

```bash
docker network create \
  --subnet 172.30.0.0/16 \
  app-net
```

Docker then manages addresses within that network.

### Conceptual Model

```text
Network
 ├── subnet
 ├── gateway
 └── container IP allocation
```

### Why It Matters

Without IP management, assigning unique addresses to containers and maintaining network topology would become difficult.

### Interview Point

**Docker IPAM manages network addressing for Docker networks.**

---

## 239. What is the Docker network gateway?

### Short Interview Answer

The gateway is the network address used by containers as the next hop for traffic that needs to leave their local container network.

### Example

Conceptually:

```text
Network:
172.30.0.0/16

Gateway:
172.30.0.1

Container:
172.30.0.2
```

Traffic from:

```text
172.30.0.2
```

to another network can be sent through:

```text
172.30.0.1
```

### Important Caveat

The exact addresses depend on the Docker network configuration.

### Interview Point

**The gateway provides the next-hop path out of the container's local Docker subnet.**

---

## 240. What is the difference between container-to-container and host-to-container communication?

### Short Interview Answer

Container-to-container communication can use a shared Docker network directly. Host-to-container communication often uses published ports or appropriate host/network configuration.

### Container → Container

```text
app → db:5432
```

when both are on the same network.

No `-p` is necessarily required.

### Host → Container

```text
Host → published host port → container port
```

Example:

```bash
docker run -p 8080:80 nginx
```

### Conceptual Difference

```text
Container-to-container
    ↓
Docker network

Host-to-container
    ↓
published port / host networking / other configured path
```

### Common Interview Trap

Do not assume `-p` is required for all Docker communication.

### Interview Point

**Publishing is primarily about exposing a container service through the host; internal network communication can work without it.**

---

# Quick Revision

| Concept | Key Point |
|---|---|
| Network namespace | Isolated Linux network stack |
| Docker network driver | Defines networking model |
| Bridge | Common single-host networking |
| User-defined bridge | Better isolation + DNS |
| `docker0` | Traditional default Linux bridge |
| veth pair | Virtual connection between namespaces |
| Docker DNS | Name-based container/service resolution |
| Container IP | Dynamically assigned by network IPAM |
| `localhost` | Current network namespace |
| `-p` | Publish host port to container port |
| `EXPOSE` | Documents intended container port |
| Same network | Enables direct container communication |
| Host networking | Uses host network namespace |
| `none` | No normal Docker network |
| Overlay | Multi-host logical networking |
| Multiple networks | Network segmentation/multi-tier connectivity |
| `network connect` | Attach container to another network |
| `network inspect` | Inspect network topology/config |
| IPAM | Manages Docker network addressing |
| Gateway | Next hop out of local Docker subnet |
| Container → container | Usually uses Docker network directly |
| Host → container | Often uses published ports |

---

# High-Value Interview Traps

### Trap 1 — "`EXPOSE 8080` makes port 8080 accessible from the host."

**Wrong.**

`EXPOSE` documents the intended port.

Publishing is done with:

```bash
-p 8080:8080
```

---

### Trap 2 — "Containers need published ports to communicate."

**Wrong.**

Containers on the same appropriate Docker network can communicate directly.

---

### Trap 3 — "`localhost` means the Docker host."

**Wrong for normal containers.**

```text
container localhost → container
host localhost      → host
```

---

### Trap 4 — "All containers use `docker0`."

**Wrong.**

User-defined bridges create their own networking configuration, and other network drivers use different mechanisms.

---

### Trap 5 — "A veth pair is a Docker network driver."

**Wrong.**

It is a Linux networking mechanism used to connect network namespaces.

---

### Trap 6 — "Container IP addresses are stable."

**Wrong.**

They can change when containers are recreated.

Prefer DNS/service names.

---

### Trap 7 — "Bridge and overlay are basically the same."

**Wrong.**

```text
bridge  → typically single-host
overlay → multi-host logical network
```

---

### Trap 8 — "Host networking means the container has no network."

**Wrong.**

It means the container uses the host's network namespace rather than a separate normal container namespace.

---

### Trap 9 — "Two containers cannot both listen on port 80."

**Wrong.**

They can both listen on:

```text
container:80
```

because they have separate network namespaces.

They cannot normally both publish the same host binding such as:

```text
host:8080
```

---

### Trap 10 — "Docker DNS gives containers permanent IP addresses."

**Wrong.**

DNS provides name resolution; container IPs can change when containers are recreated.

---

### Trap 11 — "A container can communicate with every other container automatically."

**Wrong.**

Network membership and network configuration determine connectivity.

---

# Interview Follow-Up Questions

After answering Docker networking fundamentals, an interviewer may ask:

1. How does Docker implement bridge networking using Linux primitives?
2. How does a veth pair connect a container to a bridge?
3. How does Docker perform NAT for outbound traffic?
4. How does published port forwarding work?
5. What happens when a container connects to multiple networks?
6. How does Docker's embedded DNS work?
7. What is the difference between bridge, host, none, and overlay?
8. What is Docker IPAM?
9. How would you configure a custom subnet and gateway?
10. How do you prevent containers from communicating with each other?
11. How do you troubleshoot container-to-container connectivity?
12. How does Docker networking differ from Kubernetes networking?
13. What happens to a container IP after the container is recreated?
14. Why is hard-coding container IPs a bad practice?
15. What happens when two containers publish the same host port?

---

# Final Interview Answer

> **"Docker networking is built around Linux networking primitives such as network namespaces, virtual Ethernet pairs, bridges, routing, and NAT. A normal container gets its own network namespace, so its localhost and port space are isolated from the host and other containers. Containers attached to the same user-defined network can communicate directly and use Docker's embedded DNS for name resolution, so I generally use service or container names rather than hard-coded IP addresses. Port publishing with `-p` is used when a container service needs to be exposed through the Docker host; it is not required for normal container-to-container communication. For multi-host networking, Docker can use overlay networks, while host and none modes provide different isolation/connectivity trade-offs."**

---

# One-Line Memory Map

```text
CONTAINER
   ↓
NETWORK NAMESPACE
   ↓
veth pair
   ↓
Docker bridge
   ↓
routing / NAT
   ↓
host / external network
```

## Communication Rule

```text
Container → Container
       ↓
Same Docker network
       ↓
Service/container name
       ↓
Container port

Host → Container
       ↓
Published host port (-p)
       ↓
Container port
```

## Three Ports to Remember

```text
EXPOSE 80
→ documentation/metadata

-p 8080:80
→ host 8080 → container 80

db:5432
→ container-to-container communication
   over a shared Docker network
```
