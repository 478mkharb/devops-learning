# Docker Interview Preparation — Topic 5: Dockerfile Fundamentals

> **Goal:** Understand how a Dockerfile turns application source and dependencies into a Docker image.
>
> **Interview focus:** Know what each common instruction does, when it executes, what it affects, and the traps around `RUN`, `CMD`, `ENTRYPOINT`, `COPY`, `ADD`, `ARG`, `ENV`, `EXPOSE`, `VOLUME`, `WORKDIR`, and `USER`.

---

## Q87. What is a Dockerfile?

### Short Interview Answer

A Dockerfile is a text file containing **instructions used by a Docker build system to create a container image**.

### Example

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

USER 1000

CMD ["python", "app.py"]
```

This describes how to build an image for a Python application.

### Build Flow

```text
Dockerfile
     +
Build Context
     │
     ▼
Docker Build System
     │
     ▼
Docker Image
     │
     ▼
Container
```

### Important Point

A Dockerfile describes **image construction**.

It does not directly start the application when you execute:

```bash
docker build
```

The application is normally started later when a container is created/run.

### Interview Point

> **Dockerfile = instructions for building an image.**

---

## Q88. What is the difference between a Dockerfile and a Docker image?

### Short Interview Answer

A **Dockerfile** is the set of build instructions; a **Docker image** is the resulting packaged artifact produced from those instructions and the build context.

### Comparison

| Dockerfile | Docker Image |
|---|---|
| Text instructions | Built artifact |
| Describes how to build | Contains resulting filesystem/configuration |
| Human-readable | Stored in image format |
| Input to build | Output of build |
| Can be version-controlled | Can be tagged/pushed/pulled |

### Example

```text
Dockerfile
    │
    │ docker build
    ▼
Docker Image
    │
    │ docker run
    ▼
Container
```

### Interview Point

This three-step relationship is fundamental:

```text
Dockerfile → Image → Container
```

---

## Q89. What is the purpose of the `FROM` instruction?

### Short Interview Answer

`FROM` specifies the **base image** for a Docker build stage and establishes the initial filesystem/environment from which that stage is built.

### Example

```dockerfile
FROM ubuntu:24.04
```

The build starts from the specified Ubuntu image.

Another example:

```dockerfile
FROM python:3.12
```

The stage starts with the filesystem and configuration provided by that Python image.

### Why `FROM` Is Important

Most Dockerfiles build on an existing base image rather than constructing an entire filesystem from scratch.

Conceptually:

```text
Base Image
     ↓
Install dependencies
     ↓
Copy application
     ↓
Configure runtime
     ↓
Final Image
```

### `FROM scratch`

You can also use:

```dockerfile
FROM scratch
```

which starts from an empty base.

This is useful for specially prepared minimal images, such as certain statically compiled applications.

### Multiple `FROM`

A Dockerfile can contain multiple `FROM` instructions.

This is the basis of **multi-stage builds**:

```dockerfile
FROM golang:1.24 AS builder
# build application

FROM alpine:latest
# copy application from builder
```

### Interview Trap

`FROM` does not mean:

> "Start a VM using Ubuntu."

It establishes the base filesystem/image for a build stage.

### Interview Point

```text
FROM → base image / build stage
```

---

## Q90. What is the purpose of the `RUN` instruction?

### Short Interview Answer

`RUN` executes a command **during image build time** and records the resulting filesystem changes in the image build result.

### Example

```dockerfile
FROM ubuntu:24.04

RUN apt-get update
RUN apt-get install -y nginx
```

These commands execute while the image is being built.

### Build Time vs Runtime

This is critical:

```text
docker build
    │
    └── RUN executes here
             ↓
         Image created
             ↓
docker run
    │
    └── CMD/ENTRYPOINT normally starts here
```

### Example

```dockerfile
RUN apt-get install -y curl
```

means:

> Install `curl` into the image during the build.

It does not mean:

> Install `curl` every time a container starts.

### Why This Matters

If you put setup work in `RUN`, it becomes part of the image build process and can be cached/reused depending on the build inputs and cache state.

### Common Mistake

```dockerfile
RUN python app.py
```

would execute the application **during image build**, which is usually not what you want.

Normally:

```dockerfile
CMD ["python", "app.py"]
```

starts the application when the container runs.

### Interview Point

```text
RUN → build time
```

---

## Q91. What is the purpose of the `CMD` instruction?

### Short Interview Answer

`CMD` specifies the **default command and/or default arguments** to use when a container is started from the image.

### Example

```dockerfile
CMD ["python", "app.py"]
```

When you run:

```bash
docker run myapp
```

Docker uses that default command.

### Important Override Behavior

If you run:

```bash
docker run myapp python test.py
```

the runtime command supplied after the image name replaces the image's default `CMD` behavior.

Conceptually:

```text
Image
 └── CMD ["python", "app.py"]

