# Docker Interview Preparation — Topic 18: Docker Compose & Orchestration

> **Interview focus:** Understand Docker Compose as a tool for defining and running **multi-container applications**, while clearly distinguishing it from Docker Swarm and Kubernetes.

---

# 491. What is Docker Compose?

## Short Interview Answer

Docker Compose is a tool for defining and running **multi-container Docker applications** using a YAML configuration file.

A Compose application can define:

- services
- networks
- volumes
- environment variables
- healthchecks
- dependencies
- ports
- resource-related configuration

Typical command:

```bash
docker compose up -d
```

## Example

```yaml
services:
  frontend:
    image: nginx:alpine
    ports:
      - "8080:80"

  backend:
    image: my-api:1.0

  db:
    image: postgres:16
```

The YAML describes the desired application topology.

## Why It Matters

Instead of manually running several commands:

```bash
docker run ...
docker run ...
docker network create ...
docker volume create ...
```

Compose lets you define the application declaratively.

## Common Interview Trap

Docker Compose is **not itself a container runtime**. It uses Docker's container functionality to create and manage application resources.

## Interview Point

> **Compose is primarily a declarative tool for defining and running multi-container applications.**

---

# 492. What problem does Docker Compose solve?

## Short Interview Answer

Compose simplifies the management of applications consisting of multiple related containers.

For example:

```text
Frontend
   ↓
Backend API
   ↓
Database
```

Instead of manually creating each container, network and volume, I define them in one Compose file.

## Example

```yaml
services:
  api:
    image: my-api:1.0

  db:
    image: postgres:16
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

Then:

```bash
docker compose up -d
```

## Interview Point

> **Compose turns a multi-container setup into a repeatable declarative configuration.**

---

# 493. What is a Compose file?

## Short Interview Answer

A Compose file is a YAML file that defines the application's services and their configuration.

Common filename:

```text
compose.yaml
```

Another commonly used filename is:

```text
docker-compose.yml
```

Example:

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
```

## Typical Sections

```yaml
services:
volumes:
networks:
configs:
secrets:
```

Not every application needs every section.

## Interview Point

> **The Compose file is the declarative definition of the multi-container application.**

---

# 494. What is a service in Docker Compose?

## Short Interview Answer

A service represents a logical application component that Compose manages as one part of the application.

Example:

```yaml
services:
  api:
    image: my-api:1.0

  database:
    image: postgres:16
```

Here:

```text
api       → application service
database  → database service
```

A service can result in one or more containers depending on how it is configured and deployed.

## Common Interview Trap

A Compose **service is not exactly the same thing as a container**.

The service is the declarative definition; containers are runtime instances created from that definition.

## Interview Point

> **Service = desired configuration; container = runtime instance.**

---

# 495. What is the difference between a Compose service and a container?

| Service | Container |
|---|---|
| Declarative application component | Runtime object |
| Defined in Compose YAML | Created by Docker |
| Describes image/config/network/etc. | Executes a process |
| Can have scaled instances | Individual runtime instance |
| Represents desired state | Actual running object |

Conceptually:

```text
Compose service
      ↓
Container instance(s)
```

## Interview Point

> **Do not use "service" and "container" interchangeably in Compose discussions.**

---

# 496. How do you start a Compose application?

## Short Interview Answer

Use:

```bash
docker compose up
```

For detached mode:

```bash
docker compose up -d
```

Compose reads the project configuration and creates/starts the required resources.

## Useful Commands

```bash
docker compose ps
docker compose logs
docker compose down
```

## Interview Point

> **`docker compose up -d` is the common command to start a Compose application in the background.**

---

# 497. What does `docker compose up` do?

## Short Interview Answer

It creates and starts the services described by the Compose configuration, creating required networks, volumes and containers as needed.

If an image needs to be built and the service has a `build:` definition, Compose can build it.

Example:

```yaml
services:
  api:
    build: .
```

Then:

```bash
docker compose up
```

can build the image and start the service.

## Important Distinction

`up` is concerned with bringing the Compose application to the desired running state.

## Interview Point

> **`up` is the primary "bring this application stack up" command.**

---

# 498. What does `docker compose down` do?

## Short Interview Answer

It stops and removes the Compose application's containers and networks created for the project.

Typical command:

```bash
docker compose down
```

## Important Volume Behavior

By default, named volumes are not removed.

