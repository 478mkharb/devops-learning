# Docker Interview Preparation — Topic 07: Docker Builds

> **Purpose:** Understand how Docker turns a Dockerfile and build context into an image, how build cache works, and how to troubleshoot and optimize builds.
>
> **Scope:** This topic focuses on the Docker build process, cache behavior, build context, BuildKit/buildx concepts, reproducibility, and practical build troubleshooting. Multistage builds are covered separately.

---

## 136. What happens when you run `docker build`?

### Short Interview Answer

`docker build` sends the Dockerfile and build context to the Docker build system. The builder processes the instructions, executes build steps, creates filesystem layers/configuration, and produces a Docker image.

### Example

```bash
docker build -t myapp:1.0 .
```

Here:

```text
docker build
    ↓
Dockerfile + build context
    ↓
Builder
    ↓
Execute instructions
    ↓
Create/reuse build layers
    ↓
Image
    ↓
myapp:1.0
```

The final `.` means the current directory is the build context.

### Important Point

The build process does **not** simply execute the Dockerfile as a normal shell script.

Instructions such as:

```dockerfile
FROM
RUN
COPY
ENV
WORKDIR
CMD
```

have Docker-specific semantics.

### Common Interview Trap

> "`docker build` creates a container and then converts that container into an image."

That is an oversimplification and should not be used as the modern mental model.

### Interview Point

**Docker build transforms build inputs into an image using a builder and reusable build cache.**

---

## 137. What is the Docker build context?

### Short Interview Answer

The build context is the set of files and directories made available to the builder for a build.

For:

```bash
docker build -t myapp .
```

`.` specifies the current directory as the context.

### Example

Suppose:

```text
project/
├── Dockerfile
├── app.py
├── requirements.txt
├── .git/
├── logs/
└── .venv/
```

Running:

```bash
docker build -t myapp .
```

makes the project directory the build context, subject to `.dockerignore`.

Then:

```dockerfile
COPY . .
```

can copy files from that context into the image.

### Important Distinction

```text
Build context
     ↓
Files available to Docker build instructions

Image
     ↓
Final artifact produced by the build
```

They are not the same thing.

### Common Interview Trap

The build context is not automatically "the directory containing the Dockerfile."

You can specify a different context:

```bash
docker build -f docker/Dockerfile .
```

Here:

- Dockerfile = `docker/Dockerfile`
- context = `.`

### Interview Point

**`-f` selects the Dockerfile; the final positional argument selects the build context.**

---

## 138. Why is build context important?

### Short Interview Answer

A large build context increases the amount of data Docker must process/send and can slow builds. It can also expose unnecessary files to build instructions.

### Bad Example

```text
project/
├── .git/
├── node_modules/
├── .venv/
├── build/
├── logs/
├── backups/
└── source/
```

Using:

```bash
docker build .
```

without an appropriate `.dockerignore` can create a large context.

### Better

```text
.dockerignore
.git
node_modules
.venv
logs
backups
```

### Why It Matters

```text
Large context
    ↓
more data to process
    ↓
slower build startup
    ↓
unnecessary files available to COPY
```

### Common Interview Trap

Do not say:

> "Everything in the build context automatically becomes part of the image."

False.

A file can be in the context without ever being copied into the image.

### Interview Point

**Context controls what the builder can access; Dockerfile instructions determine what becomes part of the image.**

---

## 139. What is Docker build cache?

### Short Interview Answer

Docker build cache allows previously completed build steps to be reused when their relevant inputs have not changed.

### Example

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

If only:

```text
app.py
```

changes, Docker may reuse the cached dependency-installation step.

### Conceptual Flow

```text
requirements.txt unchanged
        ↓
dependency layer can be reused
        ↓
source-copy layer changes
        ↓
later steps rebuild as necessary
```

### Why It Matters

Build cache can dramatically reduce CI/CD build time.

### Common Interview Trap

> "Docker caches only the RUN command text."

No. Cache reuse depends on the instruction and its relevant inputs/state, not merely the command string.

### Interview Point

**Build cache avoids repeating work whose inputs have not meaningfully changed.**

---

## 140. What causes Docker build cache invalidation?

### Short Interview Answer

Cache is invalidated when the relevant inputs to a build step change. This can include the Dockerfile instruction itself, files used by `COPY` or `ADD`, build arguments, base-image changes, and other build inputs depending on the instruction and builder.

### Example

```dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt
```

If:

```text
requirements.txt
```

changes, the `COPY` step changes and the dependency installation step must be reconsidered.