docker run myapp
        ↓
python app.py
```

But:

```bash
docker run myapp python test.py
```

results in the runtime command:

```text
python test.py
```

### CMD Is a Default

Think:

```text
CMD = "If the user does not specify otherwise, run/use this."
```

### JSON/Exec Form

Preferred for executable commands:

```dockerfile
CMD ["python", "app.py"]
```

### Shell Form

```dockerfile
CMD python app.py
```

This involves shell-form command processing and can affect signal handling.

### Interview Point

> **CMD provides the default runtime command/arguments and can be overridden at container startup.**

---

## Q92. What is the purpose of the `ENTRYPOINT` instruction?

### Short Interview Answer

`ENTRYPOINT` defines the **primary executable** for the container and is designed to remain the main command even when additional arguments are supplied at runtime.

### Example

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

Running:

```bash
docker run myapp
```

results conceptually in:

```text
python app.py
```

Running:

```bash
docker run myapp test.py
```

results conceptually in:

```text
python test.py
```

The runtime argument replaces the default `CMD` arguments while the `ENTRYPOINT` remains.

### Why Combine ENTRYPOINT and CMD?

This pattern is extremely useful:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

Think:

```text
ENTRYPOINT → executable
CMD        → default arguments
```

### Overriding ENTRYPOINT

Docker can explicitly override the entrypoint:

```bash
docker run --entrypoint /bin/sh myapp
```

Now `/bin/sh` becomes the entrypoint for that container invocation.

### Interview Point

> **ENTRYPOINT defines the primary executable; CMD commonly supplies default arguments.**

---

## Q93. What is the difference between `CMD` and `ENTRYPOINT`?

### Short Interview Answer

`CMD` provides **default command/arguments that are easy to override**, while `ENTRYPOINT` defines the container's primary executable and is normally preserved when runtime arguments are supplied.

### Example

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

Normal:

```bash
docker run myapp
```

Conceptually:

```text
python app.py
```

Override default arguments:

```bash
docker run myapp test.py
```

Conceptually:

```text
python test.py
```

Override entrypoint:

```bash
docker run --entrypoint /bin/sh myapp
```

Conceptually:

```text
/bin/sh
```

### Comparison

| `CMD` | `ENTRYPOINT` |
|---|---|
| Default command/arguments | Primary executable |
| Easily replaced by runtime command | Usually remains unless explicitly overridden |
| Can provide default arguments | Commonly combined with CMD |
| Useful for defaults | Useful for executable identity |

### Interview Trap

Do not say:

> "ENTRYPOINT cannot be overridden."

It can be overridden with:

```bash
docker run --entrypoint ...
```

### Interview Point

Memorize:

```text
ENTRYPOINT = executable
CMD        = default arguments
```

This is the cleanest mental model when they are combined.

---

## Q94. What is the difference between shell form and exec form in Dockerfile?

### Short Interview Answer

**Exec form** uses a JSON array and starts the specified executable directly, while **shell form** passes the command through a shell. Exec form is generally preferred for long-running applications because it avoids an unnecessary intermediate shell and gives more predictable signal behavior.

### Exec Form

```dockerfile
CMD ["python", "app.py"]
```

Conceptually:

```text
Container PID 1
      ↓
python app.py
```

### Shell Form

```dockerfile
CMD python app.py
```

Conceptually, shell processing is involved:

```text
Container
    ↓
/bin/sh -c "python app.py"
    ↓
