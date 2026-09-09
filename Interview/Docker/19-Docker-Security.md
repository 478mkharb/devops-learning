# Docker Interview Preparation — Topic 19: Docker Security

> **Interview focus:** Understand Docker security as a combination of **image security, container isolation, Linux privileges, runtime configuration, secrets, networking, host security, and supply-chain controls**.

---

# 549. Why is Docker security important?

## Short Interview Answer

Docker improves isolation and operational consistency, but a container is **not a security boundary equivalent to a separate physical machine**.

Docker security must address:

- image vulnerabilities
- container privileges
- Linux capabilities
- namespaces
- cgroups
- seccomp
- AppArmor/SELinux
- secrets
- network exposure
- host filesystem access
- Docker daemon access
- supply-chain integrity

## Interview Point

> **Container security is layered security; no single Docker feature provides complete protection.**

---

# 550. Is a Docker container a security boundary?

## Short Interview Answer

A container provides significant isolation, but it shares the host kernel.

Therefore, container isolation is not equivalent to a VM's hardware/kernel boundary.

```text
Container
   ↓
shares host kernel
```

while a VM generally has:

```text
VM
 ↓
guest kernel
 ↓
virtual hardware
```

A container escape caused by a kernel/runtime vulnerability can potentially affect the host.

## Interview Trap

Do not say:

> "Containers are completely isolated from the host."

That is false.

## Interview Point

> **Containers isolate processes and resources using kernel mechanisms while sharing the host kernel.**

---

# 551. What are the main layers of Docker security?

## Short Interview Answer

I think about Docker security in layers:

```text
Image / Supply Chain
        ↓
Container Configuration
        ↓
Linux Isolation
        ↓
Capabilities / Seccomp / MAC
        ↓
Filesystem / Secrets
        ↓
Network
        ↓
Docker Daemon
        ↓
Host OS
```

## Examples

### Image layer

- trusted base images
- vulnerability scanning
- signed/provenance-aware artifacts
- pinned versions/digests

### Runtime layer

- non-root user
- dropped capabilities
- read-only filesystem where practical
- resource limits

### Host layer

- patched kernel
- protected Docker socket
- least privilege
- host monitoring

## Interview Point

> **Secure the entire container lifecycle, not just the image.**

---

# 552. Why should containers run as a non-root user?

## Short Interview Answer

Running the application as a non-root user reduces the privileges available to the application if it is compromised.

Dockerfile:

```dockerfile
FROM alpine:3.20

RUN adduser -D appuser
USER appuser

CMD ["./app"]
```

The application then does not run as root inside the container.

## Important

Non-root is a strong baseline but is not a complete security solution.

A container can still have:

- dangerous capabilities
- host mounts
- excessive permissions
- vulnerable kernel/runtime paths

## Interview Point

> **Use least privilege at the application-user level and combine it with other controls.**

---

# 553. What is the difference between root inside a container and root on the host?

## Short Interview Answer

Root inside a normal container is constrained by the container's isolation and security configuration, but it should **not** be treated as harmless.

Depending on configuration:

```text
container root
      ↓
host kernel + container isolation
```

The container's root user does not automatically have unrestricted access to the host.

However, dangerous capabilities, privileged mode, device access or host mounts can substantially increase its power.

## Interview Trap

Do not say:

> "Root inside a container is exactly the same as root on the host."

Also do not say:

> "Container root can never affect the host."

Both are oversimplifications.

## Interview Point

> **Container root is privileged within the container context, and its effective power depends heavily on runtime configuration.**

---

# 554. What is `USER` in a Dockerfile and why is it a security control?

## Short Interview Answer

`USER` selects the user/group under which subsequent Dockerfile instructions and the default container process run.

Example:

```dockerfile
RUN adduser --disabled-password appuser
USER appuser

CMD ["./app"]
```

This prevents the application from unnecessarily running as root.

## Interview Point

> **`USER` is a simple and important least-privilege control.**

---

# 555. What are Linux capabilities in Docker?

## Short Interview Answer

Linux capabilities divide traditionally broad root privileges into smaller permission sets.

Docker containers normally run with a restricted capability set rather than unrestricted host-level capabilities.

You can inspect or modify capabilities using Docker runtime options.

Example:

```bash
docker run --cap-drop=ALL myapp
```

Then add only what is actually required.

## Principle

```text
Default privileges
      ↓
remove unnecessary capabilities
      ↓
add only required capabilities
```