A more destructive command is:

```bash
docker compose down -v
```

which also removes the Compose-managed named volumes associated with the application.

## Common Interview Trap

Do not casually use:

```bash
docker compose down -v
```

for a database workload.

## Interview Point

> **`down` removes the application resources it manages; `-v` makes the operation more destructive by removing volumes.**

---

# 499. What is the difference between `docker compose stop` and `docker compose down`?

## Short Interview Answer

### `stop`

Stops containers but keeps the containers and other Compose resources.

```bash
docker compose stop
```

### `down`

Stops and removes the Compose containers and networks.

```bash
docker compose down
```

Conceptually:

```text
stop → stop runtime
down → stop + remove application resources
```

## Interview Point

> **Use `stop` when you intend to preserve the container objects; use `down` when you want to tear down the Compose application.**

---

# 500. What is `docker compose start`?

## Short Interview Answer

`docker compose start` starts existing stopped service containers.

It does **not** perform the same create/reconcile behavior as `up`.

```bash
docker compose start
```

Useful distinction:

```text
up    → create/recreate as necessary + start
start → start existing containers
```

## Interview Point

> **`start` is for existing stopped Compose containers; `up` is the normal application bring-up command.**

---

# 501. What is the difference between `docker compose run` and `docker compose up`?

## Short Interview Answer

`docker compose up` starts the application's defined services.

`docker compose run` creates a **one-off container** for a service, commonly for tasks such as migrations or administrative commands.

Example:

```bash
docker compose run --rm api python manage.py migrate
```

This is different from permanently changing the service's normal startup behavior.

## Interview Point

> **Use `up` for the application stack; use `run` for one-off service tasks.**

---

# 502. How does Compose networking work?

## Short Interview Answer

Compose normally creates a project network for the application's services.

Services connected to the same network can communicate using service names.

Example:

```yaml
services:
  api:
    image: my-api

  db:
    image: postgres
```

The API can normally reach the database using:

```text
db
```

rather than a hard-coded container IP.

## Example

```text
api
 ↓
db:5432
```

## Common Interview Trap

Do not configure application-to-application communication using dynamic container IPs.

## Interview Point

> **Use Compose service names for service discovery rather than hard-coded container IP addresses.**

---

# 503. Why can one Compose service reach another by service name?

## Short Interview Answer

Compose attaches services to a common Docker network and provides service-name-based DNS resolution.

For:

```yaml
services:
  api:
    image: my-api

  db:
    image: postgres
```

the API can use:

```text
db
```

as the database hostname.

## Conceptual Flow

```text
api
 ↓
Docker embedded DNS
 ↓
db
 ↓
database container IP
```

The actual IP can change without requiring application configuration changes.

## Interview Point

> **Compose service discovery is name-based, not IP-based.**

---

# 504. Do Compose services need published ports to communicate with each other?

## Short Interview Answer

**No.**

If two services are on the same Docker network, they can communicate using the container's listening port.

Example:

```yaml
services:
  api:
    image: my-api

  db:
    image: postgres:16
```

The API can connect to:

```text
db:5432
```

without:

```yaml
ports:
  - "5432:5432"
```

## Why?

`ports` publishes a port to the host/external network. Internal service-to-service communication uses the Docker network directly.

## Interview Point

> **`ports` is for publishing; service-to-service communication does not require host port publishing.**

---

# 505. What is the difference between `ports` and `expose` in Compose?

## Short Interview Answer

### `ports`

Publishes a container port to the host:

```yaml
ports:
  - "8080:80"
```

### `expose`

Documents/makes the intended internal service port explicit but does not publish it to the host in the same way:

```yaml
expose:
  - "80"
```

For Compose networking, services on the same network can normally communicate without either service publishing its port.

## Interview Point

> **`ports` creates host publishing; `expose` is not a host-port publishing mechanism.**

---

# 506. What is a named volume in Compose?

## Short Interview Answer

A named volume is persistent Docker-managed storage declared by Compose.

Example:

```yaml
services:
  db:
    image: postgres:16
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

The volume exists independently from the container lifecycle.

## Interview Point

> **Compose volumes allow persistent data to survive container replacement.**

---

# 507. How do you persist database data in Compose?

## Short Interview Answer

Use a named volume or an appropriate external persistent-storage mechanism.

Example:

```yaml
services:
  db:
    image: postgres:16
    volumes:
      - postgres-data:/var/lib/postgresql/data