python app.py
```

The exact process tree depends on the shell and command.

### Why Exec Form Is Usually Better

Docker's graceful-stop behavior relies on signals reaching the container's main process.

With exec form, the intended application can run directly as PID 1:

```text
PID 1 → application
```

This generally makes signal handling more predictable.

### Shell Wrapper Example

If a script is PID 1:

```dockerfile
ENTRYPOINT ["./start.sh"]
```

and the script launches:

```bash
python app.py
```

without replacing itself, the application may become a child process.

Using:

```bash
exec python app.py
```

allows the shell to replace itself with Python.

### Interview Trap

Avoid saying:

> "Shell form never receives signals."

That is too absolute.

The precise point is:

> **Exec form avoids an intermediate shell, allowing the intended application to run directly as PID 1 and generally making signal handling more predictable.**

### Interview Point

```text
Exec form  → direct executable
Shell form → shell command processing
```

---

## Q95. What is the purpose of `COPY`?

### Short Interview Answer

`COPY` copies files and directories from the **build context** or from an available build stage into the image filesystem.

### Basic Example

```dockerfile
COPY app.py /app/
```

This copies:

```text
Build context
└── app.py
```

into:

```text
Image
└── /app/app.py
```

### Copying a Build Artifact

With multi-stage builds:

```dockerfile
FROM golang:1.24 AS builder
WORKDIR /src
COPY . .
RUN go build -o /out/app

FROM alpine:latest
COPY --from=builder /out/app /app
```

Here:

```text
Builder stage
      ↓
Compiled binary
      ↓
Final stage
```

### Why `COPY` Is Commonly Preferred

It has straightforward semantics:

> **Copy files from the build context or another stage.**

### Interview Point

Use `COPY` when you simply need to copy application files or build artifacts.

---

## Q96. What is the difference between `COPY` and `ADD`?

### Short Interview Answer

Both can copy files into an image, but `ADD` has additional behavior such as support for extracting local tar archives and certain URL-related behavior, while `COPY` is intentionally simpler and is generally preferred when plain copying is all that is required.

### `COPY`

```dockerfile
COPY app.py /app/
```

Simple file copy.

### `ADD`

```dockerfile
ADD archive.tar /app/
```

Depending on the build behavior, a local tar archive can be automatically extracted.

### Why Prefer COPY?

For ordinary file copying:

```dockerfile
COPY . /app/
```

is clearer about intent.

Using `ADD` when its extra behavior is not needed can make the Dockerfile less explicit.

### Interview Trap

Do not say:

> "`ADD` is always better because it does more."

More behavior is not automatically better.

### Interview Point

```text
COPY → straightforward copy
ADD  → copy + additional Docker build behavior
```

---

## Q97. What is the Docker build context?

### Short Interview Answer

The build context is the set of files made available to the Docker build process for instructions that reference build-context files, such as `COPY`.

### Example

```bash
docker build -t myapp .
```

Here:

```text
. → build context
```

Conceptually:

```text
Current directory
├── Dockerfile
├── app.py
├── requirements.txt
└── README.md
       │
       ▼
   Build Context
```

### Why Context Matters

If you write:

```dockerfile
COPY . /app/
```

Docker can copy files from the build context into the image.

It cannot arbitrarily copy files from anywhere on the host filesystem outside the allowed build context.

### `.dockerignore`

You can exclude files:

```text
.git
node_modules
*.log
build/
```

This reduces the context sent to the builder and prevents unnecessary files from being considered by build instructions.

### Interview Trap

In:

```bash
docker build -t myapp .
```

the `.` is not the Dockerfile itself.

It specifies the **build context**.

### Interview Point

```text
docker build ... .
              ↑
          build context
```

---

## Q98. What is `.dockerignore`?

### Short Interview Answer

`.dockerignore` specifies files and directories that should be excluded from the Docker build context.

### Example

```text
.git
.gitignore
node_modules
*.log
.env
coverage/
```

### Why Use It?

Suppose your project contains:

```text
Project
├── .git/
├── node_modules/
├── logs/
├── coverage/
├── source/
└── Dockerfile
```

You may not want all of that sent as build context.

A `.dockerignore` can reduce unnecessary context.

### Benefits

It can improve:

- Build performance
- Context transfer time
- Cache behavior
- Build reproducibility
- Accidental inclusion control

### Important Security Point

Do not rely on `.dockerignore` as a complete secret-management mechanism.

For example:

```text
.env
```

being excluded from the context is useful, but secrets should still be handled through appropriate secret mechanisms rather than copied into images.

### Interview Point

> **`.dockerignore` controls what enters the build context; it does not delete files from your host.**

---

## Q99. What is the purpose of `WORKDIR`?

### Short Interview Answer

`WORKDIR` sets the **working directory** for subsequent Dockerfile instructions and the default working directory of the resulting container unless overridden.

### Example

```dockerfile
WORKDIR /app

