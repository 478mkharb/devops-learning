# Docker Interview Preparation — Topic 16: Docker Container Lifecycle

> **Interview focus:** Understand exactly how a Docker container is created, started, stopped, restarted, paused, killed, removed, and recreated — and how container state differs from the application process and image.

---

## Q396. What is the Docker container lifecycle?

### Short Interview Answer

A Docker container moves through states such as:

```text
Created
   ↓
Running
   ↓
Stopped / Exited
   ↓
Removed
```

It can also be:

```text
Running
   ↓
Paused
   ↓
Running
```

and:

```text
Running
   ↓
Restarting
   ↓
Running / Exited
```

A simplified lifecycle is:

```text
docker create
      ↓
   Created
      ↓
docker start
      ↓
   Running
      │
      ├── docker stop → Exited
      │
      ├── docker pause → Paused → docker unpause
      │
      └── docker kill → Exited
                         │
                         ↓
                    docker rm
                         ↓
                      Removed
```

### Important distinction

A **container** is an object with configuration and a writable filesystem layer.

The **main process** running inside that container can exit while the container object remains in an exited state.

### Interview Point

> Container lifecycle and application-process lifecycle are related, but they are not identical.

---

## Q397. What happens when you run `docker create`?

### Short Interview Answer

`docker create` creates a new container from an image but does **not** start its main process.

Example:

```bash
docker create --name web nginx
```

Now:

```bash
docker ps
```

may not show it because it is not running.

Use:

```bash
docker ps -a
```

and you should see:

```text
web
Created
```

Then start it:

```bash
docker start web
```

### Conceptual flow

```text
Image
  ↓
docker create
  ↓
Container object
  ↓
Created state
  ↓
docker start
  ↓
Running
```

### Interview Point

> `docker create` creates the container; `docker start` starts an existing container.

---

## Q398. What is the difference between `docker create` and `docker run`?

### Short Interview Answer

`docker run` normally combines container creation and starting it, while `docker create` only creates the container.

### `docker create`

```bash
docker create --name app myapp:1.0
```

Conceptually:

```text
create only
```

### `docker run`

```bash
docker run --name app myapp:1.0
```

Conceptually:

```text
create
  +
start
```

### Equivalent conceptual flow

```bash
docker create --name app myapp:1.0
docker start app
```

is broadly equivalent to:

```bash
docker run --name app myapp:1.0
```

for the basic create/start lifecycle.

### Interview Point

> `run` is the convenient create-and-start operation; `create` separates those lifecycle stages.

---

## Q399. What happens when you run `docker start`?

### Short Interview Answer

`docker start` starts an existing stopped container. It does not create a new container.

Example:

```bash
docker start app
```

If:

```text
app → Exited
```

it becomes:

```text
app → Running
```

### Important distinction

```bash
docker run app
```

creates a new container.

```bash
docker start app
```

starts the existing container named `app`.

### Interview Trap

If a container exits and you want to run the same container again:

```bash
docker start app
```

Do not automatically use `docker run`, because that creates another container.

### Interview Point

> `start` operates on an existing container object.

---

## Q400. What happens when the main process inside a container exits?

### Short Interview Answer

The container's main process, normally PID 1 inside the container's PID namespace, has exited. For a normal container, Docker then considers the container stopped/exited.

The container object still exists unless it was configured for automatic removal.

### Example

```bash
docker run --name test alpine sh -c 'echo hello'
```

The command finishes immediately.

Then:

```bash
docker ps -a
```

shows:

```text
test    Exited (...)
```

### Important distinction

```text
Process exits
      ↓
Container becomes Exited
      ↓
Container object still exists
```

The container is not automatically deleted unless something such as:

```bash
--rm
```

was used.

### Interview Point

> Process exit normally changes the container state to exited; it does not automatically delete the container.

---

## Q401. What is the difference between a stopped container and a removed container?

### Short Interview Answer

A stopped container still exists and can be inspected or restarted. A removed container no longer exists as a Docker container object.

### Stopped

```bash
docker stop app
```

Then:

