# Docker Interview Preparation — Topic 09: Docker Commands

> **Purpose:** Build strong command-line fluency for Docker interviews, including container, image, inspection, logs, execution, lifecycle, cleanup, and troubleshooting commands.
>
> **Scope:** This topic focuses on command usage and what each command actually does. Networking, storage, security, and advanced build workflows are covered separately.

---

## 176. What is the basic syntax of a Docker command?

### Short Interview Answer

The general structure is:

```bash
docker <command> <subcommand> [options] [arguments]
```

Examples:

```bash
docker ps
docker images
docker run nginx
docker exec -it mycontainer bash
```

Modern Docker commands are organized into command groups such as:

```bash
docker container ...
docker image ...
docker network ...
docker volume ...
```

### Example

These are related forms:

```bash
docker ps
docker container ls
```

Both list containers.

Similarly:

```bash
docker images
docker image ls
```

list images.

### Common Interview Trap

Do not assume every Docker command has exactly the same argument structure.

For example:

```bash
docker run
```

has many runtime options, while:

```bash
docker inspect
```

is primarily an inspection operation.

### Interview Point

**Know both the command and the object it operates on.**

---

## 177. What is `docker run`?

### Short Interview Answer

`docker run` creates a new container from an image and starts it.

### Example

```bash
docker run nginx
```

Conceptually:

```text
Image
  ↓
create container
  ↓
configure runtime
  ↓
start container
```

### Useful Options

```bash
docker run -d nginx
```

Run in detached mode.

```bash
docker run --name web nginx
```

Assign a name.

```bash
docker run -p 8080:80 nginx
```

Publish host port 8080 to container port 80.

```bash
docker run -e APP_ENV=prod myapp
```

Set a runtime environment variable.

### Important Point

`docker run` is effectively a convenience workflow that creates and starts a new container.

### Common Interview Trap

`docker run` does not start an existing stopped container.

For an existing stopped container:

```bash
docker start <container>
```

### Interview Point

**`docker run` = create + start a new container from an image.**

---

## 178. What is the difference between `docker create` and `docker run`?

### Short Interview Answer

`docker create` creates a container but does not start it. `docker run` creates and starts a new container.

### Example

```bash
docker create --name web nginx
```

At this point:

```text
Container exists
Container is not running
```

Start it:

```bash
docker start web
```

With:

```bash
docker run --name web nginx
```

Docker performs the create/start workflow.

### Lifecycle

```text
docker create
      ↓
Created
      ↓
docker start
      ↓
Running
```

Whereas:

```text
docker run
   ↓
Created
   ↓
Running
```

### Common Interview Trap

`docker create` does not execute the application's main process.

### Interview Point

**`create` prepares the container; `run` starts a newly created container.**

---

## 179. What does `docker ps` show?

### Short Interview Answer

`docker ps` lists running containers by default.

### Example

```bash
docker ps
```

Typical information includes:

```text
CONTAINER ID
IMAGE
COMMAND
CREATED
STATUS
PORTS
NAMES
```

### Show All Containers

```bash
docker ps -a
```

This includes:

```text
running
stopped
created
exited
```

and other non-running containers.

### Useful Filters

```bash
docker ps --filter "status=exited"
```

### Common Interview Trap

```bash
docker ps
```

does **not** show all containers by default.

### Interview Point

**`docker ps` = running containers; `docker ps -a` = all containers.**

---

## 180. How do you start an existing stopped container?

### Short Interview Answer

Use:

```bash
docker start <container>
```

### Example

```bash
docker start web
```

If the container was created with:

```bash
docker run --name web nginx
```

and later stopped:

```bash
docker stop web
```

you can restart that same container:

```bash
docker start web
```

### Important Difference

```text
docker start
    ↓
existing container

docker run
    ↓
new container from image
```

### Common Interview Trap

Running:

```bash
docker run nginx
```

after stopping `web` creates another container. It does not restart `web`.

### Interview Point

**Use `start` for an existing container; use `run` for a new container.**

---

## 181. What is the difference between `docker stop` and `docker kill`?

### Short Interview Answer

`docker stop` requests graceful termination using the container's configured stop behavior, while `docker kill` sends a signal immediately, with `SIGKILL` commonly used by default on Linux.

