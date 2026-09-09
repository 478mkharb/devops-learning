# Docker Interview Preparation — Topic 08: Multistage Builds

> **Purpose:** Understand Docker multistage builds, why they are used, how stages work, how artifacts move between stages, and how to design production images using separate build and runtime environments.
>
> **Scope:** This topic focuses specifically on multistage Dockerfiles. General Docker build/cache topics are covered in Topic 07; detailed security mechanisms are covered separately.

---

## 156. What is a multistage Docker build?

### Short Interview Answer

A multistage Docker build uses multiple `FROM` instructions in one Dockerfile to create separate build stages. Artifacts from an earlier stage can then be copied into a later runtime stage.

### Example

```dockerfile
FROM golang:1.25 AS builder

WORKDIR /src

COPY . .
RUN go build -o myapp .

FROM debian:13-slim

WORKDIR /app

COPY --from=builder /src/myapp .

CMD ["./myapp"]
```

Conceptually:

```text
             Dockerfile
                 │
        ┌────────┴────────┐
        ↓                 ↓
   Build Stage        Runtime Stage
   golang image       slim image
        │                 │
        │ build           │
        └──── binary ────→│
                          ↓
                    Final Image
```

### Why It Matters

The compiler, source code, package managers, and other build tools do not need to be present in the final runtime image.

### Interview Point

**Build with a large toolchain; run with only what the application needs.**

---

## 157. Why are multistage builds used?

### Short Interview Answer

They separate build-time dependencies from runtime dependencies, allowing the final image to be smaller, cleaner, and easier to operate.

### Example

A Go application may need:

```text
Go compiler
Go modules
source code
build tools
```

during the build.

But production may need only:

```text
compiled binary
runtime libraries/certificates
```

A multistage build keeps those concerns separate.

### Benefits

- smaller final image
- fewer unnecessary runtime tools
- reduced dependency footprint
- clearer build/runtime separation
- easier production image design

### Common Interview Trap

Do not say:

> "Multistage builds make the application itself smaller."

They primarily reduce the **container image contents** by excluding build-time material from the final stage.

### Interview Point

**Multistage builds optimize the final artifact, not the application binary automatically.**

---

## 158. Can a Dockerfile have multiple `FROM` instructions?

### Short Interview Answer

Yes. Each `FROM` starts a new build stage.

### Example

```dockerfile
FROM node:22 AS frontend-builder

FROM golang:1.25 AS backend-builder

FROM nginx:alpine AS runtime
```

This creates three stages:

```text
frontend-builder
backend-builder
runtime
```

### Important Point

A stage can be named:

```dockerfile
FROM golang:1.25 AS builder
```

The name can then be referenced:

```dockerfile
COPY --from=builder /src/app /app
```

### Common Interview Trap

Multiple `FROM` instructions do not mean that all previous stages automatically become part of the final image.

Only content explicitly brought into the final stage is included.

### Interview Point

**Each `FROM` starts a new stage; stages are independent unless artifacts are explicitly shared.**

---

## 159. What is a build stage?

### Short Interview Answer

A build stage is the portion of a multistage Dockerfile beginning with a `FROM` instruction and ending before the next `FROM`.

### Example

```dockerfile
FROM golang:1.25 AS builder

WORKDIR /src
COPY . .
RUN go build -o app .
```

This is one stage.

Then:

```dockerfile
FROM debian:13-slim
```

starts another stage.

### Stage Model

```text
Stage 1
FROM ...
...
RUN ...

Stage 2
FROM ...
...
COPY --from=...
...
```

### Why It Matters

Each stage can have:

- its own base image
- its own filesystem state
- its own instructions
- its own dependencies

### Interview Point

**A stage is an independent image-building environment within the Dockerfile.**

---

## 160. What does `AS builder` mean?

### Short Interview Answer

`AS builder` assigns a name to a build stage so later instructions can refer to that stage.

### Example

```dockerfile
FROM golang:1.25 AS builder
```

Later:

```dockerfile
COPY --from=builder /src/app /app
```

### Without a Name

You can also reference a stage by numeric index:

```dockerfile
FROM golang:1.25
...
FROM debian:13-slim
COPY --from=0 /src/app /app
```

But named stages are easier to understand and maintain.

### Common Interview Trap

`builder` is not a special reserved keyword.

You could write:

```dockerfile
FROM golang:1.25 AS compile-stage
```

### Interview Point

**`AS name` gives a human-readable identifier to a build stage.**

---

## 161. What does `COPY --from` do?

### Short Interview Answer

`COPY --from` copies files from another build stage, image, or supported build source into the current stage.

### Example

```dockerfile
FROM golang:1.25 AS builder

WORKDIR /src
COPY . .
RUN go build -o app .

FROM debian:13-slim

WORKDIR /app
COPY --from=builder /src/app .

CMD ["./app"]
```

The final stage receives:

```text
/src/app
```

from the `builder` stage.

### Important Point

The entire builder filesystem is **not** automatically included in the final image.

Only what you copy is brought across.

### Common Interview Trap

> "`COPY --from` copies the container."

No.

It copies files/directories from the specified stage or image filesystem.

### Interview Point

**`COPY --from` is the bridge that selectively transfers build artifacts between stages.**

---

## 162. Can you copy files from an external image using `COPY --from`?

### Short Interview Answer

Yes. A `COPY --from` source can refer to an external image, not only a stage in the current Dockerfile.

### Example

Conceptually:

```dockerfile
FROM nginx:alpine

COPY --from=someimage:1.0 /app/static /usr/share/nginx/html
```

Docker can obtain the referenced image if it is not already available locally.

### Why It Can Be Useful

It allows an image to provide artifacts without requiring that image to be the current build stage.

### Common Interview Trap

`--from` does not mean only:

> "from an earlier `FROM`."

It can also refer to an image or other supported build source.

### Interview Point

**`COPY --from` can selectively import files from another stage or image.**

---

## 163. Does every stage become part of the final image?

### Short Interview Answer

No. Only the final stage's filesystem and explicitly copied artifacts become part of the final image.

### Example

```dockerfile
FROM ubuntu:24.04 AS builder

RUN apt-get update && apt-get install -y gcc
COPY source.c .
RUN gcc source.c -o app

FROM ubuntu:24.04

COPY --from=builder /app /app
```

The final image does not automatically contain:

```text
gcc
source.c
build tools
builder filesystem
```

unless they are copied into the final stage.

### Conceptual Model

```text
Builder stage
 ├── compiler
 ├── source
 ├── dependencies
 └── binary
          │
          │ COPY --from
          ↓
Runtime stage
 └── binary
```

### Interview Point

**Intermediate stages are build resources; they are not automatically runtime contents.**

---

## 164. What is the difference between a builder stage and a runtime stage?

### Short Interview Answer

The builder stage contains tools and dependencies needed to create the application artifact. The runtime stage contains only what is needed to execute that artifact.

### Builder

```text
compiler
source code
development dependencies
test/build tools
```

### Runtime

```text
application
runtime dependencies
certificates/configuration
startup command
```

### Example

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

### Why It Matters

The runtime image does not need:

```text
node
npm
source code
development dependencies
```

if the output is static frontend content.

### Interview Point

**Builder = compile/package; runtime = execute/serve.**

---

## 165. How do multistage builds reduce image size?

### Short Interview Answer

They reduce image size by allowing the final stage to start from a runtime-oriented base image and copy only the required application artifacts.

### Example

Without multistage:

```dockerfile
FROM golang:1.25

COPY . .
RUN go build -o app .

CMD ["./app"]
```

The resulting image can contain the Go toolchain and build-related contents.

With multistage:

```dockerfile
FROM golang:1.25 AS builder

COPY . .
RUN go build -o app .

FROM debian:13-slim

COPY --from=builder /app .
CMD ["./app"]
```

### Important Point

The reduction comes from **what the final stage contains**, not from magically compressing the builder stage.

### Common Interview Trap

Do not say:

> "Docker deletes the builder's files from the image."

The better explanation is that the final image is built from the final stage and only selected artifacts are copied into it.