COPY . .

CMD ["python", "app.py"]
```

The application runs with:

```text
Working directory = /app
```

### Why Prefer WORKDIR?

Instead of:

```dockerfile
RUN cd /app
RUN ...
```

use:

```dockerfile
WORKDIR /app
```

The `cd` in one `RUN` instruction does not persist as shell state into a later `RUN` instruction.

### Example

This is misleading:

```dockerfile
RUN cd /app
RUN pwd
```

The second `RUN` should not be expected to inherit the shell's directory change from the first command.

Use:

```dockerfile
WORKDIR /app
RUN pwd
```

### Interview Point

```text
WORKDIR → persistent working-directory setting for later instructions/runtime
```

---

## Q100. What is the purpose of `ENV`?

### Short Interview Answer

`ENV` sets environment variables in the image configuration, making them available to subsequent build steps and, by default, to containers created from that image.

### Example

```dockerfile
ENV APP_ENV=production
```

A container created from the image can see:

```text
APP_ENV=production
```

unless overridden at runtime.

### Runtime Override

For example:

```bash
docker run -e APP_ENV=staging myapp
```

can override the image's default environment value.

### Important Security Warning

Do not use `ENV` for secrets.

For example:

```dockerfile
ENV DB_PASSWORD=secret123
```

is a bad practice because image configuration/history and related metadata can expose sensitive information.

### Build-Time vs Runtime

```text
ENV
 ↓
Image configuration
 ↓
Container environment
```

It is not the same as `ARG`.

### Interview Point

> **ENV is for environment configuration; it is not a secure secret store.**

---

## Q101. What is the difference between `ARG` and `ENV`?

### Short Interview Answer

`ARG` defines a **build-time variable**, while `ENV` defines an environment variable that is part of the image configuration and is available to containers by default.

### Example

```dockerfile
ARG APP_VERSION=1.0
ENV APP_ENV=production
```

### ARG

Used during image build:

```bash
docker build \
  --build-arg APP_VERSION=2.0 \
  -t myapp:2.0 .
```

The value is available to relevant build instructions.

### ENV

Available in the resulting container environment:

```bash
docker run myapp
```

can expose:

```text
APP_ENV=production
```

### Comparison

| `ARG` | `ENV` |
|---|---|
| Build-time variable | Image/runtime environment variable |
| Mainly for build configuration | Runtime configuration |
| Not automatically available in final container environment | Available by default in containers |
| Can influence build results | Becomes part of image configuration |

### Important Security Warning

Neither should be treated as a secure secret store.

Do not put:

```text
passwords
API keys
private credentials
```

into ordinary `ARG`/`ENV` values.

Use appropriate build/runtime secret mechanisms.

### Interview Point

```text
ARG → build
ENV → runtime environment
```

---

## Q102. What is the purpose of `USER`?

### Short Interview Answer

`USER` specifies the user and optionally group under which subsequent Dockerfile instructions and the container's default process run.

### Example

```dockerfile
RUN useradd --create-home appuser

USER appuser

CMD ["python", "app.py"]
```

The application runs as:

```text
appuser
```

instead of root.

### Why It Matters

Running applications as non-root can reduce the impact of a compromise.

### Build-Time Effect

`USER` also affects subsequent Dockerfile instructions that execute commands.

For example:

```dockerfile
USER appuser
RUN touch /app/file
```

requires `appuser` to have permission to perform the operation.

### Interview Trap

Do not assume:

> "USER only affects runtime."

It can affect both subsequent build instructions and the default runtime user.

### Interview Point

```text
USER → default identity for subsequent build steps + runtime
```

---

## Q103. What is the purpose of `EXPOSE`?

### Short Interview Answer

`EXPOSE` documents the network ports that the application in the image is intended to listen on; it does **not** publish those ports to the host.

### Example

```dockerfile
EXPOSE 8080
```

This communicates:

> The application expects to listen on TCP port 8080.

### What It Does Not Do

This does not make:

```text
Host:8080
```

automatically reachable.

To publish a port:

```bash
docker run -p 8080:8080 myapp
```

### Important Distinction

```text
EXPOSE
   ↓
Image metadata/documentation

-p
   ↓
