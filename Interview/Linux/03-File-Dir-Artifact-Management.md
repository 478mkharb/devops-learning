# File, Directory and Artifact Management for DevOps

## Scope

This topic covers file operations required for deployments, CI/CD pipelines, Ansible automation, application packaging, backups, and production troubleshooting.

---

## 1. How do you list files?

```bash
ls
ls -l
ls -lah
```

Useful options:

- `-l`: long format
- `-a`: include hidden files
- `-h`: human-readable sizes
- `-t`: sort by modification time
- `-S`: sort by size

For deployment troubleshooting:

```bash
ls -lah /opt/myapp
ls -lt /var/log/myapp
```

---

## 2. How do you copy files?

```bash
cp app.conf /etc/myapp/
cp -r release/ /opt/myapp/
```

Preserve metadata:

```bash
cp -a source/ destination/
```

`-a` preserves permissions, ownership where permitted, timestamps, and symbolic links.

For large deployments, `rsync` is usually more suitable than repeated `cp`.

---

## 3. How do you move or rename files?

```bash
mv old.conf new.conf
mv release-v2 /opt/myapp/releases/
```

A move within the same filesystem is usually a metadata operation rather than copying all file contents.

Use caution when overwriting:

```bash
mv -i source destination
```

---

## 4. How do you remove files safely?

```bash
rm file.txt
rm -r directory/
```

Interactive mode:

```bash
rm -ri directory/
```

Never use destructive commands with an unvalidated variable:

```bash
rm -rf "$TARGET"/*
```

First validate:

```bash
test -n "$TARGET"
case "$TARGET" in
  /opt/myapp/releases/*) ;;
  *) echo "Unsafe target"; exit 1 ;;
esac
```

Avoid `rm -rf /` or broad wildcard cleanup in automation.

---

## 5. What is `rsync` and why is it useful in DevOps?

`rsync` synchronizes files efficiently.

```bash
rsync -av ./build/ user@server:/opt/myapp/
```

Common options:

- `-a`: archive mode
- `-v`: verbose
- `-z`: compress during transfer
- `--delete`: remove destination files absent from source
- `--dry-run`: show changes without applying them

Preview first:

```bash
rsync -av --dry-run ./build/ /opt/myapp/
```

Use `--delete` carefully because it can remove destination files.

---

## 6. What is the difference between `scp` and `rsync`?

| Feature | `scp` | `rsync` |
|---|---|---|
| Simple copy | Excellent | Excellent |
| Incremental transfer | Limited | Yes |
| Resume efficiency | Limited | Better |
| Dry run | No | Yes |
| Synchronization | Basic | Strong |
| Deployment suitability | Small transfers | Repeated deployments |

Example:

```bash
scp app.jar ubuntu@server:/opt/app/
rsync -av build/ ubuntu@server:/opt/app/
```

---

## 7. How do you create files and directories?

```bash
touch app.log
mkdir app
mkdir -p /opt/myapp/{releases,shared,config}
```

`mkdir -p` creates parent directories when needed and does not fail if the directory already exists.

This makes it useful in idempotent automation.

---

## 8. How do you find files?

```bash
find /opt/myapp -type f
find /var/log -name '*.log'
find /tmp -type f -mtime +7
find /opt/myapp -type f -size +500M
```

Useful predicates:

- `-type f`: regular file
- `-type d`: directory
- `-name`: filename pattern
- `-mtime`: modification age in days
- `-mmin`: modification age in minutes
- `-size`: file size
- `-user`: owner
- `-perm`: permissions

---

## 9. How do you find files changed in the last hour?

```bash
find /opt/myapp -type f -mmin -60
```

This is useful after:

- A deployment
- Configuration change
- Unexpected file modification
- Malware investigation
- Log-generation troubleshooting

---

## 10. How do you search inside files?

```bash
grep -n "ERROR" app.log
grep -Rni "database connection" /opt/myapp/
```

Useful options:

- `-n`: line number
- `-i`: case-insensitive
- `-R`: recursive
- `-v`: invert match
- `-E`: extended regular expressions

Avoid searching secrets indiscriminately. Restrict paths and protect command output in CI logs.

---

## 11. What is the difference between `grep`, `egrep`, and `awk`?

- `grep`: pattern matching
- `grep -E`: extended regular expressions
- `awk`: field-based processing and reporting

Examples:

```bash
grep -i 'failed' app.log
grep -E 'ERROR|CRITICAL' app.log
awk '{print $1, $5}' access.log
```

Modern scripts should prefer `grep -E` rather than the older `egrep` command.

---

## 12. What is `sed` used for?

`sed` performs stream editing.

Replace text:

```bash
sed 's/old-value/new-value/g' app.conf
```

Edit a file in place:

```bash
sed -i 's/^PORT=.*/PORT=8080/' app.env
```

Always back up or version-control configuration before automated replacements.

