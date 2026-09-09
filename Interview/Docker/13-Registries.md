# Docker Interview Preparation — Topic 13: Docker Configuration & Secrets

> **Interview focus:** Understand how Docker containers receive configuration, how environment variables and files are supplied, why secrets must be handled differently, and how configuration should be separated from the image.

---

## Q296. How do you configure a Docker container?

### Short Interview Answer

Docker configuration can be supplied through several mechanisms, including:

- Environment variables
- Command-line arguments
- Mounted configuration files
- Volumes/bind mounts
- Docker Compose configuration
- Secrets mechanisms
- Application-specific configuration systems

A key production principle is:

> **Build the image once and inject environment-specific configuration at runtime.**

### Example

Build:

```bash
docker build -t myapp:1.0 .
```

Run with environment-specific configuration:

```bash
docker run \
  -e APP_ENV=production \
  -e API_URL=https://api.example.com \
  myapp:1.0
```

The same image can then be used in another environment:

```text
same image
   │
   ├── development config
   ├── staging config
   └── production config
```

### Why it matters

You avoid rebuilding an image simply because the deployment environment changed.

### Interview Point

> Application code belongs in the image; environment-specific configuration should normally be supplied at runtime.

---

## Q297. What is the difference between build-time and runtime configuration?

### Short Interview Answer

**Build-time configuration** affects image creation, while **runtime configuration** affects how a container behaves after it starts.

### Build-time

Dockerfile:

```dockerfile
ARG APP_VERSION=1.0
RUN echo "Building $APP_VERSION"
```

Build:

```bash
docker build --build-arg APP_VERSION=2.0 -t myapp:2.0 .
```

### Runtime

```bash
docker run \
  -e APP_ENV=production \
  myapp:2.0
```

Conceptually:

```text
Build time
    ↓
Dockerfile + ARG
    ↓
Image

Runtime
    ↓
Image + ENV / config / secrets
    ↓
Container
```

### Important distinction

`ARG` is primarily a build-time variable.

`ENV` can provide environment values that are available in the image/runtime.

Runtime `-e` can override or add environment variables when the container starts.

### Interview Point

> Build-time inputs help construct the artifact; runtime inputs configure the running application.

---

# Environment Variables

## Q298. What is Docker `ENV`?

### Short Interview Answer

`ENV` sets environment variables in the image so that they are available to subsequent Dockerfile instructions and, by default, to containers created from that image.

### Dockerfile

```dockerfile
FROM alpine

ENV APP_ENV=production
ENV APP_PORT=8080

CMD ["sh"]
```

Run:

```bash
docker run --rm myapp
```

Inside:

```bash
echo "$APP_ENV"
echo "$APP_PORT"
```

Output:

```text
production
8080
```

### Runtime override

```bash
docker run \
  -e APP_ENV=staging \
  myapp
```

Now:

```text
APP_ENV=staging
```

### Important caveat

Do not use `ENV` for secrets such as:

```text
PASSWORD
API_TOKEN
PRIVATE_KEY
```

because image metadata, configuration, process environments, logs, debugging tools, or orchestration interfaces may expose them.

### Interview Point

> `ENV` is configuration, not a secure secret-storage mechanism.

---

## Q299. What is `docker run -e`?

### Short Interview Answer

`docker run -e` or `--env` supplies environment variables to a container at runtime.

### Example

```bash
docker run \
  -e APP_ENV=production \
  -e LOG_LEVEL=info \
  myapp
```

Multiple variables:

```bash
docker run \
  -e APP_ENV=production \
  -e LOG_LEVEL=info \
  -e API_URL=https://api.example.com \
  myapp
```

### Using a host variable

```bash
export API_URL=https://api.example.com

docker run \
  -e API_URL \
  myapp
```

The container receives the value from the host environment.

### Interview Point

> `-e` is a runtime configuration mechanism; it does not modify the image.

---

## Q300. What is the difference between `ENV`, `ARG`, and `docker run -e`?

### Short Interview Answer

| Mechanism | When | Main purpose |
|---|---|---|
| `ARG` | Build time | Build configuration |
| `ENV` | Build + runtime default | Environment configuration |
| `docker run -e` | Runtime | Container-specific configuration |

### Example

Dockerfile:

