# Docker Interview Preparation — Topic 12: Docker Storage

> **Interview focus:** Understand where container data lives, why container-layer data is ephemeral, when to use volumes/bind mounts/tmpfs, how storage is shared, and how to troubleshoot persistence and permission problems.

---

## Q265. What is Docker container storage?

### Short Interview Answer

Docker containers have a writable filesystem layer on top of the read-only image layers. This writable layer is part of the container's lifecycle, so data written there is generally ephemeral.

For data that must survive container deletion/recreation, Docker provides storage mechanisms such as:

- **Named/anonymous volumes**
- **Bind mounts**
- **tmpfs mounts**

### Detailed Explanation

Conceptually:

```text
Docker Image
┌──────────────────────────────┐
│ Read-only image layers       │
├──────────────────────────────┤
│ Container writable layer     │
└──────────────────────────────┘
```

A container normally gets a thin writable layer when it is created. Application writes go there unless a path is backed by a mount.

For example:

```bash
docker run --name app nginx
```

If the application writes:

```text
/app/data/file.txt
```

and `/app/data` is not mounted, that file belongs to the container's writable layer.

For persistent application data:

```bash
docker volume create app-data
docker run -v app-data:/app/data myapp
```

Now `/app/data` is backed by the Docker volume rather than the container writable layer.

### Why it matters

A production application should not normally depend on the container writable layer for important state.

### Common Interview Trap

**Wrong:** "Containers cannot write to their filesystem."

They can. The important distinction is that the default writable layer is **not the appropriate persistence mechanism for important application data**.

### Interview Point

> Container storage exists at different layers; persistence should be designed explicitly.

---

## Q266. What is the writable container layer?

### Short Interview Answer

The writable container layer is the top, writable layer added to a container's read-only image layers. Changes made inside the container that are not written through a mounted volume, bind mount, or tmpfs normally go into this layer.

### Detailed Explanation

An image can be viewed conceptually as:

```text
Image
┌─────────────────────┐
│ Application files   │
├─────────────────────┤
│ Runtime packages    │
├─────────────────────┤
│ Base OS files       │
└─────────────────────┘
        ↓ read-only
┌─────────────────────┐
│ Container writable  │
│ layer                │
└─────────────────────┘
        ↑
      writes
```

With copy-on-write storage, if a process modifies an existing file from an image layer, Docker's storage system can copy the file into the writable layer and apply the modification there.

### Example

```bash
docker run -it ubuntu bash
echo "hello" > /tmp/test.txt
```

The file is written to the container's writable storage unless `/tmp` is backed by another mount.

### Interview Point

> The writable layer belongs to the container, not to the immutable image.

---

## Q267. Why is the container writable layer considered ephemeral?

### Short Interview Answer

Because it is tied to the specific container. Removing that container removes its writable layer and therefore the data stored there.

### Example

```bash
docker run --name test ubuntu sh -c 'echo hello > /data.txt'
```

The file exists while that container exists:

```bash
docker exec test cat /data.txt
```

If the container is removed:

```bash
docker rm -f test
```

and a new container is created:

```bash
docker run --name test ubuntu cat /data.txt
```

the file will not be there.

### Important distinction

`docker stop` does **not** remove the container.

```text
stop
  ↓
container remains
  ↓
writable layer remains
```

But:

```text
rm
  ↓
container removed
  ↓
writable layer removed
```

### Common Interview Trap

Do not say:

> "Restarting a container deletes its data."

A restart normally keeps the same container and its writable layer.

### Interview Point

> Ephemeral means tied to the container lifecycle, not that data disappears whenever the process stops.

---

## Q268. What happens to the writable layer when a container is removed?

### Short Interview Answer

The container's writable layer is removed with the container, unless the data was stored in a separate persistent storage mechanism such as a volume or bind mount.

### Example

```bash
docker run --name app ubuntu sh -c 'echo data > /data.txt'
docker rm app
```

`/data.txt` was part of the container's writable layer and is lost.

But:

```bash
docker volume create appdata

docker run --name app \
  -v appdata:/data \
  ubuntu sh -c 'echo data > /data/file.txt'

docker rm app
```

The volume remains:

```bash
docker volume ls
```

A new container can reuse it:

```bash
docker run --rm \
  -v appdata:/data \
  ubuntu cat /data/file.txt
```