volumes:
  postgres-data:
```

Then:

```bash
docker compose down
docker compose up -d
```

does not normally remove the named volume.

## Important

A volume provides persistence across container replacement, but it is **not automatically a backup strategy**.

## Interview Point

> **Persistence and backup are different concerns.**

---

# 508. What happens to Compose volumes when containers are recreated?

## Short Interview Answer

A named volume can be reused by the replacement container.

For example:

```text
Container A
    ↓
postgres-data
```

Container A is removed:

```text
Container A ✗
postgres-data ✓
```

New container:

```text
Container B
    ↓
postgres-data
```

The data remains in the volume.

## Interview Point

> **The container lifecycle and named-volume lifecycle are separate.**

---

# 509. How do environment variables work in Compose?

## Short Interview Answer

Compose can pass environment variables to containers using `environment`, environment files and other Compose/environment mechanisms.

Example:

```yaml
services:
  api:
    image: my-api
    environment:
      DB_HOST: db
      DB_PORT: "5432"
```

Another approach:

```yaml
services:
  api:
    env_file:
      - .env
```

## Important

Environment variables are configuration, not automatically secure secret storage.

## Interview Point

> **Do not put sensitive credentials into ordinary configuration merely because Compose supports environment variables.**

---

# 510. What is an `.env` file in Compose?

## Short Interview Answer

An `.env` file is commonly used to provide variable values that Compose can use for interpolation.

Example:

```text
IMAGE_TAG=1.5
APP_PORT=8080
```

Compose file:

```yaml
services:
  api:
    image: my-api:${IMAGE_TAG}
    ports:
      - "${APP_PORT}:8080"
```

## Important Distinction

An `.env` file and a container's runtime environment are not automatically the same thing.

Variable interpolation and passing environment variables into containers are related but distinct operations.

## Interview Point

> **Do not assume every variable in `.env` automatically appears inside every container.**

---

# 511. What is `env_file` in Compose?

## Short Interview Answer

`env_file` specifies a file containing environment variables that are passed into the service container.

Example:

```yaml
services:
  api:
    image: my-api
    env_file:
      - app.env
```

This differs from using `.env` purely for Compose interpolation.

## Interview Point

> **`.env` can provide Compose substitution values; `env_file` is a service-level mechanism for supplying container environment variables.**

---

# 512. How do you override configuration in Compose?

## Short Interview Answer

Compose supports several mechanisms, including:

- shell environment variables
- variable interpolation
- environment configuration
- override files
- command-line options

A common development pattern is:

```text
compose.yaml
      +
compose.override.yaml
```

where the override file changes selected development settings.

## Example

Base:

```yaml
services:
  api:
    image: my-api:1.0
```

Development override:

```yaml
services:
  api:
    build: .
```

## Interview Point

> **Compose configuration can be layered rather than duplicating the entire base file.**

---

# 513. What are Compose profiles?

## Short Interview Answer

Profiles allow optional services to be enabled only for particular use cases.

Example:

```yaml
services:
  api:
    image: my-api

  adminer:
    image: adminer
    profiles:
      - debug
```

Start normally:

```bash
docker compose up -d
```

Start the debug profile:

```bash
docker compose --profile debug up -d
```

## Use Cases

- debugging tools
- development-only services
- optional monitoring components
- administrative interfaces

## Interview Point

> **Profiles let one Compose configuration support different application modes without always starting every service.**

---

# 514. What does `depends_on` do?

## Short Interview Answer

`depends_on` expresses a startup/shutdown dependency relationship between services.

Example:

```yaml
services:
  api:
    image: my-api
    depends_on:
      - db

  db:
    image: postgres:16
```

It does **not automatically mean the database is ready to accept application connections** merely because its container has started.

## Common Interview Trap

This is one of the most important Compose interview traps:

```text
container started
      ≠
application ready
```

## Interview Point

> **`depends_on` expresses dependency ordering; readiness requires an appropriate health/readiness mechanism and application retry behavior.**

---

# 515. How do healthchecks work with Compose?

## Short Interview Answer

A healthcheck tests whether the application inside a container is functioning as expected.

Example:

```yaml
services:
  db:
    image: postgres:16
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
```

A dependent service can use health information where supported by the Compose configuration.

## Important Distinction

```text
Container running
        ≠
