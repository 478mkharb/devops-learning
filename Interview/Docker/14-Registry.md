# Docker Interview Preparation — Topic 14: Docker Registries

> **Interview focus:** Understand what a container registry is, how images are named and transferred, authentication and tagging, image digests, private registries, push/pull workflows, and how registries fit into CI/CD.

---

## Q325. What is a Docker registry?

### Short Interview Answer

A Docker registry is a service that stores and distributes container images and related image artifacts.

It allows users and systems to:

- Push images
- Pull images
- Store multiple image versions
- Authenticate users
- Control access to private repositories

Examples include:

- Docker Hub
- Amazon ECR
- GitHub Container Registry
- Google Artifact Registry
- Azure Container Registry
- Self-hosted OCI-compatible registries

### Conceptual flow

```text
Developer / CI
      │
      │ docker push
      ▼
┌──────────────────┐
│ Container Registry│
└──────────────────┘
      │
      │ docker pull
      ▼
Production host
```

### Interview Point

> A registry is the distribution and storage system for container images; Docker Engine is the system that builds and runs containers.

---

## Q326. What is the difference between a Docker registry and a repository?

### Short Interview Answer

A **registry** is the overall service that stores and distributes images, while a **repository** is a named collection/location within that registry used for related image artifacts and references.

Example:

```text
registry.example.com/
└── team/
    └── payment-api
```

Here:

```text
registry.example.com → registry
team/payment-api      → repository
```

A repository can have multiple tags:

```text
payment-api:1.0
payment-api:1.1
payment-api:2.0
```

### Interview Trap

Do not use "registry" and "repository" as exact synonyms.

### Interview Point

> Registry is the service; repository is a named image namespace inside it.

---

## Q327. What is Docker Hub?

### Short Interview Answer

Docker Hub is a public container registry service commonly used to store and distribute Docker/OCI images.

Typical image references include:

```text
nginx
ubuntu
redis
```

For example:

```bash
docker pull nginx
```

If no registry hostname is explicitly supplied, Docker commonly resolves the reference to Docker Hub.

### Important distinction

Docker Hub is:

```text
a registry service
```

It is not:

```text
Docker Engine
Docker daemon
Docker image
Docker container
```

### Interview Point

> Docker Hub is one registry; Docker itself is the container platform/tooling.

---

# Image Naming

## Q328. Explain a Docker image reference.

### Short Interview Answer

An image reference identifies where an image is located and which repository/tag or digest should be used.

A common form is:

```text
[registry/]repository[:tag]
```

For example:

```text
registry.example.com/team/payment-api:1.4
```

Breakdown:

```text
registry.example.com → registry
team/payment-api      → repository
1.4                   → tag
```

A digest reference looks like:

```text
registry.example.com/team/payment-api@sha256:<digest>
```

### Example

```bash
docker pull registry.example.com/team/payment-api:1.4
```

### Interview Point

> Image references identify the registry/repository and the desired version reference.

---

## Q329. What happens if you run `docker pull nginx`?

### Short Interview Answer

Docker resolves the image reference, contacts the configured registry, retrieves the required image metadata and layers, verifies content as appropriate, and stores the image locally.

Conceptually:

```text
docker pull nginx
      ↓
resolve image reference
      ↓
contact registry
      ↓
retrieve manifest/index
      ↓
retrieve required layers
      ↓
store locally
```

If the image is multi-platform, the registry may return an image index from which Docker selects the appropriate platform-specific manifest.

### Verify

```bash
docker image ls nginx
```

### Interview Point

> `docker pull` downloads an image artifact from a registry into the local image store.

---

## Q330. What happens during `docker push`?

### Short Interview Answer

Docker authenticates with the registry, checks which image layers are already present remotely, uploads missing layers and metadata, and publishes the image reference.

Example:

```bash
docker tag myapp:1.0 registry.example.com/team/myapp:1.0
docker push registry.example.com/team/myapp:1.0
```

Conceptually:

```text
Local image
    ↓
docker push
    ↓
Registry authentication
    ↓
Check existing blobs
    ↓
Upload missing layers
    ↓
Upload/publish manifest
```

### Why only some layers may upload

Registries store content by digest. If an identical layer already exists, it can be reused rather than uploaded again.

### Interview Point

> Docker push is content-aware; already-present image layers do not necessarily need to be uploaded again.

---

## Q331. Why do you need to tag an image before pushing it to a private registry?

### Short Interview Answer

