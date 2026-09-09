# Docker Interview Preparation — Topic 06: Dockerfile Configuration & Best Practices

> **Purpose:** Production-oriented Dockerfile configuration and interview preparation.
>
> **Scope:** This topic focuses on writing, reviewing, and improving Dockerfiles. Deep BuildKit/buildx topics, multistage builds, secrets mechanisms, and Docker security internals are covered separately.

---

## 116. What is a good Dockerfile structure/order?

### Short Interview Answer

A good Dockerfile generally follows this order:

```text
1. Base image
2. Build-time arguments
3. Environment/configuration defaults
4. Working directory
5. Dependency/package installation
6. Application source copy
7. Build step
8. Runtime user
9. Metadata/ports/healthcheck
10. Entrypoint or CMD
```

The exact order depends on the application, but the main principle is to **put stable instructions before frequently changing instructions** so Docker can reuse build cache effectively.

### Example

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

USER 10001

EXPOSE 8080

CMD ["python", "app.py"]
```

### Why It Matters

If application source code changes but `requirements.txt` does not, Docker can reuse the dependency-installation layer.

### Common Interview Trap

> "There is one mandatory Dockerfile ordering."

No. Docker permits many valid instruction orders. The recommended ordering is primarily about **correctness, cache efficiency, readability, and maintainability**.

### Interview Point

**Stable files first → expensive dependency installation → frequently changing application code.**

---

## 117. Why should stable instructions come before frequently changing instructions?

### Short Interview Answer

Docker builds images in layers and can reuse cached layers. If a frequently changing instruction appears early, changes can invalidate the cache for all following instructions.

### Example

Bad:

```dockerfile
FROM python:3.12

COPY . .
RUN pip install -r requirements.txt
RUN python build.py
```

If one source file changes, the `COPY . .` layer changes, so the dependency installation layer may need to run again.

Better:

```dockerfile
FROM python:3.12

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
RUN python build.py
```

Now source-code changes do not necessarily invalidate the dependency-installation layer.

### Why It Matters

Suppose dependency installation takes 3 minutes and source changes happen frequently.

Good cache ordering means:

```text
requirements unchanged
        ↓
dependency layer reused
        ↓
only application-related layers rebuild
```

### Common Interview Trap

Cache reuse is not simply based on "the command text looks the same." Docker evaluates whether the relevant build inputs and previous layers can be reused.

### Interview Point

**Put expensive + stable operations before cheap + frequently changing operations.**

---

## 118. How do you optimize Dockerfile cache usage?

### Short Interview Answer

Use predictable layer boundaries, copy dependency manifests before application source, avoid unnecessary changes to early layers, and keep the build context small.

### Example

For Python:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

For Node.js:

```dockerfile
FROM node:22-slim

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

CMD ["npm", "start"]
```

### Cache-Friendly Pattern

```text
FROM
 ↓
OS/package dependencies
 ↓
application dependency manifest
 ↓
dependency installation
 ↓
application source
 ↓
build
 ↓
