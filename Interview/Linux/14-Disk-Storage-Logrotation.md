# Disk, Storage and Log Rotation for DevOps

## Scope

This topic covers storage operations from a DevOps Engineer perspective:

- Disk, partition and filesystem concepts
- Capacity and inode troubleshooting
- Mounts and persistent storage
- LVM basics
- File descriptors and deleted files
- Application storage planning
- Log rotation
- Disk pressure in EC2 and CI/CD systems
- Production troubleshooting scenarios

---

## 1. Why Storage Matters in DevOps

Storage problems can cause:

- Application crashes
- Database failures
- Jenkins builds failing
- Log collection failures
- Package installation failures
- Container or image build failures
- SSH login problems
- Systemd services failing to start
- Corrupted or incomplete deployments

A server can have free disk space but still fail because:

- Inodes are exhausted
- A mount is read-only
- A specific filesystem is full
- A deleted file is still held open
- The application has reached its file-descriptor limit
- Temporary storage is full

---

## 2. Disk, Partition, Filesystem and Mount

```text
Physical/Virtual Disk
        ↓
Partition
        ↓
Filesystem
        ↓
Mount Point
        ↓
Application Directory
```

Example:

```text
/dev/nvme1n1
└── /dev/nvme1n1p1
    └── ext4
        └── /data
```

Important terms:

| Term | Meaning |
|---|---|
| Disk | Block storage device |
| Partition | Logical section of a disk |
| Filesystem | Structure used to store files |
| Mount point | Directory where a filesystem is attached |
| Block | Storage allocation unit |
| Inode | Metadata structure representing a file |
| UUID | Persistent filesystem identifier |

---

## 3. Inspect Disks

List block devices:

```bash
lsblk
```

Show filesystem information:

```bash
lsblk -f
```

Show partition table:

```bash
sudo fdisk -l
```

Show disk usage:

```bash
df -h
```

Show inode usage:

```bash
df -i
```

Show mounted filesystems:

```bash
findmnt
```

Show a particular mount:

```bash
findmnt /var
```