```bash
docker start app
```

works.

### Removed

```bash
docker rm app
```

After removal:

```bash
docker start app
```

fails because the container no longer exists.

### Lifecycle

```text
Running
   ↓
stop
   ↓
Exited
   ↓
start
   ↓
Running

or:

Exited
   ↓
rm
   ↓
Removed
```

### Interview Point

> `stop` changes state; `rm` destroys the container object.

---

# Stop, Kill, and Signals

## Q402. What is `docker stop`?

### Short Interview Answer

`docker stop` requests graceful termination of the container's main process.

Conceptually:

```text
docker stop
    ↓
send configured stop signal
    ↓
wait for grace period
    ↓
if still running
    ↓
forceful termination
```

The default stop signal is normally `SIGTERM`, and Docker waits for a configurable grace period before sending `SIGKILL` if the process has not exited.

Example:

```bash
docker stop app
```

Specify timeout:

```bash
docker stop --time=30 app
```

### Why graceful shutdown matters

Applications can:

- Finish requests.
- Close connections.
- Flush buffers.
- Commit state.
- Clean up resources.

### Interview Point

> `docker stop` is the preferred normal shutdown operation because it allows graceful termination.

---

## Q403. What is `docker kill`?

### Short Interview Answer

`docker kill` sends a signal directly to the container's main process, with `SIGKILL` being the default.

Example:

```bash
docker kill app
```

This is forceful:

```text
docker kill
    ↓
SIGKILL
    ↓
process terminated
```

You can specify another signal:

```bash
docker kill --signal=SIGTERM app
```

### Difference from stop

```text
docker stop
    ↓
graceful shutdown
    ↓
wait
    ↓
SIGKILL if necessary

docker kill
    ↓
signal immediately
```

### Interview Point

> Use `kill` when you need immediate termination or when graceful shutdown is not working.

---

## Q404. What is the difference between `docker stop` and `docker kill`?

| | `docker stop` | `docker kill` |
|---|---|---|
| Purpose | Graceful shutdown | Immediate/forced signaling |
| Default initial signal | SIGTERM | SIGKILL |
| Grace period | Yes | No normal graceful wait |
| Application cleanup | Usually possible | SIGKILL prevents cleanup |
| Typical use | Normal operations | Hung/unresponsive process |

### Interview Answer

> "I use `docker stop` for normal shutdown because it gives the application an opportunity to handle termination. I use `docker kill` when the process is stuck or I need immediate termination."

---

## Q405. What is `STOPSIGNAL` in Docker?

### Short Interview Answer

`STOPSIGNAL` specifies which signal Docker should use when stopping a container.

Dockerfile:

```dockerfile
FROM nginx:alpine

STOPSIGNAL SIGQUIT
```

Now:

```bash
docker stop container
```

uses the configured stop signal rather than relying only on the default.

### Why useful?

Different applications may have different preferred graceful-shutdown signals.

### Important distinction

`STOPSIGNAL` does not make an application handle a signal correctly. The application itself must support the chosen signal.

### Interview Point

> `STOPSIGNAL` defines the signal Docker uses to request graceful shutdown.

---

# Restarting Containers

## Q406. What does `docker restart` do?

### Short Interview Answer

`docker restart` stops a container and starts it again.

Example:

```bash
docker restart app
```

Conceptually:

```text
Running
   ↓
stop
   ↓
Exited
   ↓
start
   ↓
Running
```

### Important point

It does not create a new container object.

The same container identity and writable layer remain.

### Interview Point

> Restarting is not the same as recreating a container.

---

## Q407. What is the difference between restarting and recreating a container?

### Short Interview Answer

A restart stops and starts the same container. Recreating means removing the old container and creating a new one.

### Restart

```text
Container A
   ↓ stop
Container A
   ↓ start
Container A
```

### Recreate

```text
Container A
   ↓ remove
new container
   ↓
Container B
```

### Why this matters

A restart preserves the container's:

- Container ID.
- Writable layer.
- Container metadata.

A new container gets a new identity and a new writable layer.