### Interview Point

**Choose a clean runtime base and copy only runtime artifacts.**

---

## 166. Can multistage builds improve security?

### Short Interview Answer

Yes, they can reduce the runtime image's software footprint by excluding compilers, package managers, source code, and other build-only tools. This can reduce the potential attack surface.

### Example

Builder:

```text
gcc
git
curl
package manager
source code
```

Runtime:

```text
application
required libraries
CA certificates
```

### Important Caveat

Multistage builds are **not a complete security solution**.

You still need to consider:

- base image vulnerabilities
- user privileges
- Linux capabilities
- secrets
- dependency vulnerabilities
- network exposure
- runtime configuration

### Common Interview Trap

> "Multistage automatically makes the image secure."

No.

It can reduce unnecessary runtime contents, but security requires multiple controls.

### Interview Point

**Smaller runtime footprint can improve security posture, but multistage is not a security boundary by itself.**

---

## 167. How do you build a React frontend using multistage builds?

### Short Interview Answer

Use a Node.js stage to install dependencies and build the React application, then copy the generated static files into a lightweight web-server stage such as NGINX.

### Example

```dockerfile
FROM node:22 AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM nginx:alpine

COPY --from=builder /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Flow

```text
React source
    ↓
Node builder
    ↓
npm ci
    ↓
npm run build
    ↓
dist/
    ↓
NGINX runtime image
```

### Why This Is Good

The final image does not need:

```text
Node.js
npm
React source
node_modules used only for building
```

### Common Interview Trap

The output directory depends on the frontend framework/build configuration. Do not blindly assume every React project produces exactly `dist/`.

### Interview Point

**Build frontend assets with Node; serve only the generated static assets in the runtime image.**

---

## 168. How do you build a Java application using multistage builds?

### Short Interview Answer

Use a JDK-based builder stage to compile/package the application, then use an appropriate Java runtime image in the final stage.

### Example

```dockerfile
FROM maven:3.9-eclipse-temurin-21 AS builder

WORKDIR /app

COPY pom.xml .
RUN mvn dependency:go-offline

COPY src ./src
RUN mvn package -DskipTests

FROM eclipse-temurin:21-jre

WORKDIR /app

COPY --from=builder /app/target/app.jar app.jar

USER 10001

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Builder

Contains:

```text
Maven
JDK
source
build dependencies
```

### Runtime

Contains:

```text
Java runtime
application JAR
```

### Important Caveat

The exact runtime image should match the application's requirements. Some applications need a full JDK or additional native libraries.

### Interview Point

**Compile/package with the build toolchain; run with the smallest compatible Java runtime environment.**

---

## 169. How do you build a Go application using multistage builds?

### Short Interview Answer

Compile the Go binary in a Go builder stage and copy the resulting binary into a minimal compatible runtime stage.

### Example

```dockerfile
FROM golang:1.25 AS builder

WORKDIR /src

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o app .

FROM scratch

COPY --from=builder /src/app /app

ENTRYPOINT ["/app"]
```

### Why `scratch`?

`scratch` is an empty base image. It can produce extremely small images when the binary and its runtime requirements are self-contained.

### Important Caveat

`scratch` provides no normal userland utilities, shell, package manager, or certificate store.

An application may therefore require additional files such as CA certificates.

### Example With Certificates

A common approach is to copy required certificate data from the builder or use a suitable runtime base image.

### Common Interview Trap

> "Every Go application should use `scratch`."

No.

`scratch` is appropriate only when the application and its dependencies work correctly in such a minimal environment.

### Interview Point

**Go's static-binary capability can make very small runtime images possible, but `scratch` has operational trade-offs.**

---

## 170. Can you use different base images for different stages?

### Short Interview Answer

Yes. Each stage can use a different base image suited to its purpose.

### Example

```dockerfile
FROM node:22 AS frontend-builder

FROM golang:1.25 AS backend-builder

FROM nginx:alpine AS frontend-runtime
```

Or:

```dockerfile
FROM maven:3.9-eclipse-temurin-21 AS builder

FROM eclipse-temurin:21-jre AS runtime
```

