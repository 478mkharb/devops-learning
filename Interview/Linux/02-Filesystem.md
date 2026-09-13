# Linux Filesystem for DevOps

## Scope

This topic focuses on filesystem concepts that matter when deploying and operating applications with Jenkins, Ansible, systemd, NGINX, cloud VMs, containers, and monitoring tools.

---

## 1. What is the Linux filesystem?

Linux presents files, directories, devices, sockets, and many kernel interfaces through a hierarchical filesystem beginning at `/`.

```text
/
├── etc
├── var
├── home
├── opt
├── usr
├── tmp
├── proc
├── sys
├── dev
└── run
```

For DevOps, filesystem knowledge is required to understand:

- Where application binaries are installed
- Where configuration is stored
- Where logs are written
- Where Jenkins workspaces exist
- Where systemd services execute
- Where persistent data is mounted
- Why deployments fail because of permissions or disk usage

---

## 2. Explain `/etc`, `/var`, `/opt`, `/usr`, and `/run`

| Directory | Purpose | DevOps example |
|---|---|---|
| `/etc` | System and service configuration | `/etc/nginx/nginx.conf` |
| `/var` | Variable application/system data | `/var/log`, `/var/lib` |
| `/opt` | Optional or manually installed software | `/opt/liquibase` |
| `/usr` | Installed programs and libraries | `/usr/bin`, `/usr/lib` |
| `/run` | Runtime state since boot | PID files and sockets |

Do not store changing application data inside `/usr`. Keep configuration, binaries, logs, and persistent data separated.

---

## 3. Where should application files be placed?

A consistent structure is more important than one universal path.

Example:

```text
/opt/attendance-api/
├── releases/
│   ├── 2026-09-13_0900/
│   └── 2026-09-13_1000/
├── current -> releases/2026-09-13_1000
├── shared/
│   ├── logs/
│   ├── uploads/
│   └── tmp/
└── config/
    └── application.env
```

Benefits:

- Easy rollback
- Clear ownership
- Safer deployments
- Separation of persistent data
- Easier Ansible automation

Never put secrets directly inside a publicly served web directory.

---

## 4. What is the difference between `/var/log` and `/var/lib`?

`/var/log` contains logs.

Examples:

```text
/var/log/syslog
/var/log/auth.log
/var/log/nginx/
```

`/var/lib` contains persistent state used by applications and services.

Examples:

```text
/var/lib/postgresql
/var/lib/redis
/var/lib/docker
```

Deleting `/var/lib` data can destroy application state. Deleting logs may affect troubleshooting and compliance. Always understand the application before cleaning either directory.

---

## 5. What is `/proc`?

`/proc` is a virtual filesystem exposing process and kernel information.

Examples:

```bash
cat /proc/cpuinfo
cat /proc/meminfo
cat /proc/loadavg
cat /proc/uptime
```

For a process:

```bash
cat /proc/<PID>/status
ls -l /proc/<PID>/fd
cat /proc/<PID>/cmdline
```

DevOps use cases:

- Investigating memory usage
- Checking process limits
- Inspecting open file descriptors
- Understanding CPU information
- Debugging container process visibility

`/proc` does not represent ordinary files stored on disk.

---

## 6. What is `/sys`?

`/sys` exposes kernel, device, and hardware-related information.

Examples:

```bash
ls /sys/class/net
cat /sys/class/net/eth0/operstate
```

It is useful for low-level troubleshooting, but most application deployment work uses higher-level commands such as:

```bash
ip addr
ip link
lsblk
udevadm info
```

---

## 7. What is `/dev`?

`/dev` contains device files.

Examples:

```text
/dev/sda
/dev/nvme0n1
/dev/null
/dev/zero
/dev/random
```

Common special devices:

```bash
echo test > /dev/null
```

`/dev/null` discards data.

```bash
head -c 10 /dev/zero
```

`/dev/zero` provides zero bytes.

Cloud DevOps relevance:

- EBS volumes appear as block devices
- Filesystems are created on devices
- Mount failures may be caused by incorrect device names
- Device names can differ between instance types and virtualization platforms