The image reference used by `docker push` tells Docker exactly which registry and repository should receive the image.

Example local image:

```text
myapp:1.0
```

Tag it:

```bash
docker tag myapp:1.0 \
  registry.example.com/team/myapp:1.0
```

Push:

```bash
docker push \
  registry.example.com/team/myapp:1.0
```

The tag creates another local reference to the same image content.

### Interview Point

> The registry hostname in the image reference determines where the push is targeted.

---

## Q332. Does `docker tag` copy or rebuild an image?

### Short Interview Answer

No. `docker tag` creates another name/reference for the same local image content. It does not rebuild the image.

Example:

```bash
docker tag myapp:1.0 myapp:stable
```

Now both references can point to the same image:

```text
myapp:1.0
      \
       → same image content

myapp:stable
```

Verify:

```bash
docker image ls
```

### Interview Point

> Tagging changes the reference, not the underlying image content.

---

# Authentication

## Q333. How do you authenticate to a private registry?

### Short Interview Answer

Use the registry's authentication mechanism, commonly through:

```bash
docker login
```

Example:

```bash
docker login registry.example.com
```

Docker stores credentials through its configured credential-management mechanism.

### CI/CD consideration

Avoid putting passwords directly into scripts or command arguments when possible.

Prefer:

- CI/CD secret stores
- Registry-specific tokens
- Cloud IAM mechanisms
- Credential helpers
- Short-lived credentials where supported

### Interview Point

> Registry authentication should use protected credentials and should not be hard-coded into CI/CD source code.

---

## Q334. What does `docker login` do?

### Short Interview Answer

`docker login` authenticates the Docker client against a registry so it can perform operations requiring authorization, such as pulling private images or pushing images.

Example:

```bash
docker login registry.example.com
```

After authentication:

```bash
docker pull registry.example.com/team/private-app:1.0
```

or:

```bash
docker push registry.example.com/team/private-app:1.0
```

may be authorized according to the account's permissions.

### Important distinction

Authentication answers:

> "Who are you?"

Authorization answers:

> "What are you allowed to do?"

### Interview Point

> Login establishes credentials; the registry still applies authorization policies.

---

## Q335. What is the difference between authentication and authorization in a registry?

### Short Interview Answer

**Authentication** verifies identity. **Authorization** determines what that identity can access or modify.

Example:

```text
Developer logs in
       ↓
Authentication
       ↓
Identity = mukesh
       ↓
Authorization
       ↓
Can push team/payment-api?
```

A user may successfully authenticate but still receive:

```text
denied
```

when attempting to push to a repository they cannot modify.

### Interview Point

> Successful login does not automatically imply push permission.

---

# Private Registries

## Q336. What is a private container registry?

### Short Interview Answer

A private registry restricts access to authorized users, systems, or identities.

It is commonly used for:

- Internal applications
- Proprietary software
- Production images
- Security-controlled CI/CD pipelines
- Organization-specific base images

Conceptually:

```text
Developer
    │
    │ authenticated push
    ▼
Private Registry
    │
    │ authorized pull
    ▼
Production
```

### Interview Point

> Private registries provide controlled distribution of non-public container artifacts.

---

## Q337. Why would an organization use a private registry instead of Docker Hub?

### Strong Interview Answer

An organization may need:

- Private image storage.
- Fine-grained access control.
- Integration with cloud IAM.
- Network restrictions.
- Vulnerability scanning.
- Auditability.
- Organization-specific repositories.
- Integration with CI/CD.
- Control over image retention and lifecycle.

### Example architecture

```text
Git repository
      ↓
CI/CD
      ↓
Build image
      ↓
Private registry
      ↓
Deployment platform
```

### Interview Point

> The registry becomes a controlled artifact repository in the software supply chain.

---

# Image Digests

## Q338. What is an image digest?

### Short Interview Answer

An image digest is a content-addressed cryptographic identifier, commonly represented using SHA-256, for an image artifact or related registry object.

Example:

```text
myapp@sha256:abcdef...
```

Unlike a tag, a digest identifies specific content.

### Tag

```text
myapp:1.0
```

can be moved to different content.

### Digest

```text
myapp@sha256:abcd...
```

identifies the referenced content.

### Interview Point

> Tags are mutable references; digests provide content-addressed identity.

---

## Q339. What is the difference between an image tag and digest?

### Short Interview Answer

A tag is a human-friendly mutable reference, while a digest is a content-addressed immutable identifier for the referenced artifact.