## Interview Point

> **Capabilities implement fine-grained privilege control below the all-powerful-root model.**

---

# 556. What is `--cap-drop`?

## Short Interview Answer

`--cap-drop` removes Linux capabilities from the container.

Example:

```bash
docker run --cap-drop=ALL myapp
```

A specific capability can be added back if required:

```bash
docker run \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  myapp
```

This follows least privilege.

## Common Interview Trap

Do not blindly add capabilities just because an application fails. Determine exactly why the capability is required.

## Interview Point

> **Drop everything unnecessary and add only the capabilities required by the workload.**

---

# 557. What does `--privileged` do?

## Short Interview Answer

`--privileged` substantially relaxes the normal container isolation and capability restrictions.

Example:

```bash
docker run --privileged myapp
```

It can provide broad access to devices and capabilities that normal containers do not receive.

## Security Risk

A privileged container has a much larger attack surface and significantly greater ability to interact with the host.

## Interview Trap

Never use:

```bash
--privileged
```

as the default solution for a permission problem.

First identify the exact capability or resource required.

## Interview Point

> **`--privileged` is a broad security relaxation and should be avoided unless there is a justified requirement.**

---

# 558. What is the difference between capabilities and `--privileged`?

| Capabilities | `--privileged` |
|---|---|
| Fine-grained | Broad privilege relaxation |
| Add/drop specific permissions | Enables a much wider set of permissions |
| Supports least privilege | Increases attack surface |
| Preferred where possible | Use only when justified |

Example:

```bash
--cap-drop=ALL
--cap-add=NET_BIND_SERVICE
```

is much more targeted than:

```bash
--privileged
```

## Interview Point

> **Prefer narrowly scoped capabilities over privileged mode.**

---

# 559. What is seccomp in Docker?

## Short Interview Answer

Seccomp is a Linux kernel security mechanism that restricts which system calls a process can make.

Docker can use a seccomp profile to reduce the system-call attack surface.

Conceptually:

```text
Container process
       ↓
syscall
       ↓
seccomp policy
       ↓
allow / block
       ↓
Linux kernel
```

## Why It Matters

A compromised process may attempt to use dangerous system calls. Restricting unnecessary syscalls can reduce exploitation opportunities.

## Interview Point

> **Seccomp controls system-call access; capabilities control specific privileges.**

---

# 560. What is AppArmor?

## Short Interview Answer

AppArmor is a Linux Mandatory Access Control framework that can restrict what applications are allowed to access.

It uses profiles describing permitted behavior.

Conceptually:

```text
Process
  ↓
AppArmor profile
  ↓
allowed / denied operations
```

Docker can integrate with AppArmor on supported Linux systems.

## Interview Point

> **AppArmor provides policy-based application confinement at the Linux security layer.**

---

# 561. What is SELinux and how does it relate to Docker?

## Short Interview Answer

SELinux is a Linux Mandatory Access Control system based on security labels and policy.

It can restrict access even when normal UNIX permissions would otherwise allow an operation.

Docker on SELinux-enabled systems can use SELinux labeling to improve container isolation.

## Important Distinction

```text
DAC
 ↓
traditional owner/group/mode permissions

MAC
 ↓
security policy such as SELinux
```

## Interview Point

> **SELinux adds policy-based access control beyond normal UNIX permissions.**

---

# 562. What is the difference between seccomp, capabilities and SELinux/AppArmor?

| Control | Primarily controls |
|---|---|
| Namespaces | Visibility/isolation |
| cgroups | Resource control |
| Capabilities | Privileged operations |
| seccomp | System calls |
| AppArmor | Application access policy |
| SELinux | Label/policy-based access control |

## Memory Map

```text
Namespaces → Where can I see?
cgroups    → How much can I use?
Capabilities → What privileged operation can I perform?
seccomp    → Which syscalls can I make?
MAC        → Which resources/operations does policy allow?
```

## Interview Point

> **These controls complement each other; they are not interchangeable.**

---

# 563. Why should you avoid running a container with `--privileged`?

## Short Interview Answer

Because it weakens multiple security boundaries at once.

It can increase access to:

- kernel capabilities
- devices
- host resources
- privileged operations

If the application is compromised, the attacker may gain substantially more control over the host.

## Better Approach

Identify the exact requirement and grant the minimum permission necessary.

## Interview Point

> **Least privilege is safer than broad privilege escalation.**

---

# 564. What is the Docker socket and why is it sensitive?

