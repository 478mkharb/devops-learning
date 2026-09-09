# Docker Interview Preparation — Topic 20: Docker vs Kubernetes + Real-World Scenarios

> **Interview focus:** Explain where Docker fits, where Kubernetes fits, and solve realistic production scenarios by reasoning from **requirements → architecture → failure mode → appropriate tool/control**.

---

# 602. What is the difference between Docker and Kubernetes?

## Short Interview Answer

Docker is primarily a platform/tooling ecosystem for **building, packaging, and running containers**.

Kubernetes is a **container orchestration platform** designed to manage containerized workloads across a cluster.

Conceptually:

```text
Docker
  ├── Build images
  ├── Run containers
  ├── Manage networks
  └── Manage storage

Kubernetes
  ├── Schedule workloads
  ├── Maintain desired state
  ├── Scale workloads
  ├── Provide service discovery
  ├── Perform rolling updates
  └── Self-heal workloads
```

## Interview Point

> **Docker focuses heavily on container lifecycle and packaging; Kubernetes focuses on cluster-level orchestration and desired-state management.**

---

# 603. Is Kubernetes a replacement for Docker?

## Short Interview Answer

Not exactly.

Kubernetes is an orchestration platform, while Docker is a container platform/tooling ecosystem.

Modern Kubernetes does not require the Docker Engine as its node-level container runtime. Kubernetes can use OCI-compatible runtimes through the Container Runtime Interface (CRI), such as `containerd` or CRI-O.

## Important Distinction

```text
Application
    ↓
Container image
    ↓
Kubernetes
    ↓
CRI runtime
    ↓
Linux kernel
```

Docker can still be used extensively in the workflow to:

- build images
- test containers locally
- push images to registries

## Interview Trap

Do not say:

> "Kubernetes runs Docker containers using Docker Engine on every node."

That is not the modern Kubernetes architecture.

## Interview Point

> **Kubernetes orchestrates container workloads; the node runtime does not have to be Docker Engine.**

---

# 604. What is a container runtime?

## Short Interview Answer

A container runtime is the software responsible for executing containers.

Examples include:

```text
containerd
CRI-O
```

At a lower level, an OCI runtime such as:

```text
runc
```

can create the actual container process and isolation.

## Conceptual Stack

```text
Kubernetes
    ↓
CRI
    ↓
containerd / CRI-O
    ↓
OCI runtime
    ↓
Linux kernel
```

## Interview Point

> **Container runtime is the execution layer; Kubernetes is the orchestration layer.**

---

# 605. What is CRI?

## Short Interview Answer

CRI stands for **Container Runtime Interface**.

It is the interface Kubernetes uses to communicate with container runtimes.

Conceptually:

```text
Kubernetes kubelet
        ↓
       CRI
        ↓
containerd / CRI-O
        ↓
OCI runtime
```

## Interview Point

> **CRI decouples Kubernetes from a specific container runtime implementation.**

---

# 606. What is OCI?

## Short Interview Answer

OCI stands for **Open Container Initiative**.

It defines open standards around container images and runtimes.

The goal is interoperability between container tooling and runtimes.

Examples of OCI-related concepts:

```text
OCI image specification
OCI runtime specification
```

## Interview Point

> **OCI provides standards; it is not itself a container runtime.**

---

# 607. Docker image vs Kubernetes Pod — are they the same?

## Short Interview Answer

No.

A Docker image is a packaged application artifact.

A Kubernetes Pod is the smallest deployable unit in Kubernetes and can contain one or more containers.

```text
Docker image
   ↓
container

Kubernetes Pod
   ├── container
   └── optional sidecar container(s)
```

## Interview Point

> **Image = packaged artifact; Pod = Kubernetes execution/deployment unit.**

---

# 608. Why does Kubernetes use Pods instead of directly managing individual containers?

## Short Interview Answer

A Pod provides a shared execution context for one or more closely coupled containers.

Containers in the same Pod can share:

- network namespace
- IP address
- localhost
- volumes

Example:

```text
Pod
 ├── Application container
 └── Sidecar container
```

They can communicate through:

```text
localhost:<port>
```

because they share the Pod's network namespace.

## Interview Point

> **A Pod groups containers that need to share networking, storage or lifecycle semantics.**

---

# 609. Docker container vs Kubernetes Pod?

| Docker Container | Kubernetes Pod |
|---|---|
| Runtime container object | Smallest Kubernetes deployable unit |
| Managed directly by Docker/container runtime | Managed by Kubernetes |
| Normally one main application process | Can contain multiple containers |
| Has its own network namespace | Containers in Pod share Pod network namespace |
| Docker lifecycle commands | Kubernetes desired-state model |

## Important

A Pod is **not itself a container**.

## Interview Point

> **Kubernetes manages Pods; containers execute inside Pods.**

---

# 610. Why does Kubernetes usually run one main application container per Pod?

## Short Interview Answer

Because the Pod is intended to group containers that are tightly coupled.

For example:

```text
Pod
 ├── application
 └── log/metrics/security sidecar
```

Putting unrelated applications in the same Pod creates unnecessary coupling.

A good rule is:

> **One main workload per Pod, with additional containers only when they genuinely share lifecycle/network/storage requirements.**

## Interview Trap

Do not say:

> "Kubernetes allows only one container per Pod."

It does not.

## Interview Point

> **One-container Pods are common; multi-container Pods are useful when containers are tightly coupled.**

---

# 611. What is a Kubernetes Deployment?

## Short Interview Answer

A Deployment manages a set of replicated Pods and provides declarative updates.

It is commonly used for stateless applications.

Conceptually:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
    ↓