For YAML, JSON, or complex configuration, use a format-aware tool rather than fragile text replacement.

---

## 13. What is `awk` used for?

`awk` is useful for structured text processing.

```bash
awk '{print $1}' access.log
```

Filter records:

```bash
awk '$9 >= 500 {print $0}' access.log
```

Calculate values:

```bash
awk '{sum += $10} END {print sum}' access.log
```

It is useful for:

- Log analysis
- CSV-like files
- Metrics extraction
- Quick operational reports

---

## 14. What is `cut`?

`cut` extracts columns or character ranges.

```bash
cut -d: -f1 /etc/passwd
cut -d, -f2 employees.csv
```

It works best with simple, consistently delimited data. For quoted CSV containing commas inside fields, use a CSV-aware parser.

---

## 15. What is `sort`, `uniq`, and `wc`?

```bash
sort names.txt
sort names.txt | uniq
wc -l app.log
wc -c file.bin
```

Count frequent values:

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
```

This can identify top client IPs or frequently requested endpoints.

---

## 16. What is `tar`?

`tar` packages files and directories.

Create archive:

```bash
tar -czf release.tar.gz release/
```

Extract:

```bash
tar -xzf release.tar.gz
```

List contents:

```bash
tar -tzf release.tar.gz
```

Common formats:

- `.tar`: archive only
- `.tar.gz`: gzip compression
- `.tar.bz2`: bzip2 compression
- `.tar.xz`: xz compression

Validate archive contents before extracting into production directories.

---

## 17. What is the difference between `tar` and `zip`?

| Feature | `tar` | `zip` |
|---|---|---|
| Unix metadata | Strong | Variable |
| Linux packaging | Common | Common for cross-platform use |
| Compression | Separate option | Built in |
| Deployment archives | Very common | Useful for portability |

Examples:

```bash
tar -czf app.tar.gz app/
zip -r app.zip app/
```

---

## 18. How do you calculate checksums?

```bash
sha256sum app.tar.gz
md5sum app.tar.gz
```

SHA-256 is preferred for integrity verification.

Verify:

```bash
sha256sum -c checksums.sha256
```

Checksums help confirm:

- Artifact was not corrupted
- Download completed correctly
- Expected binary was deployed
- Two files are identical

A checksum alone does not prove authenticity unless the checksum itself is trusted.

---

## 19. What is an artifact?

An artifact is a build output that can be stored, promoted, deployed, or released.

Examples:

- JAR file
- Python wheel
- Docker image reference
- React build directory
- Terraform package
- Helm chart
- ZIP or TAR archive
- Test report

Good artifact practices:

- Immutable versioning
- Build metadata
- Checksums
- Traceability to Git commit
- Retention policy
- Promotion across environments

---

## 20. What is the difference between source code and an artifact?

Source code is the input to a build.

An artifact is the resulting deployable output.

Example:

```text
Git repository → Maven build → app.jar → deployment
```

A production deployment should preferably use the same tested artifact rather than rebuilding separately in every environment.

---

## 21. What is an immutable artifact?

An immutable artifact is not modified after publication.

Bad:

```text
app-latest.jar
```

where the same filename is overwritten repeatedly.

Better:

```text
attendance-api-1.4.2.jar
attendance-api-git-a81f9c2.jar
```

Immutable artifacts improve:

- Rollbacks
- Auditability
- Reproducibility
- Incident investigation
- Environment consistency

---

## 22. How do you compare two files?

```bash
diff -u old.conf new.conf
cmp file1 file2
sha256sum file1 file2
```

For directories:

```bash
diff -ruN release-v1/ release-v2/
```

Use `diff` to inspect configuration changes before restarting services.

---

## 23. How do you preserve permissions while copying?

```bash
cp -p file1 file2
cp -a source/ destination/
rsync -a source/ destination/
```

Be aware that ownership preservation may require root privileges.

After deployment, verify:

```bash
stat /opt/myapp/app.jar
namei -l /opt/myapp/app.jar
```

---

## 24. What is atomic file replacement?

Atomic replacement means readers see either the old complete file or the new complete file, not a partially written file.

A common pattern:

```bash
tmp=$(mktemp)
generate_config > "$tmp"
chmod 0644 "$tmp"
mv -f "$tmp" /etc/myapp/app.conf
```

The `mv` is atomic when source and destination are on the same filesystem.

This is safer than writing directly to a live configuration file.

---

## 25. How do you safely update a symlink?

```bash
ln -sfn /opt/myapp/releases/v2 /opt/myapp/current
```

Validate first:

```bash
test -x /opt/myapp/releases/v2/bin/start
```

Then switch and health-check:

```bash
ln -sfn /opt/myapp/releases/v2 /opt/myapp/current
systemctl restart myapp
curl -f http://127.0.0.1:8080/health
```

If health checks fail, roll back to the previous release.

---

## 26. What is a sparse file?

A sparse file has logical size larger than its actual disk allocation.

Create one:

```bash
truncate -s 10G sparse.img
```

Compare:

```bash
ls -lh sparse.img
du -h sparse.img
```

`ls` reports logical size, while `du` reports allocated blocks.

Sparse files matter for:

- VM images
- Database files
- Disk images
- Backup storage
- Container layers

---

## 27. What is a deleted-but-open file?

A process may keep an open file after its directory entry is deleted.

Find such files:

```bash
sudo lsof +L1
```

The disk space is released when the process closes the file descriptor.

Typical fix:

- Reload or restart the responsible service
- Configure proper log rotation
- Ensure applications reopen rotated logs

---

## 28. How do you manage temporary files in scripts?

Use:

```bash
tmpdir=$(mktemp -d)
trap 'rm -rf "$tmpdir"' EXIT
```

This provides:

- Unique temporary directory
- Cleanup on normal exit
- Reduced collision risk

For sensitive data, also consider permissions, filesystem location, and whether temporary storage is encrypted or persistent.

---

## 29. How do you transfer artifacts to an EC2 server?

Possible approaches:

- Jenkins agent running in the target network
- S3 artifact storage
- `scp` or `rsync` through a bastion
- AWS Systems Manager
- Configuration-management tooling
- Artifact repository such as Nexus or Artifactory

For production, prefer a traceable artifact repository or object storage instead of manually copying random local files.

---

## 30. Scenario: The deployed JAR is corrupted

Check:

```bash
ls -lh app.jar
file app.jar
sha256sum app.jar
jar tf app.jar | head
```

Compare the deployed checksum with the CI artifact checksum.

Possible causes:

- Interrupted transfer
- Wrong file copied
- Partial upload
- Disk full
- Proxy or repository issue
- Deployment script continued after a failed copy

Use strict error handling and verify the artifact before starting the service.

---

## 31. Scenario: The deployment copied files but the old files remain

Possible causes:

- `rsync` was used without `--delete`
- Old release directory was reused
- Deployment did not clean the destination
- Hidden files were missed
- Symlink points to an older release

Inspect:

```bash
ls -lah destination/
readlink -f /opt/myapp/current
```

Prefer versioned release directories over repeatedly overwriting a live directory.

---

## 32. Scenario: `rsync --delete` removed user uploads

This usually means code and persistent data were placed in the same directory.

Bad structure:

```text
/var/www/app/
├── code
└── uploads
```

Better:

```text
/opt/app/releases/v1
/opt/app/releases/v2
/opt/app/shared/uploads
```

Synchronize only the release directory. Keep uploads and other persistent data outside the deployment target.

---

## 33. Scenario: A script says “No such file or directory” although the file exists

Check:

```bash
pwd
ls -l ./script.sh
file ./script.sh
head -1 ./script.sh
```

Possible causes:

- Wrong working directory
- Incorrect relative path
- Missing interpreter
- Windows CRLF line endings
- Broken symlink
- Missing dynamic loader

Check line endings:

```bash
file script.sh
sed -n '1p' script.sh | cat -A
```

Convert CRLF if appropriate:

```bash
sed -i 's/\r$//' script.sh
```

---

## 34. Scenario: A deployment script deletes the wrong directory

Prevent this with:

```bash
set -Eeuo pipefail