| | Tag | Digest |
|---|---|---|
| Human-friendly | Yes | Less so |
| Mutable | Yes | No, for the referenced content |
| Content-addressed | No | Yes |
| Good for reproducibility | Less reliable | Strong |
| Example | `myapp:1.0` | `myapp@sha256:...` |

### Example

```bash
docker pull myapp:1.0
```

may resolve to one digest today and another later if the tag is republished.

Digest:

```bash
docker pull myapp@sha256:...
```

pins the reference to specific content.

### Interview Point

> For reproducible deployments, digest pinning is stronger than relying only on tags.

---

## Q340. Why is using only `latest` risky in production?

### Short Interview Answer

`latest` is a mutable tag, not a guarantee that the image is the newest, safest, or previously deployed version.

For example:

```text
myapp:latest
```

could point to:

```text
version A
```

today and:

```text
version B
```

later.

### Problems

- Unclear version history.
- Unexpected deployments.
- Difficult rollback.
- Reduced reproducibility.
- Harder incident analysis.

### Better

Use meaningful version tags:

```text
myapp:2.7.3
```

and, where appropriate, pin the deployment to a digest:

```text
myapp:2.7.3@sha256:...
```

### Interview Point

> A tag communicates intent; a digest provides exact content identity.

---

# Registry Storage and Layers

## Q341. Does a registry store Docker containers?

### Short Interview Answer

No. A registry stores container images and related artifacts, not running containers.

### Registry

```text
Image
├── manifest
├── configuration
└── layers
```

### Docker host

```text
Container
├── process
├── network namespace
├── writable layer
└── mounted storage
```

### Important distinction

```text
Registry → image distribution
Docker Engine → container creation/execution
```

### Interview Point

> A registry stores artifacts; the Docker host runs containers.

---

## Q342. Does a registry store image layers separately?

### Short Interview Answer

Yes. Image content is composed of content-addressed blobs/layers referenced by image metadata.

Conceptually:

```text
Manifest
   │
   ├── Layer A
   ├── Layer B
   └── Layer C
```

If another image references the same layer content, the registry can reuse that blob rather than storing duplicate content conceptually.

### Why useful?

It enables:

- Layer reuse.
- Efficient pushes.
- Efficient pulls.
- Deduplication based on content identity.

### Interview Point

> Registry storage is optimized around reusable content-addressed artifacts.

---

# Registry and CI/CD

## Q343. What is a typical Docker CI/CD registry workflow?

### Strong Interview Answer

A typical workflow is:

```text
Developer
   ↓
Git commit
   ↓
CI pipeline
   ↓
Run tests/security checks
   ↓
Build image
   ↓
Tag image with release identifier
   ↓
Authenticate to registry
   ↓
Push image
   ↓
Deployment system pulls image
   ↓
Run application
```

Example:

```bash
docker build -t registry.example.com/team/app:1.2.0 .

docker push registry.example.com/team/app:1.2.0
```

Deployment:

```bash
docker pull registry.example.com/team/app:1.2.0
```

### Better release identification

Use a unique immutable release identifier, such as:

```text
1.2.0
```

or:

```text
git-<commit-sha>
```

and optionally deploy by digest.

### Interview Point

> The registry acts as the artifact handoff point between image-building and image-running stages.

---

## Q344. Why should CI/CD avoid overwriting a released image tag?

### Short Interview Answer

Overwriting a released tag makes deployments less reproducible because the same tag can refer to different image contents at different times.

Bad:

```text
release:1.0
   ↓
image A

later

release:1.0
   ↓
image B
```

Now historical deployment records become ambiguous.

### Better

Use immutable version identifiers:

```text
app:1.0.0
app:1.0.1
app:1.1.0
```

or commit-based tags:

```text
app:git-a81f2c9
```

and optionally deploy by digest.

### Interview Point

> Immutable release references simplify rollback, auditing, and incident investigation.

---

## Q345. What tags would you use in a CI/CD pipeline?

### Strong Interview Answer

I would normally use a unique immutable build identifier, such as:

```text
app:1.4.2
```

or:

```text
app:git-8f3a21c
```

I may also maintain a convenience tag such as:

```text
app:stable
```

but production deployment should not depend solely on a mutable convenience tag.

### Example

```text
registry.example.com/team/app:1.4.2
registry.example.com/team/app:git-8f3a21c
registry.example.com/team/app:stable
```

### Deployment

Prefer:

```text
app@sha256:<digest>
```

when exact content pinning is required.