Containers
```

Example desired state:

```text
replicas: 3
```

Kubernetes works to maintain that desired number.

## Interview Point

> **Deployment provides declarative management of replicated application Pods and controlled updates.**

---

# 612. What is the difference between Docker restart policy and Kubernetes self-healing?

## Short Interview Answer

A Docker restart policy operates at the local container lifecycle level.

Kubernetes uses controllers and desired-state reconciliation.

Docker:

```text
container exits
   ↓
restart policy
   ↓
restart container
```

Kubernetes:

```text
desired replicas = 3
actual replicas = 2
       ↓
controller reconciliation
       ↓
create replacement Pod
```

## Interview Point

> **Kubernetes self-healing is broader than simply restarting a process.**

---

# 613. What is desired state in Kubernetes?

## Short Interview Answer

Desired state is what the user declares Kubernetes should maintain.

Example:

```yaml
replicas: 3
```

If only two Pods are running:

```text
Desired = 3
Actual  = 2
```

Kubernetes controllers attempt to reconcile the difference.

## Interview Point

> **Kubernetes continuously works toward the declared desired state.**

---

# 614. What is declarative configuration?

## Short Interview Answer

Declarative configuration describes **what state is required**, rather than giving every individual step to achieve it.

Example:

```yaml
replicas: 3
image: myapp:1.0
```

Instead of manually:

```text
create container 1
create container 2
create container 3
restart container 2
...
```

the system determines the actions needed to reach the desired state.

## Interview Point

> **Declarative systems describe the desired result; controllers determine how to reach it.**

---

# 615. What is the difference between Docker Compose and Kubernetes?

## Short Interview Answer

Compose is simpler and commonly used for local development, testing and suitable single-host deployments.

Kubernetes is designed for cluster orchestration and provides:

- scheduling
- service discovery
- scaling
- rolling updates
- reconciliation
- self-healing
- workload placement
- broader networking/storage abstractions

| Compose | Kubernetes |
|---|---|
| Simpler | More comprehensive |
| Commonly single-host | Cluster-oriented |
| Low operational complexity | Higher operational complexity |
| Docker-centric | Container-runtime independent through CRI |
| Good for local/many small deployments | Good for distributed orchestration |

## Interview Point

> **Choose based on operational requirements rather than simply choosing the more complex platform.**

---

# 616. When would you choose Docker Compose instead of Kubernetes?

## Short Interview Answer

I would consider Compose when:

- the application is relatively small
- workloads run on one host
- advanced scheduling is unnecessary
- simple deployment is preferred
- development/testing is the primary requirement
- operational complexity should remain low

I would consider Kubernetes when:

- multiple nodes are required
- automated scheduling is needed
- replicas and self-healing are important
- rolling deployments are required
- cluster-level service discovery is required
- workload placement and scaling are complex

## Interview Point

> **The correct answer depends on requirements, not technology preference.**

---

# 617. What is a Kubernetes Service?

## Short Interview Answer

A Kubernetes Service provides a stable network abstraction for reaching a set of Pods.

Pods are ephemeral and their IP addresses can change.

The Service provides a stable endpoint:

```text
Client
  ↓
Service
  ↓
Pod 1
Pod 2
Pod 3
```

## Interview Point

> **A Kubernetes Service decouples clients from changing Pod IP addresses.**

---

# 618. How is a Kubernetes Service conceptually different from a Docker container IP?

## Short Interview Answer

A container IP is associated with an individual container.

A Kubernetes Service represents a stable access point for a dynamic set of Pods.

```text
Container IP
    ↓
individual runtime instance

Service
    ↓
stable logical endpoint
    ↓
multiple Pods
```

## Interview Point

> **Service abstraction is designed for dynamic workloads; hard-coded container IPs are fragile.**

---

# 619. What is a Kubernetes Ingress?

## Short Interview Answer

Ingress provides HTTP/HTTPS routing from outside the cluster to services based on rules such as:

- host
- path

Conceptually:

```text
Internet
   ↓
Ingress
 ┌─┴─────────┐
 ↓           ↓
/api        /web
 ↓           ↓
API        Frontend
```

An Ingress resource requires an appropriate Ingress controller to implement the routing.

## Interview Point

> **Ingress is a routing configuration/API object; the controller performs the actual implementation.**

---

# 620. Docker `-p` vs Kubernetes Service — what is the difference?

## Short Interview Answer

Docker:

```bash
docker run -p 8080:80 nginx
```

publishes a container port through the Docker host.

Kubernetes Service provides a stable cluster-level abstraction for accessing Pods.

Depending on Service type and environment, external exposure can involve:

```text
ClusterIP
NodePort
LoadBalancer
```

## Interview Point

> **Docker port publishing is host-oriented; Kubernetes Service is workload/service-oriented.**

---

# 621. What is the difference between a Kubernetes Service and an Ingress?

| Service | Ingress |
|---|---|
| Stable endpoint for Pods | HTTP/HTTPS routing layer |
| Selects backend Pods | Routes requests to Services |
| Cluster networking abstraction | North-south application routing |
| Can expose TCP/UDP depending on implementation/type | Primarily HTTP/HTTPS |

Conceptually:

```text
Client
  ↓
Ingress
  ↓
Service
  ↓
Pods
```

## Interview Point

> **Ingress normally routes to Services; Services provide stable access to Pods.**

---

# 622. What is Kubernetes self-healing?

## Short Interview Answer

Kubernetes continuously compares desired and actual state.

Example:

```text
Deployment wants 3 Pods

Pod 1 ✓
Pod 2 ✓
Pod 3 ✗

        ↓

Controller detects mismatch

        ↓