Mounted volumes can still preserve data across recreation.

### Interview Point

> Restart preserves the container; recreation replaces the container.

---

## Q408. Does restarting a container reset its filesystem?

### Short Interview Answer

No. Restarting the same container does not normally reset its writable layer.

Example:

```bash
docker run -d --name app alpine sh -c \
  'echo data > /data.txt; sleep 10000'
```

Restart:

```bash
docker restart app
```

The same container's writable layer remains.

### However

Removing and recreating the container:

```bash
docker rm -f app
```

causes its writable layer to be removed.

### Interview Point

> Restart does not reset container storage; recreation creates a new writable layer.

---

## Q409. What is a Docker restart policy?

### Short Interview Answer

A restart policy tells Docker when it should automatically restart a container.

Common policies include:

```text
no
on-failure
always
unless-stopped
```

Example:

```bash
docker run -d \
  --restart=unless-stopped \
  --name app \
  myapp
```

### Policies

#### `no`

Default behavior:

```text
Do not automatically restart.
```

#### `on-failure`

Restart when the container exits with a non-zero status.

Example:

```bash
--restart=on-failure:5
```

means Docker can retry up to five times.

#### `always`

Docker attempts to restart the container whenever it stops, subject to Docker's restart-policy behavior.

#### `unless-stopped`

Similar to `always`, but respects an explicit administrative stop across Docker daemon restart scenarios.

### Interview Point

> Restart policies provide basic container restart behavior; they are not a full application orchestrator.

---

## Q410. What is the difference between `always` and `unless-stopped`?

### Short Interview Answer

Both automatically restart containers, but `unless-stopped` remembers an explicit administrative stop across Docker daemon restarts, whereas `always` is more aggressive about restarting.

### Conceptually

```text
always
  ↓
restart whenever stopped
```

```text
unless-stopped
  ↓
restart unless explicitly stopped
```

### Interview Point

> `unless-stopped` is useful when an administrator intentionally stopped the container and does not want Docker to automatically bring it back after a daemon restart.

---

## Q411. What does `on-failure` mean?

### Short Interview Answer

`on-failure` tells Docker to restart the container when its main process exits with a non-zero exit code.

Example:

```bash
docker run -d \
  --restart=on-failure:3 \
  myapp
```

Conceptually:

```text
Application exits
      ↓
exit code?
      ↓
non-zero
      ↓
restart
```

If the application exits successfully with:

```text
exit code 0
```

the restart policy does not normally restart it based on `on-failure`.

### Interview Point

> `on-failure` is intended for failed process exits, not successful completion.

---

# Automatic Removal

## Q412. What does `docker run --rm` do?

### Short Interview Answer

`--rm` tells Docker to automatically remove the container when it exits.

Example:

```bash
docker run --rm alpine echo hello
```

After the process exits:

```text
container
   ↓
automatically removed
```

### Useful for

- One-off commands.
- Temporary build/test containers.
- Interactive troubleshooting environments.
- CI jobs where container persistence is unnecessary.

### Important caveat

Do not use `--rm` when you need to inspect the stopped container later.

### Interview Point

> `--rm` makes the container lifecycle temporary by automatically deleting the container after exit.

---

## Q413. What happens to volumes when a `--rm` container exits?

### Short Interview Answer

A named volume has an independent lifecycle and is not simply equivalent to the container writable layer.

For example:

```bash
docker volume create appdata

docker run --rm \
  -v appdata:/data \
  alpine sh -c 'echo hello > /data/file'
```

The temporary container is removed, but:

```bash
docker volume ls
```

still shows `appdata`.

### Important distinction

```text
--rm
  ↓
container automatically removed

named volume
  ↓
normally remains
```

### Interview Point

> `--rm` removes the container; persistent storage has its own lifecycle.

---

# Pause and Unpause

## Q414. What does `docker pause` do?

### Short Interview Answer

`docker pause` suspends processes in a container using the Linux freezer/cgroup mechanism where supported, without normally terminating them.

Example:

```bash
docker pause app
```

State becomes:

```text
Paused
```

Resume:

```bash
docker unpause app
```

### Conceptual difference

```text
stop
  ↓
processes terminate

pause
  ↓
processes remain but execution is suspended
```

### Interview Point

> Pause suspends execution; stop terminates the container's main process gracefully.

---

## Q415. What happens to application connections when a container is paused?

### Short Interview Answer

The processes are suspended, so application activity stops while paused. Existing network connections may remain established at the networking layer, but the application cannot process traffic while its processes are frozen.

When the container is unpaused, processes continue from their previous state.

### Interview Point

> Pause is not a graceful shutdown; it freezes the workload.

---

# Container Identity

## Q416. Does restarting a container create a new container ID?

### Short Interview Answer

No.

A restart operates on the same container object, so the container ID remains the same.

### Example

```bash
docker inspect --format '{{.Id}}' app
```

Record the ID.

Then:

```bash
docker restart app
```

Inspect again:

```bash
docker inspect --format '{{.Id}}' app
```

The ID remains the same.

### Recreate

If you:

```bash
docker rm -f app
docker run --name app myapp
```

the new container receives a new ID.

### Interview Point

> Restart preserves container identity; recreation does not.

---

## Q417. Does recreating a container create a new writable layer?

### Short Interview Answer

Yes.

A newly created container gets its own container-specific writable layer.

### Example

```text
Container A
   ↓
writable layer A
```

Remove it:

```bash
docker rm A
```

Create another:

```text
Container B
   ↓
writable layer B
```

The old writable layer does not become the new container's writable layer.

### Persistent volume

If both containers mount:

```text
appdata:/data
```

the persistent data can remain available through the volume.

### Interview Point

> Recreating a container replaces its writable layer; volumes can preserve application data independently.

---

# Container Configuration

## Q418. What container configuration is fixed at creation time?

### Short Interview Answer

Many important container settings are established when the container is created, such as:

- Image reference/configuration.
- Environment variables.
- Mounts.
- Network attachments/configuration.
- Port publishing.
- Resource limits.
- Restart policy.
- Entrypoint/command configuration.

This is why changing these settings commonly requires recreating the container.

### Example

Suppose a container was created with:

```bash
docker run -d \
  --name app \
  -p 8080:80 \
  myapp
```

Changing the published port generally means creating a new container with the desired mapping rather than editing the existing container in place.

### Interview Point

> Docker containers are largely immutable runtime objects; changing creation-time configuration generally means recreation.

---

## Q419. Can you change a container's image after it has been created?

### Short Interview Answer

No, not in the normal Docker container lifecycle model.

A container is created from a particular image configuration. To use a new image version, create a new container from that image.

### Example

Old:

```text
myapp:1.0
   ↓
container A
```

New:

```text
myapp:2.0
   ↓
container B
```

Typical deployment:

```bash
docker stop app
docker rm app
docker run --name app myapp:2.0
```

If persistent data exists:

```text
volume
  ↓
new container
```

can reuse it.

### Interview Point

> Updating an application image normally means replacing the container, not mutating its image in place.

---

## Q420. Why is container recreation common in Docker deployments?

### Short Interview Answer

Containers are designed to be replaceable. Instead of modifying a running container's immutable image, you deploy a new container from the desired image and configuration.

### Deployment model

```text
Old image
   ↓
Old container
   ↓
remove/replace
   ↓
New image
   ↓
New container
```

Persistent state lives separately:

```text
New container
      │
      ▼
same persistent volume
```

### Benefits

- Predictable deployments.
- Easy rollback.
- Immutable artifacts.
- Less configuration drift.
- Cleaner CI/CD.

### Interview Point

> Replace rather than mutate is a core container deployment principle.

---

# Inspecting Lifecycle State

## Q421. How do you check a container's current state?

### Short Interview Answer

Use:

```bash
docker ps
```

for running containers, or:

```bash
docker ps -a
```

for all containers.

For detailed state:

```bash
docker inspect <container>
```

Useful fields include:

```text
State.Status
State.Running
State.Paused
State.Restarting
State.ExitCode
State.OOMKilled
State.StartedAt
State.FinishedAt
```

### Example

```bash
docker inspect \
  --format '{{.State.Status}}' \
  app
```

### Interview Point

> `docker ps` gives a quick lifecycle view; `docker inspect` provides detailed state information.

---

## Q422. How do you find why a container exited?

### Strong Interview Answer

I would inspect the container state, exit code, logs, and application behavior.

### Step 1 — Check state

```bash
docker inspect app
```

Look at:

```text
ExitCode
Error
FinishedAt
OOMKilled
```

### Step 2 — Check logs

```bash
docker logs app
```

### Step 3 — Identify the command

```bash
docker inspect \
  --format '{{json .Config.Cmd}}' \
  app
```

and:

```bash
docker inspect \
  --format '{{json .Config.Entrypoint}}' \
  app
```

### Step 4 — Check restart policy

```bash
docker inspect \
  --format '{{json .HostConfig.RestartPolicy}}' \
  app
```

### Step 5 — Consider external causes

- OOM.
- Signal termination.
- Application configuration.
- Missing dependency.
- Failed health logic.
- Permission problems.
- Host resource pressure.

### Interview Point

> An exited container is a symptom; the exit code, logs, signals, and configuration identify the cause.

---

# Exit Codes and Signals

## Q423. What does a container exit code mean?

### Short Interview Answer

The container's exit code normally reflects how its main process terminated.

Conventionally:

```text
0
```

means successful completion.

Non-zero values usually indicate an error or signal-related termination.

### Signal convention

When a process is terminated by signal `N`, the conventional shell exit status is often:

```text
128 + N
```

Examples:

```text
SIGTERM = 15 → 143
SIGKILL = 9  → 137
```

### Important caveat

The exit code is evidence, not always a complete explanation of the failure.

### Interview Point

> Exit codes tell you how the main process ended, but you should correlate them with logs and container state.

---

## Q424. What does exit code 0 mean for a container?

### Short Interview Answer

It normally means the container's main process exited successfully.

Example:

```bash
docker run --name test alpine echo hello
```

The command completes:

```text
ExitCode = 0
```

The container is still shown as:

```text
Exited (0)
```

### Important point

`Exited (0)` does not mean:

```text
container is still running
```

It means:

```text
process completed successfully
```

### Interview Point

> A successful process can still leave the container in the Exited state.

---

## Q425. What does exit code 143 usually indicate?

### Short Interview Answer

143 commonly corresponds to:

```text
128 + 15
```

where signal 15 is:

```text
SIGTERM
```

This is commonly seen when a process receives a graceful termination request, such as during:

```bash
docker stop
```

### Important nuance

The exact application behavior and wrapper processes matter, so use the exit code together with logs and container state.

### Interview Point

> Exit 143 is commonly associated with SIGTERM-based termination.

---

## Q426. What does exit code 137 usually indicate?

### Short Interview Answer

137 commonly corresponds to:

```text
128 + 9
```

where signal 9 is:

```text
SIGKILL
```

A common cause is an OOM kill.

However:

> **137 does not prove OOM.**

It can also occur when something else sends SIGKILL.

Verify:

```bash
docker inspect app
```

and check:

```text
OOMKilled
```

### Interview Point

> Exit 137 means SIGKILL at the conventional exit-status level; OOM is a common cause but must be verified.

---

# Health and Lifecycle

## Q427. Does a Docker healthcheck restart a failed container automatically?

### Short Interview Answer

No. A `HEALTHCHECK` reports application health; it does not by itself restart an unhealthy container.

A container can be:

```text
Running
Health = unhealthy
```

while its main process is still alive.

### Example

```dockerfile
HEALTHCHECK CMD curl -f http://localhost:8080/health || exit 1
```

Possible state:

```text
Container = Running
Health    = unhealthy
```

Docker does not automatically equate "unhealthy" with "stopped."

### Interview Point

> Health status and container lifecycle state are separate concepts.

---

## Q428. What is the difference between a container being unhealthy and being stopped?