### Example

```bash
docker stop web
```

versus:

```bash
docker kill web
```

### Conceptual Difference

```text
docker stop
    ↓
termination signal
    ↓
application gets opportunity to shut down
    ↓
forceful termination if it does not stop in time
```

`docker kill` is more abrupt.

### Why It Matters

Graceful shutdown allows an application to:

- close connections
- flush buffers
- finish cleanup
- release resources

### Common Interview Trap

Do not say "`docker stop` always waits forever."

Docker has a stop timeout; behavior can be configured.

### Interview Point

**Prefer graceful shutdown with `stop`; use `kill` when immediate termination is required.**

---

## 182. What does `docker restart` do?

### Short Interview Answer

`docker restart` stops and starts an existing container.

### Example

```bash
docker restart web
```

It is useful when you want to restart the same container rather than create a new one.

### Conceptual Flow

```text
Running
   ↓
stop
   ↓
Stopped
   ↓
start
   ↓
Running
```

### Important Point

Restarting a container does not create a new container identity.

The container remains the same container.

### Common Interview Trap

A restart does not automatically rebuild the image or recreate the container from a newer image.

### Interview Point

**`restart` operates on an existing container.**

---

## 183. What is `docker rm`?

### Short Interview Answer

`docker rm` removes a container.

### Example

```bash
docker rm web
```

Usually a running container must first be stopped:

```bash
docker stop web
docker rm web
```

You can force removal:

```bash
docker rm -f web
```

### Important Point

Removing a container does not remove the image from which it was created.

```text
container → removed
image     → remains
```

### Common Interview Trap

`docker rm` is not the command for deleting images.

Use:

```bash
docker rmi <image>
```

### Interview Point

**`rm` removes containers; `rmi` removes images.**

---

## 184. What is `docker rm -f`?

### Short Interview Answer

It force-removes a container and can stop a running container as part of the operation.

### Example

```bash
docker rm -f web
```

This is useful when a container will not stop normally or when you intentionally want immediate removal.

### Important Caveat

Force removal should not be treated as the normal graceful-shutdown mechanism.

For applications that need cleanup:

```bash
docker stop web
```

is generally preferable.

### Common Interview Trap

Do not use `rm -f` as the first response to every container problem.

First understand why the container is not stopping.

### Interview Point

**`rm -f` is a forceful lifecycle operation, not a graceful shutdown strategy.**

---

## 185. What is `docker exec`?

### Short Interview Answer

`docker exec` runs a new process inside an already running container.

### Example

```bash
docker exec -it web sh
```

This starts an interactive shell process inside `web`.

Run a single command:

```bash
docker exec web ls /app
```

Run as a specific user:

```bash
docker exec -u 1000 web id
```

### Important Concept

The command launched by `docker exec` is **not the container's main process**.

For example:

```text
Container
 ├── PID 1 → application
 └── exec process → shell/diagnostic command
```

### Common Interview Trap

`docker exec` requires the container to be running.

### Interview Point

**`docker exec` starts an additional process inside a running container.**

---

## 186. What is the difference between `docker exec` and `docker attach`?

### Short Interview Answer

`docker exec` starts a new process inside a running container. `docker attach` connects your terminal to the container's existing main process streams.

### Example

Exec:

```bash
docker exec -it web sh
```

Attach:

```bash
docker attach web
```

### Conceptual Difference

```text
docker exec
     ↓
new process
     ↓
shell/command
```

versus:

```text
docker attach
     ↓
existing container process
     ↓
its stdin/stdout/stderr
```

### Why It Matters

For troubleshooting, `exec` is generally safer because you can launch a separate shell without taking over the main process's terminal.

### Common Interview Trap

`docker attach` does not launch a new shell.

### Interview Point

**Exec = new process; attach = connect to existing process streams.**

---

## 187. What does `docker logs` do?

### Short Interview Answer

`docker logs` retrieves the logs available through the container's configured logging mechanism, commonly the container's stdout and stderr streams.

### Example

```bash
docker logs web
```

Follow logs:

```bash
docker logs -f web
```

Show timestamps:

```bash
docker logs -t web
```

Show recent lines:

```bash
docker logs --tail 100 web
```