```dockerfile
ARG BUILD_VERSION
ENV APP_ENV=production
```

Build:

```bash
docker build \
  --build-arg BUILD_VERSION=1.5 \
  -t myapp:1.5 .
```

Run:

```bash
docker run \
  -e APP_ENV=staging \
  myapp:1.5
```

Conceptually:

```text
ARG
 ↓
image build

ENV
 ↓
image/runtime default

-e
 ↓
specific container runtime
```

### Interview Trap

Do not say:

> "`ARG` and `ENV` are both runtime environment variables."

They have different scopes and purposes.

### Interview Point

> `ARG` is build-time; `ENV` establishes environment defaults; `-e` supplies runtime values.

---

## Q301. Can `docker run -e` override an `ENV` value?

### Short Interview Answer

Yes. A runtime environment variable can override the image's default `ENV` value.

Dockerfile:

```dockerfile
FROM alpine
ENV APP_ENV=production
CMD ["sh", "-c", "echo $APP_ENV"]
```

Run normally:

```bash
docker run --rm myapp
```

Output:

```text
production
```

Override:

```bash
docker run --rm \
  -e APP_ENV=staging \
  myapp
```

Output:

```text
staging
```

### Interview Point

> Image-level defaults can be overridden by runtime configuration.

---

## Q302. Should application configuration be hard-coded in a Docker image?

### Short Interview Answer

Environment-specific configuration generally should not be hard-coded into the image. The image should contain the application artifact, while deployment-specific configuration should normally be injected at runtime.

### Bad pattern

```dockerfile
ENV DATABASE_HOST=prod-db.example.com
ENV API_URL=https://prod.example.com
```

This makes the image tightly coupled to production.

### Better

```dockerfile
ENV APP_ENV=production
```

Then:

```bash
docker run \
  -e DATABASE_HOST=db.example.com \
  -e API_URL=https://api.example.com \
  myapp:1.0
```

### Benefits

The same image can be promoted:

```text
Build once
   ↓
Development
   ↓
Staging
   ↓
Production
```

### Interview Point

> Separate immutable application artifacts from mutable deployment configuration.

---

# Configuration Files

## Q303. How can you provide a configuration file to a container?

### Short Interview Answer

You can provide configuration files through:

- Bind mounts
- Volumes
- Image `COPY`
- Orchestration configuration mechanisms
- Secret/config management systems

For environment-specific configuration, a mount is often preferable to rebuilding the image.

### Bind mount example

Host:

```text
/opt/myapp/config/application.yaml
```

Container:

```text
/etc/myapp/application.yaml
```

Run:

```bash
docker run \
  --mount type=bind,source=/opt/myapp/config/application.yaml,target=/etc/myapp/application.yaml,readonly \
  myapp
```

The application reads:

```text
/etc/myapp/application.yaml
```

### Why read-only?

If the container only needs to read configuration:

```text
readonly
```

reduces accidental modification.

### Interview Point

> Configuration files can be mounted independently of the application image.

---

## Q304. When should configuration be copied into the image?

### Short Interview Answer

Copy configuration into the image when it is genuinely part of the immutable application artifact and is not environment-specific or sensitive.

### Example

A static NGINX configuration that is identical for every deployment can reasonably be included:

```dockerfile
FROM nginx:alpine

COPY nginx.conf /etc/nginx/nginx.conf
COPY dist/ /usr/share/nginx/html/
```

But an environment-specific endpoint such as:

```text
production database URL
```

usually should not be baked into the image.

### Interview Point

> The question is not "file or environment variable?" but "is this configuration part of the immutable artifact or deployment environment?"

---

# Secrets

## Q305. What is a Docker secret?

### Short Interview Answer

A secret is sensitive configuration such as a password, API token, certificate key, or credential that should be protected from ordinary configuration exposure.

Examples:

```text
Database password
API token
TLS private key
Cloud credential
SSH private key
```

Secrets should not normally be baked into images or committed to source control.

### Core principle

```text
Image
  └── application code

Runtime secret
  └── injected separately
```

### Interview Point

> Secrets are sensitive runtime data and should have a stricter handling mechanism than ordinary configuration.

---

## Q306. Why should secrets not be stored in a Dockerfile?

### Short Interview Answer