Replacement Pod
```

The exact behavior depends on the controller and failure type.

## Interview Point

> **Self-healing is controller-driven reconciliation.**

---

# 623. What happens when a Kubernetes Pod dies?

## Short Interview Answer

The answer depends on how the Pod is managed.

If a Deployment manages it, the Deployment/ReplicaSet mechanism ensures the desired number of Pods is maintained.

Conceptually:

```text
Pod fails
  ↓
ReplicaSet detects fewer replicas
  ↓
Replacement Pod created
  ↓
Scheduler places it on a suitable node
```

## Interview Point

> **Do not say "Kubernetes restarts the same Pod" in every failure scenario; controllers may create replacement Pods.**

---

# 624. Why are Kubernetes Pods considered ephemeral?

## Short Interview Answer

Pods are designed to be replaceable.

Their:

- IP address
- identity
- local writable storage
- runtime instance

should not be treated as permanent application identity.

Persistent state should be stored through appropriate storage abstractions.

## Interview Point

> **Design Kubernetes applications so Pods can disappear and be replaced without losing required persistent state.**

---

# 625. What is the difference between stateless and stateful containers?

## Short Interview Answer

### Stateless

The container does not depend on local container filesystem state for durable application data.

Example:

```text
API server
```

### Stateful

The workload requires persistent identity/data.

Example:

```text
database
```

Stateful applications need an appropriate persistent-storage and lifecycle strategy.

## Interview Point

> **Do not rely on a container's writable layer for durable application state.**

---

# 626. Scenario: You have a frontend, backend API and database. Would you use one container?

## Short Interview Answer

No.

I would normally separate them:

```text
Frontend
   ↓
Backend API
   ↓
Database
```

Each component has a different lifecycle, scaling requirement and security boundary.

With Docker Compose:

```text
frontend service
api service
db service
```

With Kubernetes:

```text
frontend Deployment
api Deployment
database StatefulSet or managed database
```

depending on the architecture.

## Interview Point

> **Separate independently deployable and scalable components.**

---

# 627. Scenario: Your API needs three replicas. How would you design it?

## Docker/Compose

For a suitable single-host deployment:

```bash
docker compose up -d --scale api=3
```

But I would need to consider:

- load balancing
- port publishing
- shared state
- session handling
- storage
- resource limits

## Kubernetes

A Deployment is a natural fit:

```yaml
spec:
  replicas: 3
```

Then expose it through a Service.

```text
Client
  ↓
Service
  ↓
API Pod 1
API Pod 2
API Pod 3
```

## Interview Point

> **Scaling replicas requires a traffic-distribution and state-management strategy, not just more containers.**

---

# 628. Scenario: Your container stores uploaded files locally. What happens if the container is recreated?

## Short Interview Answer

Files stored only in the container's writable layer are lost when the container is removed.

For durable data, use:

- Docker volumes
- object storage
- external persistent storage
- another appropriate storage system

depending on architecture.

## Interview Point

> **Container filesystem state is not automatically durable application storage.**

---

# 629. Scenario: Your application needs a database. Should the database run in Docker?

## Short Interview Answer

It can, but the correct decision depends on operational requirements.

Running a database in a container is technically valid, but production considerations include:

- persistent storage
- backup/restore
- replication
- upgrades
- monitoring
- recovery
- performance
- operational expertise

For cloud environments, a managed database may reduce operational burden.

## Interview Point

> **Containerizing a database is not inherently wrong; the important question is whether the operational model is appropriate.**

---

# 630. Scenario: A Docker container works locally but fails in production.

## Strong Interview Approach

Compare:

```text
Image
Architecture
Environment variables
Secrets
Config files
Mounted volumes
User/permissions
Network
DNS
Dependencies
CPU/memory limits
External services
```

Commands:

```bash
docker inspect <container>
docker logs <container>
docker stats <container>
```

## Interview Point

> **Do not assume the image is the only difference; runtime configuration is part of the application environment.**

---

# 631. Scenario: Your container can reach another container but not the Internet.

## Strong Interview Approach

Separate:

```text
Container → internal service
```

from:

```text
Container → Internet
```

Check:

```bash
ip route
cat /etc/resolv.conf
getent hosts example.com
curl -v https://example.com
```

Then investigate Docker network gateway, host forwarding/NAT, firewall and upstream network controls.

## Interview Point

> **Internal connectivity does not prove Internet egress works.**

---

# 632. Scenario: A container can resolve DNS but cannot connect to the application.

## Strong Interview Approach

Test layers independently:

```bash
getent hosts service
```

then:

```bash
nc -vz service 8080
```

then:

```bash
curl -v http://service:8080
```

If TCP fails, inspect listener/network.

If TCP succeeds but HTTP fails, investigate the application protocol.

## Interview Point

> **DNS, TCP and application protocol are separate troubleshooting layers.**

---

# 633. Scenario: The container is healthy but users still cannot access the application.

## Strong Interview Answer

A container healthcheck may only prove that the application is functioning **inside its local environment**.

I would trace:

```text
User
 ↓
DNS
 ↓
Load Balancer / Reverse Proxy
 ↓
Firewall / Security Group
 ↓
Published/Service endpoint
 ↓
Container network
 ↓