## Short Interview Answer

The Docker daemon commonly exposes a Unix socket such as:

```text
/var/run/docker.sock
```

Clients use it to communicate with the Docker daemon.

Access to the Docker socket is highly privileged because the daemon can create containers, mount host paths, manipulate networking and perform other host-level operations.

## Dangerous Example

```yaml
volumes:
  - /var/run/docker.sock:/var/run/docker.sock
```

Giving an application unrestricted Docker socket access can effectively provide very powerful control over the host.

## Interview Point

> **Treat Docker daemon socket access as a highly privileged capability.**

---

# 565. Why is mounting `/var/run/docker.sock` into a container dangerous?

## Short Interview Answer

Because the container can potentially communicate with the Docker daemon and request privileged operations.

For example, access to the daemon may allow an application to request containers with host filesystem mounts or other powerful configurations.

Therefore:

```text
docker.sock access
        ↓
Docker API access
        ↓
potential host control
```

## Interview Trap

Do not describe Docker socket mounting as an ordinary "Docker integration."

It is a major privilege boundary.

## Interview Point

> **Docker socket access should be granted only with a strong, explicit security justification.**

---

# 566. Why should secrets not be stored in a Dockerfile?

## Short Interview Answer

Because Dockerfile instructions can become part of the image build history/layers or otherwise expose the secret to people or systems that can access the image/build metadata.

Bad:

```dockerfile
ENV DB_PASSWORD=SuperSecret
```

Also bad:

```dockerfile
RUN echo "SuperSecret" > /tmp/password
```

Even if a later layer deletes the file, the secret may remain in earlier build artifacts.

## Better Approach

Use an appropriate secret mechanism:

- BuildKit secret mounts for build-time secrets
- runtime secret management for application secrets

## Interview Point

> **Never treat Dockerfile instructions as a secure secret store.**

---

# 567. Why are `ARG` and `ENV` not appropriate for secrets?

## Short Interview Answer

Both can expose sensitive values through image/build metadata or runtime configuration.

Example:

```dockerfile
ARG TOKEN
ENV TOKEN=$TOKEN
```

does not make the token secure.

## Better

For build-time secrets, use a secret mount mechanism supported by BuildKit.

For runtime secrets, use the secret-management capability provided by the deployment environment.

## Interview Point

> **Visibility control and secret management are different problems.**

---

# 568. How do you handle build-time secrets securely?

## Short Interview Answer

Use BuildKit secret mounts so the secret is available to the specific build step without intentionally baking it into the resulting image.

Conceptually:

```text
Build secret
    ↓
temporary build step access
    ↓
secret not intentionally copied into final image
```

The exact command/configuration depends on the build workflow.

## Important

The application should not copy the secret into:

- the final filesystem
- logs
- image metadata
- generated artifacts

## Interview Point

> **A secure secret mount controls secret exposure during the build rather than turning the secret into image content.**

---

# 569. How should runtime application secrets be handled?

## Short Interview Answer

Use a dedicated secret-management mechanism appropriate to the environment rather than hard-coding secrets into images.

Examples include:

- orchestrator secret mechanisms
- cloud secret managers
- external secret-management systems
- controlled mounted secret files

The application should receive only the secrets it actually needs.

## Interview Point

> **Separate secret storage from the container image and minimize secret exposure.**

---

# 570. What is the difference between configuration and secrets?

## Short Interview Answer

Configuration controls application behavior and is not necessarily sensitive.

Examples:

```text
LOG_LEVEL=info
PORT=8080
DB_HOST=db
```

Secrets are sensitive credentials or sensitive values:

```text
DB_PASSWORD
API_TOKEN
PRIVATE_KEY
```

They should receive stronger protection.

## Interview Point

> **Not every environment variable is a secret, and not every secret should be treated as ordinary configuration.**

---

# 571. Why should you use read-only filesystems for containers when practical?

## Short Interview Answer

A read-only root filesystem reduces the locations where a compromised application can write persistent changes.

Example:

```bash
docker run --read-only myapp
```

If the application requires temporary writes, provide an explicit writable location such as an appropriate tmpfs or volume.

## Security Benefit

```text
Compromised application
        ↓
attempts to modify filesystem
        ↓
fewer writable locations
```

## Interview Point

> **Reduce the writable attack surface, but provide explicit writable paths required by the application.**

---

# 572. What is `--read-only`?

## Short Interview Answer

It makes the container's root filesystem read-only.

