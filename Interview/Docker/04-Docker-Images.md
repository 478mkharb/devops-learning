# Docker Interview Preparation — Topic 4: Docker Images

> **Goal:** Understand Docker images deeply: image layers, tags, digests, manifests, registries, pull/build behavior, caching, image history, size optimization, and image immutability.
>
> **Interview focus:** Be able to explain what an image actually contains, how Docker stores and identifies it, and why image design directly affects build speed, deployment reliability, and security.

---

## Q59. What is a Docker image?

### Short Interview Answer

A Docker image is an **immutable, layered package containing filesystem content and metadata** used as the template for creating containers.

### Detailed Explanation

A Docker image is not simply an executable file.

It contains the filesystem content and configuration needed to start an application.

Conceptually:

```text
Docker Image
├── Base filesystem
├── Application dependencies
├── Application files
└── Image configuration/metadata
```

Images are built from layers.

For example:

```text
Application layer
-----------------
Dependency layer
-----------------
Runtime layer
-----------------
Base image layer
```

The exact number and composition of layers depend on how the image is built.

### Example

A simple Dockerfile:

```dockerfile
FROM python:3.12
COPY app.py /app/
CMD ["python", "/app/app.py"]
```

creates an image containing:

- The base image content
- The copied application file
- Configuration describing the default command

### Why Images Matter

The image is the artifact that can be:

```text
Built
  ↓
Tested
  ↓
Tagged
  ↓
Pushed to registry
  ↓
Pulled by another environment
  ↓
Used to create containers
```

This makes the image central to container-based CI/CD.

### Interview Point

Remember:

> **Image = immutable application package/template.**

---

## Q60. What are Docker image layers?

### Short Interview Answer

Image layers are **stacked filesystem changes** that together form the image filesystem. They can be reused between images and are normally read-only when used by containers.

### Example

Suppose a Dockerfile contains:

```dockerfile
FROM ubuntu:24.04
RUN apt-get update
RUN apt-get install -y nginx
COPY index.html /var/www/html/
```

Conceptually, the resulting image may contain layers corresponding to filesystem changes from these build steps.

```text
Layer 4 → index.html
---------------------
Layer 3 → nginx files
---------------------
Layer 2 → apt-related changes
---------------------
Layer 1 → Ubuntu base
```

### Why Layers Exist

Layers allow Docker/build systems to reuse unchanged content.

For example:

```text
Image A
 ├── Base
 ├── Runtime
 └── App A

Image B
 ├── Base       ← shared
 ├── Runtime    ← shared
 └── App B
```

Only the differing content needs to be added separately.

### Important Nuance

Do not assume that every Dockerfile instruction always maps one-to-one to exactly one physical storage layer in every modern build implementation.

The conceptual model is:

> **Build steps produce filesystem changes that can be represented as image layers.**

### Interview Point

Layers provide:

- Reuse
- Caching
- Efficient distribution
- Smaller incremental transfers

---

## Q61. Why does Docker use layers for images?

### Short Interview Answer

Layers enable **reuse, caching, and efficient storage/distribution** because unchanged image content can be shared between images and reused during builds and pulls.

### Example

Imagine:

```text
Base image = 100 MB
Application dependencies = 200 MB
Application = 20 MB
```

If several applications use the same base and dependencies, Docker can reuse common content instead of storing/transferring everything independently.

```text
              Common Base
             /     |     \
            ▼      ▼      ▼
          App A  App B  App C
```

### Build Cache Benefit

If an earlier build step has not changed, the build system can often reuse its cached result.

For example:

```dockerfile
COPY package.json .
RUN npm install
COPY . .
```

If only application source changes:

```text
package.json unchanged
       ↓
npm install layer/cache can be reused
       ↓
source layer rebuilt
```

This can make builds significantly faster.

### Interview Point

Layers are not just a storage feature.

They are also a major part of Docker's **build-cache and distribution model**.

---

## Q62. What is the difference between a Docker image layer and a container writable layer?

### Short Interview Answer

Image layers are normally **read-only components of the image**, while the container writable layer contains runtime filesystem changes specific to that container.

### Diagram

```text
Container
┌──────────────────────────┐
│ Writable container layer │ ← runtime changes
├──────────────────────────┤
│ Image layer 3            │
├──────────────────────────┤
│ Image layer 2            │
├──────────────────────────┤
│ Image layer 1            │
└──────────────────────────┘
```