Use stable identifiers where possible:

```bash
lsblk -f
blkid
```

---

## 8. What is `/tmp`?

`/tmp` is intended for temporary files.

Problems occur when applications store permanent data there:

- Reboots may remove files
- Cleanup jobs may delete files
- Disk usage can grow unexpectedly
- Multiple applications may collide on filenames
- Sensitive data may remain accessible

Use secure temporary-file creation:

```bash
mktemp
mktemp -d
```

Do not use predictable temporary names such as:

```bash
/tmp/app-output.txt
```

for sensitive or concurrent operations.

---

## 9. What is the difference between `df` and `du`?

`df` reports filesystem-level free and used space:

```bash
df -hT
```

`du` reports space consumed by files and directories:

```bash
du -sh /var/log
du -xhd1 /var | sort -h
```

If `df` shows 95% usage but `du` cannot explain it, check deleted files still held open:

```bash
sudo lsof +L1
```

A process may continue consuming disk space through an open file after the filename has been deleted.

---

## 10. What does `df -hT` show?

- `-h`: human-readable sizes
- `-T`: filesystem type

Example:

```bash
df -hT
```

Important columns:

- Filesystem
- Type
- Size
- Used
- Available
- Use%
- Mounted on

Filesystem type matters because behavior and supported features differ between `ext4`, `xfs`, `tmpfs`, and network filesystems.

---

## 11. What does `du -x` do?

`du -x` stays on the same filesystem.

This is important when investigating `/` because mounted filesystems such as `/data` may otherwise be included in the result.

```bash
sudo du -xhd1 / | sort -h
```

Without `-x`, the result may include unrelated mounted volumes and make root-disk analysis confusing.

---

## 12. What is an inode?

An inode stores metadata about a filesystem object, including:

- Owner
- Permissions
- File type
- Size
- Timestamps
- Link count
- Data block references

Check inode usage:

```bash
df -i
```

A system can have free gigabytes but no free inodes because millions of small files exist.

Typical causes:

- Build artifacts
- Temporary files
- Application sessions
- Cache directories
- Per-request log files

---

## 13. What is a mount point?

A mount point is a directory where a filesystem is attached.

```bash
findmnt
lsblk -f
mount
```

Example:

```text
/dev/nvme1n1 mounted on /data
```

A mount point must normally exist before mounting:

```bash
sudo mkdir -p /data
```

A dangerous failure pattern is when `/data` is not mounted and the application writes into the empty directory on the root filesystem.

---

## 14. How do you make a mount persistent?

Temporary mount:

```bash
sudo mount /dev/nvme1n1p1 /data
```

Persistent mounts are configured in `/etc/fstab`.

Before rebooting, validate:

```bash
sudo findmnt --verify
```

Use UUIDs instead of assuming device names:

```bash
sudo blkid
```

Example `/etc/fstab` entry:

```fstab
UUID=<filesystem-uuid> /data ext4 defaults,nofail 0 2
```

`nofail` can prevent a non-critical volume from blocking boot, but it should not be used blindly for a volume required by the application.

---

## 15. What is the danger of editing `/etc/fstab` incorrectly?

A malformed or incorrect entry can cause:

- Boot delays
- Emergency mode
- Missing application data
- Services failing after reboot
- Mounting the wrong device

Validate the file before reboot:

```bash
sudo findmnt --verify
sudo mount -a
```

Run `mount -a` carefully and inspect errors immediately.

---

## 16. What is a read-only filesystem?

A filesystem mounted read-only does not allow normal writes.

Check:

```bash
findmnt -o TARGET,SOURCE,FSTYPE,OPTIONS
```

Look for `ro` in mount options.

Possible causes:

- Filesystem errors
- Storage failure
- Explicit read-only mount
- Cloud volume or snapshot workflow
- Kernel protection after detecting corruption

An application may fail with:

```text
Read-only file system
```

Do not simply remount read-write without investigating the cause.

---

## 17. What is a bind mount?

A bind mount exposes an existing directory at another path.

```bash
sudo mount --bind /srv/app/shared /opt/app/shared
```