Show logs since a time:

```bash
docker logs --since 10m web
```

### Important Point

The exact behavior depends on the configured logging driver and whether logs are available through Docker's logging interface.

### Common Interview Trap

`docker logs` does not automatically read arbitrary application log files such as:

```text
/app/logs/app.log
```

unless the application's logging setup and Docker logging configuration make those logs available through the container logging mechanism.

### Interview Point

**For containerized applications, prefer logging to stdout/stderr when appropriate so the container logging system can collect it.**

---

## 188. What is `docker inspect`?

### Short Interview Answer

`docker inspect` returns low-level structured information about Docker objects.

### Example

```bash
docker inspect web
```

It can inspect:

- containers
- images
- networks
- volumes
- other supported Docker objects

### Useful Example

```bash
docker inspect -f '{{.State.Status}}' web
```

Another example:

```bash
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' web
```

### Why It Matters

When troubleshooting, inspect can reveal:

```text
container state
network configuration
mounts
environment
restart policy
image reference
runtime configuration
```

### Common Interview Trap

`docker inspect` does not mean "show application logs."

Use:

```bash
docker logs
```

for container logs.

### Interview Point

**`inspect` gives structured configuration/state; `logs` gives log output.**

---

## 189. What is `docker stats`?

### Short Interview Answer

`docker stats` provides live resource-usage information for containers.

### Example

```bash
docker stats
```

Typical metrics include:

```text
CPU %
Memory usage / limit
Memory %
Network I/O
Block I/O
PIDs
```

Inspect one container:

```bash
docker stats web
```

### Why It Matters

Useful for identifying containers consuming excessive:

```text
CPU
memory
network I/O
block I/O
processes
```

### Important Caveat

`docker stats` is a runtime observation tool. It is not a replacement for a full production monitoring system.

### Common Interview Trap

High CPU in `docker stats` does not by itself explain the root cause. You still need application/process-level investigation.

### Interview Point

**`docker stats` answers "how much resource is this container using?"**

---

## 190. What is `docker top`?

### Short Interview Answer

`docker top` displays the processes currently running inside a container.

### Example

```bash
docker top web
```

It can show process information such as:

```text
PID
USER
COMMAND
```

depending on the platform and underlying implementation.

### Why Useful?

If a container appears unhealthy:

```bash
docker top web
```

can help answer:

```text
Is the application process running?
Are there unexpected processes?
How many processes are present?
```

### Difference From `docker stats`

```text
docker top
    ↓
processes

docker stats
    ↓
resource usage
```

### Interview Point

**`top` shows container processes; `stats` shows resource metrics.**

---

## 191. What is `docker image ls` / `docker images`?

### Short Interview Answer

It lists images stored in the local Docker image store.

### Examples

```bash
docker images
```

and:

```bash
docker image ls
```

### Typical Information

```text
REPOSITORY
TAG
IMAGE ID
CREATED
SIZE
```

### Filtering

```bash
docker image ls nginx
```

### Important Point

This shows local images, not every image available in a registry.

### Common Interview Trap

`docker images` does not query Docker Hub and list every remotely available image.

### Interview Point

**`docker image ls` is primarily a local image inventory command.**

---

## 192. What is `docker image rm` / `docker rmi`?

### Short Interview Answer

It removes a local image reference/image when it is no longer needed and not blocked by dependent containers or other references.

### Example

```bash
docker image rm myapp:1.0
```

Short form:

```bash
docker rmi myapp:1.0
```

### Important Point

Removing a tag/reference does not always mean every underlying layer immediately disappears. Layers can be shared by other images or references.

### Common Interview Trap

```bash
docker rmi
```

does not delete a remote registry image.

It operates on the local image store.

### Interview Point

**`rmi` removes local image references/content subject to sharing and dependencies.**

---

## 193. What is `docker pull`?

### Short Interview Answer

`docker pull` downloads an image or image components from a registry into the local image store.

### Example

```bash
docker pull nginx:1.29
```

### Conceptual Flow

```text
Registry
   ↓
image manifest/index
   ↓
required layers
   ↓
local image store
```

### Multi-Platform Note

The selected platform can depend on the host and options such as:

```bash
docker pull --platform linux/amd64 image:tag
```