### Example

Suppose an image contains:

```text
/app/config.yaml
```

A running container changes the file.

The original image layer remains unchanged.

The container gets its own changed view through the writable layer according to the storage driver's behavior.

### Why This Matters

If Container A modifies a file:

```text
Container A → changed view
Container B → original image view
```

They do not modify the shared image itself.

### Interview Point

```text
Image layers       → immutable/shared
Writable layer     → container-specific runtime changes
```

---

## Q63. What is the difference between a Docker image and an image tag?

### Short Interview Answer

An **image** is the actual content/configuration, while a **tag** is a human-readable reference used to point to an image.

### Example

```text
nginx:1.27
```

Here:

```text
nginx → repository
1.27  → tag
```

A tag is a convenient name.

### Important Point

A tag is **mutable**.

For example:

```text
myapp:production
```

could point to image A today and image B later.

Therefore:

```text
Tag ≠ permanent identity
```

### Why This Matters in Production

Suppose a deployment uses:

```text
myapp:latest
```

and the registry later updates that tag.

A new deployment may pull a different image than the previous deployment.

This can make deployments less reproducible.

### Better Approach

Use a unique version tag:

```text
myapp:2026.09.09
```

and/or pin by digest:

```text
myapp@sha256:...
```

### Interview Point

> **Tags are convenient references; they should not be treated as immutable identities.**

---

## Q64. What is an image digest?

### Short Interview Answer

An image digest is a **content-addressed cryptographic identifier**, commonly represented using SHA-256, that identifies a specific image manifest/content.

### Example

```text
nginx@sha256:abcdef1234...
```

The digest identifies a specific content-addressed object.

### Tag vs Digest

```text
nginx:latest
     ↓
Mutable reference

nginx@sha256:...
     ↓
Specific content-addressed reference
```

### Why Digests Matter

Suppose:

```text
myapp:prod
```

points to Image A today.

Later the tag is moved:

```text
myapp:prod
     ↓
Image B
```

A deployment using the digest for Image A can continue to refer to the exact content associated with that digest.

### Reproducible Deployment

A CI/CD system can record:

```text
myapp@sha256:<digest>
```

and deploy that exact image reference.

### Interview Point

If asked:

> "How do you make an image reference more immutable?"

A strong answer is:

> **"Pin the deployment to a digest rather than relying only on a mutable tag."**

---

## Q65. What is the difference between a Docker tag and a digest?

### Short Interview Answer

A tag is a **mutable human-readable reference**, while a digest is a **content-addressed identifier for a specific image manifest/content**.

### Comparison

| Tag | Digest |
|---|---|
| Human-friendly | Content-addressed |
| Mutable | Identifies specific content |
| Easy to read | Longer |
| Useful for release naming | Useful for exact pinning |
| Can move to another image | Tied to referenced content |

### Example

```text
myapp:v1
```

versus:

```text
myapp@sha256:...
```

### Production Example

You might build:

```text
myapp:1.4.0
```

Then obtain its digest and deploy:

```text
myapp@sha256:...
```

This provides a stronger guarantee about which image content is being deployed.

### Interview Trap

Do not say:

> "`latest` is a digest."

It is a tag.

### Interview Point

```text
Tag    → name/reference
Digest → exact content identity
```

---

## Q66. What is an image repository?

### Short Interview Answer

An image repository is a named collection of related image versions/references in a container registry.

### Example

```text
registry.example.com/team/payment-api
```

The repository groups image versions such as:

```text
payment-api:1.0
payment-api:1.1
payment-api:2.0
```

### Registry vs Repository

These are different concepts.

```text
Registry
   │
   ├── Repository A
   │      ├── Tag 1
   │      └── Tag 2
   │
   └── Repository B
          ├── Tag 1
          └── Tag 2
```

A registry hosts repositories.

A repository organizes related images.

### Examples

Registries can include:

- Docker Hub
- Amazon ECR
- GitHub Container Registry
- Google Artifact Registry
- Azure Container Registry
- Private registries

### Interview Point

Think:

```text
Registry → storage/service
Repository → named image collection
Tag → reference within repository
Digest → content identity
```

---

## Q67. What is a Docker registry?

### Short Interview Answer

A Docker registry is a service that stores and distributes container images and their associated manifests and metadata.