runtime configuration
```

The exact pattern varies by language and build system.

### Additional Practices

- Use `.dockerignore`.
- Avoid copying unrelated files into early layers.
- Keep dependency manifests separate from frequently changing source code.
- Avoid unnecessary invalidation of expensive steps.
- Use BuildKit/build cache features where appropriate.

### Common Interview Trap

> "More Dockerfile layers are always bad."

Not necessarily. Layers are useful for caching and image construction. The goal is **sensible layers**, not blindly minimizing their count.

### Interview Point

**Optimize cache invalidation, not merely layer count.**

---

## 119. Why should `apt-get update` and `apt-get install` usually be in the same `RUN` instruction?

### Short Interview Answer

Because package indexes downloaded by `apt-get update` can become stale. Keeping update and install together makes the package installation use a freshly updated index within the same build step.

### Recommended Pattern

```dockerfile
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
       curl \
       ca-certificates \
    && rm -rf /var/lib/apt/lists/*
```

### Why This Is Better

Avoid:

```dockerfile
RUN apt-get update
RUN apt-get install -y curl
```

The cached `apt-get update` layer may be reused even though the package index should have been refreshed.

Instead:

```text
RUN apt-get update + apt-get install
```

makes the dependency operation one logical build step.

### Why `--no-install-recommends`?

It can reduce unnecessary packages when the application does not require Debian/Ubuntu "recommended" packages.

### Common Interview Trap

The reason is **not simply "Docker only allows apt update once."**

The real issue is **build cache + stale package indexes**.

### Interview Point

**Update and install together; clean package lists afterward.**

---

## 120. Why should apt package lists be cleaned?

### Short Interview Answer

`apt-get update` downloads package metadata. If that metadata is not needed at runtime, removing it reduces the final image size.

### Example

```dockerfile
RUN apt-get update \
    && apt-get install -y --no-install-recommends curl \
    && rm -rf /var/lib/apt/lists/*
```

### What Happens?

Conceptually:

```text
apt-get update
      ↓
download package metadata
      ↓
apt-get install
      ↓
application gets required package
      ↓
remove package metadata
      ↓
smaller image
```

### Why It Matters

Smaller images can mean:

- less registry storage
- faster image transfer
- faster deployment/pull
- smaller attack surface in some cases

### Common Interview Trap

Cleaning package lists does **not** uninstall the package itself.

For example:

```bash
rm -rf /var/lib/apt/lists/*
```

removes package metadata, not `curl`.

### Interview Point

**Install what you need, remove package-manager metadata you do not need at runtime.**

---

## 121. Why should you pin base image versions or digests?

### Short Interview Answer

Pinning improves build reproducibility and reduces unexpected changes when an upstream image is updated.

### Less Predictable

```dockerfile
FROM python:3.12
```

The tag can point to a newer image later.

### More Deterministic

```dockerfile
FROM python:3.12@sha256:<digest>
```

A digest identifies a specific image content.

### Trade-Off

Digest pinning improves reproducibility, but it creates a maintenance responsibility:

```text
Pinned digest
     ↓
security/update releases happen
     ↓
digest must be intentionally updated
```

### Important Distinction

A version tag:

```text
python:3.12
```

is a human-readable reference.

A digest:

```text
python@sha256:...
```

is content-addressed.

### Common Interview Trap

> "A version tag is always immutable."

No. Tags are references and can be moved.

### Interview Point

**Tags are convenient; digests provide stronger reproducibility.**

---

## 122. Why avoid `latest` in production Dockerfiles?

### Short Interview Answer

`latest` is a mutable tag and does not communicate an exact application or base-image version.

### Example

Avoid:

```dockerfile
FROM ubuntu:latest
```

Prefer a deliberate versioning strategy such as:

```dockerfile
FROM ubuntu:24.04
```

or, when stronger reproducibility is required:

```dockerfile
FROM ubuntu:24.04@sha256:<digest>
```

### Why `latest` Causes Problems

Today:

```text
latest → image A
```

Later:

```text
latest → image B
```

The same Dockerfile can therefore produce different results at different times.

### Common Interview Trap

`latest` does **not** mean "the image currently running in production."

It is just a tag name.

### Interview Point

**Production builds should have an explicit and controlled versioning strategy.**

---

## 123. Why should containers use non-root users?

### Short Interview Answer

Running the application as a non-root user follows the principle of least privilege and limits the impact of a container compromise.

### Example

```dockerfile
FROM python:3.12-slim

WORKDIR /app

RUN useradd --create-home appuser

COPY --chown=appuser:appuser . .

USER appuser

CMD ["python", "app.py"]
```

### Why It Matters

If an attacker compromises the application, running as a non-root user reduces the privileges available inside the container.

It does **not** make the container automatically secure.

### Important Distinction

```text
USER appuser
```

controls the default user for subsequent Dockerfile instructions and the resulting container process.

It is different from:

```bash
docker run --user ...
```

which can override the runtime user.

### Common Interview Trap

> "Non-root means the process has no privileges."

Not exactly. A non-root process can still have capabilities or access granted through other mechanisms.

### Interview Point

**Non-root is a security baseline, not a complete security model.**

---

## 124. Why use `.dockerignore`?

### Short Interview Answer

`.dockerignore` prevents unnecessary files from being sent as part of the build context.

### Example

```text
.git
.gitignore
node_modules
__pycache__
*.log
.env
.venv
```

### Why It Matters

Without `.dockerignore`, a command such as:

```bash
docker build -t myapp .
```

can send unnecessary files in `.` as build context.

That can:

- increase build time
- increase context size
- reduce cache efficiency
- accidentally expose files to build steps

### Important Security Point

`.dockerignore` is **not a complete secret-management mechanism**.

Do not rely on it as your primary protection for sensitive data.

### Common Interview Trap

> "`.dockerignore` reduces the final image size directly."

Not necessarily.

Its immediate purpose is to reduce/exclude the **build context**. Whether the final image becomes smaller depends on what the Dockerfile copies or otherwise uses.

### Interview Point

**Small build context = faster, cleaner, safer builds.**

---

## 125. Why should secrets not be stored in `ARG` or `ENV`?

### Short Interview Answer

`ARG` and `ENV` are not designed to be secure secret-storage mechanisms. Values can become exposed through image metadata, build history, configuration, or runtime inspection depending on how they are used.

### Bad Example

```dockerfile
ARG DB_PASSWORD
ENV DB_PASSWORD=$DB_PASSWORD
```

Building:

```bash
docker build --build-arg DB_PASSWORD=mysecret .
```

does not make `mysecret` a securely handled secret.

### Better Principle

Use a dedicated secret mechanism supported by your build/runtime environment.

For BuildKit-based builds, for example, secret mounts can be used for build-time secrets rather than embedding them in image layers.

At runtime, use a proper secret manager or orchestrator mechanism where appropriate.

### Important Distinction

```text
ARG
→ primarily build-time configuration

ENV
→ image/runtime environment configuration

Secret mechanism
→ designed to avoid treating credentials as ordinary build/configuration data
```

### Common Interview Trap

> "If I don't use `ENV`, `ARG` secrets are automatically safe."

No. Build arguments can still be exposed through build metadata/history depending on the build process.

### Interview Point

**Configuration is not the same thing as secret management.**

---

## 126. What makes a Dockerfile production-ready?

### Short Interview Answer

A production-ready Dockerfile is reproducible, minimal, cache-efficient, secure by default, clear, and designed for the actual runtime environment.

### Typical Checklist

```text
[ ] Appropriate base image
[ ] Controlled/pinned versions where required
[ ] Small build context
[ ] .dockerignore
[ ] Efficient dependency caching
[ ] No embedded secrets
[ ] Non-root runtime user
[ ] Correct WORKDIR
[ ] Explicit startup command
[ ] Appropriate EXPOSE documentation
[ ] HEALTHCHECK when useful
[ ] Only required OS packages
[ ] Unnecessary package metadata removed
[ ] Predictable configuration
[ ] Clear labels/metadata where useful
```

### Example

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

RUN useradd --create-home appuser

COPY --chown=appuser:appuser . .

USER appuser

EXPOSE 8080

HEALTHCHECK CMD python -c "import urllib.request; urllib.request.urlopen('http://127.0.0.1:8080/health')"

CMD ["python", "app.py"]
```

The exact implementation depends on the application.

### Common Interview Trap

> "Production-ready means the image must be as small as physically possible."

Not necessarily.

The objective is a good balance of:

```text
security
+ reproducibility
+ maintainability
+ performance
+ operational correctness
```

### Interview Point

**Production-ready ≠ merely small.**

---

## 127. What are common Dockerfile anti-patterns?

### Short Interview Answer

Common anti-patterns include using unpinned or uncontrolled dependencies, running as root unnecessarily, copying the entire context too early, embedding secrets, using `latest`, installing unnecessary packages, and creating inefficient cache boundaries.

### Examples

#### Anti-pattern 1 — Secrets

```dockerfile
ENV API_KEY=supersecret
```

#### Anti-pattern 2 — Everything copied first

```dockerfile
COPY . .
RUN pip install -r requirements.txt
```

#### Anti-pattern 3 — Unnecessary root execution

```dockerfile
USER root
CMD ["./app"]
```

when the application does not require root.

#### Anti-pattern 4 — Mutable base reference

```dockerfile
FROM ubuntu:latest
```

when reproducibility is required.

#### Anti-pattern 5 — Huge build context

No `.dockerignore` with:

```text
.git/
node_modules/
logs/
build artifacts/
virtual environments/
```

### Common Interview Trap

Avoid saying:

> "Every Dockerfile with multiple RUN instructions is bad."

The correct question is whether the instructions create useful cache boundaries and a sensible final image.

### Interview Point

**Look for unnecessary privileges, unnecessary data, unnecessary dependencies, and unnecessary rebuilds.**

---

## 128. How should application configuration be handled?

### Short Interview Answer

Application configuration should generally be supplied at runtime rather than hard-coded into the image.

### Example

Dockerfile:

```dockerfile
ENV APP_PORT=8080
```

Runtime override:

```bash
docker run -e APP_PORT=9090 myapp
```

The image contains the application, while the deployment supplies environment-specific configuration.

### Typical Separation

```text
Image
 ├── application code
 ├── runtime dependencies
 └── default configuration

Runtime
 ├── environment-specific values
 ├── service endpoints
 ├── feature flags
 └── secrets
```

### Why It Matters

The same image can then be promoted across environments:

```text
same image
   ↓
development
   ↓
staging
   ↓
production
```

while configuration changes externally.

### Common Interview Trap

`ENV` in a Dockerfile can define defaults, but it should not be treated as a secure secret store.

### Interview Point

**Build once; configure at deployment/runtime.**

---

## 129. What is the Twelve-Factor approach to container configuration?

### Short Interview Answer

The Twelve-Factor methodology recommends storing configuration that varies between deployments in the environment rather than hard-coding it into application code.

### Example

Instead of:

```python
DATABASE_HOST = "prod-db.internal"
```

use:

```python
DATABASE_HOST = os.getenv("DATABASE_HOST")
```

Then:

```bash
docker run \
  -e DATABASE_HOST=prod-db.internal \
  myapp
```

### Why This Fits Containers

Containers are frequently promoted between environments.

```text
                    ┌──────────────┐
                    │ Same Image   │
                    └──────┬───────┘
                           │
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
          Dev config   Stage config   Prod config
```

The application artifact remains the same while deployment-specific configuration changes.

### Important Caveat

Environment variables are not automatically appropriate for every kind of secret or sensitive value. Use dedicated secret-management mechanisms where needed.

### Interview Point

**Keep deployment-specific configuration outside the immutable application artifact.**

---

## 130. How do you reduce unnecessary software in a Docker image?

### Short Interview Answer

Use an appropriate minimal base image, install only required packages, avoid unnecessary recommended packages, remove package metadata, and separate build-time dependencies from runtime dependencies where the build design permits.

### Example

Instead of:

```dockerfile
RUN apt-get update && apt-get install -y \
    curl \
    vim \
    git \
    wget \
    build-essential \
    ...
```

install only what the runtime actually needs.

For Debian/Ubuntu-based images:

```dockerfile
RUN apt-get update \
    && apt-get install -y --no-install-recommends ca-certificates \
    && rm -rf /var/lib/apt/lists/*
```

### Why It Matters

Unnecessary software increases:

- image size
- patching burden
- dependency complexity
- potential attack surface

### Common Interview Trap

Do not blindly choose the smallest-looking image.

For example, a very minimal image can complicate debugging, compatibility, certificate handling, or native-library requirements.

### Interview Point

**Minimize what is unnecessary, not what is necessary.**

---

## 131. How do you make Docker builds reproducible?

### Short Interview Answer

Control the versions of important dependencies, use deterministic dependency installation, control the build context, avoid mutable references where reproducibility matters, and record the exact source/dependency state used to produce the image.

### Example

Less controlled:

```dockerfile
FROM python:3.12

RUN pip install flask
```

More controlled:

```dockerfile
FROM python:3.12@sha256:<digest>

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
```

where `requirements.txt` contains controlled dependency versions.

### Reproducibility Model

```text
Same source
+ same Dockerfile
+ same dependency versions
+ same base image
+ controlled build inputs
        ↓
more predictable image
```

### Important Caveat

Pinning everything does not magically guarantee bit-for-bit identical output in every build environment. Build tools, timestamps, generated artifacts, external inputs, and other factors can affect reproducibility.

### Common Interview Trap

> "Using a version tag guarantees reproducible builds."

A version tag is better than `latest`, but it can still be mutable.

### Interview Point

**Reproducibility means controlling the inputs to the build.**

---

## 132. How should package-manager caches be handled?

### Short Interview Answer

Package-manager caches should be handled deliberately: retain them when they improve build performance through a proper build-cache mechanism, but avoid unnecessarily carrying them into the final runtime image.

### Important Distinction

There are two different concerns:

```text
Build cache
    ↓
speed up future builds

Runtime image contents
    ↓
what gets shipped/deployed
```

A build cache can be useful without putting all cached package data into the final image.

### Example

With a simple Debian/Ubuntu build:

```dockerfile
RUN apt-get update \
    && apt-get install -y --no-install-recommends curl \
    && rm -rf /var/lib/apt/lists/*
```

For more advanced BuildKit builds, cache mounts can preserve package-manager caches across builds without necessarily embedding those caches into the final image.

### Common Interview Trap

> "Delete every cache immediately."

That can be counterproductive during builds if a proper build-cache mechanism could make subsequent builds much faster.

### Interview Point

**Optimize build cache and runtime image separately.**

---

## 133. Why should related package operations sometimes be combined into one `RUN` instruction?

### Short Interview Answer

Related package operations are often combined so that temporary metadata or package-manager state can be cleaned in the same layer and so the resulting image does not retain unnecessary intermediate data.

### Example

Preferred:

```dockerfile
RUN apt-get update \
    && apt-get install -y --no-install-recommends curl \
    && rm -rf /var/lib/apt/lists/*
```

Instead of:

```dockerfile
RUN apt-get update
RUN apt-get install -y curl
RUN rm -rf /var/lib/apt/lists/*
```

### Why?

Docker image layers are immutable.

If unwanted data is created in one layer and deleted in a later layer, the data may still exist in the earlier layer even though it is no longer visible in the final merged filesystem.

Therefore:

```text
create temporary data
        ↓
use it
        ↓
delete it
        ↓
same logical layer
```

is often preferable.

### Common Interview Trap

> "Always combine every RUN instruction."

No.

Unrelated operations may benefit from separate layers for caching, readability, and maintainability.

### Interview Point

**Combine tightly related temporary operations; preserve useful cache boundaries elsewhere.**

---

## 134. What is the difference between image build configuration and runtime configuration?

### Short Interview Answer

Build configuration controls how the image is constructed; runtime configuration controls how a container created from that image behaves.

### Build-Time Examples

```dockerfile
ARG VERSION
RUN make build
COPY app /app
```

These influence image creation.

### Runtime Examples

```dockerfile
ENV APP_ENV=production
USER appuser
WORKDIR /app
ENTRYPOINT ["./app"]
```

and:

```bash
docker run \
  -e APP_ENV=staging \
  -p 8080:8080 \
  myapp
```

### Conceptual Model

```text
Build
  ↓
Dockerfile + build context + build arguments
  ↓
Image
  ↓
Runtime configuration
  ↓
Container
```

### Important Distinction

An image is an artifact.

A container is a runtime instance of that artifact with its own runtime state/configuration.

### Common Interview Trap

`ARG` and `ENV` are not interchangeable.

- `ARG` is primarily for build-time values.
- `ENV` establishes environment variables in the image/runtime environment.
- Runtime flags/environment variables can override or supplement configuration when the container starts.

### Interview Point

**Build creates the artifact; runtime configuration controls its execution.**

---

## 135. How would you review a Dockerfile in an interview?

### Short Interview Answer

I would review it in this order:

1. Base image and version control
2. Build context and `.dockerignore`
3. Dependency installation and cache usage
4. Layer structure
5. Secret handling
6. User/privileges
7. Runtime command
8. Configuration strategy
9. Image size and unnecessary packages
10. Health/operational behavior

### Example Review

Given:

```dockerfile
FROM ubuntu:latest

COPY . .

RUN apt-get update
RUN apt-get install -y python3 python3-pip git vim

ENV DB_PASSWORD=secret

RUN pip3 install -r requirements.txt

CMD python3 app.py
```

I would identify:

### Problem 1 — Mutable base

```dockerfile
FROM ubuntu:latest
```

Use a controlled version/digest strategy.

### Problem 2 — Poor cache structure

```dockerfile
COPY . .
RUN pip3 install ...
```

Copy dependency manifests first where appropriate.

### Problem 3 — Unnecessary packages

```text
git
vim
```

may not be required at runtime.

### Problem 4 — Secret in ENV

```dockerfile
ENV DB_PASSWORD=secret
```

Do not embed credentials this way.

### Problem 5 — Root runtime

No `USER` is specified, so the application may run as root depending on the base image/default configuration.

### Problem 6 — Shell-form CMD

```dockerfile
CMD python3 app.py
```

For a long-running application, exec form is generally preferable:

```dockerfile
CMD ["python3", "app.py"]
```

### Improved Example

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

RUN useradd --create-home appuser

COPY --chown=appuser:appuser . .

USER appuser

EXPOSE 8080

CMD ["python", "app.py"]
```

Configuration and secrets should then be supplied through the deployment/runtime environment rather than hard-coded into the image.

### Common Interview Trap

Do not review a Dockerfile only by asking:

> "How many lines does it have?"

Review it by asking:

```text
Is it reproducible?
Is it cache-efficient?
Is it unnecessarily large?
Is it secure by default?
Does it run correctly?
Can the same image move across environments?
```

### Interview Point

**A strong Dockerfile review covers build efficiency, runtime correctness, reproducibility, and security—not just syntax.**

---

# Quick Revision

| Topic | Key Point |
|---|---|
| Dockerfile order | Stable instructions before frequently changing ones |
| Cache | Optimize invalidation, not simply layer count |
| `apt-get` | Update and install together |
| Apt cleanup | Remove unnecessary package metadata |
| Base image | Prefer controlled versions; use digests when needed |
| `latest` | Mutable tag; avoid uncontrolled production use |
| Non-root | Principle of least privilege |
| `.dockerignore` | Reduces build context; not a secret store |
| `ARG` | Primarily build-time |
| `ENV` | Image/runtime environment configuration |
| Secrets | Use dedicated secret mechanisms |
| Configuration | Prefer runtime/deployment configuration |
| Production Dockerfile | Reproducible + efficient + secure + operationally correct |
| Package caches | Separate build-cache concerns from runtime image contents |
| `RUN` grouping | Combine tightly related temporary package operations |
| Reproducibility | Control important build inputs |
| Dockerfile review | Check correctness, cache, size, security, and reproducibility |

---

# High-Value Interview Traps

### Trap 1 — "`latest` means newest and immutable."

**Wrong.**

`latest` is simply a tag. Tags can move.

---

### Trap 2 — "More layers always mean a bad image."

**Wrong.**

Layers provide caching and filesystem composition. The goal is sensible layer design.

---

### Trap 3 — "Delete files in a later layer to reduce the original layer."

**Not necessarily.**

If data was created in an earlier immutable layer, deleting it later does not remove the data from that earlier layer.

---

### Trap 4 — "`.dockerignore` protects secrets."

**Wrong.**

It excludes files from the build context, but it is not a substitute for proper secret management.

---

### Trap 5 — "`ENV` is the correct place for passwords."

**Wrong.**

Treat secrets separately from ordinary configuration.

---

### Trap 6 — "Non-root containers are completely secure."

**Wrong.**

Non-root reduces privileges but does not eliminate all container security risks.

---

### Trap 7 — "The smallest possible image is always the best image."

**Wrong.**

The image must still provide the libraries, certificates, runtime, diagnostics, and compatibility required by the application.

---

### Trap 8 — "Build cache and runtime cache are the same thing."

**Wrong.**

Build cache accelerates image construction; runtime filesystem/cache contents are part of the deployed container environment.

---

# Interview Follow-Up Questions

After answering Dockerfile best practices, an interviewer may continue with:

1. How does Docker determine whether a layer can use cache?
2. What causes Docker build cache invalidation?
3. How does BuildKit improve Docker builds?
4. What is a cache mount?
5. How do multistage builds reduce final image size?
6. How do you pass build-time secrets securely?
7. How do you scan a Docker image for vulnerabilities?
8. Why should containers normally run as non-root?
9. What is the difference between an image tag and digest?
10. How would you troubleshoot a Docker build that suddenly became slow?

---

# Final Interview Answer

> **"When I write a production Dockerfile, I focus on four major areas: reproducibility, build efficiency, runtime correctness, and security. I use a controlled base image, keep the build context small with `.dockerignore`, place stable dependency instructions before frequently changing source code for better caching, combine related package-manager operations and clean temporary metadata, avoid hard-coded secrets, run the application as a non-root user where possible, and provide an explicit runtime command. I also keep environment-specific configuration outside the image so the same image can be promoted across environments."**

---

# One-Line Memory Map

```text
GOOD DOCKERFILE
= controlled base
+ small context
+ smart cache order
+ minimal dependencies
+ no embedded secrets
+ non-root runtime
+ runtime configuration
+ reproducible inputs
+ explicit startup
```