Check disk model and transport:

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS,MODEL
```

Do not confuse `lsblk` output with actual free space. Use `df` for filesystem capacity.

---

## 4. `df` versus `du`

### `df`

Reports filesystem-level usage:

```bash
df -h
```

Use it to answer:

> How full is the filesystem?

### `du`

Reports directory/file usage:

```bash
sudo du -sh /var/log
sudo du -xhd1 /var | sort -h
```

Use it to answer:

> Which directories or files consume the space?

Typical workflow:

```bash
df -h
sudo du -xhd1 / | sort -h
sudo du -xhd1 /var | sort -h
```

The `-x` option prevents crossing into other mounted filesystems.

---

## 5. Human-Readable Disk Usage

```bash
df -h
du -sh /opt/myapp
du -sh /var/log/*
du -xhd1 /var | sort -h
```

Find large files:

```bash
sudo find /var -xdev -type f -size +500M -ls
```

Find files modified recently:

```bash
sudo find /var/log -type f -mmin -60 -ls
```

Find files by extension:

```bash
sudo find /opt -type f \( -name '*.log' -o -name '*.tmp' \) -ls
```

Use `ncdu` for interactive analysis if approved for the environment:

```bash
sudo apt install ncdu
sudo ncdu -x /
```

---

## 6. Inodes

Every file consumes at least one inode.

Check inode usage:

```bash
df -i
```

A filesystem can report:

```text
Use%: 100%
```

even when byte capacity is available, because too many small files consumed all inodes.

Find directories containing many files:

```bash
sudo find /var -xdev -type f | cut -d/ -f1-4 | sort | uniq -c | sort -nr | head
```

Common inode consumers:

- Millions of temporary files
- Application cache files
- Mail queues
- CI workspaces
- Session files
- Extracted artifacts
- Small log fragments

Do not delete files blindly. Identify the owner and retention requirement first.

---

## 7. Mount Points

Show mounts:

```bash
findmnt
mount
```

Check whether a directory is a separate filesystem:

```bash
findmnt /var/lib
```

Check disk usage without crossing mounts:

```bash
sudo du -xhd1 /
```

A common troubleshooting mistake is assuming `/data` is a separate disk when the mount failed and files are being written to the root filesystem instead.

Verify:

```bash
findmnt /data
df -h /data
```

---

## 8. `/etc/fstab`

Persistent mounts are commonly configured in:

```text
/etc/fstab
```

Example:

```fstab
UUID=xxxx-xxxx  /data  ext4  defaults,nofail  0  2
```

Inspect UUIDs:

```bash
sudo blkid
```

Validate mounts:

```bash
sudo mount -a
```

Important:

- Test `fstab` changes carefully.
- A syntax error can affect boot.
- `nofail` can prevent a noncritical disk from blocking boot.
- Prefer UUIDs over unstable device names where appropriate.

Check mount errors:

```bash
journalctl -b | grep -iE 'mount|fstab|filesystem'
```

---

## 9. Read-Only Filesystems

Check mount options:

```bash
findmnt -o TARGET,SOURCE,FSTYPE,OPTIONS
```

Look for:

```text
ro
```

A filesystem may become read-only because of:

- Filesystem errors
- Storage failure
- Kernel protection
- Explicit mount options
- Cloud volume or device issues

Check kernel messages:

```bash
dmesg -T | grep -iE 'error|ext4|xfs|I/O'
journalctl -k -p err
```

Do not immediately remount read-write without understanding the cause.

For filesystem repair, normally:

1. Stop affected services.
2. Unmount the filesystem if possible.
3. Use the appropriate filesystem-check tool.
4. Repair from a maintenance environment when required.
5. Re-mount and validate.

Never run filesystem repair casually on a mounted production filesystem.

---

## 10. Filesystem Types

Common Linux filesystems:

| Filesystem | Typical use |
|---|---|
| ext4 | General-purpose Linux filesystem |
| XFS | Large systems and enterprise workloads |
| tmpfs | Memory-backed temporary filesystem |
| NFS | Network filesystem |
| EBS-backed ext4/XFS | AWS EC2 persistent storage |

Identify filesystem type:

```bash
df -T
lsblk -f
```

Filesystem choice depends on:

- Workload
- Performance
- File size
- Snapshot/backup strategy
- Operational standards
- Recovery requirements

---

## 11. AWS EBS Storage Perspective

For EC2, EBS volumes are separate block devices.

Typical workflow:

```text
Create EBS volume
      ↓
Attach to EC2
      ↓
Identify device
      ↓
Partition if required
      ↓
Create filesystem
      ↓
Mount filesystem
      ↓
Configure /etc/fstab
      ↓
Validate after reboot
```

Inspect devices:

```bash
lsblk
```

Create a filesystem only on the correct unused device:

```bash
sudo mkfs.ext4 /dev/nvme1n1
```

Mount:

```bash
sudo mkdir -p /data
sudo mount /dev/nvme1n1 /data
```

Check:

```bash
df -h /data
findmnt /data
```

**Warning:** `mkfs` destroys existing filesystem data on the selected device. Verify the device carefully.

---

## 12. LVM Basics

LVM provides flexible storage management.

```text
Physical Volume (PV)
        ↓
Volume Group (VG)
        ↓
Logical Volume (LV)
        ↓
Filesystem
        ↓
Mount Point
```

Inspect:

```bash
sudo pvs
sudo vgs
sudo lvs
```

Typical commands:

```bash
sudo pvcreate /dev/nvme1n1
sudo vgcreate vg_app /dev/nvme1n1
sudo lvcreate -n lv_data -L 20G vg_app
sudo mkfs.ext4 /dev/vg_app/lv_data
sudo mkdir -p /data
sudo mount /dev/vg_app/lv_data /data
```

Extend a logical volume:

```bash
sudo lvextend -L +10G /dev/vg_app/lv_data
```

Resize an ext4 filesystem:

```bash
sudo resize2fs /dev/vg_app/lv_data
```

For XFS:

```bash
sudo xfs_growfs /data
```

LVM is useful when storage must grow without redesigning the whole disk layout.

---

## 13. Disk Expansion Workflow

When a filesystem is full:

1. Confirm the affected filesystem.
2. Identify the largest consumers.
3. Check whether cleanup is safe.
4. Decide between cleanup and expansion.
5. Expand the cloud volume if required.
6. Expand the partition if applicable.
7. Expand the filesystem.
8. Validate capacity and application behavior.

Example checks:

```bash
df -h
lsblk
findmnt /
```

After cloud-side expansion, the OS may still show the old size until the partition/filesystem is expanded.

Do not assume increasing an EBS volume automatically increases the filesystem.

---

## 14. Deleted Files Still Consuming Space

A deleted file can continue consuming disk space while a process keeps it open.

Find deleted open files:

```bash
sudo lsof +L1
```

Alternative:

```bash
sudo lsof | grep '(deleted)'
```

Typical example:

```text
app  1234  ... /var/log/app.log (deleted)
```

The process still holds the file descriptor.

Possible resolution:

- Restart or reload the responsible service.
- Use the application's supported log-reopen mechanism.
- Avoid killing processes without assessing impact.

Do not blindly delete more files; the space may not be released until the file descriptor closes.

---

## 15. File Descriptors

A file descriptor is a process handle for:

- Files
- Sockets
- Pipes
- Devices

View process limits:

```bash
ulimit -n
```

Check a process:

```bash
cat /proc/1234/limits | grep -i 'open files'
```

Count open descriptors:

```bash
ls /proc/1234/fd | wc -l
```

System-wide information:

```bash
cat /proc/sys/fs/file-nr
```

Too many open files can cause:

```text
EMFILE: Too many open files
```

Possible causes:

- File-descriptor leak
- Too many concurrent connections
- Excessive log files
- Incorrect application limits
- Connection pool problems

Investigate before increasing limits.

---

## 16. Temporary Storage

Important temporary locations:

```text
/tmp
/var/tmp
/dev/shm
```

Check:

```bash
df -h /tmp
df -h /dev/shm
mount | grep tmpfs
```

`/dev/shm` is usually a memory-backed filesystem.

Applications may fail when:

- `/tmp` is full
- `/dev/shm` is too small
- Temporary files are not cleaned
- A build uses excessive temporary space

Jenkins builds, package managers, browsers and compilers often use temporary directories.

---

## 17. Log Rotation

Log rotation prevents unbounded log growth.

Common tool:

```bash
logrotate
```

Main configuration:

```text
/etc/logrotate.conf
/etc/logrotate.d/
```

Inspect:

```bash
cat /etc/logrotate.conf
ls -l /etc/logrotate.d/
```

Dry run:

```bash
sudo logrotate -d /etc/logrotate.conf
```

Force rotation:

```bash
sudo logrotate -f /etc/logrotate.conf
```

Do not force rotation unnecessarily on production systems.

---

## 18. Example Logrotate Configuration

Example:

```text
/var/log/myapp/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    copytruncate
}
```

Meaning:

| Directive | Meaning |
|---|---|
| `daily` | Rotate daily |
| `rotate 14` | Keep 14 rotated files |
| `compress` | Compress old logs |
| `delaycompress` | Compress from the second rotated generation |
| `missingok` | Do not fail if file is missing |
| `notifempty` | Do not rotate empty files |
| `copytruncate` | Copy then truncate active file |

`copytruncate` is convenient but can lose a small amount of log data during the copy/truncate window.

Prefer application-supported log reopening when available.

---

## 19. `copytruncate` versus `postrotate`

### `copytruncate`

```text
copy active log
truncate original log
```

Pros:

- Works when application cannot reopen logs.

Cons:

- Possible log loss during rotation.
- Copy operation can be expensive.

### Signal/reload approach

```text
rotate file
signal application
application opens new file
```

Pros:

- Better for high-volume applications.

Cons:

- Application must support reopening logs.
- Correct signal or reload behavior is required.

Always verify how the application handles open log files.

---

## 20. Journald Storage

Check journal usage:

```bash
journalctl --disk-usage
```

Show persistent journal directory:

```bash
ls -ld /var/log/journal
```

Common configuration file:

```text
/etc/systemd/journald.conf
```

Example retention settings:

```text
SystemMaxUse=1G
MaxRetentionSec=14day
```

After configuration changes:

```bash
sudo systemctl restart systemd-journald
```

Remove old journal entries by time:

```bash
sudo journalctl --vacuum-time=14d
```

Or by size:

```bash
sudo journalctl --vacuum-size=1G
```

Retention must balance:

- Disk capacity
- Compliance
- Incident investigation
- Centralized logging availability

---

## 21. Jenkins and Disk Usage

Jenkins commonly consumes disk through:

- Workspaces
- Archived artifacts
- Build logs
- Test reports
- Dependency caches
- Temporary files
- Failed or abandoned builds

Inspect:

```bash
sudo du -xhd1 /var/lib/jenkins | sort -h
```

Useful locations may include:

```text
/var/lib/jenkins/workspace
/var/lib/jenkins/jobs
/var/lib/jenkins/.m2
```

Operational controls:

- Build discard policies
- Artifact retention
- Workspace cleanup
- Separate build volumes
- Monitoring disk usage
- Avoiding unbounded caches

Do not manually delete active Jenkins job data without understanding Jenkins state.

---

## 22. Application Storage Planning

Before deploying an application, identify:

- Application binaries
- Configuration
- Logs
- Uploads
- Temporary files
- Database files
- Backups
- Cache
- Artifacts
- Crash dumps

Example layout:

```text
/opt/myapp              application release
/etc/myapp              configuration
/var/log/myapp          logs
/var/lib/myapp          persistent application data
/run/myapp              runtime state
/tmp                    temporary data
```

Separate high-growth data from the root filesystem when appropriate.

Use explicit retention for:

- Logs
- Backups
- Uploads
- Build artifacts
- Temporary files

---

## 23. Disk Monitoring Metrics

Useful metrics include:

- Filesystem usage percentage
- Filesystem available bytes
- Inode usage percentage
- Disk read/write throughput
- IOPS
- Disk latency
- Queue depth
- Open file descriptors
- Log growth rate
- EBS burst balance where applicable

Example Prometheus node-exporter metrics:

```text
node_filesystem_avail_bytes
node_filesystem_size_bytes
node_filesystem_files_free
node_filesystem_files
node_disk_read_bytes_total
node_disk_written_bytes_total
node_filefd_allocated
```

A useful filesystem usage expression:

```promql
100 *
(
  1 -
  node_filesystem_avail_bytes
  /
  node_filesystem_size_bytes
)
```

Exclude pseudo-filesystems and irrelevant mounts using suitable label filters.

---

## 24. Common Troubleshooting Scenarios

### Scenario 1: `No space left on device`

Check:

```bash
df -h
df -i
sudo du -xhd1 / | sort -h
sudo lsof +L1
```

Possible causes:

- Filesystem full
- Inodes exhausted
- Deleted open files
- Temporary directory full
- Log growth
- Build artifacts

### Scenario 2: `df` says full but `du` does not

Likely causes:

- Deleted files still held open
- Hidden files under a mount point
- Different filesystem being measured
- Reserved filesystem blocks

Check:

```bash
sudo lsof +L1
findmnt
df -h
sudo du -xhd1 /
```

### Scenario 3: Application logs fill the root filesystem

Check:

```bash
sudo du -sh /var/log/*
sudo journalctl --disk-usage
```

Then:

- Fix log rotation.
- Fix excessive error loops.
- Configure retention.
- Ship logs centrally.
- Expand storage if needed.

### Scenario 4: New EBS volume is attached but unavailable

Check:

```bash
lsblk
sudo blkid
findmnt
```

Then:

- Confirm the correct device.
- Create filesystem only if it is new.
- Create mount point.
- Mount it.
- Configure and validate `fstab`.

### Scenario 5: Jenkins build fails unexpectedly

Check:

```bash
df -h
df -i
df -h /tmp
sudo du -xhd1 /var/lib/jenkins | sort -h
```

Look for:

- Workspace growth
- Artifact retention
- Dependency cache
- Temporary files
- Inode exhaustion

---

## 25. Commands to Memorize

```bash
lsblk
lsblk -f
sudo fdisk -l
df -h
df -i
df -T
du -sh DIRECTORY
du -xhd1 DIRECTORY
findmnt
findmnt /data
sudo blkid
sudo lsof +L1
ulimit -n
cat /proc/sys/fs/file-nr
sudo logrotate -d /etc/logrotate.conf
sudo logrotate -f /etc/logrotate.conf
journalctl --disk-usage
sudo journalctl --vacuum-time=14d
sudo dmesg -T
systemctl --failed
```

---

## 26. Interview Questions

### Q1. What is the difference between `df` and `du`?

`df` reports filesystem-level usage. `du` reports space consumed by files and directories.

### Q2. Can disk space be available while the filesystem is unusable?

Yes. Inodes may be exhausted, the filesystem may be read-only, or file-descriptor/resource limits may be reached.

### Q3. Why can `df` show more usage than `du`?

Deleted files may still be open, or the commands may be measuring different filesystems.

### Q4. How do you find deleted files still consuming space?

```bash
sudo lsof +L1
```

### Q5. What is log rotation?

A mechanism that renames, compresses, retains and removes old logs to prevent uncontrolled disk growth.

### Q6. What is the drawback of `copytruncate`?

A small amount of log data may be lost during the copy/truncate window.

### Q7. Does increasing an EBS volume automatically expand the filesystem?

No. The block device, partition and filesystem may each require separate expansion steps.

### Q8. What is inode exhaustion?

The filesystem has no free inode entries for new files, even if byte capacity remains.

### Q9. Why should application data be separated from the root filesystem?

To prevent high-growth data such as logs, uploads or databases from filling the OS filesystem and affecting the entire server.

### Q10. What storage metrics should DevOps monitor?

Capacity, available bytes, inode usage, I/O throughput, IOPS, latency, queue depth, file descriptors and log growth.

---

## Final DevOps Checklist

- [ ] Check `df -h` and `df -i`.
- [ ] Use `du` to identify space consumers.
- [ ] Verify mount points with `findmnt`.
- [ ] Check for deleted open files.
- [ ] Monitor `/tmp` and `/dev/shm`.
- [ ] Validate EBS device names before formatting.
- [ ] Test `/etc/fstab` changes.
- [ ] Configure log rotation and retention.
- [ ] Monitor Jenkins workspaces and artifacts.
- [ ] Separate high-growth application data where appropriate.
- [ ] Monitor filesystem, inode and I/O metrics.
- [ ] Preserve logs and evidence during incidents.

## Key DevOps Principle

> Storage is an application dependency. A production deployment is not reliable unless capacity, inodes, mounts, log growth, retention, I/O performance and recovery procedures are managed together.