Because Dockerfile instructions can become part of the image build history or image metadata, and the resulting secret can remain recoverable even if the final filesystem no longer contains the obvious file.

### Bad example

```dockerfile
ARG DB_PASSWORD
RUN echo "$DB_PASSWORD" > /tmp/password
```

Even if `/tmp/password` is later deleted, the secret may have been exposed through build metadata or intermediate layers depending on how it was used.

Another bad example:

```dockerfile
ENV DB_PASSWORD=SuperSecret
```

This places the secret into image configuration.

### Correct principle

Use a dedicated secret mechanism.

For BuildKit builds, use secret mounts:

```bash
docker build \
  --secret id=mysecret,src=secret.txt \
  -t myapp:1.0 .
```

Dockerfile:

```dockerfile
RUN --mount=type=secret,id=mysecret \
    cat /run/secrets/mysecret
```

The secret is mounted for that build step rather than intentionally becoming part of the resulting image filesystem.

### Interview Point

> Never assume deleting a secret later removes it from the image's build history.

---

## Q307. What is the difference between Docker secrets and environment variables?

### Short Interview Answer

Environment variables are convenient configuration inputs, but secrets require stronger handling because environment variables can be exposed through process inspection, debugging interfaces, logs, orchestration metadata, or accidental application output.

### Environment variable

```bash
docker run \
  -e DB_PASSWORD=secret \
  myapp
```

### Secret

A secrets mechanism provides controlled delivery of sensitive material without treating it as ordinary configuration.

### Important nuance

Do not say:

> "Environment variables are always insecure."

They are widely used and can be acceptable depending on the threat model and platform.

The stronger interview answer is:

> "Secrets should use a purpose-built secret-management mechanism when the platform provides one, rather than treating sensitive values as ordinary environment configuration."

### Interview Point

> Configuration convenience and secret protection are different concerns.

---

## Q308. How do Docker BuildKit secret mounts work?

### Short Interview Answer

BuildKit can temporarily mount a secret into a build step without intentionally storing the secret in the resulting image layer.

### Build command

```bash
docker build \
  --secret id=npmrc,src=$HOME/.npmrc \
  -t myapp:1.0 .
```

Dockerfile:

```dockerfile
# syntax=docker/dockerfile:1

RUN --mount=type=secret,id=npmrc \
    cp /run/secrets/npmrc /tmp/npmrc && \
    npm install
```

A better real-world approach is to configure the package manager to consume the mounted secret directly rather than copying it unnecessarily.

### Conceptual flow

```text
Secret on build host
        │
        ▼
BuildKit secret mount
        │
        ▼
specific RUN step
        │
        └── secret available temporarily

Final image
        │
        └── secret not intentionally stored
```

### Important distinction

This is **build-time secret handling**.

It is not the same as supplying a secret to a running container.

### Interview Point

> BuildKit secret mounts solve secret exposure during image builds; runtime secret management is a separate concern.

---

# Runtime Secrets

## Q309. How should runtime secrets be supplied to containers?

### Short Interview Answer

Use a secret-management mechanism appropriate to the deployment platform rather than baking secrets into the image.

Depending on the environment, this can include:

- Docker/Compose secrets
- Kubernetes Secrets
- Cloud secret managers
- External secret stores
- Mounted secret files
- Carefully controlled runtime environment variables where appropriate

### Example concept

```text
Secret Manager
      │
      ▼
Deployment platform
      │
      ▼
Container
      │
      └── application
```

### Important distinction

A Docker image should remain reusable:

```text
myapp:1.0
```

while the secret changes independently:

```text
dev secret
staging secret
production secret
```

### Interview Point

> Secret lifecycle should be independent from image lifecycle.

---

## Q310. What is Docker Compose `secrets`?

### Short Interview Answer

Docker Compose can define secrets separately from ordinary environment configuration and make them available to services, commonly as files under `/run/secrets/<name>`.

Example:

```yaml
services:
  app:
    image: myapp:1.0
    secrets:
      - db_password

secrets:
  db_password:
    file: ./db_password.txt
```

Inside the container, the secret can be available at:

```text
/run/secrets/db_password
```

### Important distinction

This is different from:

```yaml
environment:
  DB_PASSWORD: ...
```