### Interview Point

> Container deletion and volume deletion are separate lifecycle events.

---

# Docker Volumes

## Q269. What is a Docker volume?

### Short Interview Answer

A Docker volume is a Docker-managed persistent storage object that can be mounted into one or more containers.

### Create a volume

```bash
docker volume create app-data
```

### Use it

```bash
docker run -d \
  --name app \
  --mount source=app-data,target=/app/data \
  myapp
```

or:

```bash
docker run -d \
  --name app \
  -v app-data:/app/data \
  myapp
```

### Why volumes are useful

Volumes are appropriate when:

- Data must survive container replacement.
- Docker should manage the storage lifecycle.
- Multiple containers need shared data.
- A database needs persistent storage.
- You want to avoid coupling the container to a particular host directory.

### Interview Point

> A volume is separate from the container's writable layer and normally survives container removal.

---

## Q270. Why use Docker volumes?

### Short Interview Answer

Use volumes when application data must outlive a container and you want Docker to manage the storage rather than directly managing a host directory.

### Example

Suppose PostgreSQL stores its data at:

```text
/var/lib/postgresql/data
```

Run it with a volume:

```bash
docker volume create pgdata

docker run -d \
  --name postgres \
  -v pgdata:/var/lib/postgresql/data \
  postgres
```

If the PostgreSQL container is replaced:

```bash
docker rm -f postgres
```

the volume remains:

```bash
docker volume ls
```

A replacement container can mount the same volume.

### Why this is better than the writable layer

```text
Container A
     │
     ├── writable layer → disposable
     │
     └── pgdata volume → persistent
```

### Interview Point

> Volumes decouple persistent data from the lifecycle of an individual container.

---

## Q271. What is the difference between a named volume and an anonymous volume?

### Short Interview Answer

A **named volume** has an explicit name that you choose and can easily reuse. An **anonymous volume** is created without a user-defined name and is usually referenced by its generated identifier.

### Named volume

```bash
docker volume create mydata

docker run -v mydata:/data alpine
```

You know exactly which volume to reuse:

```bash
docker run -v mydata:/data alpine
```

### Anonymous volume

```bash
docker run -v /data alpine
```

Docker creates a volume automatically for `/data`.

### Comparison

| Feature | Named | Anonymous |
|---|---|---|
| User-defined name | Yes | No |
| Easy reuse | Yes | Less convenient |
| Explicit lifecycle management | Easy | More difficult |
| Common production choice | Yes | Less common |

### Interview Trap

An anonymous volume is still a volume. It is **not** the same thing as the container writable layer.

### Interview Point

> Named volumes are generally easier to manage explicitly because their identity is known.

---

## Q272. What is the difference between a Docker volume and a bind mount?

### Short Interview Answer

A **volume** is managed by Docker, while a **bind mount** maps a specific host filesystem path into the container.

### Volume

```bash
docker volume create appdata

docker run \
  --mount source=appdata,target=/app/data \
  myapp
```

### Bind mount

```bash
docker run \
  --mount type=bind,source=/opt/app/data,target=/app/data \
  myapp
```

### Conceptual difference

```text
Volume:

Docker
  │
  └── manages storage
          │
          └── container

Bind mount:

Host filesystem
  │
  └── /opt/app/data
          │
          └── container /app/data
```

### Comparison

| | Volume | Bind Mount |
|---|---|---|
| Managed by Docker | Yes | No |
| Source | Docker-managed storage | Specific host path |
| Host path control | Abstracted | Explicit |
| Common use | Persistent application data | Development/configuration/host integration |
| Portability | Generally better | More host-dependent |

### Interview Point

> Choose a volume when Docker-managed persistent storage is appropriate; choose a bind mount when you intentionally need a specific host path.

---

## Q273. When would you use a bind mount?

### Short Interview Answer

Use a bind mount when the container needs direct access to a specific host filesystem path.

### Common development example

```bash
docker run --rm -it \
  --mount type=bind,source="$PWD",target=/workspace \
  node:24
```

The current host directory appears inside the container as:

```text
/workspace
```

This is useful for source-code development because changes made on the host are immediately visible inside the container.

### Other examples

- Sharing host configuration files.
- Mounting certificates.
- Supplying development source code.
- Integrating with a host directory intentionally.