Host-to-container port publishing
```

### Interview Trap

**Wrong:**

> "`EXPOSE 8080` opens port 8080 on the host."

No.

### Interview Point

Memorize:

> **EXPOSE documents; `-p` publishes.**

---

## Q104. What is the purpose of `VOLUME`?

### Short Interview Answer

`VOLUME` declares a mount point intended for persistent or separately managed data and causes the container runtime to treat that path as a volume mount point.

### Example

```dockerfile
VOLUME ["/var/lib/myapp"]
```

The image declares that:

```text
/var/lib/myapp
```

is intended to be backed by separate storage.

### Why Volumes?

Application data often should not depend on the container's writable layer.

Conceptually:

```text
Container
   │
   └── /var/lib/myapp
           │
           ▼
         Volume
```

### Important Nuance

`VOLUME` is a declaration in the image.

It does not mean:

> "This data is automatically backed up."

Nor does it replace understanding how the volume is created, mounted, retained, backed up, and managed.

### Runtime Mount

You can explicitly provide storage at runtime:

```bash
docker run \
  --mount source=mydata,target=/var/lib/myapp \
  myapp
```

### Interview Point

```text
VOLUME → declare data mount point
```

---

## Q105. What is the purpose of `LABEL`?

### Short Interview Answer

`LABEL` adds metadata to an image, such as ownership, version, documentation, or organizational information.

### Example

```dockerfile
LABEL org.opencontainers.image.title="Payment API"
LABEL org.opencontainers.image.version="1.2.0"
```

### Why Use Labels?

They can help with:

- Image identification
- Ownership
- Automation
- Documentation
- Inventory
- Compliance metadata

### Inspecting Labels

```bash
docker image inspect myapp
```

can show image labels.

### Interview Point

Labels are metadata, not application environment variables.

---

## Q106. What is the purpose of `HEALTHCHECK`?

### Short Interview Answer

`HEALTHCHECK` defines a command Docker can use to determine whether the application inside a container is **healthy**, independently of whether its main process is merely running.

### Example

```dockerfile
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD curl --fail http://localhost:8080/health || exit 1
```

### Why Is This Useful?

A process can be running but the application can still be unhealthy.

For example:

```text
Process running
      ↓
Database connection broken
      ↓
Application cannot serve requests
```

A health check can detect this.

### States

A container can have a health status such as:

```text
starting
healthy
unhealthy
```

depending on the configured health check and current state.

### Important Distinction

```text
Process state
    ≠
Application health
```

A process being alive does not guarantee the service is usable.

### Interview Point

> **HEALTHCHECK tests application health, not simply process existence.**

---

## Q107. What is the purpose of `SHELL`?

### Short Interview Answer

`SHELL` changes the default shell used by shell-form commands in subsequent Dockerfile instructions.

### Example

On Linux, the default shell commonly used for shell-form instructions is:

```text
/bin/sh -c
```

A Dockerfile can specify another shell where supported:

```dockerfile
SHELL ["/bin/bash", "-c"]
```

### Why Use It?

This can matter when build commands depend on shell-specific features.

For example:

```dockerfile
SHELL ["/bin/bash", "-c"]

RUN source /etc/profile && echo "$PATH"
```

### Important Point

`SHELL` affects shell-form instruction processing.

It does not turn exec-form commands into shell commands.

### Interview Point

```text
SHELL → controls shell used by shell-form commands
```

---

## Q108. What is the purpose of `STOPSIGNAL`?

### Short Interview Answer

`STOPSIGNAL` specifies the system signal that should be used to request the container's main process to stop.

### Example

```dockerfile
STOPSIGNAL SIGTERM
```

### Why Is It Useful?

Different applications may expect different termination signals.

The signal is part of the container/image configuration and influences graceful shutdown behavior.

### Relationship to PID 1

```text
Docker stop
     ↓
Configured stop signal
     ↓
Container main process / PID 1
     ↓
Graceful shutdown
```

If the process does not exit within the applicable timeout, Docker can ultimately force termination.

### Interview Point

`STOPSIGNAL` controls the stop signal; it does not itself guarantee that the application will shut down gracefully.

---

## Q109. What is `ONBUILD`?

### Short Interview Answer

`ONBUILD` registers a build instruction that is triggered when the current image is later used as the base image of another build.

### Example

```dockerfile
ONBUILD COPY . /src
```

The instruction is not executed as part of the current image's normal build in the same way as an ordinary `COPY`.

Instead, it is stored as a trigger for a future child build using this image as a base.

### Conceptual Flow

```text
Base image build
      │
      └── stores ONBUILD trigger
                    │
                    ▼