### Typical Workflow

```text
Developer
   │
   │ docker build
   ▼
Local Image
   │
   │ docker tag
   ▼
Registry Reference
   │
   │ docker push
   ▼
Container Registry
   │
   │ docker pull
   ▼
Server / CI / Runtime
```

### Why Registries Matter

Registries allow teams to share images between environments.

For example:

```text
CI Pipeline
     ↓
Build image
     ↓
Push to registry
     ↓
Production server
     ↓
Pull exact image
```

### Public vs Private Registries

**Public:**

Anyone may be able to pull the image depending on repository permissions.

**Private:**

Authentication/authorization controls who can access it.

### Interview Point

A registry is essentially the distribution/storage endpoint for container images.

---

## Q68. What happens when you run `docker pull nginx`?

### Short Interview Answer

Docker resolves the image reference, contacts the configured registry, retrieves the image manifest and required layers, verifies content using digests, and stores the image locally.

### Conceptual Flow

```text
docker pull nginx
       │
       ▼
Resolve image reference
       │
       ▼
Contact registry
       │
       ▼
Get manifest
       │
       ▼
Determine required layers
       │
       ▼
Download missing layers
       │
       ▼
Verify/store content
       │
       ▼
Local image available
```

### Why Doesn't Docker Always Download Everything?

Docker can reuse layers already present locally.

For example:

```text
Local:
Layer A ✓
Layer B ✓
Layer C ✗
```

Only missing content needs to be downloaded.

### Important Point

The registry does not simply send one giant `.tar` file containing the entire image every time.

Container image distribution uses manifests and content-addressed layers.

### Interview Point

A good answer mentions:

> **manifest + layers + content digests + local layer reuse.**

---

## Q69. What happens when you run `docker images`?

### Short Interview Answer

`docker images` lists locally available image references and related information such as repository, tag, image identifier, creation information, and size.

### Example

```bash
docker images
```

may show:

```text
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
nginx        latest    ...            ...           ...
myapp        v1.0      ...            ...           ...
```

### Important Nuance

The displayed `IMAGE ID` is a local identifier associated with the image, while a registry digest is a content-addressed identifier for a manifest/content object.

Do not treat the displayed image ID and registry digest as automatically interchangeable.

### Interview Point

Use:

```bash
docker images
```

for a quick local image inventory.

For detailed image information:

```bash
docker image inspect <image>
```

---

## Q70. How can you inspect a Docker image?

### Short Interview Answer

Use `docker image inspect` to view detailed image metadata and configuration.

### Command

```bash
docker image inspect nginx
```

This can expose information such as:

- Image configuration
- Environment variables
- Entrypoint
- CMD
- Working directory
- User
- Root filesystem/layer information
- Labels
- Architecture/OS metadata

### Why It Matters

When debugging an image, you may want to know:

```text
What command starts?
What environment is defined?
What user is configured?
What architecture is this image for?
What layers/content are associated with it?
```

### Example

```bash
docker image inspect myapp:1.0
```

Then search relevant fields in the JSON output.

### Interview Point

`docker image inspect` is a practical diagnostic command.

---

## Q71. What is `docker history`?

### Short Interview Answer

`docker history` displays the image's historical layer/build information, helping you understand how the image was assembled.

### Command

```bash
docker history nginx
```

It can show information such as:

- Layer/image history
- Commands associated with layers
- Layer sizes
- Creation information

### Why It Is Useful

It can help answer:

> "Why is my image so large?"

For example:

```text
Image
 ├── 500 MB package installation
 ├── 300 MB build tools
 └── 20 MB application
```

The history can reveal unexpectedly large build steps.

### Important Nuance

`docker history` is useful for understanding image history, but it should not be interpreted as a perfect source-level reconstruction of the Dockerfile.

### Interview Point

Use:

```bash
docker history <image>
```

when investigating image composition and layer sizes.

---

## Q72. What is the difference between `docker pull` and `docker build`?

### Short Interview Answer

`docker pull` downloads an existing image from a registry, while `docker build` creates a new image from a build context and Dockerfile/build instructions.

### Comparison

| `docker pull` | `docker build` |
|---|---|
| Downloads existing image | Creates new image |
| Uses registry | Uses Dockerfile/build instructions |
| Retrieves manifest/layers | Executes build steps |
| No application build instructions required | Requires build context/build definition |