Container healthy
```

## Interview Point

> **Healthchecks measure application health, not merely container existence.**

---

# 516. Why is `depends_on` alone insufficient for database readiness?

## Short Interview Answer

Because a database process can be running while it is still initializing and unable to accept connections.

Example:

```text
db container starts
       ↓
PostgreSQL initialization
       ↓
database becomes ready
```

The API may start during the middle step.

A robust application should also tolerate transient dependency failures and retry appropriately.

## Interview Point

> **Startup ordering is not the same as readiness.**

---

# 517. How do you build an image with Compose?

## Short Interview Answer

Use `build` in the service:

```yaml
services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
```

Then:

```bash
docker compose build
```

or:

```bash
docker compose up --build
```

## Interview Point

> **`build` tells Compose how to construct the service image; `image` identifies an image to use/tag.**

---

# 518. What is the difference between `image` and `build` in Compose?

## Short Interview Answer

`image` specifies an image to run:

```yaml
image: nginx:alpine
```

`build` specifies how to build an image:

```yaml
build:
  context: .
  dockerfile: Dockerfile
```

A service can use both:

```yaml
services:
  api:
    build: .
    image: my-api:1.0
```

This allows the image produced by the build to have the specified image name/tag.

## Interview Point

> **`image` selects/names an image; `build` defines how to build one.**

---

# 519. How do you rebuild a Compose service?

## Short Interview Answer

Build:

```bash
docker compose build api
```

Then start/recreate as necessary:

```bash
docker compose up -d api
```

Or:

```bash
docker compose up -d --build api
```

## Interview Point

> **Rebuilding an image and recreating the running container are related but separate concepts.**

---

# 520. How do you force Compose to recreate containers?

## Short Interview Answer

Use:

```bash
docker compose up -d --force-recreate
```

This tells Compose to recreate containers even when it otherwise determines recreation is unnecessary.

## Useful Related Command

```bash
docker compose up -d --no-recreate
```

prevents recreation when possible.

## Interview Point

> **Image rebuilding and container recreation are separate operations.**

---

# 521. How do you scale a Compose service?

## Short Interview Answer

For local Compose usage, a service can be scaled using:

```bash
docker compose up -d --scale api=3
```

This creates multiple container instances for that service.

## Important

The application must be designed for multiple instances, and host port publishing can become a constraint.

For example, this can be problematic:

```yaml
ports:
  - "8080:8080"
```

if three instances all need the same host port.

## Interview Point

> **Scaling containers does not automatically solve load balancing, state management or port allocation.**

---

# 522. How does Compose handle service names when scaling?

## Short Interview Answer

A service can have multiple container instances, so applications should not depend on one specific container IP or container name.

Compose networking provides service-level discovery, but the exact DNS behavior and returned addresses depend on the Compose/Docker setup.

For scalable application designs, put an appropriate load-balancing/service-discovery layer in front of replicated instances.

## Interview Point

> **Never build application logic around a single container IP.**

---

# 523. What is a Compose project?

## Short Interview Answer

A Compose project is a logical application grouping created from a Compose configuration.

The project name is used to namespace resources such as:

- networks
- containers
- volumes

You can explicitly choose a project name:

```bash
docker compose -p myproject up -d
```

## Why It Matters

Two Compose applications can use similarly named services without necessarily colliding because their resources are project-scoped/namespaced.

## Interview Point

> **Project naming is important when running multiple Compose applications on the same Docker host.**

---

# 524. What is the difference between `docker compose config` and `docker compose up`?

## Short Interview Answer

`docker compose config` renders and validates the Compose configuration without starting the application.

Example:

```bash
docker compose config
```

It is useful for checking:

- YAML structure
- variable interpolation
- merged configuration
- effective service definitions

Then:

```bash
docker compose up -d
```

actually starts the application.

## Interview Point

> **Use `config` to inspect the effective Compose model before deploying it.**

---

# 525. How do you troubleshoot a Compose application?

## Short Interview Answer

I start with:

```bash
docker compose ps
docker compose logs
docker compose config
```

Then inspect the affected container:

```bash
docker inspect <container>
```

For networking:

```bash
docker network ls
docker network inspect <network>
```

For storage:

```bash
docker volume ls
docker volume inspect <volume>
```

For resources:

```bash
docker stats
```

## Troubleshooting Flow

```text
Compose config
      ↓