### Common Interview Trap

`docker pull` does not create a running container.

You need:

```bash
docker run ...
```

to create/start one.

### Interview Point

**Pull downloads an image; run creates/starts a container from an image.**

---

## 194. What is `docker push`?

### Short Interview Answer

`docker push` uploads a local image/tag to a registry repository.

### Example

```bash
docker tag myapp:1.0 registry.example.com/team/myapp:1.0

docker push registry.example.com/team/myapp:1.0
```

### Conceptual Flow

```text
Local image
    ↓
tag/reference
    ↓
registry repository
    ↓
push manifest + required layers
```

### Important Point

You generally need to authenticate to a private registry:

```bash
docker login registry.example.com
```

### Common Interview Trap

`docker push` does not push a container.

It pushes an image representation.

### Interview Point

**Registries store/publish images; containers are runtime instances.**

---

## 195. What is `docker tag`?

### Short Interview Answer

`docker tag` creates another name/reference for an existing local image.

### Example

```bash
docker tag myapp:1.0 \
  registry.example.com/team/myapp:1.0
```

Then:

```bash
docker push registry.example.com/team/myapp:1.0
```

### Important Point

Tagging does not rebuild the image.

It creates another reference to the same underlying image content.

### Example

```text
myapp:1.0
       │
       └──── same image content ──── registry.example.com/team/myapp:1.0
```

### Common Interview Trap

> "`docker tag` copies the image."

Not in the normal sense.

It creates another image reference/tag.

### Interview Point

**Tagging changes the reference, not the application artifact itself.**

---

## 196. What is `docker cp`?

### Short Interview Answer

`docker cp` copies files or directories between a container filesystem and the local filesystem.

### Container → Host

```bash
docker cp web:/app/log.txt ./log.txt
```

### Host → Container

```bash
docker cp ./config.yaml web:/app/config.yaml
```

### Why Useful?

It can help with:

- extracting diagnostic files
- retrieving generated artifacts
- placing temporary files into a container

### Important Caveat

For persistent application data, bind mounts or volumes are generally more appropriate than repeatedly using `docker cp`.

### Common Interview Trap

`docker cp` does not establish a persistent synchronization relationship.

### Interview Point

**`docker cp` performs an explicit file copy; it is not a volume.**

---

## 197. What is `docker rename`?

### Short Interview Answer

`docker rename` changes the name of an existing container.

### Example

```bash
docker rename old-web web
```

### Important Point

Renaming a container does not:

- create a new container
- rebuild its image
- change the underlying container ID

### Common Interview Trap

Container names are references for humans and commands. They are not the same thing as the immutable container ID.

### Interview Point

**Rename changes the container's human-readable name.**

---

## 198. What is `docker pause` and `docker unpause`?

### Short Interview Answer

`docker pause` suspends processes in a container, while `docker unpause` resumes them.

### Example

```bash
docker pause web
```

Resume:

```bash
docker unpause web
```

### Conceptual Model

```text
Running
   ↓
pause
   ↓
Processes suspended
   ↓
unpause
   ↓
Running
```

### Important Distinction

Paused is not the same as stopped.

```text
Stopped → process terminated
Paused  → process remains but execution is suspended
```

### Common Interview Trap

Do not use `pause` as a normal deployment shutdown strategy.

### Interview Point

**Pause suspends execution; stop terminates the container's main process.**

---

## 199. What is `docker version`?

### Short Interview Answer

`docker version` displays version information for the Docker client and, when reachable, the Docker server/daemon.

### Example

```bash
docker version
```

This can help identify:

```text
Client version
Server version
API versions
Go version
OS/architecture
```

### Why Useful?

When troubleshooting compatibility issues:

```text
CLI version
      ↕
daemon version
      ↕
API compatibility
```

can matter.

### Common Interview Trap

`docker --version` is more concise and generally shows the CLI version, while `docker version` provides client/server version information.

### Interview Point

**`docker version` is useful for diagnosing CLI/daemon version differences.**

---

## 200. What is `docker info`?

### Short Interview Answer

`docker info` displays system-wide information about the Docker environment and daemon.

### Example

```bash
docker info
```

It can provide information about:

- Docker server
- storage driver
- container/image counts
- cgroup configuration
- security options
- plugins
- runtime information
- Docker root directory

### Why Useful?

It is one of the first commands to run when diagnosing a Docker host.

### Example

```bash
docker info
```

can help determine whether the Docker daemon is reachable.

### Common Interview Trap

`docker info` is not the same as:

```bash
docker inspect <container>
```

`info` is primarily daemon/system-level information; `inspect` is object-specific.

### Interview Point

**`docker info` = Docker environment/daemon overview.**

---

## 200. How do you see the Docker daemon logs on Linux?

### Short Interview Answer

If Docker Engine is managed by systemd, use:

```bash
sudo journalctl -u docker
```

For recent entries:

```bash
sudo journalctl -u docker --since "10 minutes ago"
```

Follow live logs:

```bash
sudo journalctl -u docker -f
```

### Why Useful?

If:

```bash
docker ps
```

returns an error such as inability to connect to the daemon, daemon logs can reveal:

```text
startup failures
storage problems
permission issues
configuration errors
runtime failures
```

### Important Distinction

```text
docker logs <container>
        ↓
application/container logs

journalctl -u docker
        ↓
Docker daemon logs
```

### Common Interview Trap

Do not troubleshoot a Docker daemon failure only by looking at container logs. If the daemon cannot start, container-level commands may not work.

### Interview Point

**Know the difference between application logs and Docker daemon logs.**

---

## 201. What is `docker system df`?

### Short Interview Answer

`docker system df` shows disk usage consumed by Docker objects such as images, containers, and local volumes.

### Example

```bash
docker system df
```

For more detail:

```bash
docker system df -v
```

### Why Useful?

When disk usage is high, this command helps identify where Docker storage is being consumed.

### Conceptual View

```text
Docker disk usage
 ├── images
 ├── containers
 ├── local volumes
 └── build cache
```

### Common Interview Trap

Do not immediately run a destructive prune command without first understanding what is consuming space.

### Interview Point

**Inspect Docker disk usage before deleting resources.**

---

## 202. What are Docker prune commands?

### Short Interview Answer

Prune commands remove unused Docker objects.

Examples include:

```bash
docker container prune
docker image prune
docker network prune
docker volume prune
docker builder prune
docker system prune
```

### Example

```bash
docker container prune
```

removes stopped containers that are eligible for pruning.

```bash
docker system prune
```

removes various unused resources according to the command's rules.

### Important Warning

Prune operations can be destructive.

Before running them in a shared or production environment, understand exactly what will be removed.

### Common Interview Trap

> "`docker system prune` deletes everything."

No.

It removes resources considered unused according to the prune command's rules, not every Docker object.

### Interview Point

**Prune means cleanup of unused resources, not "delete everything."**

---

## 203. How do you find a container's IP address?

### Short Interview Answer

Use `docker inspect` to inspect its network configuration.

### Example

```bash
docker inspect web
```

Or format the output:

```bash
docker inspect \
  -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' \
  web
```

### Important Caveat

A container IP is often not the correct address for application clients to hard-code.

On user-defined Docker networks, containers should generally communicate using service/container names through Docker's embedded DNS.

### Better Pattern

Instead of:

```text
connect to 172.x.x.x
```

use:

```text
connect to database
```

when both containers are on the same appropriate user-defined network.

### Common Interview Trap

Container IPs can change when containers are recreated.

### Interview Point

**Inspect can show the IP; application configuration should generally prefer stable service/container naming.**

---

## 204. How do you enter a running container for troubleshooting?

### Short Interview Answer

Use `docker exec` with an available shell.

### Example

```bash
docker exec -it web sh
```

If Bash exists:

```bash
docker exec -it web bash
```

### Important Caveat

Minimal images may not contain Bash.

For example:

```text
bash: not found
```

does not mean the container is broken.

Try:

```bash
sh
```

if available.

### If No Shell Exists

For very minimal images, such as some `scratch`-based images, there may be no shell at all.

In that case, troubleshoot through:

```text
docker logs
docker inspect
docker stats
docker top
application metrics
external debugging/observability
```

### Interview Point

**`docker exec` is the normal interactive troubleshooting tool, but minimal images may have no shell.**

---