### Example

Pull:

```bash
docker pull nginx:latest
```

Build:

```bash
docker build -t myapp:1.0 .
```

### Build Flow

```text
Dockerfile + Context
       ↓
Build system
       ↓
Image
```

### Pull Flow

```text
Registry
       ↓
Manifest + layers
       ↓
Local image
```

### Interview Point

```text
pull  → obtain
build → create
```

---

## Q73. What is `docker save` and how is it different from `docker export`?

### Short Interview Answer

`docker save` exports **Docker image(s)** including their layers and image metadata, while `docker export` exports a **container's filesystem** as a tar archive and does not preserve the image's normal layer/history structure.

### `docker save`

```bash
docker save -o myapp.tar myapp:1.0
```

Conceptually:

```text
Image
 ├── Layers
 ├── Metadata
 └── Image configuration
       ↓
    tar archive
```

It can later be loaded with:

```bash
docker load -i myapp.tar
```

### `docker export`

```bash
docker export mycontainer -o container.tar
```

This exports the container filesystem.

Conceptually:

```text
Running/stopped container
        ↓
Filesystem snapshot
        ↓
tar archive
```

### Comparison

| `docker save` | `docker export` |
|---|---|
| Image-oriented | Container-oriented |
| Preserves image layers/metadata | Exports filesystem contents |
| Can save image tags/references | Does not preserve normal image history/layers |
| Use with `docker load` | Use to move a container filesystem |

### Interview Trap

Do not say:

> "`docker export` exports the image."

It exports the container filesystem.

### Interview Point

```text
save   → image
export → container filesystem
```

---

## Q74. What is `docker load`?

### Short Interview Answer

`docker load` imports a Docker image archive created by `docker save` and restores the image and its associated metadata/references.

### Example

```bash
docker load -i myapp.tar
```

Then:

```bash
docker images
```

can show the imported image.

### Typical Offline Workflow

```text
Machine A
   │
docker save
   ↓
image.tar
   │
   │ transfer
   ▼
Machine B
   │
docker load
   ↓
Docker image
```

### Why This Is Useful

It can be useful when transferring images between environments without pulling them directly from a registry.

### Interview Point

```text
docker save → create image archive
docker load → restore image archive
```

---

## Q75. What is `docker import`?

### Short Interview Answer

`docker import` creates a Docker image from a filesystem tar archive, unlike `docker load`, which restores a Docker image archive created by `docker save`.

### Example

```bash
docker import rootfs.tar myimage:1.0
```

### Important Difference

```text
docker load
     ↓
Docker image archive
     ↓
Restores image structure/metadata

docker import
     ↓
Filesystem tar
     ↓
Creates a new image
```

### Comparison

| Command | Input | Purpose |
|---|---|---|
| `docker load` | `docker save` archive | Restore image |
| `docker import` | Filesystem tar | Create image from filesystem |

### Interview Trap

Do not treat:

```text
load = import
```

They have different semantics.

### Interview Point

```text
save/load
    ↕
Docker image archive

export/import
    ↕
Container filesystem / filesystem tar
```

---

## Q76. Why should Docker images be kept small?

### Short Interview Answer

Smaller images generally improve **build speed, registry storage, image transfer time, deployment speed, and attack-surface management**.

### Example

Suppose:

```text
Image A = 1.5 GB
Image B = 150 MB
```

If a deployment repeatedly pulls the image:

```text
1.5 GB → slower transfer
150 MB → faster transfer
```

Smaller images can therefore reduce deployment latency and bandwidth usage.

### Security Benefit

A smaller runtime image can contain fewer:

- Packages
- Utilities
- Libraries
- Potentially vulnerable components

However:

> **Small does not automatically mean secure.**

An image can be tiny and still contain a critical vulnerability.

### How to Reduce Image Size

Common techniques include:

- Use an appropriate minimal runtime base
- Multi-stage builds
- Avoid unnecessary packages
- Remove unnecessary build artifacts
- Use `.dockerignore`
- Keep build tools out of runtime images

### Interview Point

Image size affects:

```text
Build → Storage → Transfer → Deployment
```

and can also influence the runtime attack surface.

---

## Q77. What is image immutability?

### Short Interview Answer

Image immutability means that once an image content is built and identified, running containers do not modify the underlying image layers; changes belong to the container runtime layer or external storage.

### Example