### Caution

Bind mounts can expose host files to containers. Permissions and path correctness must therefore be considered carefully.

### Interview Point

> Bind mounts are powerful because they expose a real host path; that is also their main operational and security consideration.

---

## Q274. What is a tmpfs mount?

### Short Interview Answer

A tmpfs mount stores data in memory rather than persistent disk storage. The data exists only while the container is running and is lost when the container is removed.

### Example

```bash
docker run -d \
  --tmpfs /app/tmp \
  nginx
```

Or:

```bash
docker run -d \
  --mount type=tmpfs,destination=/app/tmp \
  nginx
```

### Use cases

- Temporary files.
- Short-lived caches.
- Data that should not be written to disk.
- Some sensitive transient data.

### Important distinction

```text
Volume     → persistent storage
Bind mount → host filesystem path
tmpfs      → memory-backed temporary storage
```

### Interview Trap

tmpfs is **not** a persistence mechanism.

### Interview Point

> tmpfs is useful when data should be temporary and memory-backed.

---

# Volume Lifecycle and Sharing

## Q275. What is the lifecycle of a Docker volume?

### Short Interview Answer

A volume has a lifecycle independent of the container that uses it.

Typical lifecycle:

```text
create
  ↓
mount into container
  ↓
container writes data
  ↓
container removed
  ↓
volume still exists
  ↓
reuse or explicitly remove volume
```

Example:

```bash
docker volume create appdata
docker run --rm -v appdata:/data alpine sh -c 'echo hello > /data/file'
docker run --rm -v appdata:/data alpine cat /data/file
```

Output:

```text
hello
```

Remove it explicitly:

```bash
docker volume rm appdata
```

### Interview Point

> Container lifecycle and volume lifecycle are independent.

---

## Q276. Where are Docker volumes stored on Linux?

### Short Interview Answer

For Docker's default local volume driver on a typical Linux installation, volume data is stored under Docker's data root, commonly:

```text
/var/lib/docker/volumes/
```

A specific volume can be inspected with:

```bash
docker volume inspect mydata
```

Example output contains a `Mountpoint` similar to:

```text
/var/lib/docker/volumes/mydata/_data
```

### Important caveat

Do not hard-code this path in an application.

The actual Docker data root can be changed, and storage behavior can differ with the volume driver and Docker environment.

Check Docker's data root:

```bash
docker info
```

### Interview Point

> `docker volume inspect` is the authoritative way to see the mountpoint for a particular volume in that Docker environment.

---

## Q277. Can multiple containers share the same Docker volume?

### Short Interview Answer

Yes. Multiple containers can mount the same volume, subject to application and access-mode requirements.

### Example

```bash
docker volume create shared-data

docker run -d \
  --name writer \
  -v shared-data:/data \
  alpine sh -c 'while true; do date >> /data/log.txt; sleep 5; done'

docker run --rm \
  -v shared-data:/data \
  alpine cat /data/log.txt
```

Both containers access the same underlying volume data.

### Important consideration

Docker allowing multiple mounts does **not** mean the application is automatically safe for concurrent access.

For example, two processes writing to the same database files can corrupt data if the database/application does not support that access pattern.

### Interview Point

> Shared storage and safe concurrent access are different questions.

---

## Q278. How do you mount a volume as read-only?

### Short Interview Answer

Use a read-only mount so the container can read the data but cannot modify it through that mount.

### `--mount`

```bash
docker run --rm \
  --mount source=config,target=/etc/app,readonly \
  myapp
```

### `-v`

```bash
docker run --rm \
  -v config:/etc/app:ro \
  myapp
```

### Why use it?

Useful for:

- Configuration.
- Certificates.
- Shared static content.
- Data that the application should not modify.

### Security benefit

It reduces the ability of the containerized process to alter mounted data.

### Interview Point

> Read-only mounts enforce the storage access mode at the container mount boundary.

---

## Q279. What is the difference between `--mount` and `-v`?

### Short Interview Answer

Both can configure mounts, but `--mount` uses a more explicit key-value syntax and is generally easier to read and less error-prone for complex configurations.

### `-v`

```bash
docker run -v mydata:/app/data myapp
```

### `--mount`