### Important Cascade

```text
Layer A changes
    ↓
Layer B depends on A
    ↓
Layer B cannot simply reuse old result
    ↓
Later dependent steps may also rebuild
```

### Example

```dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
RUN python build.py
```

Changing only `app.py` generally does not require reinstalling dependencies.

### Common Interview Trap

> "Any source-code change invalidates the entire Dockerfile."

Not necessarily.

It depends on where the changed input enters the build and which later steps depend on it.

### Interview Point

**Cache invalidation follows dependency relationships between build steps and their inputs.**

---

## 141. Why is Dockerfile instruction order important for caching?

### Short Interview Answer

Because once a build step cannot use cache, subsequent dependent steps may also need to execute again.

### Bad Structure

```dockerfile
COPY . .
RUN pip install -r requirements.txt
```

### Better Structure

```dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
```

### Why?

If:

```text
application source changes
```

but:

```text
requirements.txt remains unchanged
```

the dependency-installation layer can remain reusable.

### General Pattern

```text
Stable input
   ↓
Expensive operation
   ↓
Frequently changing input
   ↓
Application build
```

### Common Interview Trap

The goal is not simply to minimize the number of Dockerfile lines.

The goal is to create **useful cache boundaries**.

### Interview Point

**Order instructions according to how frequently their inputs change and how expensive they are.**

---

## 142. What is BuildKit?

### Short Interview Answer

BuildKit is Docker's modern build engine that provides improved build performance, better caching, parallelism, advanced build features, and more efficient handling of build inputs.

### Conceptual View

Traditional mental model:

```text
Dockerfile
    ↓
Builder
    ↓
Image
```

Modern BuildKit-oriented model:

```text
Dockerfile / build definition
          ↓
       BuildKit
       ↙     ↘
    cache    parallel work
       ↓
     image
```

### Useful Capabilities

BuildKit supports features such as:

- efficient caching
- parallel build operations where possible
- improved build output
- secret mounts
- SSH mounts
- cache mounts
- advanced frontend/build definitions
- improved handling of build contexts

### Common Interview Trap

Do not describe BuildKit as a replacement for Docker images.

BuildKit is primarily a **build engine**, not the image format itself.

### Interview Point

**BuildKit improves how images are built; it does not replace the image artifact.**

---

## 143. What is `docker buildx`?

### Short Interview Answer

`docker buildx` is a Docker CLI plugin/interface for advanced image building, built around BuildKit.

### Example

```bash
docker buildx build -t myapp:1.0 .
```

### Why Use It?

It supports advanced build workflows such as:

```text
BuildKit-backed builds
        +
multiple builders
        +
cache export/import
        +
multi-platform builds
```

For example:

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t myrepo/myapp:1.0 \
  --push .
```

### Important Distinction

```text
buildx
   ↓
CLI/build workflow interface

BuildKit
   ↓
build engine/backend

Image
   ↓
build output
```

### Common Interview Trap

> "`buildx` is the new Docker image format."

No.

It is a build interface/tooling layer around BuildKit.

### Interview Point

**buildx exposes advanced BuildKit capabilities through Docker CLI workflows.**

---

## 144. What is a Docker builder?

### Short Interview Answer

A builder is the environment/backend that executes a Docker build definition and produces build output.

With modern Docker, builders can be backed by BuildKit and can be configured with different drivers and capabilities.

### Example

```bash
docker buildx ls
```

can show available builders.

A build can then use a selected builder.

### Conceptual Architecture

```text
docker buildx
      ↓
selected builder
      ↓
BuildKit
      ↓
build execution
      ↓
image/cache/output
```

### Why It Matters

Different builders can be useful for:

- local builds
- CI builds
- isolated build environments
- multi-platform builds
- remote/advanced build workflows

### Common Interview Trap

A builder is not the same thing as a Docker container running your application.

### Interview Point

**Builder = environment used to execute the build.**

---

## 145. What is the difference between `docker build` and `docker buildx build`?

### Short Interview Answer

`docker build` is the standard Docker build command, while `docker buildx build` exposes advanced BuildKit-based build functionality and configuration.

### Basic Build

```bash
docker build -t myapp:1.0 .
```

### Advanced Build

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t registry.example.com/myapp:1.0 \
  --push .
```

### Key Difference

`buildx` is useful when you need advanced capabilities such as:

- multi-platform builds
- custom builders
- cache import/export
- advanced BuildKit features

### Common Interview Trap