```text
Image v1
   │
   ├── Container A
   │      └── modifies /app/config
   │
   └── Container B
          └── original image content
```

Container A does not modify the image itself.

### Why Immutability Matters

It supports:

- Reproducible deployments
- Versioned artifacts
- Rollbacks
- Safer CI/CD promotion
- Easier debugging

### Example Deployment

```text
Build
 ↓
myapp:1.2.0
 ↓
Test
 ↓
Deploy
```

If the image content is kept immutable, the artifact tested is the artifact deployed.

### Interview Point

This supports the DevOps principle:

> **Build once, promote the same artifact.**

---

## Q78. What is an image manifest?

### Short Interview Answer

An image manifest is metadata describing an image artifact, including information about the content layers and configuration needed to retrieve and use it.

### Conceptual Structure

```text
Image Manifest
 ├── Configuration descriptor
 ├── Layer descriptor 1
 ├── Layer descriptor 2
 └── Layer descriptor 3
```

Each descriptor contains content-addressed information such as:

```text
media type
digest
size
```

### Why It Matters

When Docker pulls an image, it needs to know:

```text
Which configuration?
Which layers?
What are their digests?
What sizes/types are they?
```

The manifest provides this information.

### Interview Point

Think:

> **Manifest = description/index of the image content that the client/runtime needs to retrieve and assemble the image.**

---

## Q79. What is a multi-platform image?

### Short Interview Answer

A multi-platform image reference allows the same image name/tag to resolve to **platform-specific image manifests**, such as Linux `amd64` or Linux `arm64`.

### Example

```text
myapp:1.0
     │
     ▼
Image Index / Manifest List
     │
     ├── linux/amd64 → Image A
     ├── linux/arm64 → Image B
     └── ...
```

### Why Is This Useful?

A developer can use:

```bash
docker pull myapp:1.0
```

on different architectures, while Docker selects the appropriate platform-specific image when available.

### Important Nuance

A multi-platform image is **not one universal binary image** that runs identically at the CPU instruction level everywhere.

It is a higher-level reference/index that points to platform-specific images.

### Example Platforms

```text
linux/amd64
linux/arm64
```

### Interview Point

This distinction is important:

```text
One image reference
       ↓
Multiple platform-specific images
```

---

## Q80. What is the difference between an image index and an image manifest?

### Short Interview Answer

An **image manifest** describes one image artifact, while an **image index** (often called a manifest list in older terminology) can reference multiple platform-specific image manifests.

### Conceptual Diagram

```text
Image Index
    │
    ├── linux/amd64
    │      └── Manifest A
    │
    ├── linux/arm64
    │      └── Manifest B
    │
    └── ...
```

### Why This Matters

When a client requests:

```text
myapp:1.0
```

the registry can return an image index containing platform variants.

The client then selects the appropriate manifest.

### Interview Point

```text
Manifest → one image artifact
Index    → collection/reference to platform variants
```

---

## Q81. What is image caching during a Docker build?

### Short Interview Answer

Build caching allows Docker's build system to reuse previously computed results for unchanged build steps, reducing build time and unnecessary work.

### Example

Consider:

```dockerfile
COPY package.json package-lock.json ./
RUN npm ci

COPY . .
RUN npm run build
```

Suppose only:

```text
src/app.js
```

changes.

The dependency files are unchanged:

```text
package.json ✓
package-lock.json ✓
```

Therefore the dependency installation step may be reused from cache.

### Why Order Matters

Bad ordering:

```dockerfile
COPY . .
RUN npm install
```

A change to any file in the context can invalidate the `COPY`, which can invalidate later steps.

Better:

```dockerfile
COPY package*.json ./
RUN npm install

COPY . .
```

Now source changes do not necessarily invalidate dependency installation.

### Interview Point

> **Put stable, expensive build inputs earlier and frequently changing files later.**

---

## Q82. What invalidates Docker build cache?

### Short Interview Answer

Cache reuse depends on whether the build system determines that the relevant inputs for a step have changed. Changes to instruction inputs, copied files, build arguments, base image resolution, or other dependencies can cause cache invalidation.

### Example

```dockerfile
COPY package.json .
RUN npm install
```

If:

```text
package.json changes
```

the `COPY` result changes, which can cause:

```text
RUN npm install
```

to be rebuilt.

### Another Example

```dockerfile
COPY . .
RUN make
```