### Interview Point

> Use unique release tags for traceability and digests when exact artifact identity matters.

---

# Pull and Push Failures

## Q346. `docker push` returns "denied". What would you check?

### Troubleshooting sequence

### 1. Check image tag

```bash
docker image ls
```

Verify that the image is tagged for the correct registry/repository:

```bash
docker image ls \
  registry.example.com/team/app
```

### 2. Check authentication

```bash
docker login registry.example.com
```

### 3. Check authorization

Confirm the authenticated identity has permission to push to:

```text
team/app
```

### 4. Check repository/namespace

A typo can cause:

```text
wrong repository
wrong organization
wrong registry
```

### 5. Check registry policy

The registry may enforce:

- Repository existence.
- Immutable tags.
- Network restrictions.
- Token scope.
- Organization policies.

### Interview Point

> "Denied" is usually an authentication/authorization/repository-policy problem, not automatically a Docker build problem.

---

## Q347. `docker pull` returns "unauthorized". What does that mean?

### Short Interview Answer

It generally means the registry requires authorization for the requested image and the current credentials are missing, invalid, or insufficient.

Check:

```bash
docker login registry.example.com
```

Then:

```bash
docker pull registry.example.com/team/private-app:1.0
```

Also verify that the identity has **pull/read** permission.

### Interview Point

> Push and pull permissions can be different; successful login alone does not guarantee image access.

---

## Q348. `docker pull` says "manifest unknown". What could be wrong?

### Short Interview Answer

The requested repository/tag or manifest may not exist, or the requested platform/manifest may not be available.

Example:

```bash
docker pull registry.example.com/team/app:9.9.9
```

If that tag does not exist, the registry may return a manifest-related error.

### Troubleshooting

Check:

```text
Registry hostname
Repository name
Tag
Platform
Image publication status
```

### Interview Point

> A manifest error often means the requested image reference cannot be resolved to an available manifest.

---

# Multi-Platform Images

## Q349. How does a registry handle multi-platform images?

### Short Interview Answer

A registry can store an image index that points to platform-specific image manifests.

Conceptually:

```text
app:1.0
   │
   ▼
Image Index
   ├── linux/amd64 manifest
   ├── linux/arm64 manifest
   └── other platform manifest
```

A client requests an image:

```bash
docker pull app:1.0
```

and Docker selects the appropriate platform variant.

### Why useful?

The same logical image reference can work across:

```text
x86_64
ARM64
```

without requiring the user to manually choose a platform in normal cases.

### Interview Point

> Multi-platform support is represented through an index that references platform-specific manifests.

---

## Q350. What is the difference between a registry, manifest, and image index?

### Short Interview Answer

They operate at different levels.

```text
Registry
   │
   ├── Repository
   │      │
   │      ├── Image index
   │      │      ├── Manifest for amd64
   │      │      └── Manifest for arm64
   │      │
   │      └── blobs/layers
```

### Registry

Stores and distributes artifacts.

### Manifest

Describes a specific image artifact, including its configuration and layer references.

### Image index

References multiple platform-specific manifests.

### Interview Point

> Registry is the service; manifest describes an image artifact; index groups platform-specific manifests under one logical reference.

---

# Private Registry Security

## Q351. How would you secure a private Docker registry?

### Strong Interview Answer

I would consider:

- TLS for registry communication.
- Strong authentication.
- Least-privilege authorization.
- Short-lived credentials where supported.
- Vulnerability scanning.
- Image signing/verification where required.
- Network restrictions.
- Audit logging.
- Retention policies.
- Protection of registry credentials.
- Immutable release tags where supported.
- Access control for CI/CD identities.

### Supply-chain view

```text
Source
  ↓
Build
  ↓
Security scan
  ↓
Sign/attest if required
  ↓
Registry
  ↓
Verified deployment
```

### Interview Point

> Registry security is part of the software supply chain, not merely a login problem.

---

## Q352. What is image scanning in a registry?

### Short Interview Answer

Image scanning analyzes image contents for known vulnerabilities or policy violations.

It may inspect:

```text
OS packages
Application dependencies
Known CVEs
Configuration issues
Licensing/policy rules
```

Conceptually:

```text
Build image
    ↓
Push registry
    ↓
Scan
    ↓
Vulnerabilities?
   / \
 yes  no
  ↓    ↓
block  approve
```

### Important nuance

Scanning is not proof that an image is completely secure. It generally identifies known issues based on available vulnerability intelligence and scanning rules.