Application
```

Healthcheck success does not prove every external network path works.

## Interview Point

> **Health is scoped to the check being performed.**

---

# 634. Scenario: Your container is consuming all host memory.

## Strong Interview Answer

I would first identify the consuming container:

```bash
docker stats
```

Then verify its configured limit:

```bash
docker inspect <container>
```

If no limit exists, I would consider an appropriate memory limit after understanding the application's expected workload.

I would also investigate:

- memory leaks
- workload spikes
- dependency behavior
- host memory pressure

## Interview Point

> **Resource limits contain blast radius, but they do not replace application performance analysis.**

---

# 635. Scenario: Your Docker host is running out of disk.

## Strong Interview Answer

I would distinguish filesystem capacity from inode exhaustion:

```bash
df -h
df -i
```

Then:

```bash
docker system df -v
```

I would identify whether the problem is:

```text
Images
Containers
Volumes
Build cache
Logs
Application data
```

Then perform targeted cleanup.

## Interview Point

> **Never delete Docker volumes blindly when persistent data may be present.**

---

# 636. Scenario: A developer wants to mount `/var/run/docker.sock` into a CI container.

## Strong Interview Answer

I would treat this as a significant security decision.

The Docker socket gives the container access to the Docker daemon, which can provide powerful host control.

I would first ask why the CI job requires Docker access and evaluate alternatives such as:

- rootless/build-specific tooling
- remote build services
- dedicated builders
- BuildKit-based workflows
- isolated build environments

If socket access is unavoidable, it should be tightly controlled and isolated.

## Interview Point

> **Docker socket access is a privileged trust boundary, not a normal application dependency.**

---

# 637. Scenario: An image contains a known critical vulnerability.

## Strong Interview Answer

I would:

1. identify the vulnerable package/component
2. determine whether a fixed version exists
3. update the base image/dependency
4. rebuild the image
5. scan again
6. test the application
7. deploy the corrected image
8. retire the vulnerable artifact according to policy

I would avoid simply suppressing the scanner finding without understanding the actual risk.

## Interview Point

> **The remediation process is identify → update → rebuild → scan → test → deploy.**

---

# 638. Scenario: A production deployment references `latest`.

## Strong Interview Answer

I would recommend an immutable release strategy.

Instead of:

```text
myapp:latest
```

use a release identifier such as:

```text
myapp:1.4.2
```

and, where stronger immutability is required, deploy by digest:

```text
myapp@sha256:<digest>
```

This improves:

- reproducibility
- rollback
- auditability
- deployment consistency

## Interview Point

> **Mutable tags are convenient; immutable references are safer for controlled deployments.**

---

# 639. Scenario: You need to deploy a Java application efficiently.

## Strong Interview Answer

I would use a multistage build.

```text
Stage 1
JDK + Maven/Gradle
      ↓
compile/package
      ↓
Stage 2
Java runtime + application artifact
```

The final image should contain only the runtime requirements.

I would also:

- run as non-root
- avoid embedding secrets
- use a healthcheck where useful
- pin appropriate versions
- scan the final image

## Interview Point

> **Separate build dependencies from runtime dependencies.**

---

# 640. Scenario: You need to deploy a React frontend.

## Strong Interview Answer

A common pattern is:

```text
Node.js build stage
        ↓
npm ci
        ↓
npm run build
        ↓
Static assets
        ↓
NGINX runtime stage
```

Example:

```dockerfile
FROM node:22 AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
```

The production image does not need Node.js if it only serves static files.

## Interview Point

> **Build with Node; serve with a minimal static web server.**

---

# 641. Scenario: You need to deploy a Go application as a very small image.

## Strong Interview Answer

Go applications can often be compiled into a self-contained binary.

Typical pattern:

```text
Go builder
    ↓
go build
    ↓
small runtime image
```

A `scratch` runtime can be used if the binary and application do not require additional runtime files.

However, `scratch` contains no normal userland, shell or CA certificates by default, so the application must include/provide everything it requires.

## Interview Point

> **`scratch` can produce extremely small images, but only when the application is truly compatible with the minimal runtime environment.**

---

# 642. Scenario: A container exits with code 143.

## Strong Interview Answer

`143` is commonly:

```text
128 + 15
```

where signal 15 is:

```text
SIGTERM
```

This commonly indicates that the process received a graceful termination request.

I would still inspect:

```bash
docker logs <container>
docker inspect <container>
```

because the exit code must be interpreted in context.

## Interview Point

> **143 commonly means SIGTERM; it does not automatically mean application failure.**

---

# 643. Scenario: A container exits with code 137.

## Strong Interview Answer

```text
137 = 128 + 9
```

Signal 9 is:

```text
SIGKILL
```

OOM is a common reason, so I would verify:

```bash
docker inspect <container> \
  --format '{{.State.OOMKilled}}'
```

I would also investigate manual kills, runtime behavior and resource constraints.

## Interview Point

> **137 means SIGKILL; verify whether OOM actually caused it.**

---

# 644. Scenario: A container's healthcheck fails but the process remains running.

## Strong Interview Answer

The container can remain:

```text
running
```

while its health state becomes:

```text
unhealthy
```

I would inspect:

```bash
docker inspect <container>
```

and review the healthcheck output.

Then determine whether the failure is caused by:

- application failure
- incorrect healthcheck
- dependency failure
- network issue
- startup timing

## Interview Point

> **Healthcheck failure is an application-health signal, not automatically a container process failure.**

---

# 645. Scenario: Your application needs secrets during image build.

## Strong Interview Answer

I would use a build secret mechanism such as BuildKit secret mounts rather than:

```dockerfile
ARG SECRET=...
```

or:

```dockerfile
ENV SECRET=...
```

The secret should be available only to the required build step and should not become part of the resulting image.

## Interview Point

> **Build-time secret access should be ephemeral and excluded from the final artifact.**

---

# 646. Scenario: Your application needs a database password at runtime.

## Strong Interview Answer

I would keep the password out of the image and supply it through an appropriate runtime secret-management mechanism.

I would avoid:

```dockerfile
ENV DB_PASSWORD=...
```

and avoid committing credentials into Compose files or source control.

The application should receive only the secret it requires.

## Interview Point

> **Build artifacts should be reusable; environment-specific secrets belong at deployment/runtime.**

---

# 647. Scenario: The application needs to write temporary files, but you want a read-only root filesystem.

## Strong Interview Answer

I would use:

```bash
--read-only
```

and explicitly provide a writable location for temporary data, such as an appropriate tmpfs mount.

Conceptually:

```text
Root filesystem
     ↓