```bash
docker run \
  --mount type=volume,source=mydata,target=/app/data \
  myapp
```

### Bind mount example

```bash
docker run \
  --mount type=bind,source=/opt/app,target=/app \
  myapp
```

### Why interviewers care

The `--mount` syntax makes mount type and options explicit:

```text
type=
source=
target=
readonly
```

This is especially useful when troubleshooting complex mount configurations.

### Interview Point

> `--mount` is more explicit; `-v` is shorter and familiar.

---

# Permissions and Ownership

## Q280. How do Docker volume permissions work?

### Short Interview Answer

A mounted volume uses the underlying filesystem's ownership and permission model. The process inside the container still has a UID/GID, and access is determined using normal Linux permission checks.

### Important concept

Container username and host username are not the key issue.

Linux primarily evaluates:

```text
UID
GID
mode bits
ACLs/capabilities where applicable
```

Suppose the application runs as:

```text
UID 1000
GID 1000
```

but the mounted directory is owned by:

```text
UID 0
GID 0
```

with restrictive permissions.

The application may receive:

```text
Permission denied
```

### Troubleshooting

Check container user:

```bash
docker exec app id
```

Check mount:

```bash
docker inspect app
```

For bind mounts, inspect the host directory:

```bash
ls -ld /opt/app/data
ls -ln /opt/app/data
```

### Common fixes

- Match UID/GID appropriately.
- Adjust ownership/permissions deliberately.
- Use an initialization process where appropriate.
- Run the application with the intended user.

### Interview Trap

Do not solve every permission problem by doing:

```bash
chmod 777
```

That may hide the real ownership problem and weaken security.

### Interview Point

> Docker does not replace Linux filesystem permission semantics.

---

# Backup and Restore

## Q281. How would you back up a Docker volume?

### Short Interview Answer

A common approach is to mount the volume into a temporary container and create an archive of its contents.

### Example

Create a volume:

```bash
docker volume create appdata
```

Back it up:

```bash
docker run --rm \
  -v appdata:/data:ro \
  -v "$PWD":/backup \
  alpine \
  tar czf /backup/appdata.tar.gz -C /data .
```

Conceptually:

```text
appdata volume
      │
      ▼
temporary container
      │
      └── tar
           │
           ▼
host /backup/appdata.tar.gz
```

### Restore

Create another volume:

```bash
docker volume create restored-data
```

Extract the archive:

```bash
docker run --rm \
  -v restored-data:/data \
  -v "$PWD":/backup \
  alpine \
  tar xzf /backup/appdata.tar.gz -C /data
```

### Production caveat

For databases, filesystem copying alone may not guarantee application-consistent backups. Prefer the database's native backup mechanism when appropriate.

For example, PostgreSQL commonly uses tools such as:

```bash
pg_dump
pg_restore
```

### Interview Point

> Volume backup is a filesystem operation; database backup consistency is an application-level concern.

---

## Q282. What is a Docker volume driver?

### Short Interview Answer

A volume driver controls how Docker volumes are provisioned and accessed. The default local driver stores data locally, while other drivers/plugins can integrate with external storage systems.

### Example

```bash
docker volume create mydata
```

Inspect it:

```bash
docker volume inspect mydata
```

You may see:

```json
{
  "Driver": "local"
}
```

### Why drivers matter

They can provide storage backed by systems such as:

- Local host storage.
- Networked storage.
- Vendor/plugin-managed storage.

### Interview Point

> The volume abstraction separates the container from the underlying storage implementation.

---

## Q283. What is the difference between a volume and the container writable layer?

### Short Interview Answer

The writable layer belongs to a particular container and disappears when that container is removed. A volume is a separate storage object whose lifecycle is independent of the container.

| | Writable Layer | Volume |
|---|---|---|
| Belongs to | Container | Separate storage object |
| Survives container removal | No | Yes |
| Intended for persistent data | No | Yes |
| Managed as separate Docker object | No | Yes |
| Reusable by replacement container | No | Yes |

### Interview Point

> If the container can be deleted and recreated without losing the data, that data should normally be outside the container writable layer.

---

# Database Storage

## Q284. What is the recommended storage approach for a database container?

### Short Interview Answer

Put database data on persistent storage such as a Docker volume or an appropriate external storage system, and use database-native backup/restore procedures.

### Example