### Why?

Builder images need development tools.

Runtime images should contain only what execution requires.

### Common Interview Trap

All stages do not need to use the same base image.

### Interview Point

**Choose the base image independently for each stage according to that stage's purpose.**

---

## 171. Can one build stage copy artifacts from another build stage?

### Short Interview Answer

Yes. A stage can copy artifacts from an earlier stage using `COPY --from=<stage>`.

### Example

```dockerfile
FROM node:22 AS frontend

WORKDIR /app
COPY . .
RUN npm run build

FROM nginx:alpine

COPY --from=frontend /app/dist /usr/share/nginx/html
```

### Important Point

This creates a controlled artifact flow:

```text
Stage A
  ↓
build artifact
  ↓
COPY --from
  ↓
Stage B
```

### Common Interview Trap

The source stage's entire filesystem is not merged into the destination stage.

### Interview Point

**Stages communicate through explicitly copied artifacts.**

---

## 172. What happens to intermediate stages after the build?

### Short Interview Answer

Intermediate stages are used during the build but are not part of the final image unless their contents are copied into the final stage.

### Example

```dockerfile
FROM golang:1.25 AS builder
...
FROM debian:13-slim
COPY --from=builder /app/app /app/app
```

The final image is based on:

```text
debian:13-slim
```

not:

```text
golang:1.25
```

### Important Operational Point

Intermediate build data may still be represented in build cache depending on the builder/cache configuration. That is different from being part of the final runtime image.

### Common Interview Trap

Do not confuse:

```text
final image contents
```

with:

```text
build cache/storage
```

### Interview Point

**Not in the final image does not necessarily mean "never existed anywhere during the build."**

---

## 173. Can you target a specific stage?

### Short Interview Answer

Yes. Docker can build up to a named stage using a target option.

### Example

Dockerfile:

```dockerfile
FROM node:22 AS builder
...
FROM nginx:alpine AS production
...
```

Build the production target:

```bash
docker build --target production -t frontend:prod .
```

### Why Useful?

It can be useful for:

- debugging
- development images
- testing intermediate stages
- inspecting build output
- CI workflows

### Example Structure

```text
base
 ↓
dependencies
 ↓
builder
 ↓
test
 ↓
production
```

A CI workflow may target:

```text
test
```

while deployment targets:

```text
production
```

### Interview Point

**`--target` lets you stop the build at a selected stage.**

---

## 174. How can multistage builds be used for testing?

### Short Interview Answer

A dedicated test stage can contain test tools and dependencies, while the final production stage excludes them.

### Example

```dockerfile
FROM python:3.12 AS test

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

RUN pytest
```

Then:

```dockerfile
FROM python:3.12-slim AS production

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

USER 10001

CMD ["python", "app.py"]
```

The test stage can be targeted:

```bash
docker build --target test -t myapp:test .
```

### Why It Matters

Testing dependencies do not have to be included in the production image.

### Common Interview Trap

A test stage does not automatically mean tests run in every production build unless the build graph requires that stage.

### Interview Point

**Use dedicated stages for testing without contaminating the runtime image.**

---

## 175. What are common multistage build mistakes?

### Short Interview Answer

Common mistakes include copying unnecessary files into the runtime stage, using an incompatible runtime base, forgetting runtime dependencies, using an unnecessarily complex stage structure, and assuming the smallest base is always appropriate.

### Example Mistake

```dockerfile
FROM node:22 AS builder
...
FROM nginx:alpine

COPY --from=builder /app /usr/share/nginx/html
```

If `/app` contains unnecessary source files or build artifacts, the final image may still contain more than required.

Better:

```dockerfile
COPY --from=builder /app/dist /usr/share/nginx/html
```

### Other Mistakes

#### Mistake 1 — Missing runtime library

A compiled application may require shared libraries not present in the runtime image.

#### Mistake 2 — Missing certificates

Applications making HTTPS requests may need CA certificates.

#### Mistake 3 — Wrong architecture

A binary built for one architecture cannot necessarily run in a runtime image for another.

#### Mistake 4 — Debugging difficulty