Read-only

/tmp
     ↓
Explicit writable temporary storage
```

This preserves the security benefit while satisfying the application's runtime requirement.

## Interview Point

> **Security hardening should constrain writes without breaking legitimate application behavior.**

---

# 648. Scenario: The container needs port 80 but should not run as root.

## Strong Interview Answer

I would first consider changing the application to listen on an unprivileged port such as:

```text
8080
```

and publish:

```text
80 → 8080
```

For example:

```bash
docker run -p 80:8080 myapp
```

Alternatively, where genuinely required, a narrowly scoped capability such as:

```text
NET_BIND_SERVICE
```

can allow binding to privileged ports without granting broad privileged mode.

## Interview Point

> **Prefer architectural changes or narrowly scoped capabilities over running the whole application as root.**

---

# 649. Scenario: Two containers need to communicate securely.

## Strong Interview Answer

I would:

1. put them on an appropriate private Docker network
2. avoid unnecessary host port publishing
3. use service-name DNS
4. restrict network membership
5. use TLS/mTLS if the threat model and application require encryption/authentication
6. manage credentials through appropriate secret mechanisms

Conceptually:

```text
Service A
   │
Private Network
   │
Service B
```

## Interview Point

> **Network isolation reduces exposure; encryption/authentication protects traffic and identity where required.**

---

# 650. Scenario: You need zero-downtime application deployment.

## Short Interview Answer

On a simple Docker host, I can use a controlled replacement strategy behind a reverse proxy/load balancer.

For example:

```text
Load Balancer
   ├── old container
   └── new container
```

Deploy the new version, verify health, shift traffic, then remove the old version.

In Kubernetes, a Deployment can provide rolling-update behavior.

## Interview Point

> **Zero-downtime deployment requires traffic management and health verification, not merely starting a new container.**

---

# 651. Scenario: You need automatic rollback after a bad deployment.

## Strong Interview Answer

I would use immutable versioned artifacts and a deployment system capable of tracking rollout health.

The strategy is:

```text
Version N
   ↓
Deploy N+1
   ↓
Health / metrics validation
   ↓
Success → continue
Failure → rollback to N
```

In Kubernetes, Deployment rollout mechanisms can support controlled updates and rollback workflows.

With simple Docker deployments, rollback can be implemented by explicitly redeploying the previous known-good image.

## Interview Point

> **Rollback is much easier when releases are immutable and versioned.**

---

# 652. Scenario: Your application has local session state but you want multiple replicas.

## Short Interview Answer

I would identify the state dependency first.

Options include:

- external session store
- stateless authentication such as appropriately designed tokens
- shared state service
- carefully configured sticky sessions where justified

The preferred architecture is usually to make application instances as stateless as practical.

## Interview Point

> **Horizontal scaling requires state to be externalized or deliberately managed.**

---

# 653. Scenario: A container needs persistent database storage.

## Strong Interview Answer

I would not use the container writable layer as durable database storage.

For a Docker-hosted database, I would use a persistent volume and a tested backup strategy.

For production cloud architecture, I would also evaluate a managed database because backups, replication, patching and recovery can be operationally significant.

## Interview Point

> **Persistent storage solves durability; backup/replication solve availability and recovery.**

---

# 654. Scenario: The same image must run on AMD64 and ARM64.

## Strong Interview Answer

I would build a multi-platform image.

Conceptually:

```text
myapp
 ├── linux/amd64
 └── linux/arm64
```

Tools such as BuildKit/buildx can build and publish the platform variants.

The registry stores the appropriate image manifests/index metadata so clients can select the matching platform.

## Interview Point

> **A multi-platform image reference represents platform-specific image variants rather than one universal binary.**

---

# 655. Scenario: A container works on your laptop but gives `exec format error` in production.

## Strong Interview Answer

I would immediately suspect an architecture mismatch.

Check:

```bash
uname -m
docker image inspect <image>
```

and inspect the image's supported platforms.

For example:

```text
Laptop → arm64
Production → amd64
```

or the reverse.

Then rebuild/publish the correct platform variant or a multi-platform image.

## Interview Point

> **Architecture mismatch is one of the classic causes of "works locally, fails in production."**

---

# 656. Scenario: You need to expose only the frontend publicly.

## Strong Interview Answer

I would avoid publishing the database and backend directly.

Conceptually:

```text
Internet
   ↓
Frontend / Reverse Proxy
   ↓
Backend
   ↓
Database
```

Use network segmentation and expose only the necessary public endpoint.

## Interview Point

> **Public exposure should follow the minimum required attack surface.**

---

# 657. Scenario: Your Docker host is running many unrelated applications.

## Strong Interview Answer

I would evaluate whether a single Docker host is still appropriate.

Considerations:

- resource isolation
- failure blast radius
- security boundaries
- deployment independence
- monitoring
- backup/recovery
- operational ownership

Possible solutions include:

```text
separate hosts
VM isolation
orchestrator
managed services
```

depending on requirements.

## Interview Point

> **Container isolation does not automatically make one host equivalent to independent infrastructure boundaries.**

---

# 658. Scenario: When would you move from Docker Compose to Kubernetes?

## Strong Interview Answer

I would move when operational requirements justify cluster orchestration, for example:

- multiple nodes
- automated scheduling
- higher availability requirements
- frequent rolling deployments
- horizontal scaling
- self-healing
- workload placement
- complex service discovery
- centralized cluster management

I would not migrate merely because Kubernetes is more popular.

## Interview Point

> **Migrate when operational complexity requires orchestration capabilities that Compose does not provide.**

---

# 659. Give a complete Docker-to-Kubernetes architecture flow.

## Strong Interview Answer

A typical modern flow is:

```text
Developer
   ↓