## 205. How do you see the environment variables of a container?

### Short Interview Answer

Use `docker inspect` to inspect the container configuration, or execute a command such as `env` inside the running container.

### Example

```bash
docker exec web env
```

Or:

```bash
docker inspect web
```

and inspect the environment configuration.

### Important Security Point

Environment variables may contain sensitive values.

Do not casually paste:

```bash
docker inspect
```

output into tickets, logs, or public channels.

### Common Interview Trap

Environment variables are not inherently secret merely because they are inside a container.

### Interview Point

**Inspect runtime configuration carefully because it may expose credentials or sensitive settings.**

---

## 206. How do you see the command with which a container was started?

### Short Interview Answer

Use:

```bash
docker inspect <container>
```

and examine the container's configuration, including its command and entrypoint.

### Example

```bash
docker inspect web
```

Useful fields include concepts corresponding to:

```text
Config.Cmd
Config.Entrypoint
```

### Why Useful?

It helps answer:

```text
What executable is configured?
What arguments were supplied?
What entrypoint is active?
```

### Common Interview Trap

The image's Dockerfile `CMD` and `ENTRYPOINT` are defaults. Runtime options can override or supplement them.

### Interview Point

**Inspect the actual container configuration rather than assuming the Dockerfile is the complete runtime configuration.**

---

## 207. How do you copy an image from one machine to another without using a registry?

### Short Interview Answer

Use `docker save` to create an image archive and `docker load` on the destination.

### Source Machine

```bash
docker save -o myapp.tar myapp:1.0
```

Transfer:

```bash
scp myapp.tar user@server:/tmp/
```

Destination:

```bash
docker load -i /tmp/myapp.tar
```

### Flow

```text
Local image
    ↓
docker save
    ↓
tar archive
    ↓
transfer
    ↓
docker load
    ↓
local image
```

### Important Distinction

This is different from:

```bash
docker export
docker import
```

which operate on a container filesystem archive rather than preserving an image's normal image metadata/layer structure.

### Interview Point

**`save/load` = image transfer; `export/import` = container filesystem transfer.**

---

## 208. How do you inspect an image's build history?

### Short Interview Answer

Use:

```bash
docker history <image>
```

### Example

```bash
docker history myapp:1.0
```

It can show information about image layers and the commands associated with those layers.

### Why Useful?

It helps investigate:

- unexpectedly large layers
- image construction
- build instructions
- layer history

### Important Caveat

Image history is useful for analysis but should not be treated as a complete security guarantee or as a substitute for proper secret handling.

### Interview Point

**`docker history` helps understand how an image was constructed layer by layer.**

---

## 209. How do you inspect an image's metadata?

### Short Interview Answer

Use:

```bash
docker image inspect <image>
```

or:

```bash
docker inspect <image>
```

### Example

```bash
docker image inspect nginx:1.29
```

Information can include:

```text
image ID
architecture
OS
config
environment
entrypoint
command
root filesystem information
labels
```

### Difference From `docker history`

```text
docker image inspect
    ↓
current image metadata/configuration

docker history
    ↓
image layer/history information
```

### Interview Point

**Inspect tells you what the image is configured to be; history helps show how it was constructed.**

---

## 210. What is the difference between `docker inspect`, `docker logs`, `docker stats`, and `docker top`?

### Short Interview Answer

They answer different troubleshooting questions:

| Command | Primary Purpose |
|---|---|
| `docker inspect` | Configuration/state/details |
| `docker logs` | Container log output |
| `docker stats` | Resource usage |
| `docker top` | Running processes |

### Troubleshooting Example

Container is unhealthy:

```text
docker inspect web
    ↓
What is configured?

docker logs web
    ↓
What is the application reporting?

docker stats web
    ↓
Is it consuming excessive resources?

docker top web
    ↓
Which processes are running?
```

### Interview Point

**Choose the command based on the question you are trying to answer.**

---

# Quick Revision