Do not imply that ordinary `docker build` and `buildx` are completely unrelated build systems. Modern Docker uses BuildKit for standard builds as well, depending on configuration/version.

### Interview Point

**`buildx` is the advanced build interface; `docker build` is the standard interface.**

---

## 146. What is a cache mount in BuildKit?

### Short Interview Answer

A cache mount provides a persistent build cache directory that can be reused across builds without making that cache part of the resulting image layer.

### Example

For an appropriate BuildKit-enabled build:

```dockerfile
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt
```

The package cache can be reused by later builds.

### Conceptual Difference

Normal image layer:

```text
command
  ↓
filesystem changes
  ↓
image layer
```

Cache mount:

```text
command
  ↓
temporary/reusable build cache
  ↓
not intended as application runtime data
```

### Why It Matters

It can improve repeated builds, especially when package managers download many dependencies.

### Common Interview Trap

A cache mount does not mean the cached files become guaranteed application data inside the final image.

### Interview Point

**Build cache can accelerate builds without becoming part of the runtime artifact.**

---

## 147. How can you pass secrets securely during a Docker build?

### Short Interview Answer

Use a dedicated build-secret mechanism, such as BuildKit secret mounts, rather than embedding credentials in `ARG`, `ENV`, or image layers.

### Example

Dockerfile:

```dockerfile
RUN --mount=type=secret,id=npmrc,target=/root/.npmrc \
    npm ci
```

Build:

```bash
docker buildx build \
  --secret id=npmrc,src=$HOME/.npmrc \
  -t myapp .
```

### Why?

The application may need credentials temporarily during a build, for example to download a private dependency.

The credential should not become a normal image-layer artifact.

### Common Interview Trap

Avoid:

```dockerfile
ARG TOKEN
RUN echo "$TOKEN"
```

or:

```dockerfile
ENV TOKEN=secret
```

These are not appropriate secret-management mechanisms.

### Interview Point

**Secrets needed during builds should be mounted into the specific build step, not baked into the image.**

---

## 148. What is a multi-platform Docker build?

### Short Interview Answer

A multi-platform build produces image variants for multiple CPU/OS platforms, such as `linux/amd64` and `linux/arm64`, and publishes them so clients can pull the appropriate variant.

### Example

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t myrepo/myapp:1.0 \
  --push .
```

### Conceptual Result

```text
                 myapp:1.0
                     │
              image index
               /         \
              /           \
linux/amd64 image      linux/arm64 image
```

When a client pulls the image, the registry/client can select the appropriate platform-specific image.

### Important Point

This does not mean one image contains one universal executable that runs identically on every architecture.

### Common Interview Trap

> "Multi-platform means Docker converts the x86 binary to ARM automatically."

Not generally.

The build must produce a compatible image for each requested platform.

### Interview Point

**Multi-platform images provide platform-specific image variants under one logical image reference.**

---

## 149. What is `--platform` used for during a build?

### Short Interview Answer

`--platform` specifies the target platform for the build or image output.

### Example

```bash
docker buildx build \
  --platform linux/amd64 \
  -t myapp:1.0 .
```

Multiple platforms:

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t myapp:1.0 \
  --push .
```

### Why It Matters

A developer may build on:

```text
ARM64 laptop
```

while production runs:

```text
AMD64 servers
```

Explicit platform selection helps create the correct artifact.

### Important Caveat

Cross-platform builds may require emulation or native builders depending on the build and dependencies.

### Common Interview Trap

`--platform` does not magically make architecture-specific application binaries compatible.

### Interview Point

**Platform selection controls the target OS/architecture of the resulting image variant.**

---

## 150. What is `--build-arg`?

### Short Interview Answer

`--build-arg` supplies a value for an `ARG` declared in the Dockerfile during the build.

### Dockerfile

```dockerfile
ARG APP_VERSION=1.0
RUN echo "Building version $APP_VERSION"
```

Build:

```bash
docker build \
  --build-arg APP_VERSION=2.0 \
  -t myapp:2.0 .
```

### Important Characteristics

`ARG` is primarily build-time configuration.

It can influence:

```dockerfile
RUN
FROM
COPY
other build logic
```

depending on where it is declared and used.

### Common Interview Trap

Do not use:

```bash
--build-arg DB_PASSWORD=secret
```

as a secure secret-delivery mechanism.

### Interview Point

**`--build-arg` supplies build-time variables; it is not a secret store.**

---

## 151. What is the difference between `--build-arg` and runtime `-e`?