The latter treats the value as ordinary environment configuration.

### Interview Point

> Compose secrets provide a dedicated configuration path for sensitive values.

---

# `.env` and Environment Files

## Q311. What is a Docker environment file?

### Short Interview Answer

An environment file contains key-value pairs that can be supplied as container environment variables.

Example:

```text
APP_ENV=development
LOG_LEVEL=debug
API_URL=https://api.example.com
```

Run:

```bash
docker run \
  --env-file .env \
  myapp
```

### Why useful?

Instead of writing:

```bash
docker run \
  -e APP_ENV=development \
  -e LOG_LEVEL=debug \
  -e API_URL=https://api.example.com \
  myapp
```

you can keep the variables in a file.

### Security warning

Do not assume `.env` means "secret."

If it contains credentials:

```text
DB_PASSWORD=secret
```

it is still sensitive data and should be protected appropriately.

### Interview Point

> An env file is a configuration convenience, not automatically a secure secret store.

---

## Q312. What is the difference between `.env` and Docker secrets?

### Short Interview Answer

`.env` is commonly used to supply ordinary environment configuration, while secrets are designed for sensitive values and should have more controlled handling.

### `.env`

```text
APP_ENV=production
LOG_LEVEL=info
```

Usage:

```bash
docker run --env-file .env myapp
```

### Secret

Conceptually:

```text
/run/secrets/db_password
```

The application reads the secret as sensitive data.

### Interview Trap

Do not say:

> "Everything in `.env` is secret."

An `.env` file is just a configuration file.

### Interview Point

> Environment configuration and secret management should not be treated as identical problems.

---

# Configuration Precedence

## Q313. What happens when the same environment variable is defined at multiple levels?

### Short Interview Answer

The effective value depends on the configuration source and how the container is started. Runtime configuration can override image defaults.

For a simple example:

Dockerfile:

```dockerfile
ENV APP_ENV=production
```

Runtime:

```bash
docker run -e APP_ENV=staging myapp
```

Effective value:

```text
APP_ENV=staging
```

### Practical troubleshooting

Inspect the actual container:

```bash
docker inspect app
```

and:

```bash
docker exec app env
```

The second command shows what the running process environment contains from inside the container.

### Interview Point

> When configuration conflicts, inspect the effective runtime configuration instead of reasoning only from the Dockerfile.

---

# Immutable Images and Twelve-Factor Principles

## Q314. What does "build once, configure at runtime" mean?

### Short Interview Answer

It means creating one immutable application image and supplying environment-specific settings when the container is deployed rather than rebuilding the image for every environment.

### Bad pipeline

```text
Build Dev Image
      ↓
Build Staging Image
      ↓
Build Production Image
```

Each image may contain different configuration.

### Better pipeline

```text
Source
  ↓
Build
  ↓
myapp:1.0
  │
  ├── Dev config
  ├── Staging config
  └── Production config
```

### Benefits

- Consistent artifact promotion.
- Reduced configuration drift.
- Easier rollback.
- Better reproducibility.
- Cleaner CI/CD pipeline.

### Interview Point

> Promote the same artifact through environments; change deployment configuration, not application binaries.

---

## Q315. What is the Twelve-Factor principle for configuration?

### Short Interview Answer

The Twelve-Factor methodology recommends storing configuration that varies between deployments in the environment rather than hard-coding it into application code.

Typical examples:

```text
Database URL
API endpoint
Log level
Feature flags
External service configuration
```

Conceptually:

```text
Application artifact
       +
environment configuration
       ↓
running application
```

### Docker application

```bash
docker run \
  -e DATABASE_URL=... \
  -e LOG_LEVEL=info \
  myapp
```

### Important nuance

Not every configuration item must literally be an environment variable. Mounted configuration files and external configuration systems can also be appropriate.

### Interview Point

> The principle is separation of deploy-specific configuration from the application artifact, not "everything must be an environment variable."

---

# Configuration and Secrets in Production

## Q316. How would you design configuration for a production Docker application?

### Strong Interview Answer

I would separate configuration into three categories:

### 1. Immutable application content

Stored in the image:

```text
Application binary
Libraries
Static assets
Required non-environment-specific defaults
```

### 2. Non-sensitive runtime configuration

Supplied through:

```text
Environment variables
Config files
Compose configuration
Deployment configuration
```

Examples:

```text
LOG_LEVEL
API_URL
APP_ENV
FEATURE_FLAG
```

### 3. Sensitive runtime data

Supplied through:

```text
Secret manager
Secrets mechanism
Controlled secret files
```

Examples:

```text
DB_PASSWORD
API_TOKEN
TLS_PRIVATE_KEY
```

### Architecture

```text
                ┌───────────────┐
                │ Docker Image  │
                │ app + runtime │
                └───────┬───────┘
                        │
             ┌──────────┴──────────┐
             │                     │
       Config values          Secrets
             │                     │
             ▼                     ▼
       runtime config       secret mechanism
             │                     │
             └──────────┬──────────┘
                        ▼
                    Container
```

### Interview Point

> Keep application artifact, ordinary configuration, and sensitive secrets as separate lifecycle concerns.

---

## Q317. What are common mistakes with Docker configuration?

### Common mistakes

#### 1. Hard-coding production endpoints

```dockerfile
ENV API_URL=https://prod.example.com
```

#### 2. Putting passwords in Dockerfiles

```dockerfile
ENV DB_PASSWORD=secret
```

#### 3. Using `ARG` as a secret store

```dockerfile
ARG TOKEN
RUN some-command --token "$TOKEN"
```

#### 4. Committing `.env` files containing credentials

```text
.env
DB_PASSWORD=real-password
```

#### 5. Rebuilding the image for every environment

```text
dev image
staging image
prod image
```

#### 6. Passing secrets into shell commands unnecessarily

```bash
docker run -e PASSWORD=secret ...
```

can expose sensitive information depending on how the command is handled and inspected.

#### 7. Giving containers writable access to configuration unnecessarily

Prefer read-only mounts when appropriate.

### Interview Point

> Good configuration design minimizes coupling, exposure, and environment-specific image builds.

---

# Troubleshooting

## Q318. A container is using the wrong configuration. How would you troubleshoot it?

### Strong Interview Answer

I would first identify the configuration source and then inspect the effective configuration inside the running container.

### Step 1 — Inspect environment

```bash
docker exec app env
```

### Step 2 — Inspect container metadata

```bash
docker inspect app
```

Look for:

```text
Config.Env
Mounts
Cmd
Entrypoint
```

### Step 3 — Check mounted configuration files

```bash
docker exec app ls -la /etc/myapp
docker exec app cat /etc/myapp/application.yaml
```

### Step 4 — Check runtime command

```bash
docker inspect app \
  --format '{{json .Config.Cmd}}'
```

and:

```bash
docker inspect app \
  --format '{{json .Config.Entrypoint}}'
```

### Step 5 — Check Compose/deployment configuration

Verify:

```text
environment:
env_file:
secrets:
configs:
volumes:
command:
entrypoint:
```

### Step 6 — Check application logs

```bash
docker logs app
```

### Troubleshooting model

```text
Image defaults
      ↓
container runtime configuration
      ↓
mounted config
      ↓
secret/config injection
      ↓
application startup arguments
      ↓
effective application configuration
```

### Interview Point

> Troubleshoot the effective configuration, not just the Dockerfile.

---

## Q319. A secret was accidentally committed to a Dockerfile. What would you do?

### Strong Interview Answer

I would treat the secret as compromised rather than simply deleting the Dockerfile line.

### Immediate actions

1. **Rotate/revoke the exposed secret.**
2. Remove it from the current source.
3. Determine whether it entered image layers, build logs, cache, registry artifacts, or CI logs.
4. Rebuild the image without the secret.
5. Replace affected images/artifacts as necessary.
6. Check source-control history and access logs.
7. Move future secret handling to a proper secret mechanism.

### Important principle

```text
Delete secret
      ≠
secret is no longer compromised
```

If a credential was exposed, assume it may have been copied.

### Interview Point

> Secret remediation starts with rotation/revocation, not merely deleting the visible line.

---

## Q320. How would you prevent secrets from entering Docker image layers during builds?

### Strong Interview Answer

Use BuildKit secret mounts instead of `ARG`, `ENV`, or copying secret files into the build context.

Example:

```bash
docker build \
  --secret id=aws,src=$HOME/.aws/credentials \
  -t myapp:1.0 .
```