Dockerfile
   ↓
Docker / BuildKit
   ↓
OCI Image
   ↓
Container Registry
   ↓
Kubernetes
   ↓
Deployment
   ↓
ReplicaSet
   ↓
Pods
   ↓
Container runtime
   ↓
OCI runtime
   ↓
Linux kernel
```

Traffic:

```text
User
 ↓
Ingress / Load Balancer
 ↓
Service
 ↓
Pod
 ↓
Container
 ↓
Application
```

## Interview Point

> **Docker commonly participates in image creation; Kubernetes consumes and orchestrates those images.**

---

# 660. Give a real-world example of where Docker and Kubernetes work together.

## Example

Suppose the application is:

```text
React frontend
Go API
Python worker
PostgreSQL
Redis
```

### Build phase

Dockerfiles produce:

```text
frontend image
api image
worker image
```

These are pushed to a registry.

### Deployment phase

Kubernetes defines:

```text
frontend Deployment
api Deployment
worker Deployment
Services
Ingress
ConfigMaps
Secrets
Persistent storage
```

### Runtime

```text
Registry
   ↓
Kubernetes node
   ↓
container runtime
   ↓
Pods
   ↓
Containers
```

## Interview Point

> **Docker can be part of the image build/package workflow while Kubernetes provides production orchestration.**

---

# 661. What is the biggest conceptual mistake candidates make about Docker and Kubernetes?

## Short Interview Answer

They treat Docker, containers, Pods and Kubernetes as interchangeable concepts.

They are different layers:

```text
Dockerfile
   ↓
Image
   ↓
Container
   ↓
Pod
   ↓
Deployment
   ↓
Kubernetes cluster
```

Each abstraction solves a different problem.

## Interview Point

> **Good interviews require explaining the boundaries between these abstractions.**

---

# 662. Explain Docker → Container → Pod → Deployment in one flow.

## Answer

```text
Dockerfile
    ↓
build
    ↓
Docker/OCI image
    ↓
container runtime
    ↓
Container
    ↓
Kubernetes Pod
    ↓
Deployment manages Pods
    ↓
Kubernetes cluster
```

### Meaning

```text
Image       = packaged artifact
Container   = running instance
Pod         = Kubernetes execution unit
Deployment  = desired-state controller for replicated Pods
Cluster     = collection of nodes/control-plane components
```

## Interview Point

> **Know the abstraction hierarchy.**

---

# 663. If Kubernetes can run containers without Docker Engine, why do DevOps engineers still use Docker?

## Short Interview Answer

Docker remains extremely useful for:

- writing Dockerfiles
- building images
- local container development
- debugging
- image inspection
- registry workflows
- CI pipelines

The important distinction is:

```text
Docker CLI/BuildKit
       ↓
build artifact

Kubernetes
       ↓
orchestrate artifact
```

The production cluster does not need Docker Engine merely because Docker was used to build the image.

## Interview Point

> **Build tooling and production runtime are separate concerns.**

---

# 664. What would you choose for a small three-container application?

## Strong Interview Answer

If it runs on one host and does not require complex orchestration, I would likely choose Docker Compose.

For example:

```text
NGINX
API
PostgreSQL
```

Compose provides:

- service definitions
- networking
- volumes
- environment configuration
- healthchecks
- simple lifecycle management

I would consider Kubernetes only if the application's operational requirements justify the additional complexity.

## Interview Point

> **Simple requirements should not automatically lead to complex infrastructure.**

---

# 665. What would you choose for a highly available multi-node application?

## Strong Interview Answer

I would consider Kubernetes or another suitable orchestration platform.

I would design for:

- multiple nodes
- replicated workloads
- service discovery
- load balancing
- rolling updates
- health checks
- persistent storage
- observability
- failure recovery
- security controls

The exact architecture depends on workload and organizational requirements.

## Interview Point

> **High availability is an architecture property, not simply a container count.**

---

# 666. How would you explain Docker vs Kubernetes to a non-technical interviewer?

## Short Interview Answer

I would use an analogy carefully:

> "Docker is like the technology used to package and run individual application units, while Kubernetes is like the system that manages a large fleet of those application units across multiple machines."

Then I would clarify that Kubernetes uses container images and a container runtime underneath.

## Interview Point

> **Use the analogy only as an introduction, then return to the technical architecture.**

---

# 667. What is the correct answer if an interviewer asks, "Does Kubernetes create containers?"

## Short Interview Answer

Kubernetes **orchestrates Pods**, and the node's container runtime creates and manages the actual containers.

Conceptually:

```text
Kubernetes
   ↓
Pod specification
   ↓
kubelet
   ↓
CRI
   ↓
container runtime
   ↓
container
```

## Interview Point

> **Kubernetes is the orchestrator; the runtime performs container execution.**

---

# 668. What is the correct answer if an interviewer asks, "Is a Pod a container?"

## Short Interview Answer

No.

A Pod is a Kubernetes abstraction that can contain one or more containers.

```text
Pod
 ├── Container A
 └── Container B
```

Containers inside the same Pod share the Pod's network namespace and can share volumes.

## Interview Point

> **Pod ≠ container.**

---

# 669. What is the correct answer if an interviewer asks, "Why not run the container directly on a node?"

## Short Interview Answer

Kubernetes adds abstractions and control mechanisms around container execution.

The Pod provides:

- shared network context
- shared storage
- lifecycle grouping
- scheduling unit
- metadata and workload semantics

The Deployment/controller layer adds:

- desired state
- replicas
- updates
- reconciliation

The node runtime ultimately executes the containers.

## Interview Point

> **Kubernetes adds workload management and reconciliation around container execution.**

---

# 670. Final scenario: Design a production container platform.

## Strong Interview Answer

I would design the platform in layers.

### 1. Build

```text
Source
  ↓