### Short Interview Answer

**Unhealthy** means the configured healthcheck is failing while the container may still be running.

**Stopped** means the container's main process has exited.

### Example

```text
Container A
  Running
  Unhealthy

Container B
  Exited
  ExitCode 1
```

These require different troubleshooting.

### Interview Point

> A failed healthcheck does not necessarily mean the main process has exited.

---

# Practical Lifecycle Scenarios

## Q429. A container exited. Should you use `docker start` or `docker run`?

### Strong Interview Answer

If I want to start the same existing container with the same creation-time configuration:

```bash
docker start <container>
```

If I intentionally want a new container with different configuration or a different image:

```bash
docker run ...
```

### Example

Existing:

```bash
docker start app
```

New image:

```bash
docker run --name app-v2 myapp:2.0
```

### Interview Point

> Use `start` to reuse; use `run` to create.

---

## Q430. A production container is running the wrong image version. How would you update it?

### Strong Interview Answer

I would not modify the image inside the existing container.

I would:

1. Pull the desired image.
2. Verify its tag/digest.
3. Prepare the required configuration and persistent storage.
4. Stop/remove or replace the old container according to the deployment strategy.
5. Start the new container.
6. Verify health and application behavior.
7. Keep rollback capability.

Example:

```bash
docker pull registry.example.com/team/app:2.0
```

Then:

```text
old container
      ↓
new container from app:2.0
      ↓
same persistent volume if required
```

### Production improvement

For critical services, prefer a controlled deployment strategy rather than a manual stop/remove sequence.

### Interview Point

> Image upgrades are normally container replacement operations.

---

## Q431. A container keeps restarting. How would you troubleshoot it?

### Strong Interview Answer

I would determine whether the restart is caused by the application exiting, an OOM condition, a restart policy, or another lifecycle issue.

### Step 1 — Check status

```bash
docker ps -a
```

Look for:

```text
Restarting
Exited
```

### Step 2 — Check logs

```bash
docker logs --tail=200 app
```

### Step 3 — Inspect state

```bash
docker inspect app
```

Check:

```text
ExitCode
OOMKilled
RestartCount
Error
```

### Step 4 — Check restart policy

```bash
docker inspect \
  --format '{{json .HostConfig.RestartPolicy}}' \
  app
```

### Step 5 — Check resources

```bash
docker stats app
```

Look for:

```text
memory pressure
CPU pressure
```

### Step 6 — Check application startup

Common causes:

```text
bad configuration
missing secret
missing dependency
port conflict
permission problem
application crash
OOM
```

### Interview Point

> "Restarting" is a symptom; inspect the underlying exit reason and restart policy.

---

## Q432. What happens if the Docker daemon restarts while containers are running?

### Short Interview Answer

The exact behavior depends on Docker configuration and restart policies, but Docker can restore/restart containers according to their configured restart policy after the daemon becomes available again.

A container's filesystem and metadata are not normally erased merely because the daemon restarts.

### Example

```text
Docker daemon
      ↓
restart
      ↓
daemon unavailable temporarily
      ↓
daemon starts
      ↓
containers reconciled according to configuration
```

### Important distinction

```text
Docker daemon restart
≠
container deletion
```

### Interview Point

> Restarting `dockerd` is different from stopping or deleting containers.

---

## Q433. What happens if the host machine reboots?

### Short Interview Answer

Containers stop when the host goes down. When Docker starts again, containers with appropriate restart policies can be started automatically.

For example:

```bash
--restart=unless-stopped
```

can be used for workloads intended to return after Docker/host recovery.

### Important distinction

Persistent data on a Docker volume is separate from the container lifecycle and normally remains across host reboots.

### Interview Point

> Host reboot stops running containers temporarily; restart policy determines automatic recovery after Docker comes back.

---

## Q434. Does deleting a container delete its image?

### Short Interview Answer

No.

A container and its image are separate Docker objects.

```bash
docker rm app
```

removes the container.

The image remains:

```bash
docker image ls
```

### Example