Service state
      ↓
Container logs
      ↓
Container configuration
      ↓
Dependencies
      ↓
Network
      ↓
Storage
      ↓
Resources
      ↓
Application
```

## Interview Point

> **Troubleshoot the Compose layer and the underlying Docker objects separately.**

---

# 526. A Compose service is restarting. What do you check?

## Short Interview Answer

First:

```bash
docker compose ps
docker compose logs --tail 100 api
```

Then:

```bash
docker inspect <container>
```

Check:

- exit code
- OOMKilled
- restart policy
- command/entrypoint
- environment variables
- mounted files
- dependency availability
- health status

## Interview Point

> **Compose does not eliminate the need to understand normal Docker container troubleshooting.**

---

# 527. A Compose service cannot connect to the database. How do you troubleshoot it?

## Short Interview Answer

I verify:

1. database service is running
2. database is ready
3. both services share a network
4. application uses the correct service hostname
5. correct container port is used
6. credentials are correct
7. database accepts the connection

Example:

```bash
docker compose ps
docker compose logs db
docker compose exec api getent hosts db
docker compose exec api nc -vz db 5432
```

The application should generally use:

```text
db:5432
```

rather than:

```text
localhost:5432
```

## Interview Trap

Inside the API container:

```text
localhost
```

means the API container itself.

## Interview Point

> **For Compose service-to-service communication, use the service name and container port.**

---

# 528. Why does `localhost` usually break service-to-service communication in Compose?

## Short Interview Answer

Each container normally has its own network namespace.

Therefore:

```text
api container:
localhost → api container

db container:
localhost → db container
```

So:

```text
api → localhost:5432
```

does not normally mean:

```text
api → db
```

Instead:

```text
api → db:5432
```

## Interview Point

> **Use the database service name, not `localhost`, from another container.**

---

# 529. How do you view logs for one Compose service?

## Short Interview Answer

Use:

```bash
docker compose logs api
```

Follow logs:

```bash
docker compose logs -f api
```

Limit output:

```bash
docker compose logs --tail 100 api
```

All services:

```bash
docker compose logs
```

## Interview Point

> **Compose gives a project/service-level interface over container logs.**

---

# 530. How do you execute a command inside a Compose service?

## Short Interview Answer

Use:

```bash
docker compose exec api sh
```

or:

```bash
docker compose exec api env
```

This targets a running service container.

For a one-off container:

```bash
docker compose run --rm api <command>
```

## Important Distinction

```text
exec → existing running container
run  → new one-off container
```

## Interview Point

> **Do not confuse `exec` with `run`.**

---

# 531. What is the difference between Compose and Docker Swarm?

## Short Interview Answer

Docker Compose is primarily used to define and run multi-container applications, commonly on a single Docker host or development environment.

Docker Swarm is a cluster orchestration system that manages services across multiple Docker nodes.

Conceptually:

```text
Compose
  ↓
Application definition / local multi-container management

Swarm
  ↓
Cluster orchestration
  ↓
Multiple Docker nodes
```

## Interview Point

> **Compose and Swarm can both use declarative service definitions, but they solve different operational problems.**

---

# 532. What is Docker Swarm?

## Short Interview Answer

Docker Swarm is Docker's native cluster orchestration mode.

It provides concepts such as:

- manager nodes
- worker nodes
- services
- replicas
- scheduling
- service discovery
- overlay networking
- rolling updates

Example:

```text
Manager
  ├── Worker
  ├── Worker
  └── Worker
```

## Interview Point

> **Swarm extends Docker from single-host container management toward multi-node orchestration.**

---

# 533. What is the difference between a Docker container and a Swarm service?

## Short Answer

A container is an individual runtime instance.

A Swarm service defines the desired state of replicated tasks.

Conceptually:

```text
Swarm service
      ↓
desired replicas
      ↓
tasks
      ↓
containers
```

## Interview Point

> **A Swarm service expresses desired state; containers are runtime instances of scheduled tasks.**

---

# 534. What is a Swarm task?

## Short Interview Answer

A task is the atomic scheduling unit created by a Swarm service.

A service might request:

```text
3 replicas
```

and Swarm schedules three tasks.

Each task normally corresponds to one container instance.

## Conceptual Flow

```text
Service
  ↓
3 desired replicas
  ↓
3 tasks
  ↓