Child Dockerfile
FROM base-image
                    │
                    ▼
ONBUILD instruction triggers
```

### Why Is It Less Common?

Modern build practices often favor explicit multi-stage builds and clearer Dockerfiles.

`ONBUILD` can still be useful for specialized base images, but it should be used carefully because behavior is deferred to downstream builds.

### Interview Point

```text
ONBUILD → instruction for a future child build
```

---

## Q110. What is the difference between `RUN`, `CMD`, and `ENTRYPOINT`?

### Short Interview Answer

- `RUN` executes during **image build**.
- `CMD` defines a **default runtime command/arguments**.
- `ENTRYPOINT` defines the **primary runtime executable**.

### Comparison

| Instruction | When | Purpose |
|---|---|---|
| `RUN` | Build time | Create/configure image |
| `CMD` | Container runtime | Default command/arguments |
| `ENTRYPOINT` | Container runtime | Primary executable |

### Example

```dockerfile
FROM python:3.12

RUN pip install flask

WORKDIR /app
COPY app.py .

ENTRYPOINT ["python"]
CMD ["app.py"]
```

### What Happens?

Build:

```text
RUN pip install flask
```

creates/configures the image.

Runtime:

```text
ENTRYPOINT + CMD
       ↓
python app.py
```

### Interview Point

This is one of the most important Dockerfile distinctions:

```text
RUN        → build
CMD        → default runtime
ENTRYPOINT → primary runtime executable
```

---

## Q111. What is the difference between `COPY`, `ADD`, and `RUN`?

### Short Interview Answer

`COPY` and `ADD` place content into the image, while `RUN` executes a command during image construction.

### Comparison

| Instruction | Main Purpose |
|---|---|
| `COPY` | Copy files into image |
| `ADD` | Copy files + additional behavior |
| `RUN` | Execute build-time command |

### Example

```dockerfile
COPY requirements.txt /app/
RUN pip install -r /app/requirements.txt
```

Flow:

```text
requirements.txt
       ↓
COPY
       ↓
Image filesystem
       ↓
RUN pip install
       ↓
Dependencies installed
```

### Interview Trap

Do not say:

> "`COPY` installs files."

It copies them.

`RUN` performs the installation command.

### Interview Point

```text
COPY/ADD → put files there
RUN      → do something with them
```

---

## Q112. What is the difference between `ARG`, `ENV`, and runtime `-e`?

### Short Interview Answer

`ARG` is primarily for build-time variables, `ENV` defines image/container environment defaults, and `docker run -e` supplies or overrides environment variables at container runtime.

### Example

```dockerfile
ARG APP_VERSION=1.0
ENV APP_ENV=production
```

Build:

```bash
docker build \
  --build-arg APP_VERSION=2.0 \
  -t myapp:2.0 .
```

Runtime:

```bash
docker run \
  -e APP_ENV=staging \
  myapp:2.0
```

Conceptually:

```text
ARG
 ↓
Build configuration

ENV
 ↓
Image default environment

-e
 ↓
Runtime environment override
```

### Important Security Point

Do not use any of these ordinary mechanisms as a secure place for secrets.

### Interview Point

```text
ARG → build
ENV → image/runtime default
-e  → runtime override/input
```

---

## Q113. What is the difference between `WORKDIR` and `RUN cd`?

### Short Interview Answer

`WORKDIR` sets the working directory for subsequent Dockerfile instructions and the container runtime, while `RUN cd` changes directory only within that particular shell/process executing that `RUN` instruction.

### Example

Not a persistent approach:

```dockerfile
RUN cd /app
RUN pwd
```

The second command should not be expected to run from `/app`.

Correct:

```dockerfile
WORKDIR /app
RUN pwd
```

Now later instructions use `/app` as the working directory.

### Why?

Each `RUN` instruction is executed in its own build step/process context.

The shell's current directory change does not become a Dockerfile-wide setting.

### Interview Point

```text
RUN cd   → temporary command-shell change
WORKDIR  → Dockerfile/runtime configuration
```

---

## Q114. Does `EXPOSE` publish a port?

### Short Interview Answer

**No.** `EXPOSE` documents the port intended for the application; it does not publish the port to the host.

### Example

Dockerfile:

```dockerfile
EXPOSE 8080
```

Runtime:

```bash
docker run myapp
```

does not by itself mean:

```text
Host port 8080 → Container port 8080
```

To publish:

```bash
docker run -p 8080:8080 myapp
```

### Mental Model

```text
EXPOSE
  ↓