Dockerfile
  ↓
BuildKit
  ↓
Security scanning
  ↓
Image signing/provenance
```

### 2. Registry

```text
Immutable versioned image
        ↓
Container registry
```

### 3. Orchestration

For a multi-node production environment:

```text
Kubernetes
    ↓
Deployments / Stateful workloads
    ↓
Pods
```

### 4. Networking

```text
Internet
   ↓
Load Balancer / Ingress
   ↓
Services
   ↓
Pods
```

### 5. Configuration

```text
Config
  ↓
ConfigMap / equivalent

Secrets
  ↓
Secret management
```

### 6. Storage

Use persistent storage for stateful workloads and tested backup/recovery processes.

### 7. Security

```text
Non-root
Capabilities
Seccomp/MAC
Network policies
Image scanning
Secrets
Least privilege
```

### 8. Observability

Monitor:

- application metrics
- logs
- container health
- node resources
- cluster state
- deployment health

### 9. Deployment

```text
Build
 ↓
Scan
 ↓
Push
 ↓
Deploy
 ↓
Health validation
 ↓
Progressive/rolling rollout
 ↓
Rollback if required
```

## Interview Point

> **A production container platform is not just "Docker running containers"; it is an integrated build, registry, orchestration, networking, storage, security and observability system.**

---

# Quick Revision

| Concept | Interview Answer |
|---|---|
| Docker | Container build/run platform/tooling |
| Kubernetes | Container orchestration platform |
| Image | Immutable-style packaged artifact |
| Container | Runtime instance |
| Pod | Smallest Kubernetes deployable unit |
| Deployment | Declarative controller for replicated Pods |
| Service | Stable network endpoint for Pods |
| Ingress | HTTP/HTTPS routing abstraction |
| CRI | Kubernetes container runtime interface |
| OCI | Open container standards |
| containerd | Container runtime component |
| runc | OCI runtime |
| Compose | Multi-container application management |
| Swarm | Docker-native cluster orchestration |
| Desired state | State Kubernetes attempts to maintain |
| Self-healing | Reconciliation toward desired state |
| Stateless | No required durable local runtime state |
| Stateful | Requires durable data/identity |
| Rolling update | Gradual replacement of workload versions |
| Registry | Stores/publishes image artifacts |

---

# Docker vs Kubernetes — High-Value Comparison

| Area | Docker | Kubernetes |
|---|---|---|
| Primary role | Build/run/manage containers | Orchestrate workloads |
| Image building | Yes | Not its primary role |
| Single container | Excellent | Possible but usually workload abstraction is Pod |
| Multi-container local app | Compose | Possible but more complex |
| Multi-node scheduling | Not Docker Engine's primary role | Yes |
| Desired-state reconciliation | Limited/local tooling | Core capability |
| Self-healing | Restart policies/local mechanisms | Controllers |
| Scaling | Manual/Compose mechanisms | Native workload scaling |
| Service discovery | Docker networking/DNS | Kubernetes Services/DNS |
| Rolling deployment | Manual/tooling-dependent | Built-in workload mechanisms |
| Container runtime | Docker Engine can run containers | Uses CRI-compatible runtime |
| Complexity | Lower | Higher |
| Typical use | Build/local/single-host | Cluster production orchestration |

---

# High-Value Interview Traps

## Trap 1 — Kubernetes is a container runtime

False.

Kubernetes is an orchestrator.

---

## Trap 2 — Kubernetes requires Docker Engine

False.

Modern Kubernetes uses CRI-compatible runtimes such as containerd or CRI-O.

---

## Trap 3 — Pod = container

False.

A Pod can contain one or more containers.

---

## Trap 4 — Deployment = container

False.

A Deployment manages replicated Pods through controllers.

---

## Trap 5 — Service = Pod

False.

A Service provides a stable network abstraction in front of Pods.

---

## Trap 6 — Ingress replaces Service

Not generally.

Typical flow:

```text
Ingress
   ↓
Service
   ↓