Example:

```bash
docker run --read-only myapp
```

The application may still need writable locations for:

- temporary files
- runtime sockets
- caches
- application data

Those should be explicitly provided where required.

## Interview Trap

Read-only root filesystem does not mean the container has no writable storage anywhere.

## Interview Point

> **Read-only root filesystem is a least-write principle, not an absolute no-write rule.**

---

# 573. Why are bind mounts a security concern?

## Short Interview Answer

A bind mount exposes a host filesystem path inside the container.

Example:

```bash
docker run \
  -v /host/config:/app/config \
  myapp
```

If the mount is writable, the container can modify the host files available through that path.

A dangerous example is exposing sensitive host directories.

## Better

Use:

```bash
--mount type=bind,src=/host/config,dst=/app/config,readonly
```

when write access is not required.

## Interview Point

> **Host filesystem mounts expand the container's access beyond its own filesystem.**

---

# 574. Why is mounting the host root filesystem dangerous?

## Short Interview Answer

A mount such as:

```bash
-v /:/host
```

exposes the host filesystem to the container.

If the container has write access and sufficient privileges, compromise of the application can become a host compromise.

## Interview Point

> **Never expose broad host filesystem paths unless there is an exceptional, well-understood requirement.**

---

# 575. What is the principle of least privilege in Docker?

## Short Interview Answer

Give a container only the permissions, filesystem access, network access and resources it actually requires.

Examples:

```text
non-root user
drop capabilities
read-only filesystem
minimal host mounts
restricted networks
minimal image
limited resources
controlled secrets
```

## Memory Map

```text
Need it?
  ├─ No → don't grant it
  └─ Yes → grant the smallest useful permission
```

## Interview Point

> **Least privilege should apply to identity, capabilities, filesystem, network and secrets.**

---

# 576. Why should Docker images be minimal?

## Short Interview Answer

A smaller image can reduce:

- attack surface
- number of vulnerable packages
- image size
- transfer time
- maintenance burden

For example, a runtime image does not normally need:

```text
compiler
source tree
package manager cache
debugging utilities
```

if they are unnecessary at runtime.

## Important

Minimal does not mean blindly removing everything. The image must still contain required runtime dependencies, certificates, users and configuration.

## Interview Point

> **Minimize unnecessary software while keeping the runtime complete and supportable.**

---

# 577. How does multistage build improve security?

## Short Interview Answer

Multistage builds allow build tools to remain in a builder stage while only runtime artifacts are copied into the final image.

Example:

```text
Builder
  ├── compiler
  ├── source code
  └── dependencies
        ↓
      binary
        ↓
Runtime image
  └── binary + runtime requirements
```

This reduces unnecessary software in the production image.

## Important

Multistage builds reduce image contents; they do not automatically make the application secure.

## Interview Point

> **Multistage builds reduce runtime attack surface by separating build dependencies from runtime dependencies.**

---

# 578. Why should production images be pinned instead of using `latest`?

## Short Interview Answer

`latest` is a mutable tag.

A future pull can resolve to different image content.

Using explicit version tags:

```text
myapp:1.4.2
```

or, for stronger immutability, a digest:

```text
myapp@sha256:...
```

makes deployments more predictable.

## Interview Point

> **Tags improve readability; digests provide content-addressed identity.**

---

# 579. What is image vulnerability scanning?

## Short Interview Answer

Image scanning analyzes image contents for known vulnerabilities in:

- OS packages
- application dependencies
- libraries

Example tools include:

```text
Trivy
Docker Scout
registry-integrated scanners
```

The exact findings depend on the scanner and vulnerability database.

## Important

Scanning is not a guarantee that an image is secure.

It should be combined with:

- trusted sources
- patching
- minimal images
- SBOM/provenance
- runtime controls
- secure deployment practices

## Interview Point

> **Scanning reduces known-vulnerability risk; it does not prove absence of vulnerabilities.**

---

# 580. What is an SBOM and why is it useful for Docker security?

## Short Interview Answer

SBOM means **Software Bill of Materials**.

It describes the software components and dependencies contained in an artifact.

Conceptually:

```text
Image
  ↓
SBOM
  ├── OS packages
  ├── libraries
  └── application dependencies
```

It helps with:

- vulnerability analysis
- incident response
- dependency visibility
- compliance
- supply-chain management

## Interview Point

> **An SBOM improves visibility into what is actually inside an image.**

---

# 581. What is image signing or verification?