```bash
docker volume create pgdata

docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=example \
  --mount source=pgdata,target=/var/lib/postgresql/data \
  postgres
```

### Why?

A database is stateful:

```text
PostgreSQL process
      │
      ▼
persistent data
      │
      ▼
volume / external storage
```

The container itself should be replaceable:

```text
Container A
   ↓ delete
Container B
   ↓ same persistent storage
same database data
```

### Production consideration

For serious production workloads, also consider:

- Storage performance.
- Durability.
- Replication.
- Snapshots.
- Database-consistent backups.
- Recovery testing.
- Failure domains.

### Interview Point

> Containerize the database process if appropriate, but do not make the container writable layer the database's persistence strategy.

---

## Q285. What happens if a container is recreated with the same volume?

### Short Interview Answer

If the same volume is mounted at the same application path, the replacement container can access the data already stored in that volume.

### Example

```bash
docker volume create appdata
```

Container 1:

```bash
docker run --name app1 \
  -v appdata:/data \
  alpine sh -c 'echo persistent > /data/file.txt'
```

Remove container:

```bash
docker rm app1
```

Container 2:

```bash
docker run --name app2 \
  -v appdata:/data \
  alpine cat /data/file.txt
```

Output:

```text
persistent
```

### Key idea

```text
Container 1 ──┐
              ├── appdata volume
Container 2 ──┘
```

### Interview Point

> Recreating the container does not recreate the volume unless you explicitly remove/recreate the volume.

---

# Volume Commands

## Q286. How do you list Docker volumes?

```bash
docker volume ls
```

Example:

```text
DRIVER    VOLUME NAME
local     appdata
local     pgdata
```

Filter:

```bash
docker volume ls --filter name=app
```

### Interview Point

> `docker volume ls` shows Docker-managed volumes, not arbitrary host directories used by bind mounts.

---

## Q287. How do you inspect a Docker volume?

```bash
docker volume inspect appdata
```

Typical information includes:

- Name
- Driver
- Mountpoint
- Scope
- Driver-specific options/labels where applicable

Example:

```bash
docker volume inspect appdata
```

The `Mountpoint` tells you where the volume is represented in the Docker host for the relevant local storage configuration.

### Interview Point

> When a volume behaves unexpectedly, inspect the volume and the container mount configuration rather than guessing the host path.

---

## Q288. How do you create a Docker volume?

```bash
docker volume create appdata
```

Then:

```bash
docker run --rm \
  --mount source=appdata,target=/data \
  alpine
```

You can also create a volume with options when supported by the selected driver.

### Interview Point

> Creating the volume separately makes its lifecycle explicit and simplifies reuse.

---

## Q289. How do you remove a Docker volume?

```bash
docker volume rm appdata
```

Docker normally prevents removal when the volume is still in use by a container.

Check usage with:

```bash
docker ps -a
docker inspect <container>
```

### Dangerous command

```bash
docker volume prune
```

This removes unused local volumes.

Always understand what "unused" means before pruning production storage.

### Interview Trap

Removing a container does not automatically mean its named volume is removed.

### Interview Point

> Volume deletion is an explicit destructive storage operation.

---

## Q290. What does `docker volume prune` do?

### Short Interview Answer

It removes unused local Docker volumes according to Docker's pruning rules.

```bash
docker volume prune
```

Docker asks for confirmation unless appropriate flags/options are used.

### Why it matters

Unused volumes can consume significant disk space.

However:

```text
unused ≠ unimportant
```

A volume may be intentionally retained for future recovery or reuse.

### Interview Point

> Never treat pruning as harmless cleanup on a production host.

---

# Storage Driver vs Volumes

## Q291. What is the difference between a storage driver and a volume?

### Short Interview Answer

A **storage driver** manages the container/image filesystem layers, while a **volume** is a separate persistent storage mechanism mounted into the container.

### Conceptual model

```text
Docker Engine
│
├── Image/container filesystem
│      └── storage driver
│          └── overlay2, etc.
│
└── Persistent storage
       └── volume
           └── volume driver
```

### Why this distinction matters

For example, on a Linux Docker installation using `overlay2`:

```text
image layers
     ↓
overlay2
     ↓
container writable layer
```

A volume is mounted separately:

```text
volume
   ↓
/app/data
```