Dockerfile:

```dockerfile
RUN --mount=type=secret,id=aws \
    some-build-command
```

Also:

- Keep secrets outside the build context.
- Do not commit them to source control.
- Avoid printing them in build logs.
- Review CI/CD logs and cache handling.
- Use appropriate external secret managers where possible.

### Interview Point

> Secure build handling means preventing the secret from becoming part of the image, build output, or source context.

---

# Scenario Questions

## Q321. Why is this Dockerfile insecure?

```dockerfile
FROM ubuntu:24.04

ARG DB_PASSWORD
ENV API_TOKEN=secret-token

COPY . /app

RUN echo "$DB_PASSWORD" > /app/password.txt

CMD ["./app"]
```

### Strong Interview Answer

There are several issues:

### Problem 1 — `ARG DB_PASSWORD`

Build arguments are not a secure secret store.

### Problem 2 — `ENV API_TOKEN`

The token becomes part of the image configuration.

### Problem 3 — Secret copied into filesystem

```dockerfile
RUN echo "$DB_PASSWORD" > /app/password.txt
```

This risks placing the secret into an image layer.

### Problem 4 — `COPY .`

Without an appropriate `.dockerignore`, sensitive files such as:

```text
.env
.git
credentials
private keys
```

may enter the build context.

### Better approach

Use:

```text
.dockerignore
BuildKit secrets
Runtime secret management
```

and keep credentials outside the image.

### Interview Point

> A secret is not safe merely because the final Dockerfile line does not visibly contain the literal password.

---

## Q322. Why is this Dockerfile problematic?

```dockerfile
FROM node:24

COPY . /app
WORKDIR /app

RUN npm install

ENV API_URL=https://production.example.com

CMD ["npm", "start"]
```

### Problems

#### 1. Environment-specific configuration is baked into the image

```dockerfile
ENV API_URL=https://production.example.com
```

This couples the image to production.

#### 2. `COPY .` may send unnecessary or sensitive files

Use:

```text
.dockerignore
```

#### 3. Dependency installation can be improved

For a Node application, copy dependency manifests before application source when appropriate:

```dockerfile
COPY package*.json ./
RUN npm ci
COPY . .
```

This can improve build-cache reuse.

### Better configuration model

```bash
docker run \
  -e API_URL=https://api.example.com \
  myapp:1.0
```

### Interview Point

> Configuration design and Docker build-cache design often intersect.

---

## Q323. An application requires a database password. Would you put it in `ENV`, `ARG`, `.env`, or a secret?

### Strong Interview Answer

For a real production credential, I would prefer a **dedicated secret-management mechanism**.

I would not put the password into:

```text
Dockerfile ENV
```

or:

```text
Docker build ARG
```

because these are not appropriate secret-storage mechanisms.

An `.env` file can technically provide the value, but if it contains a real production secret, it must be protected like any other sensitive credential and is generally less desirable than a purpose-built secret mechanism.

### Interview-quality answer

> "For local development I may use an ignored `.env` file, but for production I would use the platform's secret-management facility and inject the credential at runtime."

### Interview Point

> The correct answer depends on environment and threat model, but production credentials deserve dedicated secret handling.

---

## Q324. Can you change `ENV` in a running container permanently?

### Short Interview Answer

You can change environment variables for a process when launching a new container, but changing the environment of an already-running container process does not modify the image's `ENV` definition.

If configuration must change, normally recreate the container with the new runtime configuration.

### Example

Original:

```bash
docker run -d \
  --name app \
  -e LOG_LEVEL=info \
  myapp
```

Reconfigure:

```bash
docker rm -f app

docker run -d \
  --name app \
  -e LOG_LEVEL=debug \
  myapp
```

### Better production model

Use an orchestrator/deployment system to manage configuration and perform controlled replacement/rolling updates.

### Interview Point

> Containers are intended to be replaceable; configuration changes normally result in a new container instance.

---

# Quick Revision