3 containers
```

## Interview Point

> **Task is a Swarm scheduling concept between service desired state and container execution.**

---

# 535. What is a Swarm manager?

## Short Interview Answer

A Swarm manager participates in cluster control-plane operations such as:

- maintaining cluster state
- scheduling services
- managing membership
- coordinating desired state

Manager nodes use a consensus mechanism based on Raft for cluster state.

## Interview Point

> **Managers provide the Swarm control plane; workers execute tasks.**

---

# 536. What is a Swarm worker?

## Short Interview Answer

A worker node runs tasks assigned by the Swarm manager.

Conceptually:

```text
Manager
   ↓
schedule task
   ↓
Worker
   ↓
container
```

A worker does not independently decide the desired cluster state.

## Interview Point

> **Workers provide execution capacity; managers maintain orchestration control.**

---

# 537. What is a Swarm overlay network?

## Short Interview Answer

An overlay network provides container/service connectivity across multiple Docker hosts in a Swarm cluster.

Example:

```text
Node A                 Node B

Container A  ←──────→  Container B
       \                /
        \ overlay net  /
```

This is different from a normal single-host bridge network.

## Interview Point

> **Bridge networking is primarily single-host; overlay networking enables multi-host container networking.**

---

# 538. What is the difference between Compose and Kubernetes?

## Short Interview Answer

Compose is a lightweight tool for defining/running multi-container Docker applications.

Kubernetes is a full container orchestration platform designed for cluster-wide scheduling, networking, service discovery, scaling, rolling deployments, self-healing and more.

| Compose | Kubernetes |
|---|---|
| Simpler | More comprehensive |
| Excellent for local/dev/small deployments | Designed for cluster orchestration |
| YAML-based | YAML/API-based |
| Docker-centric | Container-runtime ecosystem |
| Lower operational complexity | Higher operational complexity |
| Limited orchestration scope | Extensive orchestration features |

## Interview Point

> **Do not present Compose as a replacement for Kubernetes in large-scale orchestration.**

---

# 539. Can Docker Compose be used in production?

## Short Interview Answer

**Yes, depending on the workload and operational requirements.**

Compose can be appropriate for:

- small deployments
- single-host applications
- development/staging
- internal tools
- simple production services

For large distributed environments requiring advanced orchestration, Kubernetes or another orchestrator may be more appropriate.

## Interview Trap

Do not say:

> "Compose is only for development."

That is too absolute.

## Interview Point

> **Choose the orchestration platform based on operational requirements, not fashion.**

---

# 540. What does "orchestration" mean?

## Short Interview Answer

Container orchestration means automatically managing containers across an environment according to a desired state.

Typical orchestration capabilities include:

- scheduling
- scaling
- service discovery
- health management
- rolling updates
- rescheduling
- networking
- workload placement

Conceptually:

```text
Desired state
      ↓
Orchestrator
      ↓
Actual state
      ↓
Reconcile
      ↓
Desired state
```

## Interview Point

> **Orchestration is fundamentally about maintaining desired state and managing workloads at scale.**

---

# 541. What is desired state?

## Short Interview Answer

Desired state is the condition the orchestrator is instructed to maintain.

Example:

```text
api replicas = 3
```

If one instance fails:

```text
Desired = 3
Actual = 2
```

An orchestrator can detect the difference and create another instance.

## Interview Point

> **Desired state enables reconciliation and self-healing behavior.**

---

# 542. What is self-healing in container orchestration?

## Short Interview Answer

Self-healing means the orchestrator detects that actual state differs from desired state and takes corrective action.

Example:

```text
Desired: 3 replicas

Replica 1 ✓
Replica 2 ✗
Replica 3 ✓

        ↓

Orchestrator schedules replacement

Replica 1 ✓
Replica 2 ✓
Replica 3 ✓
```

## Important

The exact behavior depends on the orchestration platform and workload configuration.

## Interview Point

> **Self-healing is reconciliation of actual state toward desired state.**

---

# 543. Does Docker Compose provide full self-healing orchestration?

## Short Interview Answer

No—not in the same cluster-orchestration sense as Kubernetes.

Compose can use Docker restart policies and healthchecks, but it does not provide the same cluster-level scheduling, reconciliation and multi-node orchestration capabilities as Kubernetes.

## Interview Point

> **Restart policies are not equivalent to full orchestration.**

---

# 544. What is a restart policy in a Compose application?

## Short Interview Answer

A restart policy controls when Docker should restart a container.

Example:

```yaml
services:
  api:
    image: my-api:1.0
    restart: unless-stopped