### Short Interview Answer

`--build-arg` supplies build-time values to `ARG`, while `docker run -e` supplies environment variables to the running container.

### Build-Time

```bash
docker build \
  --build-arg APP_VERSION=2.0 \
  -t myapp .
```

Dockerfile:

```dockerfile
ARG APP_VERSION
```

### Runtime

```bash
docker run \
  -e APP_ENV=production \
  myapp
```

Application receives:

```text
APP_ENV=production
```

### Lifecycle

```text
Build time:
--build-arg → ARG

Image:
      ↓

Runtime:
docker run -e → container environment
```

### Common Interview Trap

A build argument is not automatically a runtime environment variable.

### Interview Point

**ARG = build time; `-e` = container runtime.**

---

## 152. What is the difference between `docker build --no-cache` and normal caching?

### Short Interview Answer

Normal builds attempt to reuse eligible cached results. `--no-cache` tells Docker not to use the existing build cache for the build steps.

### Normal

```bash
docker build -t myapp .
```

### No Cache

```bash
docker build --no-cache -t myapp .
```

### When Useful

Use `--no-cache` when:

- debugging cache-related behavior
- intentionally forcing package/dependency installation
- validating that a build works from a clean cache state

### Important Caveat

`--no-cache` does not mean "ignore every external state."

For example, if your Dockerfile explicitly references a mutable tag, that tag's current state still matters.

### Common Interview Trap

`--no-cache` is not a general reproducibility switch.

Reproducibility requires controlling build inputs.

### Interview Point

**`--no-cache` disables reuse of existing build cache; it does not make an uncontrolled build deterministic.**

---

## 153. What is `--pull` during a Docker build?

### Short Interview Answer

`--pull` asks Docker to attempt to pull a newer version of the base image referenced by the Dockerfile before building.

### Example

```bash
docker build --pull -t myapp .
```

If the Dockerfile contains:

```dockerfile
FROM python:3.12-slim
```

Docker attempts to obtain the current available image for that reference.

### Why Use It?

Useful when you want to make sure the build considers a newer base image instead of relying only on a locally cached base image.

### Important Distinction

```text
--pull
    ↓
refresh/consider base image

--no-cache
    ↓
don't reuse existing build cache
```

They solve different problems.

### Common Interview Trap

`--pull` does not automatically update every dependency in your application.

### Interview Point

**`--pull` concerns base-image freshness; `--no-cache` concerns build-cache reuse.**

---

## 154. What is `docker build --progress`?

### Short Interview Answer

`--progress` controls how build progress is displayed, particularly with BuildKit.

### Example

```bash
docker buildx build --progress=plain .
```

`plain` produces detailed, line-oriented build output that is often useful in CI logs and troubleshooting.

### Common Modes

Depending on the Docker/buildx version and environment, modes include:

```text
auto
tty
plain
quiet
```

### Why `plain` Helps

Instead of compact interactive output, CI logs can show individual build steps and command output.

### Common Interview Trap

`--progress` does not change the build result by itself. It mainly controls build output presentation.

### Interview Point

**Use `plain` when you need detailed build logs for CI or debugging.**

---

## 155. How do you troubleshoot a Docker build that suddenly became slow?

### Short Interview Answer

I first identify which build step became slow, then check cache invalidation, build-context size, dependency downloads, base-image pulls, network access, and builder/cache configuration.

### Step 1 — Inspect Build Output

```bash
docker buildx build --progress=plain .
```

Find the slow step.

### Step 2 — Check Cache

Ask:

```text
Did a Dockerfile instruction change?
Did requirements/package files change?
Did the build context change?
Did a build argument change?
Is the builder using its previous cache?
```

### Step 3 — Check Build Context

Look for:

```text
node_modules
.git
large logs
build artifacts
virtual environments
```

and improve `.dockerignore`.

### Step 4 — Check Dependency Downloads

A slow:

```dockerfile
RUN npm ci
```

or:

```dockerfile
RUN pip install ...
```

may indicate network/package-registry problems or lost package cache.

### Step 5 — Check Base Image

A fresh or changed base image can require additional download/build work.

### Step 6 — Check CI Builder

In CI, determine whether the build cache is persistent.

```text
Ephemeral builder
      ↓
cache disappears
      ↓
every build repeats expensive work
```

### Common Interview Trap

Do not immediately conclude:

> "Docker is slow."

First identify **which build phase is slow and why its previous result was not reused**.

### Interview Point

**Troubleshoot Docker builds by locating the cache miss or expensive external operation first.**