| Topic | Key Point |
|---|---|
| Container configuration | Supplied through runtime inputs, files, secrets, etc. |
| `ARG` | Build-time variable |
| `ENV` | Image/runtime environment default |
| `docker run -e` | Runtime environment variable |
| Runtime override | `-e` can override image `ENV` |
| Config file | Can be copied or mounted |
| Bind-mounted config | Useful for environment-specific files |
| Read-only config | Prevents unnecessary modification |
| Secret | Sensitive runtime/build data |
| Dockerfile secret | Unsafe |
| `ARG` secret | Not a secure secret mechanism |
| `ENV` secret | Not a secure secret mechanism |
| BuildKit secret | Temporary build-time secret mount |
| Runtime secret | Inject separately from image |
| Compose secret | Dedicated secret configuration |
| `.env` | Environment configuration convenience |
| `.env` with password | Still sensitive |
| Build once | Same artifact promoted across environments |
| Twelve-Factor config | Deployment-specific config separated from code |
| Troubleshooting | Inspect effective runtime configuration |
| Secret leak | Rotate/revoke first |

---

# High-Value Interview Traps

### Trap 1 — "`ARG` is safe for passwords"

**Incorrect.**

Build arguments are not a secret-management mechanism.

---

### Trap 2 — "`ENV` is safe because users cannot see it"

**Incorrect.**

Image configuration and runtime environment can be inspected.

---

### Trap 3 — "Deleting the secret file in a later Dockerfile layer makes it secure"

**Incorrect.**

The secret may remain in an earlier image layer or build metadata.

---

### Trap 4 — "`.env` means secrets"

**Incorrect.**

`.env` is a configuration file convention. If it contains credentials, those credentials are still sensitive.

---

### Trap 5 — "Everything should be an environment variable"

Not necessarily.

Configuration files and external configuration systems can be better for structured or large configuration.

---

### Trap 6 — "Build-time and runtime secrets are the same"

**Incorrect.**

BuildKit secret mounts address secrets needed while building an image. Runtime secret mechanisms address secrets needed by the running application.

---

### Trap 7 — "Changing the Dockerfile is enough to change a running container"

**Incorrect.**

You normally build a new image and recreate the container.

---

### Trap 8 — "Same image means same configuration"

Not necessarily.

The same image can behave differently because runtime configuration can differ:

```text
same image
  +
different environment
  =
different behavior
```

---

# High-Value Interview Follow-Up Questions

1. Why should secrets not be stored in Docker image layers?
2. What is the difference between `ARG` and `ENV`?
3. Can `docker run -e` override an image `ENV`?
4. How do you inject configuration files into containers?
5. Why use read-only configuration mounts?
6. What is a BuildKit secret mount?
7. How is a build secret different from a runtime secret?
8. Is an `.env` file secure?
9. What is the Twelve-Factor configuration principle?
10. How would you handle different configuration for dev, staging, and production?
11. How would you rotate a leaked Docker secret?
12. How would you find the effective environment of a running container?
13. Where can Docker configuration be inspected with `docker inspect`?
14. Why is "build once, configure at runtime" useful in CI/CD?
15. What happens if the image contains a default `ENV` and the container supplies `-e`?
16. When would you use a configuration file instead of environment variables?
17. Why should `.dockerignore` be used when secrets exist in a repository?
18. How can a secret accidentally enter the Docker build context?
19. What is the difference between Compose `environment`, `env_file`, and `secrets`?
20. How would you design configuration for a production microservice?

---

# Final Interview Answer

If asked **"How do you handle configuration and secrets in Docker?"**, a strong answer is:

> "I separate the application artifact from environment-specific configuration and secrets. The Docker image should contain the application and its runtime dependencies, while non-sensitive deployment configuration can be supplied through environment variables or mounted configuration files. I avoid baking environment-specific values into the image because I want to build the image once and promote the same artifact across environments. For secrets such as database passwords, API tokens, and private keys, I don't use Dockerfile `ARG` or `ENV` as a secret store. During builds, I can use BuildKit secret mounts, and at runtime I use the secret-management mechanism provided by the deployment platform. If a secret is accidentally committed or included in an image, I treat it as compromised and rotate it rather than simply deleting the visible value."

---

# One-Line Memory Map

```text
ARG      = build-time input
ENV      = image/runtime default
-e       = runtime override
Config   = runtime behavior
Secret   = sensitive data → dedicated secret mechanism
BuildKit = secure build-time secret mount
Image    = immutable application artifact
```