| Command | Meaning |
|---|---|
| `docker run` | Create + start new container |
| `docker create` | Create container without starting |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers |
| `docker start` | Start existing container |
| `docker stop` | Gracefully stop container |
| `docker kill` | Forcefully signal container |
| `docker restart` | Stop/start existing container |
| `docker rm` | Remove container |
| `docker rm -f` | Force-remove container |
| `docker exec` | Run new process in running container |
| `docker attach` | Attach to existing process streams |
| `docker logs` | Read container logs |
| `docker inspect` | Inspect object configuration/state |
| `docker stats` | Live resource metrics |
| `docker top` | Processes in container |
| `docker images` | List local images |
| `docker rmi` | Remove local image/reference |
| `docker pull` | Download image from registry |
| `docker push` | Upload image to registry |
| `docker tag` | Create another image reference |
| `docker cp` | Copy files between host/container |
| `docker rename` | Rename container |
| `docker pause` | Suspend container processes |
| `docker unpause` | Resume suspended processes |
| `docker version` | Client/server version information |
| `docker info` | Docker environment/daemon information |
| `docker system df` | Docker disk-usage information |
| `docker * prune` | Remove eligible unused resources |
| `docker save` | Export image to archive |
| `docker load` | Import image archive |
| `docker history` | Image layer/build history |

---

# High-Value Interview Traps

### Trap 1 — "`docker run` restarts a stopped container."

**Wrong.**

```text
docker run   → new container
docker start → existing container
```

---

### Trap 2 — "`docker ps` shows all containers."

**Wrong.**

```bash
docker ps
```

shows running containers by default.

Use:

```bash
docker ps -a
```

for all containers.

---

### Trap 3 — "`docker exec` connects to PID 1."

**Not necessarily.**

`docker exec` creates a new process inside the running container.

---

### Trap 4 — "`docker attach` starts a shell."

**Wrong.**

It attaches to the existing container process streams.

---

### Trap 5 — "`docker logs` reads every log file inside the container."

**Wrong.**

It accesses logs available through Docker's logging mechanism.

---

### Trap 6 — "`docker stop` and `docker kill` are identical."

**Wrong.**

`stop` is intended for graceful termination; `kill` is for immediate signaling.

---

### Trap 7 — "`docker rmi` removes the container."

**Wrong.**

```text
rm  → container
rmi → image
```

---

### Trap 8 — "`docker pull` starts a container."

**Wrong.**

It downloads the image.

---

### Trap 9 — "`docker tag` creates a new image build."

**Wrong.**

It creates another reference to existing image content.

---

### Trap 10 — "Container IP should be hard-coded between containers."

**Usually a bad design.**

Use Docker's network/service naming and DNS on appropriate user-defined networks.

---

### Trap 11 — "`docker system prune` deletes everything."

**Wrong.**

It removes resources considered unused according to its pruning rules.

---

### Trap 12 — "`docker exec -it container bash` always works."

**Wrong.**

Minimal images may not contain Bash.

Try:

```bash
docker exec -it container sh
```

if available.

---

# Troubleshooting Command Matrix

| Problem | First Commands |
|---|---|
| Container not running | `docker ps -a`, `docker inspect` |
| Application error | `docker logs` |
| Need shell access | `docker exec -it ... sh` |
| High CPU/memory | `docker stats` |
| Process investigation | `docker top` |
| Network/config details | `docker inspect` |
| Image unexpectedly large | `docker history`, `docker image inspect` |
| Disk space problem | `docker system df` |
| Docker daemon problem | `docker info`, `docker version`, `journalctl -u docker` |
| Need image on another host | `docker save` → transfer → `docker load` |

---

# Final Interview Answer

> **"I use Docker commands according to the object and troubleshooting question. For container lifecycle, `docker run` creates and starts a new container, while `start` operates on an existing container; `stop` provides graceful termination and `kill` is more forceful. For troubleshooting, I use `docker logs` for application output, `docker inspect` for configuration and state, `docker stats` for resource usage, `docker top` for processes, and `docker exec` to run a diagnostic process inside a running container. For images, I use `pull`, `tag`, `push`, `history`, and `inspect`, while `save` and `load` are useful for moving images without a registry."**

---

# One-Line Memory Map

```text
CONTAINER
run → start → exec/logs/inspect/stats/top → stop → rm

IMAGE
pull → inspect/history → tag → push

TROUBLESHOOT
logs   = what happened
inspect = what is configured
stats   = how much resource
top     = which processes
exec    = run a command inside
```