The application sees one filesystem namespace, but the underlying storage mechanisms are different.

### Interview Point

> Storage drivers handle layered image/container filesystems; volumes handle separately managed persistent data.

---

## Q292. What is `overlay2`, and how is it different from a volume?

### Short Interview Answer

`overlay2` is a Linux storage driver commonly used by Docker to implement image layers and container writable layers. A volume is separate storage mounted into the container filesystem.

### Conceptual view

```text
Image layers
   │
   ▼
overlay2
   │
   └── container writable layer

Volume
   │
   └── mounted at /var/lib/app/data
```

### Why databases should not rely on the writable layer

Layered copy-on-write storage is designed around container/image filesystem semantics. Persistent database storage is better separated into a volume or appropriate external storage.

### Interview Point

> `overlay2` is a storage implementation for layered container filesystems, not a replacement for persistent application storage.

---

# Practical Scenarios

## Q293. Why should you not store important database data in the container writable layer?

### Short Interview Answer

Because the writable layer is tied to the container lifecycle. Removing or replacing the container removes that layer and can therefore destroy the database data.

### Bad design

```text
PostgreSQL
   │
   └── container writable layer
             │
             └── container deleted
                    ↓
                 data lost
```

### Better design

```text
PostgreSQL container
       │
       ▼
/var/lib/postgresql/data
       │
       ▼
persistent volume
```

Then:

```text
Container replaced
       │
       ▼
same volume
       │
       ▼
same data
```

### Interview Point

> Stateless containers should be replaceable; state must live outside the replaceable container layer.

---

## Q294. How would you troubleshoot a Docker volume mount problem?

### Short Interview Answer

I would verify the mount definition, volume existence, container path, permissions, and what the application actually sees inside the container.

### Step 1 — Verify the volume

```bash
docker volume ls
docker volume inspect appdata
```

### Step 2 — Verify the container mount

```bash
docker inspect app
```

Look under:

```text
Mounts
```

Check:

```text
Type
Source
Destination
RW
```

### Step 3 — Check from inside the container

```bash
docker exec -it app sh
```

Then:

```bash
mount
df -h
ls -la /app/data
id
```

### Step 4 — Check permissions

For bind mounts:

```bash
ls -ld /host/path
ls -ln /host/path
```

Inside container:

```bash
id
ls -ld /app/data
```

Compare UID/GID and permissions.

### Step 5 — Check whether the application is using the expected path

A correctly mounted volume does not help if the application writes somewhere else.

For example:

```text
Expected:
  /var/lib/app/data

Actual:
  /tmp/data
```

### Step 6 — Check read-only configuration

If the mount is read-only:

```text
RW: false
```

the application cannot write through that mount.

### Interview troubleshooting chain

```text
Volume exists?
      ↓
Correct container mounted it?
      ↓
Correct destination path?
      ↓
Read/write mode correct?
      ↓
UID/GID permissions correct?
      ↓
Application writing to that path?
      ↓
Storage has capacity?
```

### Interview Point

> Storage troubleshooting is not just "check the volume"; verify the complete path from Docker configuration to application behavior.

---

## Q295. Scenario: "My container was recreated and all my data disappeared." What would you investigate?

### Strong Interview Answer

First I would determine **where the data was stored**.

If the application wrote into the container writable layer:

```text
container writable layer
        ↓
container removed
        ↓
data removed
```

then the behavior is expected.

I would verify:

```bash
docker inspect <container>
```

and inspect the `Mounts` section.

If there is no mount covering the application's data directory, the application may have been writing to the writable layer.

If a volume was expected, I would check:

```bash
docker volume ls
docker volume inspect <volume>
docker inspect <container>
```

Then I would verify:

1. Was the correct volume mounted?
2. Was it mounted at the correct destination?
3. Was the volume itself deleted?
4. Did the application write to another directory?
5. Did a deployment script accidentally create a new anonymous volume?
6. Were host/bind-mount paths changed?
7. Did permissions prevent the application from writing to the expected storage?

### Important interview distinction

```text
Container recreated
       ↓
same volume attached
       ↓
data should remain

Container recreated
       ↓
new volume / no volume
       ↓
old data may not be visible
```

### Interview Point

> "Data disappeared" is a storage-path/lifecycle investigation, not automatically a Docker bug.