```text
Image: myapp:1.0
   │
   ├── Container A
   └── Container B
```

Removing Container A does not delete the image.

### Interview Point

> Container deletion and image deletion are independent operations.

---

## Q435. Does deleting a container delete its named volume?

### Short Interview Answer

Normally, no.

For example:

```bash
docker volume create appdata

docker run --name app \
  -v appdata:/data \
  myapp
```

Then:

```bash
docker rm app
```

does not normally remove:

```text
appdata
```

The volume must be explicitly removed:

```bash
docker volume rm appdata
```

### Interview Point

> Container, image, and volume lifecycles are separate.

---

# Lifecycle Command Map

## Q436. What are the most important Docker lifecycle commands?

| Command | Purpose |
|---|---|
| `docker create` | Create stopped container |
| `docker run` | Create + start container |
| `docker start` | Start existing container |
| `docker stop` | Gracefully stop container |
| `docker kill` | Forcefully signal container |
| `docker restart` | Stop + start same container |
| `docker pause` | Suspend container processes |
| `docker unpause` | Resume paused processes |
| `docker rm` | Remove container |
| `docker rm -f` | Force-remove running container |
| `docker inspect` | Inspect lifecycle/configuration |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers |
| `docker logs` | View container logs |

### Interview Point

> Know what each lifecycle command does to the container object, its process, and its writable layer.

---

# Lifecycle Deep-Dive Scenario

## Q437. Explain this sequence:

```bash
docker run --name app myapp
docker stop app
docker start app
docker restart app
docker rm app
```

### Step-by-step

### 1. `docker run`

```text
Image
 ↓
create container
 ↓
start process
 ↓
Running
```

### 2. `docker stop`

```text
Running
 ↓
SIGTERM / graceful shutdown
 ↓
Exited
```

The container object remains.

### 3. `docker start`

```text
Exited
 ↓
same container starts again
 ↓
Running
```

### 4. `docker restart`

```text
Running
 ↓
stop
 ↓
Exited
 ↓
start
 ↓
Running
```

Still the same container.

### 5. `docker rm`

```text
Running
 ↓
rm
 ↓
Removed
```

If necessary, `docker rm -f` can stop/remove a running container forcefully.

### Final state

```text
Container no longer exists
```

The image still exists unless separately removed.

Any named volume remains unless separately removed.

### Interview Point

> This sequence tests whether you understand container state transitions versus image and volume lifecycles.

---

# Quick Revision

| Concept | Key Point |
|---|---|
| `create` | Creates container without starting it |
| `run` | Create + start |
| `start` | Starts existing container |
| Process exit | Container normally becomes Exited |
| `stop` | Graceful shutdown |
| `kill` | Immediate signal, SIGKILL by default |
| `restart` | Stop + start same container |
| Recreate | Remove old + create new container |
| `--rm` | Automatically remove container after exit |
| `pause` | Freeze processes |
| `unpause` | Resume processes |
| Restart policy | Controls automatic restart |
| `on-failure` | Restart after non-zero exit |
| `always` | Aggressive automatic restart behavior |
| `unless-stopped` | Restart unless explicitly stopped |
| Container ID | Preserved across restart |
| Recreated container | New container ID |
| Writable layer | Preserved across restart, removed with container |
| Volume | Separate lifecycle from container |
| Image | Separate lifecycle from container |
| `inspect` | Detailed state/configuration |
| Exit 0 | Successful process completion |
| Exit 143 | Commonly SIGTERM |
| Exit 137 | SIGKILL; OOM common but not proof |
| Healthcheck | Reports health, does not itself restart |
| Docker daemon restart | Does not inherently delete containers |
| Host reboot | Containers stop; restart policy can recover them |

---

# High-Value Interview Traps

### Trap 1 — "`docker start` creates a new container"

**Incorrect.**

It starts an existing container.

---

### Trap 2 — "`docker restart` creates a new container"

**Incorrect.**

It stops and starts the same container.

---

### Trap 3 — "Container exit means container deletion"

**Incorrect.**

The container normally remains in the Exited state.