## Short Interview Answer

Image signing helps establish trust in the origin and integrity of container artifacts.

The concept is:

```text
Publisher
   ↓
sign artifact
   ↓
Registry
   ↓
verify before deployment
```

This addresses a different problem from vulnerability scanning.

## Distinction

```text
Scanning → Is known-vulnerable software present?
Signing  → Can I trust the artifact's identity/integrity?
```

## Interview Point

> **Scanning and signing solve different supply-chain security problems.**

---

# 582. Why should base images come from trusted sources?

## Short Interview Answer

The base image becomes part of the application's software supply chain.

An untrusted or compromised base image can introduce:

- malicious code
- vulnerable packages
- unexpected utilities
- compromised dependencies

Use trusted publishers and controlled image sources.

## Interview Point

> **Your application image inherits the security posture of its base image and dependencies.**

---

# 583. Why should Docker images be regularly updated?

## Short Interview Answer

Base images and dependencies receive security fixes over time.

A secure image process should include:

```text
Monitor vulnerabilities
      ↓
Update dependencies/base image
      ↓
Rebuild
      ↓
Scan
      ↓
Test
      ↓
Release
```

## Interview Point

> **Security is a continuous lifecycle, not a one-time image build.**

---

# 584. How do you secure Docker networking?

## Short Interview Answer

I apply network segmentation and expose only required ports.

Principles:

- do not publish internal services unnecessarily
- separate frontend/backend/database networks where appropriate
- restrict external access with host/cloud firewalls
- avoid exposing databases directly to the Internet
- use TLS where required
- restrict inter-service communication according to application needs

Example:

```text
Internet
   ↓
Reverse Proxy
   ↓
Application Network
   ↓
Backend
   ↓
Database Network
```

## Interview Point

> **Network security starts with reducing unnecessary exposure.**

---

# 585. Why should databases generally not publish ports to the Internet?

## Short Interview Answer

A database normally needs to be reachable only by trusted application components.

Instead of:

```text
Internet → Database
```

prefer:

```text
Internet
   ↓
Application
   ↓
Database
```

This reduces attack surface and allows database access to be controlled by application/network boundaries.

## Interview Point

> **Only expose services that genuinely require external access.**

---

# 586. How do Docker networks contribute to security?

## Short Interview Answer

Separate Docker networks can limit which services can communicate.

For example:

```text
frontend-net:
frontend ↔ api

backend-net:
api ↔ db
```

The database does not need to be attached to the public-facing network.

## Interview Point

> **Network segmentation reduces unnecessary service-to-service reachability.**

---

# 587. What is Docker Content Trust?

## Short Interview Answer

Docker Content Trust is a mechanism historically associated with signing and verifying Docker image metadata using Notary.

The broader security principle is:

```text
publisher identity
      +
artifact integrity
      ↓
verification
```

When discussing modern production supply-chain security, also consider current signing, provenance and verification tooling rather than assuming Content Trust alone covers every requirement.

## Interview Point

> **The important interview concept is trusted artifact verification, not simply the name of one signing mechanism.**

---

# 588. How do you secure the Docker daemon?

## Short Interview Answer

I protect the Docker daemon and its control interface because daemon access is highly privileged.

Practices include:

- restrict Docker socket access
- use least privilege
- avoid exposing the daemon API unnecessarily
- secure remote daemon communication with appropriate TLS/authentication when required
- patch Docker and the host
- monitor daemon activity
- protect the host itself

## Interview Point

> **Protecting the Docker daemon is effectively protecting a privileged host-control interface.**

---

# 589. Why is exposing the Docker daemon API over an unsecured TCP port dangerous?

## Short Interview Answer

The Docker API provides powerful control over containers and host resources.

If exposed without proper authentication and transport security, an attacker may be able to:

- create containers
- mount host paths
- manipulate workloads
- access sensitive data
- potentially compromise the host

## Interview Point

> **The Docker API is a privileged management interface, not a normal application endpoint.**

---

# 590. How do resource limits contribute to security?

## Short Interview Answer

Resource limits can reduce the impact of resource-exhaustion attacks and runaway applications.

Examples:

```bash
docker run \
  --memory=512m \
  --cpus=1.0 \
  --pids-limit=200 \
  myapp
```

They can limit:

- memory
- CPU
- process count
- some I/O resources

## Important

Resource limits are availability controls, not complete security controls.

## Interview Point

> **Resource limits help contain blast radius from runaway workloads and some denial-of-service conditions.**