```

Common policies include:

```text
no
on-failure
always
unless-stopped
```

## Important

Restart policies operate at the Docker container lifecycle level. They are not equivalent to cluster scheduling.

## Interview Point

> **Restart policy provides local recovery behavior, not full orchestration.**

---

# 545. What is a good Compose architecture for a typical three-tier application?

## Example

```text
                 Internet
                    │
                    ▼
              Reverse Proxy
                    │
                    ▼
                 Frontend
                    │
                    ▼
                   API
                    │
                    ▼
                Database
```

Compose:

```yaml
services:
  proxy:
    image: nginx:alpine

  frontend:
    image: my-frontend:1.0

  api:
    image: my-api:1.0

  db:
    image: postgres:16
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

## Network Design

A more controlled architecture can use separate networks:

```text
proxy-net
   ↓
proxy ↔ frontend/api

backend-net
   ↓
api ↔ db
```

This reduces unnecessary network exposure.

## Interview Point

> **Use network boundaries to express which services actually need to communicate.**

---

# 546. How would you design a Compose file for development versus production?

## Short Interview Answer

I would keep the application definition reusable and separate environment-specific configuration.

Typical approach:

```text
compose.yaml
compose.override.yaml
```

or explicit additional Compose files.

Development may include:

- source bind mounts
- debugging tools
- hot reload
- development databases

Production may use:

- immutable image tags/digests
- minimal configuration overrides
- persistent volumes
- secrets
- restart policies
- healthchecks
- resource constraints
- external monitoring

## Interview Point

> **Do not turn the development Compose file into an accidental production configuration.**

---

# 547. What are common Docker Compose mistakes?

## High-Value Mistakes

### 1. Using `localhost` between services

Wrong:

```text
api → localhost:5432
```

Better:

```text
api → db:5432
```

### 2. Publishing every service

Database/internal services usually do not need host publishing.

### 3. Hard-coding container IPs

Use service names.

### 4. Assuming `depends_on` means ready

Startup order ≠ application readiness.

### 5. Using `down -v` casually

Can remove persistent volumes.

### 6. Putting secrets in ordinary environment configuration

Use an appropriate secret-management mechanism.

### 7. Assuming `up` always means "start existing containers only"

Compose may create or recreate resources according to configuration.

### 8. Scaling services without considering ports/state

Multiple instances introduce load-balancing and state-management concerns.

### 9. Treating Compose as Kubernetes

They solve different levels of orchestration problems.

### 10. Duplicating large YAML files

Prefer reusable configuration and overrides where appropriate.

---

# 548. Give a production-style Compose troubleshooting scenario.

## Scenario

```text
API container is running.
Database container is running.
API reports "connection refused".
```

## Strong Interview Approach

### Step 1 — Check service state

```bash
docker compose ps
```

### Step 2 — Check database logs

```bash
docker compose logs db
```

### Step 3 — Check readiness

If configured:

```bash
docker inspect <db-container>
```

Look at health status.

### Step 4 — Verify DNS

```bash
docker compose exec api getent hosts db
```

### Step 5 — Verify TCP

```bash
docker compose exec api nc -vz db 5432
```

### Step 6 — Verify application configuration

Check:

```text
DB_HOST=db
DB_PORT=5432
```

not:

```text
DB_HOST=localhost
```

### Step 7 — Verify database listener

Inside DB container, use an appropriate database/listener diagnostic.

## Conclusion

The problem may be:

```text
database not ready
```

rather than:

```text
Docker network broken
```

## Interview Point

> **Use layered tests to identify the first failing boundary.**

---

# Quick Revision

| Concept | Key Answer |
|---|---|
| Compose | Multi-container application definition/management |
| Compose file | Declarative YAML configuration |
| Service | Logical application component |
| Container | Runtime instance |
| `up` | Create/reconcile/start application |
| `down` | Stop/remove Compose resources |
| `stop` | Stop containers, preserve objects |
| `start` | Start existing stopped containers |
| `run` | One-off service container |
| `ports` | Host port publishing |
| `expose` | Internal/documentation-oriented port declaration |
| Service DNS | Service-name-based discovery |
| Named volume | Persistent Docker-managed storage |
| `depends_on` | Dependency ordering/relationship |
| Healthcheck | Application health signal |
| Profiles | Optional service groups |
| `config` | Render/validate effective Compose config |
| `--scale` | Multiple service instances |
| Project | Logical Compose application grouping |
| Compose | Multi-container definition/management |
| Swarm | Docker-native cluster orchestration |
| Kubernetes | Full-featured container orchestration platform |