Metadata/documentation

-p
  ↓
Port publishing
```

### Interview Point

This is a classic Docker interview trap.

---

## Q115. Does `VOLUME` guarantee data persistence?

### Short Interview Answer

No. `VOLUME` declares a mount point intended for separate storage, but persistence still depends on the actual volume/storage lifecycle and how it is mounted and managed.

### Important Distinction

```text
VOLUME instruction
       ↓
Declares mount point

Actual volume
       ↓
Stores data independently of container writable layer
```

### Example

```dockerfile
VOLUME ["/data"]
```

This does not mean:

```text
Automatic backup
```

It means the path is intended to use volume storage.

### Interview Point

Never equate:

```text
VOLUME = backup
```

They are completely different concepts.

---

# Dockerfile Example — Putting the Fundamentals Together

Consider:

```dockerfile
FROM python:3.12-slim

ARG APP_VERSION=1.0

ENV APP_ENV=production

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

RUN useradd --create-home appuser

USER appuser

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://127.0.0.1:8080/health')"

ENTRYPOINT ["python"]

CMD ["app.py"]
```

### What Happens During Build?

```text
FROM
 ↓
Base image

ARG
 ↓
Build variable

ENV
 ↓
Image environment

WORKDIR
 ↓
Set working directory

COPY requirements.txt
 ↓
Copy dependency definition

RUN pip install
 ↓
Install dependencies

COPY .
 ↓
Copy application

RUN useradd
 ↓
Create user

USER
 ↓
Subsequent build/runtime identity

EXPOSE
 ↓
Document port

HEALTHCHECK
 ↓
Configure health test

ENTRYPOINT + CMD
 ↓
Configure runtime
```

### What Happens During Runtime?

```text
docker run myapp
       ↓
ENTRYPOINT + CMD
       ↓
python app.py
       ↓
Application listens on 8080
```

If you run:

```bash
docker run -p 8080:8080 myapp
```

then Docker also publishes the host port.

---

# Quick Revision — Topic 5

| Instruction | Core Meaning |
|---|---|
| `FROM` | Base image/build stage |
| `RUN` | Execute during build |
| `CMD` | Default runtime command/arguments |
| `ENTRYPOINT` | Primary runtime executable |
| `COPY` | Copy files into image |
| `ADD` | Copy + additional behavior |
| `WORKDIR` | Set working directory |
| `ENV` | Image/runtime environment variable |
| `ARG` | Build-time variable |
| `USER` | Build/runtime user identity |
| `EXPOSE` | Document intended container port |
| `VOLUME` | Declare mount point |
| `LABEL` | Image metadata |
| `HEALTHCHECK` | Application health test |
| `SHELL` | Shell for shell-form instructions |
| `STOPSIGNAL` | Stop signal configuration |
| `ONBUILD` | Deferred instruction for child builds |
| `.dockerignore` | Exclude files from build context |

---

# The Most Important Dockerfile Distinctions

## 1. Build Time vs Runtime

```text
                 Dockerfile
                     │
          ┌──────────┴──────────┐
          │                     │
       BUILD                  RUNTIME
          │                     │
      FROM/RUN             ENTRYPOINT/CMD
      COPY/ADD
      ARG
```

## 2. CMD vs ENTRYPOINT

```text
ENTRYPOINT → executable
CMD        → default arguments
```

Example:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

Result:

```text
python app.py
```

Override CMD:

```bash
docker run myapp test.py
```

Result:

```text
python test.py
```

Override ENTRYPOINT:

```bash
docker run --entrypoint /bin/sh myapp
```

---

## 3. EXPOSE vs `-p`

```text
EXPOSE
   ↓
Documentation / metadata

-p
   ↓
Actual host port publishing
```

---

## 4. ARG vs ENV

```text
ARG
 ↓
Build-time

ENV
 ↓
Image/container environment
```

Neither should be used as a secure secret store.

---

## 5. COPY vs ADD

```text
COPY
 ↓