---

# Quick Revision

| Concept | Key Point |
|---|---|
| `docker build` | Converts Dockerfile + build inputs into an image |
| Build context | Files available to the builder |
| `-f` | Selects Dockerfile |
| `.` | Commonly specifies build context |
| Build cache | Reuses eligible previous build results |
| Cache invalidation | Happens when relevant inputs change |
| Instruction order | Controls useful cache reuse |
| BuildKit | Modern build engine |
| buildx | Advanced BuildKit-oriented build interface |
| Builder | Environment/backend executing the build |
| Cache mount | Reusable build cache, separate from final image contents |
| Build secrets | Use dedicated secret mounts/mechanisms |
| Multi-platform | Produces platform-specific image variants |
| `--platform` | Specifies target platform(s) |
| `--build-arg` | Supplies Dockerfile `ARG` values |
| `docker run -e` | Supplies runtime environment variables |
| `--no-cache` | Prevents reuse of existing build cache |
| `--pull` | Attempts to refresh base-image references |
| `--progress=plain` | Detailed build logs |
| Build troubleshooting | Find the slow/cache-missed step first |

---

# High-Value Interview Traps

### Trap 1 — "`docker build` executes the Dockerfile like a shell script."

**Wrong.**

Dockerfile instructions have Docker-specific build semantics.

---

### Trap 2 — "The Dockerfile directory is always the build context."

**Wrong.**

Example:

```bash
docker build -f docker/Dockerfile .
```

Dockerfile and context are different paths.

---

### Trap 3 — "Everything in the build context goes into the image."

**Wrong.**

Only files referenced by build instructions such as `COPY`/`ADD` become image contents.

---

### Trap 4 — "Any source change rebuilds the whole Dockerfile."

**Wrong.**

Only affected steps and their dependent later steps need rebuilding.

---

### Trap 5 — "`--no-cache` makes builds reproducible."

**Wrong.**

It disables cache reuse; it does not control mutable tags, external dependencies, timestamps, or other build inputs.

---

### Trap 6 — "`--pull` and `--no-cache` do the same thing."

**Wrong.**

```text
--pull     → base-image freshness
--no-cache → build-cache reuse
```

---

### Trap 7 — "`buildx` and BuildKit are the same thing."

**Not exactly.**

```text
buildx   → build interface/tooling
BuildKit → build engine
```

---

### Trap 8 — "Multi-platform means one binary runs on every architecture."

**Wrong.**

Multi-platform images contain/select platform-specific image variants.

---

### Trap 9 — "`ARG` is safe for passwords because it disappears after the build."

**Wrong.**

Do not treat build arguments as secure secret storage.

---

# Interview Follow-Up Questions

After answering Docker builds, an interviewer may ask:

1. How exactly does Docker determine whether a `RUN` instruction can use cache?
2. What happens to cache when a `COPY` source file changes?
3. What is BuildKit's DAG-based build model?
4. What are BuildKit cache exporters and importers?
5. What is a cache mount?
6. How do you share build cache between Jenkins agents?
7. How do multi-platform builds work internally?
8. What is QEMU and when is it used?
9. What is the difference between a builder and a build context?
10. How would you secure private package credentials during a build?
11. How do you reduce build time in CI/CD?
12. How would you diagnose repeated cache misses?

---

# Final Interview Answer

> **"When I run a Docker build, Docker uses the Dockerfile together with the specified build context and passes those inputs to the builder. The builder executes the instructions and produces the image while reusing eligible cached results. For efficient builds, I structure the Dockerfile so stable dependency inputs are processed before frequently changing source code, and I keep the build context small with `.dockerignore`. For advanced workflows I use BuildKit/buildx for features such as improved caching, cache mounts, build secrets, and multi-platform builds. If a build becomes slow, I first identify the slow step and determine whether it is caused by cache invalidation, a large context, dependency downloads, base-image pulls, or the build environment."**

---

# One-Line Memory Map

```text
DOCKER BUILD
= Dockerfile
+ build context
+ build arguments
+ builder/BuildKit
+ cache
+ dependencies
        ↓
     IMAGE
```

## Build Optimization Memory Map

```text
SMALL CONTEXT
      +
GOOD .dockerignore
      +
STABLE INPUTS FIRST
      +
EFFECTIVE CACHE
      +
BUILD CACHE MOUNTS
      +
CONTROLLED DEPENDENCIES
      ↓
FASTER + MORE PREDICTABLE BUILDS
```