Pods
```

---

## Trap 7 — Kubernetes automatically makes applications highly available

Not by itself.

You need appropriate:

- replicas
- node topology
- load balancing
- health checks
- storage design
- failure-domain planning

---

## Trap 8 — More replicas automatically means scalability

False.

You also need:

- traffic distribution
- sufficient resources
- statelessness/state strategy
- database capacity
- application concurrency

---

## Trap 9 — Docker Compose is useless in production

Too absolute.

It can be appropriate for some single-host/simple production workloads.

---

## Trap 10 — Docker and Kubernetes compete at exactly the same layer

False.

They overlap in the container ecosystem but primarily solve different layers of the problem.

---

## Trap 11 — Kubernetes creates images

Not its primary responsibility.

Build images using Docker/BuildKit or another image-building system, then deploy them.

---

## Trap 12 — Kubernetes manages individual containers directly

The important abstraction is the Pod and its controllers; the container runtime performs container execution.

---

# Interview Follow-Up Questions

1. What is the difference between Docker and Kubernetes?
2. Is Kubernetes a replacement for Docker?
3. What is a container runtime?
4. What is CRI?
5. What is OCI?
6. Why does Kubernetes use Pods?
7. Is a Pod a container?
8. Why are most Pods single-container?
9. What is a Deployment?
10. What is desired state?
11. What is reconciliation?
12. What is self-healing?
13. What is a Kubernetes Service?
14. What is Ingress?
15. Service vs Ingress?
16. Compose vs Kubernetes?
17. Swarm vs Kubernetes?
18. When would you choose Compose?
19. When would you choose Kubernetes?
20. What happens when a Pod dies?
21. Why are Pods ephemeral?
22. How do you handle persistent data?
23. How do you scale an API?
24. How do you achieve zero-downtime deployment?
25. How do you roll back a bad deployment?
26. How do you handle application sessions with multiple replicas?
27. How do you build multi-platform images?
28. What causes `exec format error`?
29. How do Docker and Kubernetes work together?
30. What happens from Dockerfile to running Pod?
31. How would you design a production container platform?
32. What are the major Docker security controls?
33. What would make you migrate from Compose to Kubernetes?
34. How would you troubleshoot a production containerized application?
35. What is the difference between a container restart and Pod replacement?

---

# Final Interview Answer

If asked:

> **"Explain Docker and Kubernetes and how they work together."**

Answer:

> "Docker is primarily used for building, packaging and running containerized applications. A Dockerfile is used to build an image, which is stored in a registry and can then be run as a container. Kubernetes operates at a higher orchestration layer. It manages workloads using abstractions such as Pods, Deployments and Services and continuously reconciles actual state toward the desired state. Modern Kubernetes does not require Docker Engine as its node runtime; it communicates through the Container Runtime Interface with runtimes such as containerd or CRI-O. In a typical workflow, Docker or BuildKit builds the OCI image, the image is pushed to a registry, and Kubernetes pulls that image and schedules it into Pods across cluster nodes. Kubernetes then provides capabilities such as service discovery, scaling, rolling updates and self-healing."

---

# Final Scenario Answer

If asked:

> **"You have a Dockerized application. How would you decide whether to use Compose or Kubernetes?"**

Answer:

> "I would start with operational requirements rather than technology preference. If the application is small, runs on one host and needs straightforward networking, volumes and lifecycle management, Docker Compose may be sufficient. If I need multiple nodes, automated scheduling, replicated workloads, rolling deployments, service discovery, self-healing and more complex scaling, I would consider Kubernetes. I would also consider operational cost and team expertise because Kubernetes introduces significant complexity. The goal is to use the simplest platform that reliably satisfies the application's availability, scalability, security and operational requirements."

---

# Complete Docker Interview Memory Map

```text
                         DOCKER
                           │
          ┌────────────────┼────────────────┐
          │                │                │
       Build             Run             Manage
          │                │                │
      Dockerfile        Container       Network
          │                │             Volume
          ↓                ↓             Config
        Image          Process          Resource
          │
          ↓
      Registry
          │
          ↓
   ┌────────────────────────────────────┐
   │         ORCHESTRATION              │
   │                                    │
   │  Compose → Multi-container app     │
   │  Swarm   → Docker cluster          │
   │  K8s     → Cluster orchestration   │
   └────────────────────────────────────┘
                          │
                          ↓
                      Kubernetes
                          │
                    ┌─────┴─────┐
                    ↓           ↓
               Deployment    Service
                    ↓           ↓
                ReplicaSet    DNS
                    ↓
                   Pods
                    ↓
                Containers
                    ↓
             Container Runtime
                    ↓
               Linux Kernel
```

---

# Docker Interview — End-to-End Mental Model

```text
1. WRITE
   Dockerfile
      ↓

2. BUILD
   Docker / BuildKit
      ↓

3. ARTIFACT
   OCI Image
      ↓

4. STORE
   Registry
      ↓

5. DEPLOY
   Compose / Kubernetes / Swarm
      ↓

6. EXECUTE
   Container Runtime
      ↓

7. ISOLATE
   Namespaces + Capabilities + seccomp/MAC
      ↓

8. CONTROL
   cgroups + resource limits
      ↓

9. CONNECT
   Network + DNS + Service discovery
      ↓

10. STORE
    Volumes / Persistent Storage
      ↓

11. OBSERVE
    Logs + Metrics + Health
      ↓

12. SECURE
    Least Privilege + Scanning + Secrets
      ↓

13. OPERATE
    Troubleshoot → Recover → Rollback
```

---

# Topic 20 Complete

**Questions covered: Q602–Q670**

**Core skill:**

> **Know exactly where Docker ends, where orchestration begins, and how to reason through real production container scenarios instead of memorizing isolated commands.**

---

# Docker Interview Preparation — Complete

## Topics Completed

| # | Topic | Questions |
|---:|---|---:|
| 01 | Docker Fundamentals | Q1–Q12 |
| 02 | Docker Architecture | Q13–Q27 |
| 03 | Docker Internals | Q28–Q58 |
| 04 | Docker Images | Q59–Q86 |
| 05 | Dockerfile Fundamentals | Q87–Q115 |
| 06 | Dockerfile Configuration & Best Practices | Q116–Q135 |
| 07 | Docker Builds | Q136–Q155 |
| 08 | Multistage Builds | Q156–Q175 |
| 09 | Docker Commands | Q176–Q210 |
| 10 | Networking Fundamentals | Q211–Q240 |
| 11 | Networking Troubleshooting | Q241–Q264 |
| 12 | Storage | Q265–Q295 |
| 13 | Configuration & Secrets | Q296–Q324 |
| 14 | Registries | Q325–Q357 |
| 15 | Resource Management | Q358–Q395 |
| 16 | Container Lifecycle | Q396–Q437 |
| 17 | Troubleshooting | Q438–Q490 |
| 18 | Compose & Orchestration | Q491–Q548 |
| 19 | Security | Q549–Q601 |
| 20 | Docker vs Kubernetes + Scenarios | Q602–Q670 |

> **Final preparation principle:**  
> **Understand the abstraction → understand what happens internally → know the command → know the failure mode → explain the production trade-off.**