Straightforward copy

ADD
 ↓
Copy + additional behavior
```

Use `COPY` when you simply need to copy files.

---

# High-Value Interview Traps

### Trap 1 — `RUN` starts the application

❌ No.

`RUN` executes during image build.

### Trap 2 — `CMD` is mandatory

❌ No.

An image can have neither `CMD` nor `ENTRYPOINT`, although a runnable application image normally needs an appropriate startup configuration or runtime command.

### Trap 3 — `ENTRYPOINT` cannot be overridden

❌ No.

Use:

```bash
docker run --entrypoint ...
```

### Trap 4 — `EXPOSE` opens the host port

❌ No.

Use:

```bash
docker run -p host:container
```

### Trap 5 — `ENV` is a secret store

❌ No.

Image configuration can expose environment values.

### Trap 6 — `ARG` is automatically available in the running container

❌ No.

`ARG` is primarily a build-time variable.

### Trap 7 — `RUN cd /app` changes the working directory permanently

❌ No.

Use:

```dockerfile
WORKDIR /app
```

### Trap 8 — `ADD` should always replace `COPY`

❌ No.

Prefer `COPY` for straightforward copying.

### Trap 9 — `VOLUME` means automatic backup

❌ No.

A volume provides separate storage; backup is a separate operational concern.

### Trap 10 — Shell form is always wrong

❌ No.

Shell form can be useful when shell features are intentionally required. Exec form is generally preferable for long-running applications when direct process/signal behavior is desired.

---

# Interview Follow-Up Questions

After explaining Dockerfile fundamentals, expect questions such as:

1. What is a Dockerfile?
2. What does `FROM` do?
3. What does `RUN` do?
4. What does `CMD` do?
5. What does `ENTRYPOINT` do?
6. Difference between `CMD` and `ENTRYPOINT`?
7. Difference between shell and exec form?
8. Why is exec form preferred for application containers?
9. What happens to PID 1?
10. What does `COPY` do?
11. Difference between `COPY` and `ADD`?
12. What is build context?
13. What is `.dockerignore`?
14. What does `WORKDIR` do?
15. Difference between `WORKDIR` and `RUN cd`?
16. What does `ENV` do?
17. Difference between `ARG` and `ENV`?
18. What does `USER` do?
19. Why run as non-root?
20. What does `EXPOSE` do?
21. Does `EXPOSE` publish a port?
22. What does `VOLUME` do?
23. Does `VOLUME` guarantee persistence?
24. What does `HEALTHCHECK` do?
25. Difference between process running and container healthy?
26. What does `STOPSIGNAL` do?
27. What does `SHELL` do?
28. What does `ONBUILD` do?
29. How would you optimize Dockerfile layer caching?
30. How would you prevent secrets from entering an image?

---

# Final Interview Answer — "Explain the Important Dockerfile Instructions"

If the interviewer asks:

> **"Explain the important Dockerfile instructions."**

A strong answer is:

> "A Dockerfile contains instructions used to build an image. `FROM` selects the base image or starts a build stage. `RUN` executes commands during image construction. `COPY` and `ADD` place files into the image, with `ADD` providing additional behavior beyond straightforward copying. `WORKDIR` sets the working directory. `ARG` provides build-time variables, while `ENV` provides environment defaults that are available to containers. `USER` sets the user for subsequent build steps and the default runtime process. `EXPOSE` documents an intended container port but does not publish it. `VOLUME` declares a mount point for separately managed storage. `ENTRYPOINT` defines the primary runtime executable and `CMD` supplies the default command or arguments. For application containers, exec-form `ENTRYPOINT`/`CMD` is generally preferred because it avoids an unnecessary shell and gives more predictable PID 1 and signal behavior. `HEALTHCHECK` can report application health independently of whether the main process is running."

---

# One-Line Memory Map

```text
Dockerfile
    │
    ├── FROM       → base
    ├── RUN        → build
    ├── COPY/ADD   → files
    ├── WORKDIR    → directory
    ├── ARG        → build variable
    ├── ENV        → runtime environment
    ├── USER       → identity
    ├── EXPOSE     → document port
    ├── VOLUME     → mount point
    ├── HEALTHCHECK→ health
    ├── LABEL      → metadata
    │
    └── ENTRYPOINT + CMD
              ↓
         runtime command
              ↓
           Container
```