---

# Quick Revision

| Topic | Key Point |
|---|---|
| Writable layer | Container-specific and normally ephemeral |
| Volume | Docker-managed persistent storage |
| Named volume | Explicit reusable name |
| Anonymous volume | Automatically generated volume identity |
| Bind mount | Specific host path mounted into container |
| tmpfs | Memory-backed temporary storage |
| Volume lifecycle | Independent of container lifecycle |
| Shared volume | Multiple containers can mount it |
| Read-only mount | Prevents writes through that mount |
| `--mount` | Explicit mount syntax |
| `-v` | Short mount syntax |
| Volume permissions | Linux UID/GID/mode semantics still apply |
| Volume backup | Can archive mounted contents |
| Database backup | Prefer application-consistent backup methods |
| Volume driver | Controls volume storage implementation |
| Storage driver | Handles image/container filesystem layers |
| `overlay2` | Common Linux layered filesystem storage driver |
| `docker volume ls` | List volumes |
| `docker volume inspect` | Inspect volume details |
| `docker volume create` | Create volume |
| `docker volume rm` | Delete volume |
| `docker volume prune` | Remove unused local volumes |

---

# High-Value Interview Traps

### Trap 1 — "Container restart deletes data"

**Incorrect.**

Stopping/restarting a container does not normally remove its writable layer.

Container removal is the important lifecycle event.

---

### Trap 2 — "A volume is just the writable layer"

**Incorrect.**

A volume is separate persistent storage mounted into the container.

---

### Trap 3 — "Bind mount and volume are the same"

**Incorrect.**

A bind mount points to a specific host path; a volume is managed through Docker's volume abstraction.

---

### Trap 4 — "tmpfs persists after container deletion"

**Incorrect.**

tmpfs is temporary memory-backed storage.

---

### Trap 5 — "chmod 777 fixes Docker storage"

Not necessarily.

First identify the actual UID/GID, mount type, permissions, and application path.

---

### Trap 6 — "If a volume exists, the application is using it"

Not necessarily.

The application must write to the path covered by the mount.

---

### Trap 7 — "Docker volume backup is automatically database-consistent"

Not necessarily.

A live database may require a database-aware backup mechanism.

---

### Trap 8 — "overlay2 is a persistent database storage mechanism"

It is a storage driver for layered container/image filesystems, not the preferred persistence boundary for important application state.

---

# High-Value Interview Follow-Up Questions

An interviewer may continue with:

1. What happens when you delete a container but keep its volume?
2. How do you find which containers are using a volume?
3. Can two containers mount the same volume?
4. Can a volume be mounted read-only?
5. What happens if the target directory already contains files before mounting?
6. How would you migrate a Docker volume to another host?
7. How do UID/GID mismatches cause volume permission problems?
8. What is the difference between a volume and a bind mount in production?
9. How would you back up a PostgreSQL container?
10. What happens if the Docker host itself fails?
11. Would a Docker volume survive a host reboot?
12. Would a Docker volume survive Docker daemon restart?
13. What happens if you delete a volume accidentally?
14. How would you monitor Docker disk usage?
15. Why might `docker system df` show significant reclaimable storage?
16. What is the relationship between Docker volumes and storage drivers?
17. How would you migrate from bind mounts to named volumes?
18. What happens when a volume is mounted over a directory containing image data?

---

# Final Interview Answer

If asked **"Explain Docker storage"**, a strong answer is:

> "Docker has a container writable layer on top of the image's read-only layers. That writable layer is tied to the container lifecycle, so important application data should not normally be stored there. For persistent data, Docker provides volumes, bind mounts, and tmpfs mounts. A volume is Docker-managed storage and is independent of the container lifecycle, while a bind mount exposes a specific host filesystem path. tmpfs is memory-backed and temporary. For databases, I would normally use persistent storage and database-consistent backup mechanisms. When troubleshooting storage, I check the volume, container Mounts configuration, destination path, read/write mode, UID/GID permissions, filesystem capacity, and whether the application is actually writing to the mounted path."

---

# One-Line Memory Map

```text
Writable Layer = container lifecycle
Volume         = Docker-managed persistent data
Bind Mount     = specific host path
tmpfs          = temporary memory
Storage Driver = image/container filesystem layers
```