If files included in the relevant build context change, the `COPY` result may change and invalidate subsequent steps.

### Why Cache Invalidation Cascades

Build steps generally depend on earlier results:

```text
Step 1
  ↓
Step 2
  ↓
Step 3
  ↓
Step 4
```

If Step 2 changes:

```text
Step 2 → rebuilt
Step 3 → may need rebuild
Step 4 → may need rebuild
```

### Interview Point

Cache optimization is about controlling **what changes when**.

---

## Q83. What is `docker tag`?

### Short Interview Answer

`docker tag` creates another local image reference pointing to an existing image.

### Example

```bash
docker tag myapp:latest \
  123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:1.0
```

This does not rebuild the image.

Conceptually:

```text
Existing image
      │
      ├── myapp:latest
      └── registry.example.com/myapp:1.0
```

Both references can point to the same image content.

### Why Tag Before Push?

Registries commonly expect the image reference to contain the destination repository.

Typical workflow:

```bash
docker build -t myapp:1.0 .
docker tag myapp:1.0 registry.example.com/myapp:1.0
docker push registry.example.com/myapp:1.0
```

### Interview Trap

`docker tag` does **not** modify the image's filesystem.

It creates a reference.

### Interview Point

```text
tag → rename/reference
not
tag → rebuild
```

---

## Q84. What is the difference between an image ID and a digest?

### Short Interview Answer

An image ID is a local identifier associated with a Docker image configuration/content in the local image store, while a digest is a content-addressed identifier used for registry-distributed artifacts such as image manifests.

### Why This Can Be Confusing

You may see:

```bash
docker images
```

showing:

```text
IMAGE ID
```

while registry references use:

```text
@sha256:...
```

They are related to image identity but are not simply two names for exactly the same object in every context.

### Interview Point

Avoid saying:

> "Image ID and digest are always identical."

The safer explanation is:

> **"An image ID is a local image-store identifier, while a digest is a content-addressed identifier for a specific registry artifact such as a manifest."**

---

## Q85. What is the difference between `latest` and a version tag?

### Short Interview Answer

`latest` is simply a conventional tag name; it does not inherently mean "newest." A version tag such as `1.4.2` communicates a specific release naming convention, but tags in general remain mutable unless protected by registry policy.

### Example

```text
myapp:latest
myapp:1.4.2
```

Both are tags.

The registry owner can potentially move either tag.

### Why `latest` Can Be Risky

Suppose:

```text
Monday:
latest → v1

Wednesday:
latest → v2
```

A new deployment pulling `latest` on Wednesday may get a different image.

### Better Production Practice

Use:

```text
myapp:1.4.2
```

and preferably record/pin the digest for exact deployment identity.

### Interview Point

> **`latest` is a tag, not a guarantee about recency or immutability.**

---

## Q86. How do you optimize Docker image builds?

### Short Interview Answer

Use effective layer/cache ordering, a precise `.dockerignore`, appropriate base images, multi-stage builds, and BuildKit/build cache features while avoiding unnecessary files and packages.

### Main Techniques

#### 1. Order instructions for cache reuse

```dockerfile
COPY package*.json ./
RUN npm ci

COPY . .
```

#### 2. Use `.dockerignore`

Avoid sending unnecessary files such as:

```text
.git
node_modules
build/
*.log
```

into the build context.

#### 3. Use multi-stage builds

```text
Builder image
   ↓
Compile/build
   ↓
Runtime image
   ↓
Only runtime artifacts
```

#### 4. Avoid unnecessary packages

Don't install development utilities into a production runtime image unless required.

#### 5. Choose an appropriate base

A smaller base can reduce image size, but compatibility and security support matter more than size alone.

#### 6. Use build cache

Modern Docker builds commonly use BuildKit/buildx capabilities for more advanced caching.

### Interview Point

Image optimization is a combination of:

```text
Smaller context
+
Better cache reuse
+
Fewer runtime dependencies
+
Separate build/runtime environments
```

---

# Quick Revision — Topic 4