---

# 591. What is a PID limit and why is it useful?

## Short Interview Answer

A PID limit restricts how many processes a container can create.

Example:

```bash
docker run --pids-limit=200 myapp
```

This can reduce the impact of process-exhaustion attacks or accidental process explosions.

## Interview Point

> **PID limits protect the host and workload from uncontrolled process creation.**

---

# 592. What is Docker rootless mode?

## Short Interview Answer

Rootless Docker runs the Docker daemon and containers without requiring the daemon to run as root.

This can reduce the impact of certain container/runtime compromises because the Docker daemon itself does not have normal root privileges.

## Important

Rootless mode has compatibility and feature differences depending on workload and host configuration.

It should not be treated as a replacement for all other security controls.

## Interview Point

> **Rootless mode reduces daemon privilege, adding another security layer.**

---

# 593. What is the difference between rootless Docker and a non-root container?

## Short Interview Answer

They solve different problems.

### Non-root container

The application process inside the container runs as a non-root user.

### Rootless Docker

The Docker daemon and containers are operated without requiring a root daemon.

```text
Non-root container
→ application privilege

Rootless Docker
→ daemon/runtime privilege
```

They can be used together.

## Interview Point

> **Container user identity and Docker daemon privilege are separate security layers.**

---

# 594. How would you secure a production Dockerfile?

## Strong Interview Answer

I would consider:

```dockerfile
FROM trusted-base:version

# install only required runtime dependencies

RUN adduser --disabled-password appuser

COPY --chown=appuser:appuser app /app

WORKDIR /app

USER appuser

CMD ["./app"]
```

Then I would also:

- use a minimal appropriate base
- pin versions where practical
- use multistage builds
- exclude unnecessary files with `.dockerignore`
- avoid secrets in `ARG`/`ENV`
- run vulnerability scanning
- generate/track SBOM where required
- keep runtime image minimal
- configure healthchecks where useful

## Interview Point

> **A secure Dockerfile follows least privilege, minimality, reproducibility and supply-chain hygiene.**

---

# 595. Give an insecure Dockerfile and explain why it is insecure.

## Example

```dockerfile
FROM ubuntu:latest

ENV DB_PASSWORD=SuperSecret

COPY . /app

RUN apt-get update

RUN apt-get install -y curl vim gcc

USER root

CMD ["./start.sh"]
```

## Problems

### `ubuntu:latest`

Mutable base reference.

### Secret in `ENV`

Sensitive information is placed into image configuration.

### `COPY . /app`

May include:

- `.git`
- credentials
- logs
- build artifacts

if `.dockerignore` is not configured.

### Unnecessary packages

Increase image size and attack surface.

### `USER root`

Application runs with unnecessary privilege.

### Package cleanup/reproducibility

The package installation is not optimized or tightly controlled.

## Better Direction

Use:

```text
trusted/pinned base
↓
multistage build
↓
minimal runtime
↓
non-root user
↓
no baked secrets
↓
small build context
↓
security scanning
```

---

# 596. What is the Docker security checklist you would use in production?

## Image

- trusted base image
- minimal runtime image
- pinned version/digest where appropriate
- vulnerability scanning
- SBOM/provenance where required
- image signing/verification where required
- regular rebuilds

## Runtime

- non-root user
- drop unnecessary capabilities
- avoid `--privileged`
- read-only filesystem where practical
- limit resources
- limit PIDs
- use appropriate seccomp/MAC policies

## Filesystem

- avoid broad host bind mounts
- use read-only mounts where possible
- protect sensitive host paths
- use persistent volumes for required data

## Secrets

- never bake secrets into images
- avoid secrets in Dockerfile `ARG`/`ENV`
- use dedicated secret-management mechanisms
- rotate compromised credentials

## Network

- expose only required ports
- segment services
- do not expose databases unnecessarily
- restrict ingress/egress
- use TLS where required

## Daemon/Host

- protect Docker socket
- secure remote Docker API
- patch Docker and kernel
- least-privilege host access
- monitor the host and daemon

---

# 597. Scenario: An interviewer says "Make this container more secure." What would you do?

## Strong Interview Answer

I would first identify the application's actual requirements and then reduce privileges.

My baseline would be:

```text
Trusted minimal image
        ↓
Multistage build
        ↓
Non-root USER
        ↓
Drop unnecessary capabilities
        ↓
Avoid privileged mode
        ↓
Read-only root filesystem where practical
        ↓
Minimal host mounts
        ↓
Restricted networks
        ↓
Resource/PID limits
        ↓
Secure secret handling
        ↓
Image scanning/signing/provenance
        ↓
Patched Docker host
```

I would verify that the security controls do not break required functionality and document any justified exceptions.

## Interview Point

> **Security hardening should be requirement-driven and least-privilege based.**

---

# 598. Scenario: A developer asks for `--privileged` because the container gets "permission denied." What do you do?

## Strong Interview Answer

I would not immediately grant `--privileged`.

First I would identify the exact denied operation.

Then check whether it requires:

- a specific Linux capability
- a device
- a filesystem permission
- a security policy adjustment
- a user/UID change
- a kernel feature

If one capability is sufficient:

```bash
--cap-add=<required-capability>
```

is preferable to:

```bash
--privileged
```

## Interview Point

> **Fix the exact permission requirement instead of granting broad privileges.**

---

# 599. Scenario: A secret was accidentally baked into an image. What do you do?

## Strong Interview Answer

I treat it as a potential credential compromise.

### Immediate action

1. Rotate/revoke the exposed secret.
2. Identify where the secret was exposed.
3. Remove the secret from the Dockerfile/build process.
4. Rebuild the image without the secret.
5. Replace deployed instances using the compromised artifact.
6. Review logs, registry access and CI/CD systems as appropriate.
7. Verify the secret is no longer being used.

## Critical Point

Simply deleting the latest image tag is not enough if the secret was already exposed or the image was distributed.

## Interview Point

> **Secret remediation starts with rotation/revocation, not merely deleting the image.**

---

# 600. Scenario: A container needs to write to the host. How do you evaluate the risk?

## Strong Interview Answer

I ask whether host write access is actually required.

If required:

1. identify the smallest host path
2. use the narrowest permissions
3. prefer read-only access when possible
4. align UID/GID permissions
5. avoid sensitive host directories
6. document why the mount is required
7. consider an alternative such as a Docker volume

Example:

```bash
--mount type=bind,src=/safe/app-data,dst=/app/data
```

rather than:

```bash
-v /:/host
```

## Interview Point

> **Host filesystem access should be narrowly scoped because it expands the container's trust boundary.**

---

# 601. How would you explain Docker security in one interview answer?

## Final Interview Answer

> "Docker security is layered. Containers share the host kernel, so I don't treat them as equivalent to a separate VM security boundary. At the image level I use trusted and minimal base images, pinned versions where appropriate, vulnerability scanning and supply-chain controls. At runtime I run applications as non-root, drop unnecessary Linux capabilities, avoid privileged mode, use read-only filesystems where practical and apply resource limits. I protect secrets using dedicated secret-management mechanisms rather than baking them into images. I restrict network exposure and avoid unnecessary host bind mounts or Docker socket access. Finally, I secure and patch the Docker daemon and host kernel. The overall principle is least privilege and minimizing the container's attack surface."

---

# Quick Revision

| Security Control | Primary Purpose |
|---|---|
| Non-root `USER` | Reduce application privilege |
| Capabilities | Fine-grained privileged operations |
| `--cap-drop` | Remove unnecessary capabilities |
| `--privileged` | Broad privilege relaxation; high risk |
| Namespaces | Isolation/visibility |
| cgroups | Resource control |
| seccomp | Restrict system calls |
| AppArmor | Application confinement |
| SELinux | Mandatory access control |
| `--read-only` | Reduce writable filesystem |
| Volumes | Controlled persistent storage |
| Bind mounts | Host path access; security-sensitive |
| Docker socket | Highly privileged daemon access |
| Rootless Docker | Reduce daemon privilege |
| Image scanning | Find known vulnerabilities |
| SBOM | Component/dependency visibility |
| Signing | Artifact trust/integrity |
| Minimal image | Reduce attack surface |
| Multistage build | Keep build tools out of runtime |
| Resource limits | Reduce resource-exhaustion impact |
| Network segmentation | Reduce unnecessary reachability |
| Secret management | Protect sensitive credentials |

---

# High-Value Interview Traps

## Trap 1 — Containers are completely isolated

False.

They share the host kernel.

---

## Trap 2 — Container root is harmless

False.

Root inside a container is still privileged within the container, and dangerous runtime configuration can greatly expand its effective power.

---

## Trap 3 — `--privileged` fixes permissions

Technically it may make many permissions available, but it is usually an unnecessarily broad security relaxation.