---

# High-Value Interview Traps

## Trap 1 — `depends_on` means ready

**Incorrect:**

```text
depends_on = database is ready
```

**Correct:**

```text
depends_on = dependency relationship/order
```

Readiness requires healthchecks/application retry logic.

---

## Trap 2 — Services communicate through published ports

Usually unnecessary.

```text
api → db:5432
```

does not require:

```yaml
ports:
  - "5432:5432"
```

---

## Trap 3 — `localhost` means the Docker host

Inside a container:

```text
localhost = that container
```

---

## Trap 4 — Compose service = container

Not exactly.

```text
Service definition
       ↓
Container instance(s)
```

---

## Trap 5 — `docker compose down -v` is harmless

It can remove named volumes and therefore persistent data.

---

## Trap 6 — Restart policy = orchestration

Restart policy handles container restart behavior.

It does not provide the full scheduling/reconciliation capabilities of a cluster orchestrator.

---

## Trap 7 — Compose is only for development

Too absolute.

Compose can be suitable for some production workloads, especially single-host/simple deployments.

---

## Trap 8 — Scaling automatically provides load balancing

Scaling creates multiple instances, but the application still needs an appropriate traffic-distribution mechanism.

---

## Trap 9 — Container running = application healthy

False.

Use healthchecks and application-level tests.

---

## Trap 10 — `up` and `start` are identical

They are not:

```text
up    → bring application configuration to desired running state
start → start existing stopped containers
```

---

# Interview Follow-Up Questions

1. What happens when you run `docker compose up -d`?
2. What is the difference between `up`, `start`, `run` and `down`?
3. How do Compose services communicate?
4. Why should you not use container IPs?
5. Why does `localhost` fail for database connections?
6. Do internal services need published ports?
7. What does `depends_on` actually guarantee?
8. How do you implement readiness?
9. How do healthchecks work?
10. What happens to named volumes after `docker compose down`?
11. What does `down -v` do?
12. How do you troubleshoot a Compose application?
13. What is `docker compose config` useful for?
14. How do you scale a Compose service?
15. What happens when a scaled service publishes a fixed host port?
16. What are Compose profiles?
17. What is the difference between Compose and Swarm?
18. What is the difference between Compose and Kubernetes?
19. Can Compose be used in production?
20. What is desired state?
21. What is self-healing?
22. Why isn't a restart policy equivalent to orchestration?
23. How would you design networks for frontend/API/database?
24. How would you handle persistent database data?
25. How would you separate development and production configuration?

---

# Final Interview Answer

If asked:

> **"Explain Docker Compose and how it differs from orchestration platforms."**

Answer:

> "Docker Compose is a declarative tool for defining and running multi-container applications. I can define services, networks, volumes, environment configuration and healthchecks in a Compose YAML file and bring the application up with `docker compose up`. Services normally communicate over Docker networks using service-name DNS, so they don't need to publish ports to communicate internally. Compose is very useful for development, testing and suitable single-host deployments. It should not be confused with a full cluster orchestrator. Docker Swarm and Kubernetes provide broader orchestration capabilities such as multi-node scheduling, desired-state reconciliation, service placement, scaling and self-healing. So I would choose Compose when the application's operational requirements are relatively simple and use a cluster orchestrator when distributed workload management is required."

---

# One-Line Memory Map

```text
Compose File
     ↓
Project
     ↓
Services
 ┌───┼────────┐
 ↓   ↓        ↓
API  DB    Frontend
 │    │        │
 └────┴────────┘
       ↓
   Docker Network
       ↓
  Volumes / Config
       ↓
   Containers

Compose
  ↓
Multi-container application management

Swarm / Kubernetes
  ↓
Cluster orchestration
  ↓
Scheduling + Desired State + Scaling + Recovery
```

---

# Topic 18 Complete

**Questions covered: Q491–Q548**

**Core skill:**

> **Understand the difference between defining an application, running containers, and orchestrating workloads across a cluster.**