| Concept | Remember |
|---|---|
| Image | Immutable, layered package/template |
| Layer | Filesystem change/content layer |
| Image layer | Normally read-only |
| Writable layer | Runtime changes for a container |
| Tag | Mutable human-readable reference |
| Digest | Content-addressed identifier |
| Repository | Named collection of related images |
| Registry | Stores/distributes image artifacts |
| Manifest | Describes one image artifact |
| Image index | References multiple platform-specific manifests |
| `docker pull` | Downloads an existing image |
| `docker build` | Creates an image |
| `docker tag` | Creates another image reference |
| `docker inspect` | Detailed image metadata |
| `docker history` | Image/layer history |
| `docker save` | Exports image archive |
| `docker load` | Imports image archive |
| `docker export` | Exports container filesystem |
| `docker import` | Creates image from filesystem tar |
| `latest` | Conventional tag; not guaranteed newest |
| Multi-platform image | One reference → platform-specific images |
| Cache | Reuses unchanged build results |

---

# Image Mental Model

```text
                       Registry
                          │
                     Manifest/Index
                          │
               ┌──────────┴──────────┐
               ▼                     ▼
          Image Manifest A      Image Manifest B
           linux/amd64            linux/arm64
               │
               ▼
          Image Layers
        ┌───────────────┐
        │ Application   │
        ├───────────────┤
        │ Dependencies  │
        ├───────────────┤
        │ Runtime       │
        ├───────────────┤
        │ Base          │
        └───────────────┘
               │
               ▼
           Container
               │
        Writable Layer
               │
               ▼
        Runtime changes
```

---

# High-Value Interview Traps

### Trap 1 — `latest` means newest

❌ Not guaranteed.

It is just a tag.

### Trap 2 — Tag is immutable

❌ No.

Tags can be moved unless registry controls prevent that.

### Trap 3 — Digest is just another tag

❌ No.

A digest is content-addressed.

### Trap 4 — `docker tag` rebuilds the image

❌ No.

It creates another reference to existing image content.

### Trap 5 — `docker pull` downloads one giant image file

❌ Oversimplified.

Images are distributed using manifests/indexes and content-addressed layers.

### Trap 6 — `docker export` exports an image

❌ No.

It exports a container filesystem.

### Trap 7 — `docker save` and `docker export` are the same

❌ No.

```text
save   → image
export → container filesystem
```

### Trap 8 — `docker load` and `docker import` are the same

❌ No.

```text
load   → restore Docker image archive
import → create image from filesystem tar
```

### Trap 9 — Smaller image automatically means safer image

❌ No.

Smaller can reduce attack surface, but vulnerability/security status still depends on the actual contents and configuration.

### Trap 10 — Multi-platform image is one universal binary

❌ No.

It is generally an index/reference that points to platform-specific images.

---

# Interview Follow-Up Questions

After explaining Docker images, expect questions such as:

1. What are Docker image layers?
2. Why does Docker use layers?
3. What is copy-on-write?
4. What is `overlay2`?
5. What is the container writable layer?
6. What is the difference between a tag and a digest?
7. Why is `latest` risky in production?
8. What is a Docker registry?
9. What is a repository?
10. What is an image manifest?
11. What is an image index?
12. How does a multi-platform image work?
13. What happens during `docker pull`?
14. How does Docker cache image layers?
15. What invalidates build cache?
16. How do you reduce image size?
17. What is `docker history`?
18. What is `docker image inspect`?
19. Difference between `docker save` and `docker export`?
20. Difference between `docker load` and `docker import`?
21. What does `docker tag` do?
22. What is an image ID?
23. What is an image digest?
24. Why should production deployments pin image versions/digests?
25. How would you troubleshoot a large Docker image?

---

# Final Interview Answer — "Explain Docker Images"

If the interviewer asks:

> **"Explain Docker images and how they are used."**

A strong answer is:

> "A Docker image is an immutable, layered package containing an application's filesystem content and metadata. Images are built from filesystem changes that can be represented as layers, allowing common content to be reused and build/cache operations to be more efficient. A container is created from the image and gets its own writable runtime layer. Images can be tagged with human-readable references, but tags are mutable, so a digest can be used when an exact content-addressed reference is required. Images are distributed through registries using manifests and layers, and multi-platform image indexes can point to different platform-specific images. In CI/CD, we can build an image once, test it, push it to a registry, and deploy the same image—preferably pinned to an exact version and digest."

---

# One-Line Memory Map

```text
Image
  ↓
Layers
  ↓
Manifest / Index
  ↓
Registry
  ↓
Pull
  ↓
Local Image
  ↓
Container
  ↓
Writable Runtime Layer
```