---

### Trap 4 — "Restarting resets the writable filesystem"

**Incorrect.**

The same container and writable layer remain.

---

### Trap 5 — "Recreating the container preserves the writable layer"

**Incorrect.**

A new container gets a new writable layer.

Persistent volumes are the mechanism used to preserve important data.

---

### Trap 6 — "Exit 137 always means OOM"

**Incorrect.**

137 conventionally means SIGKILL. OOM is a common cause but must be verified.

---

### Trap 7 — "Unhealthy means stopped"

**Incorrect.**

A container can be:

```text
Running + Unhealthy
```

---

### Trap 8 — "`docker stop` immediately kills the process"

**Incorrect.**

It normally requests graceful termination first and waits before forceful termination.

---

### Trap 9 — "`docker pause` gracefully stops the application"

**Incorrect.**

Pause freezes processes; it does not perform a graceful shutdown.

---

### Trap 10 — "Deleting a container deletes its named volume"

**Incorrect.**

The volume normally has an independent lifecycle.

---

### Trap 11 — "Deleting a container deletes its image"

**Incorrect.**

Images and containers are separate Docker objects.

---

### Trap 12 — "Restart policy is the same as orchestration"

**Incorrect.**

Restart policies provide basic local container restart behavior. They do not provide the full scheduling, service discovery, rollout, scaling, and reconciliation capabilities of an orchestrator.

---

# High-Value Interview Follow-Up Questions

1. Explain the complete Docker container lifecycle.
2. `docker create` vs `docker run`?
3. `docker start` vs `docker run`?
4. What happens when PID 1 exits?
5. What is the Exited state?
6. `docker stop` vs `docker kill`?
7. What signal does `docker stop` normally use?
8. What is `STOPSIGNAL`?
9. What is a Docker restart policy?
10. `always` vs `unless-stopped`?
11. What does `on-failure` mean?
12. What does `--rm` do?
13. What happens to a named volume when a `--rm` container exits?
14. What does `docker pause` do?
15. Pause vs stop?
16. Does restart create a new container ID?
17. Does recreation create a new writable layer?
18. What configuration normally requires container recreation?
19. Can you change a container's image in place?
20. Why are containers normally replaced rather than modified?
21. How do you find why a container exited?
22. What does exit code 0 mean?
23. What does exit 143 mean?
24. What does exit 137 mean?
25. How do you verify an OOM kill?
26. Does a healthcheck restart an unhealthy container?
27. Unhealthy vs exited?
28. What happens when Docker daemon restarts?
29. What happens when the host reboots?
30. What happens to volumes when a container is deleted?
31. What happens to images when a container is deleted?
32. How would you troubleshoot a continuously restarting container?
33. How would you safely upgrade a production container to a new image?
34. Restart vs recreate?
35. Why is container recreation fundamental to immutable deployments?

---

# Final Interview Answer

If asked **"Explain the Docker container lifecycle"**, a strong answer is:

> "A Docker container is created from an image and can move through states such as Created, Running, Paused, and Exited before being Removed. `docker create` creates the container without starting it, while `docker run` normally creates and starts it. `docker start` starts an existing stopped container. For normal shutdown I use `docker stop`, which sends the configured stop signal and allows a grace period before forceful termination if necessary. `docker kill` sends a signal directly, with SIGKILL as the default. `docker restart` stops and starts the same container, so the container ID and writable layer remain. Recreating a container is different: the old container is removed and a new container with a new writable layer and ID is created. Persistent application data should therefore be stored in volumes or other external storage rather than the writable layer. Docker restart policies can automatically restart containers, while healthchecks only report application health and do not themselves restart unhealthy containers."

---

# One-Line Memory Map

```text
create   = make container
run      = create + start
start    = start existing
stop     = graceful shutdown
kill     = forceful signal
restart  = stop + start same container
pause    = freeze processes
rm       = remove container
--rm     = remove automatically after exit
restart policy = automatic recovery
healthcheck    = health status, not restart
volume         = persistent data
writable layer = container-specific lifecycle data
```