Find the exact required capability instead.

---

## Trap 4 — Non-root means fully secure

False.

Non-root is one layer.

You still need:

```text
capabilities
seccomp/MAC
filesystem controls
network controls
host security
image security
```

---

## Trap 5 — `ARG` is safe for secrets

False.

Build arguments are not a secure secret-management mechanism.

---

## Trap 6 — Delete the secret from the final layer

Not sufficient.

Secrets can remain in previous image/build artifacts.

---

## Trap 7 — Docker socket is just an API file

False.

Access to the Docker daemon can provide extremely powerful control over the host.

---

## Trap 8 — Image scanning means the image is secure

False.

Scanning mainly detects known vulnerabilities.

It does not prove:

```text
no vulnerability
no malicious code
no configuration problem
no runtime exploit
```

---

## Trap 9 — Read-only filesystem means no writable storage

False.

You can explicitly provide writable volumes/tmpfs where required.

---

## Trap 10 — Rootless Docker = non-root container

False.

They address different privilege layers.

---

## Trap 11 — Smaller image automatically means secure image

False.

Small size helps reduce attack surface, but security also depends on configuration, dependencies, runtime privileges and host security.

---

## Trap 12 — Docker security is only container security

False.

The Docker daemon, host kernel, registry, CI/CD pipeline and software supply chain are all part of the security boundary.

---

# Interview Follow-Up Questions

1. Are containers a security boundary?
2. Why should containers run as non-root?
3. What is the difference between root in a container and root on the host?
4. What are Linux capabilities?
5. Why use `--cap-drop=ALL`?
6. What does `--privileged` do?
7. Why is `--privileged` dangerous?
8. What is seccomp?
9. What is AppArmor?
10. What is SELinux?
11. How do seccomp and capabilities differ?
12. Why is Docker socket access dangerous?
13. Why should secrets not be stored in `ENV`?
14. Why should secrets not be passed through `ARG`?
15. How do you handle build-time secrets?
16. How do you handle runtime secrets?
17. Why use a read-only filesystem?
18. Why are bind mounts security-sensitive?
19. How does multistage build improve security?
20. Why should `latest` be avoided in production?
21. What is image vulnerability scanning?
22. What is an SBOM?
23. What is image signing?
24. How do you secure Docker networking?
25. How do resource limits improve security?
26. What is rootless Docker?
27. Rootless Docker vs non-root container?
28. How would you harden a production Dockerfile?
29. How would you respond to a leaked Docker image secret?
30. How would you respond to a developer requesting `--privileged`?

---

# Final Interview Answer

If asked:

> **"What are the most important Docker security best practices?"**

Answer:

> "My baseline is least privilege and attack-surface reduction. I use trusted, minimal and regularly rebuilt images, scan them for known vulnerabilities, and apply appropriate supply-chain controls. Containers should run as non-root users, unnecessary capabilities should be dropped, and privileged mode should be avoided. I use seccomp and Linux MAC controls where appropriate, make the root filesystem read-only where practical, minimize host filesystem mounts, and never expose the Docker socket unnecessarily. Secrets should not be baked into images or stored casually in Dockerfile arguments or environment variables. I also restrict network exposure, apply resource limits, protect the Docker daemon, and keep the host kernel and Docker components patched. Finally, I validate the security controls against the application's actual requirements rather than granting broad privileges as a shortcut."

---

# One-Line Memory Map

```text
SECURE DOCKER
     │
     ├── Image
     │    ├── Trusted
     │    ├── Minimal
     │    ├── Scanned
     │    └── Signed / Provenance
     │
     ├── Runtime
     │    ├── Non-root
     │    ├── Drop capabilities
     │    ├── No privileged mode
     │    ├── Read-only FS
     │    └── Resource limits
     │
     ├── Kernel Security
     │    ├── Namespaces
     │    ├── seccomp
     │    └── SELinux/AppArmor
     │
     ├── Secrets
     │    └── Dedicated secret management
     │
     ├── Network
     │    ├── Least exposure
     │    └── Segmentation
     │
     └── Host / Daemon
          ├── Protect docker.sock
          ├── Patch kernel/Docker
          └── Least privilege
```

---

# Topic 19 Complete

**Questions covered: Q549–Q601**

**Core skill:**

> **Reduce privilege, reduce attack surface, protect secrets, restrict connectivity, secure the daemon/host, and verify the entire image-to-runtime supply chain.**