### Interview Point

> Vulnerability scanning reduces risk but does not guarantee an image is vulnerability-free.

---

## Q353. What is an image signing/verification concept?

### Short Interview Answer

Image signing allows an organization to establish trust in the provenance or publisher of an image and verify that the deployed artifact meets its trust policy.

Conceptually:

```text
Build
  ↓
Sign artifact
  ↓
Registry
  ↓
Verify signature
  ↓
Deploy
```

This can help answer:

> "Did this image come from a trusted build process or signer?"

### Important distinction

A digest answers:

> "What exact content is this?"

A signature helps answer:

> "Who/what attested or signed this content?"

### Interview Point

> Integrity and provenance are related but distinct from simple image tagging.

---

# Registry Storage and Cleanup

## Q354. Why do registries need image retention policies?

### Short Interview Answer

CI/CD pipelines can create many images and tags. Without lifecycle management, registry storage can grow continuously.

Example:

```text
Every commit
   ↓
new image
   ↓
new layers/tags
   ↓
registry growth
```

Retention policies can remove artifacts that are no longer required according to organizational rules.

### Important consideration

Do not delete artifacts still needed for:

- Production rollback.
- Disaster recovery.
- Compliance.
- Reproducible builds.
- Active deployments.

### Interview Point

> Registry cleanup should balance storage cost with rollback, audit, and recovery requirements.

---

## Q355. What happens if you delete a tag from a registry?

### Short Interview Answer

It removes that tag/reference, but the underlying image content may still be retained if other tags or references point to it or if the registry's garbage-collection rules have not removed the underlying blobs.

### Conceptually

```text
app:1.0 ─────┐
             ├── image content
app:stable ──┘
```

Delete:

```text
app:1.0
```

The content may still be reachable through:

```text
app:stable
```

### Important nuance

Registry implementations differ in how tag deletion and garbage collection are handled.

### Interview Point

> Deleting a tag is not necessarily equivalent to immediately deleting all underlying image data.

---

# Practical Registry Scenario

## Q356. Your application works locally, but production cannot pull the image. How would you troubleshoot?

### Strong Interview Answer

I would separate the problem into image identity, registry access, network access, and authorization.

### Step 1 — Verify the image reference

```text
registry.example.com/team/app:1.2.0
```

Check:

```text
registry hostname
repository
tag
```

### Step 2 — Verify the image exists

From an authorized environment:

```bash
docker pull registry.example.com/team/app:1.2.0
```

### Step 3 — Check production authentication

Verify the production host/deployment identity has registry credentials or cloud IAM access as appropriate.

### Step 4 — Check authorization

The identity needs permission to pull the repository.

### Step 5 — Check network connectivity

Verify the production environment can reach the registry endpoint over the required protocol/port.

For HTTPS:

```bash
curl -v https://registry.example.com
```

The exact endpoint/API behavior varies by registry, so HTTP success alone is not sufficient proof of image-pull authorization.

### Step 6 — Check platform compatibility

If the image is multi-platform, verify that the required platform is published.

### Step 7 — Check registry/image policy

Look for:

- Deleted tag.
- Immutable-tag policy.
- Expired credential.
- Repository restriction.
- Registry outage.
- Network policy.

### Interview Point

> "Works locally" proves the image exists in the local image store; it does not prove production can authenticate, reach, authorize, and resolve the same registry reference.

---

## Q357. What would you do if production is running an image that has since been deleted from the registry?

### Short Interview Answer

If the image is already present on the production host, the running container can continue using its local image content even if the registry copy is later unavailable. However, future container recreation or deployment may fail if the image cannot be pulled again.

### Risk

```text
Registry image deleted
       ↓
Existing container
       ↓
may continue running

Container recreated
       ↓
pull required
       ↓
image unavailable
       ↓
deployment failure
```

### Better operational practice

Maintain:

- Proper retention policies.
- Rollback images.
- Immutable artifact references.
- Registry backups/replication where required.
- Deployment records containing image digests.

### Interview Point

> Registry availability is especially important when containers must be recreated, scaled, or recovered.

---

# Quick Revision