Use cases:

- Sharing application data
- Exposing host directories to containers
- Presenting a directory at a service-specific path
- Migration without changing application configuration

Inspect:

```bash
findmnt
```

---

## 18. What is a symbolic link?

A symbolic link stores a target path.

```bash
ln -s /opt/app/releases/v2 /opt/app/current
```

Inspect:

```bash
ls -l /opt/app/current
readlink -f /opt/app/current
```

Common DevOps uses:

- Release switching
- Current-version pointers
- Standardized binary names
- Linking configuration into application directories

A symlink can become broken if its target is removed.

---

## 19. How do you find broken symbolic links?

```bash
find /opt/app -xtype l -print
```

A broken symlink can cause:

- systemd startup failure
- NGINX document-root errors
- Missing configuration
- Deployment rollback failure

Always validate the target after changing release links.

---

## 20. What is a hard link?

A hard link is another directory entry for the same inode.

```bash
ln original.txt copy.txt
```

Both names refer to the same underlying file data.

Hard links generally:

- Cannot cross filesystems
- Cannot normally link directories
- Continue working if one filename is deleted

Release deployments normally use symlinks, not hard links, because symlinks clearly represent a release target.

---

## 21. What are file timestamps?

Linux commonly exposes:

- `atime`: access time
- `mtime`: content modification time
- `ctime`: inode metadata change time

Display them:

```bash
stat file.txt
```

`ctime` is not creation time. Birth time support depends on the filesystem and tools.

Find recently modified files:

```bash
find /var/log -type f -mmin -30
```

Useful during incident response and deployment verification.

---

## 22. What is `stat` useful for?

```bash
stat /etc/nginx/nginx.conf
```

It shows:

- File type
- Permissions
- UID/GID
- Size
- Inode
- Link count
- Timestamps

When a service cannot read a file, `stat` helps confirm ownership, permissions, and whether the file is actually the expected object.

---

## 23. What is `namei`?

`namei` resolves each component of a path and displays permissions.

```bash
namei -l /opt/app/current/config/app.yaml
```

This is extremely useful when a file appears readable but a parent directory lacks execute permission.

For directory traversal, the user needs execute permission on every parent directory in the path.

---

## 24. What is a filesystem quota?

A quota limits disk space or inode usage for users, groups, or projects.

Quotas can cause an application to fail even when the filesystem appears to have free space.

Possible symptoms:

```text
Disk quota exceeded
```

In shared environments, check:

- User quota
- Group quota
- Project quota
- Container storage limits
- Cloud volume capacity

---

## 25. What is `tmpfs`?

`tmpfs` is a memory-backed temporary filesystem.

Check:

```bash
df -hT
mount | grep tmpfs
```

Common locations:

```text
/run
/dev/shm
```

`tmpfs` consumes memory and may use swap depending on system behavior. Large temporary files in `/dev/shm` can cause memory pressure.

---

## 26. What is `/dev/shm`?

`/dev/shm` is a shared-memory filesystem, commonly used for POSIX shared memory and temporary high-speed files.

Some applications, browsers, databases, and test tools use it.

Check:

```bash
df -h /dev/shm
```

A full `/dev/shm` can cause unexpected application failures even when `/` has free space.

---

## 27. How do you locate large files?

```bash
sudo find / -xdev -type f -size +1G -ls
```

For the current filesystem:

```bash
sudo du -ahx / | sort -rh | head -30
```

Be cautious with:

- `/proc`
- `/sys`
- Mounted volumes
- Deleted files
- Sparse files

Do not delete files solely because they are large. Confirm ownership and application impact first.

---

## 28. How do you find files changed during a deployment?

```bash
find /opt/app -type f -newermt '2026-09-13 09:00'
```

Or use a reference file:

```bash
touch /tmp/deploy-start
# run deployment
find /opt/app -newer /tmp/deploy-start
```

This helps verify whether the expected files were updated.

---

## 29. What is log rotation?

Log rotation prevents logs from filling the filesystem.

Ubuntu commonly uses `logrotate`.

Configuration locations:

```text
/etc/logrotate.conf
/etc/logrotate.d/
```

Test configuration:

```bash
sudo logrotate -d /etc/logrotate.conf
```

Force a rotation for testing:

```bash
sudo logrotate -f /etc/logrotate.conf
```

Application logging should also support:

- Retention
- Compression
- Rotation by size/time
- Centralized collection
- Structured output

---

## 30. Scenario: Root filesystem is 100% full

Initial checks:

```bash
df -hT
df -i
sudo du -xhd1 / | sort -h
sudo lsof +L1
```

Then inspect:

```bash
sudo du -xhd1 /var | sort -h
sudo du -xhd1 /tmp | sort -h
sudo journalctl --disk-usage
```

Possible causes:

- Application logs
- Journal logs
- Package cache
- Temporary files
- Deleted files held open
- Container layers
- Core dumps

Do not blindly delete `/var/lib` or system logs. Apply a controlled cleanup and fix the source of growth.

---

## 31. Scenario: Application writes to the root disk instead of `/data`

Likely causes:

- Volume was not mounted
- Wrong mount path
- Service started before mount
- Incorrect `fstab`
- Application uses a different path
- Mount failed silently

Check:

```bash
findmnt /data
df -h /data
systemctl status data.mount
```

For systemd services, use dependencies such as:

```ini
RequiresMountsFor=/data
```

This helps ensure the required mount is available before the service starts.

---

## 32. Scenario: Jenkins workspace is full

Inspect workspace:

```bash
du -sh "$WORKSPACE"
du -xhd1 "$WORKSPACE" | sort -h
```

Prevention:

- Clean workspaces after builds
- Retain only required artifacts
- Use Jenkins artifact retention policies
- Remove old dependency caches carefully
- Put workspaces on suitable storage
- Monitor disk and inode usage

Do not delete another job's workspace while its build is running.

---

## 33. Scenario: NGINX returns 403 for a static file

Check:

```bash
namei -l /var/www/html/index.html
ls -l /var/www/html/index.html
```

Possible causes:

- NGINX user cannot traverse a parent directory
- File permissions deny reading
- Directory index is missing
- SELinux/AppArmor policy blocks access
- Incorrect `root` or `alias` configuration

Review:

```bash
sudo nginx -t
sudo journalctl -u nginx
```

Avoid making the entire directory world-writable.

---

## 34. Scenario: Deployment rollback points to a missing release

Check:

```bash
readlink -f /opt/app/current
ls -lah /opt/app/releases
find /opt/app/releases -maxdepth 1 -type d
```

A safe rollback process should:

1. Confirm the target release exists
2. Validate configuration
3. Switch the symlink atomically
4. Restart or reload the service if required
5. Run a health check
6. Revert if validation fails

Example atomic switch:

```bash
ln -sfn /opt/app/releases/v1 /opt/app/current
```

Test the target before switching production traffic.

---

## 35. Practical command reference

```bash
pwd
ls -lah
tree -L 2
stat file
namei -l /path/to/file
readlink -f link
find /path -type f
find /path -xtype l
df -hT
df -i
du -sh /path
du -xhd1 /path
findmnt
lsblk -f
blkid
mount
sudo mount -a
sudo lsof +L1
journalctl --disk-usage
```

---

## Interview checklist

You should be able to explain:

- Linux filesystem hierarchy
- Application directory structure
- `/etc` versus `/var`
- `/var/log` versus `/var/lib`
- `/proc`, `/sys`, and `/dev`
- `df` versus `du`
- Inodes
- Mount points
- `/etc/fstab`
- UUID-based mounting
- Read-only filesystems
- Bind mounts
- Symlinks and hard links
- `stat` and `namei`
- `tmpfs` and `/dev/shm`
- Log rotation
- Deleted files held open
- Disk-full incidents
- Jenkins workspace cleanup
- Application data mounted on separate volumes
- NGINX permission troubleshooting
- Release rollback using symlinks

## Key DevOps principle

> Filesystem design is part of deployment design. Separate code, configuration, logs, temporary files, and persistent data so that one failure does not take down the entire server.