TARGET="${TARGET:-}"

if [[ -z "$TARGET" || "$TARGET" == "/" ]]; then
  echo "Unsafe target"
  exit 1
fi

rm -rf -- "$TARGET"
```

Also validate that the path belongs to the expected deployment root.

Never trust an empty variable in a destructive command.

---

## 35. Practical command reference

```bash
ls -lah
cp -a source destination
mv source destination
rm -ri directory
mkdir -p /opt/app/{releases,shared,config}
find /opt/app -type f
grep -Rni "error" /var/log/myapp
sed -n '1,100p' file
awk '{print $1}' file
sort file | uniq -c
tar -czf release.tar.gz release/
tar -xzf release.tar.gz
sha256sum artifact
diff -u old new
rsync -av --dry-run source/ destination/
stat file
namei -l /path/to/file
mktemp
lsof +L1
```

---

## Interview checklist

You should be able to explain:

- `cp`, `mv`, `rm`, and `mkdir`
- Safe deletion in scripts
- `rsync` versus `scp`
- `find`, `grep`, `sed`, and `awk`
- `tar` and artifact packaging
- Checksums
- Immutable artifacts
- Atomic file replacement
- Symlink-based deployments
- Sparse files
- Deleted open files
- Temporary-file safety
- Artifact transfer to EC2
- Rollback design
- Why persistent data must be separated from application releases
- Why `rsync --delete` is dangerous
- How to troubleshoot corrupted or incomplete artifacts

## Key DevOps principle

> Treat build outputs as immutable, versioned artifacts; deploy them into isolated release directories and keep persistent data outside the deployment path.