| Topic | Key Point |
|---|---|
| Registry | Stores/distributes container artifacts |
| Repository | Named image namespace within registry |
| Docker Hub | Public registry service |
| Image reference | Registry + repository + tag/digest |
| `docker pull` | Downloads image artifact |
| `docker push` | Uploads/publishes image artifact |
| `docker tag` | Creates another image reference |
| `docker login` | Authenticates with registry |
| Authentication | Establishes identity |
| Authorization | Determines allowed actions |
| Private registry | Restricted image distribution |
| Tag | Mutable human-readable reference |
| Digest | Content-addressed identity |
| `latest` | Mutable tag; not a release guarantee |
| Registry container storage | Registry stores artifacts, not running containers |
| Layers | Reusable content-addressed image components |
| CI/CD | Registry acts as artifact handoff |
| Multi-platform | Image index references platform manifests |
| Manifest | Describes a specific image artifact |
| Image index | Groups platform-specific manifests |
| Scanning | Finds known vulnerabilities/policy issues |
| Signing | Establishes trust/provenance |
| Retention | Controls registry growth |
| Tag deletion | Does not necessarily immediately remove blobs |
| Production pull failure | Check reference, auth, authorization, network, platform, policy |

---

# High-Value Interview Traps

### Trap 1 — "Registry and repository are the same thing"

**Incorrect.**

A registry is the service; a repository is a namespace within it.

---

### Trap 2 — "`docker tag` rebuilds the image"

**Incorrect.**

It creates another reference to existing image content.

---

### Trap 3 — "`latest` means newest"

**Incorrect.**

`latest` is just a tag and can point to arbitrary content.

---

### Trap 4 — "A tag is immutable"

**Incorrect.**

Tags can be moved unless the registry enforces immutability.

---

### Trap 5 — "A digest is just another tag"

**Incorrect.**

A digest is content-addressed.

---

### Trap 6 — "Successful `docker login` means push is allowed"

**Incorrect.**

Authentication and authorization are separate.

---

### Trap 7 — "Registry stores containers"

**Incorrect.**

The registry stores images/artifacts; Docker hosts run containers.

---

### Trap 8 — "Deleting a tag immediately deletes the image"

Not necessarily.

Other references may still exist, and registry garbage collection behavior varies.

---

### Trap 9 — "Scanning means the image is secure"

**Incorrect.**

Scanning mainly identifies known issues; it cannot prove absence of all vulnerabilities.

---

### Trap 10 — "If the image works locally, production can pull it"

**Incorrect.**

Production still needs:

```text
correct reference
+
network access
+
authentication
+
authorization
+
compatible platform
```

---

# High-Value Interview Follow-Up Questions

1. What is a Docker registry?
2. Registry vs repository?
3. What happens during `docker push`?
4. What happens during `docker pull`?
5. Why tag an image before pushing?
6. Does `docker tag` rebuild an image?
7. What is an image digest?
8. Tag vs digest?
9. Why is `latest` risky?
10. How do you authenticate to a private registry?
11. Authentication vs authorization?
12. Why use a private registry?
13. How does Docker store image layers in a registry?
14. What is an image manifest?
15. What is an image index?
16. How do multi-platform images work?
17. What happens if `docker push` returns denied?
18. What does `manifest unknown` mean?
19. How would you troubleshoot an unauthorized pull?
20. How would you secure a private registry?
21. What is registry image scanning?
22. What is image signing?
23. Why use immutable tags?
24. What is a registry retention policy?
25. What happens if a registry tag is deleted?
26. Why can a running container continue after the registry image is deleted?
27. How would you design registry usage in CI/CD?
28. How would you make production deployments reproducible?
29. What happens if the production environment cannot reach the registry?
30. How would you troubleshoot a multi-platform image pull failure?

---

# Final Interview Answer

If asked **"Explain Docker registries and how they are used in CI/CD"**, a strong answer is:

> "A Docker registry is an artifact repository used to store and distribute container images. A registry contains repositories, and repositories contain image references such as tags and digests. In a CI/CD pipeline, I would build the image, run tests and security checks, tag it with a unique release or commit identifier, authenticate to the private registry, and push the image. The deployment environment then pulls that image from the registry. I prefer immutable version tags and, when exact reproducibility is important, digest-based deployment because tags such as `latest` are mutable. For private registries, I would use least-privilege authorization, protected credentials, TLS, scanning, retention policies, and appropriate signing or provenance controls."

---

# One-Line Memory Map

```text
Registry     = artifact storage/distribution
Repository   = image namespace
Tag          = mutable reference
Digest       = exact content identity
Manifest     = describes one image artifact
Index        = groups platform-specific manifests
Pull         = registry → host
Push         = host → registry
Login        = authentication
Permission   = authorization
CI/CD        = build → scan → tag → push → deploy
```