An extremely minimal runtime image may not contain a shell or diagnostic tools.

### Interview Point

**The runtime stage must be minimal but complete.**

---

# Quick Revision

| Concept | Key Point |
|---|---|
| Multistage build | Multiple `FROM` stages in one Dockerfile |
| Stage | Section beginning with `FROM` |
| `AS builder` | Names a stage |
| `COPY --from` | Copies artifacts from another stage/image |
| Builder stage | Contains build tools/dependencies |
| Runtime stage | Contains runtime requirements |
| Final image | Based on the final selected stage |
| Intermediate stages | Not automatically included in final image |
| Image size | Reduced by excluding build-time contents |
| Security | Smaller runtime footprint can reduce attack surface |
| Different base images | Allowed and often desirable |
| `--target` | Build up to a selected stage |
| Test stage | Keeps test tooling out of production image |
| `scratch` | Empty base; extremely minimal but has trade-offs |
| Frontend pattern | Node builder → NGINX runtime |
| Java pattern | Maven/JDK builder → Java runtime |
| Go pattern | Go builder → minimal runtime |
| Runtime stage | Must be minimal **and** complete |

---

# High-Value Interview Traps

### Trap 1 — "Every `FROM` becomes part of the final image."

**Wrong.**

Only the final stage and explicitly copied artifacts form the final image.

---

### Trap 2 — "`COPY --from=builder` copies the whole builder."

**Wrong.**

It copies only the specified files/directories.

---

### Trap 3 — "Multistage builds make the binary smaller."

**Not necessarily.**

They primarily make the **image** smaller by excluding build-time contents.

---

### Trap 4 — "Use `scratch` whenever you want the smallest image."

**Wrong.**

`scratch` may be unsuitable if the application requires certificates, shared libraries, shell utilities, or other runtime components.

---

### Trap 5 — "Intermediate stage means temporary container."

**Not exactly.**

A build stage is a logical stage in the image build process. Do not equate it directly with a long-running application container.

---

### Trap 6 — "Multistage automatically provides security."

**Wrong.**

It can reduce runtime footprint, but it does not replace vulnerability management, least privilege, capabilities, secret management, and other controls.

---

### Trap 7 — "The runtime image must use the same base image as the builder."

**Wrong.**

Different stages can use completely different base images.

---

### Trap 8 — "If a file is not in the final image, it never existed during the build."

**Wrong.**

It may have existed in an intermediate stage or build cache.

---

# Interview Follow-Up Questions

After answering multistage builds, an interviewer may ask:

1. How does `COPY --from` work internally?
2. What is the difference between a build stage and an image?
3. Can stages be built independently?
4. What is `--target` used for?
5. Can one stage use another stage as its base?
6. How would you design a multistage Dockerfile for a Spring Boot application?
7. How would you design one for a Go microservice?
8. Why might `scratch` fail for an otherwise working Go binary?
9. How do you debug an application in a minimal runtime image?
10. How do multistage builds interact with Docker build cache?
11. How do you keep development and production dependencies separate?
12. How would you copy only static frontend assets into an NGINX image?

---

# Final Interview Answer

> **"I use multistage Docker builds to separate build-time and runtime concerns. The first stage contains the compiler, package manager, source code, and other tools needed to build the application. A later runtime stage starts from an appropriate production base image and uses `COPY --from` to bring in only the artifacts required to run the application. This keeps compilers, source code, and build dependencies out of the final image, which can reduce image size and runtime attack surface. I also make sure the runtime stage still contains every dependency the application actually needs, because the smallest image is not automatically the correct image."**

---

# One-Line Memory Map

```text
MULTISTAGE BUILD

SOURCE
  ↓
BUILDER STAGE
  ├── compiler
  ├── package manager
  ├── build dependencies
  └── source
          │
          │ COPY --from
          ↓
RUNTIME STAGE
  ├── runtime
  ├── application artifact
  └── required runtime dependencies
          ↓
      FINAL IMAGE
```

## Core Rule

```text
BUILD STAGE = everything needed to BUILD

RUNTIME STAGE = only what is needed to RUN
```
