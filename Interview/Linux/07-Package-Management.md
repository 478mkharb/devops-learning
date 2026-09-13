# Package Management and Software Installation for DevOps

## Scope

This guide covers Linux package management from a **DevOps Engineer's perspective**.

The focus is on:

- Installing runtime dependencies
- Managing package versions
- Reproducible server provisioning
- Ubuntu and RHEL-based systems
- Ansible automation
- Repository management
- Package troubleshooting
- Security updates
- Application deployment dependencies

---

# 1. What is package management?

Package management is the process of installing, updating, removing and verifying software packages.

A package normally contains:

- Compiled binaries
- Libraries
- Configuration files
- Service definitions
- Documentation
- Metadata
- Dependency information

Examples of packages:

- `nginx`
- `openjdk-17-jre`
- `python3`
- `redis-server`
- `postgresql-client`
- `unzip`
- `curl`
- `git`

## DevOps relevance

Package management is used during:

- EC2 bootstrapping
- Ansible provisioning
- Packer image creation
- Jenkins agent setup
- Application deployment
- Security patching
- Golden AMI creation
- Disaster recovery

---

# 2. What are common Linux package managers?

| Distribution family | Package manager | Package format |
|---|---|---|
| Ubuntu/Debian | `apt`, `apt-get`, `dpkg` | `.deb` |
| RHEL/CentOS/Rocky/Alma | `dnf`, `yum`, `rpm` | `.rpm` |
| Alpine | `apk` | `.apk` |
| Arch | `pacman` | Arch package |

Check the operating system:

```bash
cat /etc/os-release
```

Check available commands:

```bash
command -v apt
command -v dnf
command -v rpm
```

## DevOps interview point

Package commands are distribution-specific. A provisioning script written only for Ubuntu may fail on RHEL or Amazon Linux.

---

# 3. `apt` versus `apt-get`

Both work with Debian-based packages.

## `apt`

Designed for interactive use:

```bash
sudo apt update
sudo apt install nginx
sudo apt remove nginx
```

## `apt-get`

Common in automation and scripts:

```bash
sudo apt-get update
sudo apt-get install -y nginx
```

## Practical rule

Use:

- `apt` for interactive administration
- `apt-get` for scripts and automation where stable command behavior is preferred

Never assume that `apt update` installs updates. It refreshes package metadata.

---

# 4. What does `apt update` do?

```bash
sudo apt update
```

It downloads current package metadata from configured repositories.

It does not normally upgrade installed packages.

Check available upgrades:

```bash
apt list --upgradable
```

Upgrade packages:

```bash
sudo apt upgrade
```

For automation:

```bash
sudo apt-get update
sudo apt-get upgrade -y
```

## Common mistake

This command:

```bash
sudo apt update
```

does not mean:

```text
Update every installed application
```

It means:

```text
Refresh the local package index
```

---

# 5. Install, remove and purge packages

Install:

```bash
sudo apt install nginx
```

Install without interactive confirmation:

```bash
sudo apt-get install -y nginx
```

Remove the package but usually keep configuration:

```bash
sudo apt remove nginx
```

Remove package and configuration:

```bash
sudo apt purge nginx
```

Remove unused dependencies:

```bash
sudo apt autoremove
```

## DevOps caution

Do not run `autoremove` blindly on production systems. Review the proposed package list first.

---

# 6. Search for packages

Search package names and descriptions:

```bash
apt search nginx
```

Show package information:

```bash
apt show nginx
```

List installed packages:

```bash
dpkg -l
```

Check whether a package is installed:

```bash
dpkg -s nginx
```

Find which package owns a file:

```bash
dpkg -S /usr/sbin/nginx
```

List files installed by a package:

```bash
dpkg -L nginx
```

---

# 7. Installing a specific package version

List available versions:

```bash
apt-cache policy nginx
```

Install a specific version:

```bash
sudo apt-get install nginx=1.24.0-2ubuntu7
```

Check installed version:

```bash
nginx -v
dpkg-query -W nginx
```

## Why version pinning matters

Without version control, a provisioning run may install a newer package tomorrow than it installed today.

This can cause:

- Unexpected behavior changes
- Compatibility problems
- Different application runtimes
- Failed builds
- Difficult rollback

For reproducible environments, define and control versions.

---

# 8. Package pinning and holding

Hold a package:

```bash
sudo apt-mark hold nginx
```

View held packages:

```bash
apt-mark showhold
```

Remove hold:

```bash
sudo apt-mark unhold nginx
```

Use holds carefully. A held package may miss important security updates.

## Better DevOps approach

Use:

- Tested package versions
- Golden images
- Configuration management
- Repository snapshots where required
- Automated validation
- Controlled promotion between environments

---

# 9. Updating packages safely

Before updating:

```bash
apt list --upgradable
```

Review package changes:

```bash
sudo apt-get -s upgrade
```

The `-s` option simulates the operation.

Update:

```bash
sudo apt-get upgrade -y
```

For broader dependency changes:

```bash
sudo apt-get dist-upgrade
```

or:

```bash
sudo apt-get full-upgrade
```

## Production practice

For critical servers:

1. Test updates in a lower environment.
2. Take a backup or snapshot when appropriate.
3. Patch a small batch first.
4. Validate services.
5. Continue gradually.
6. Monitor errors and performance.

---

# 10. Security updates

List available security updates:

```bash
apt list --upgradable 2>/dev/null
```

Check unattended-upgrades status:

```bash
systemctl status unattended-upgrades
```

Install security update tooling:

```bash
sudo apt-get install unattended-upgrades
```

Security patching should be coordinated with:

- Maintenance windows
- Application compatibility
- Reboot requirements
- Vulnerability severity
- Rollback plans
- Compliance requirements

A package update is not complete if a required reboot or service restart was skipped.

---

# 11. Detect whether a reboot is required

On Ubuntu:

```bash
test -f /var/run/reboot-required && echo "Reboot required"
```

Show affected packages:

```bash
cat /var/run/reboot-required.pkgs
```

In production, do not reboot automatically without considering:

- Load balancer draining
- Instance replacement
- Cluster quorum
- Stateful workloads
- Maintenance windows
- Monitoring suppression

For immutable infrastructure, replacing an instance with a new tested image may be safer than patching it in place.

---

# 12. RHEL-family package management with `dnf`

Install:

```bash
sudo dnf install nginx
```

Update metadata:

```bash
sudo dnf makecache
```

Update packages:

```bash
sudo dnf upgrade
```

Remove:

```bash
sudo dnf remove nginx
```

Search:

```bash
dnf search nginx
```

Package information:

```bash
dnf info nginx
```

List installed packages:

```bash
rpm -qa
```

Check package:

```bash
rpm -qi nginx
```

List files:

```bash
rpm -ql nginx
```

Find package owning a file:

```bash
rpm -qf /usr/sbin/nginx
```

---

# 13. What is the difference between `dpkg` and `apt`?

| Tool | Responsibility |
|---|---|
| `dpkg` | Low-level `.deb` package installation and database |
| `apt` | Higher-level dependency resolution and repository management |

For example:

```bash
sudo dpkg -i package.deb
```

may fail if dependencies are missing.

Then:

```bash
sudo apt-get -f install
```

attempts to fix missing dependencies.

## Interview point

Use `apt` for normal package installation. Use `dpkg` for low-level inspection or manual package handling.

---

# 14. Installing a local `.deb` package

Modern Ubuntu:

```bash
sudo apt install ./myapp_1.0.0_amd64.deb
```

Alternative:

```bash
sudo dpkg -i myapp_1.0.0_amd64.deb
sudo apt-get -f install
```

Inspect package contents:

```bash
dpkg-deb -c myapp_1.0.0_amd64.deb
```

Inspect metadata:

```bash
dpkg-deb -I myapp_1.0.0_amd64.deb
```

Verify architecture:

```bash
dpkg-deb -f myapp_1.0.0_amd64.deb Architecture
```

---

# 15. Installing a local `.rpm` package

Inspect:

```bash
rpm -qip myapp.rpm
```

Install with dnf:

```bash
sudo dnf install ./myapp.rpm
```

Install directly with rpm:

```bash
sudo rpm -ivh myapp.rpm
```

Upgrade:

```bash
sudo rpm -Uvh myapp.rpm
```

Use `dnf` when possible because it can resolve dependencies.

---

# 16. Repository configuration

Ubuntu repository configuration is commonly under:

```text
/etc/apt/sources.list
/etc/apt/sources.list.d/
```

Inspect:

```bash
grep -Rhv '^#' /etc/apt/sources.list /etc/apt/sources.list.d/ 2>/dev/null
```

RHEL-family repositories can be inspected with:

```bash
dnf repolist
```

Repository problems can cause:

- Package not found
- GPG signature errors
- Slow installation
- Version mismatch
- Dependency conflicts
- Installation failure

Do not bypass signature verification to “make it work.”

---

# 17. GPG signatures and package trust

Package managers verify repository metadata and package signatures.

A signature helps verify:

- Package authenticity
- Repository trust
- Integrity of downloaded metadata

Never solve a package-signature problem by blindly disabling verification.

Investigate:

- Incorrect system time
- Expired signing key
- Wrong repository
- MITM or proxy issue
- Corrupt metadata
- Unsupported distribution release

---

# 18. Package cache and cleanup

APT cache location:

```text
/var/cache/apt/archives/
```

Show cache usage:

```bash
du -sh /var/cache/apt
```

Clean downloaded package files:

```bash
sudo apt-get clean
```

Remove obsolete package files:

```bash
sudo apt-get autoclean
```

Use cleanup carefully on systems where packages may be needed for offline recovery.

---

# 19. Package installation in Ansible

Example:

```yaml
- name: Install required packages
  ansible.builtin.apt:
    name:
      - curl
      - unzip
      - nginx
      - python3
    state: present
    update_cache: true
    cache_valid_time: 3600
  become: true
```

For a specific version:

```yaml
- name: Install pinned nginx version
  ansible.builtin.apt:
    name: nginx=1.24.0-2ubuntu7
    state: present
  become: true
```

RHEL-family example:

```yaml
- name: Install packages on RHEL family
  ansible.builtin.dnf:
    name:
      - curl
      - nginx
    state: present
  become: true
```

## Idempotency

Ansible should be safe to run repeatedly. It should not reinstall or change packages unnecessarily when the desired state is already present.

---

# 20. Package installation in Packer

A Packer provisioner may install baseline software:

```bash
sudo apt-get update
sudo DEBIAN_FRONTEND=noninteractive apt-get install -y \
  awscli \
  curl \
  unzip \
  nginx \
  python3 \
  python3-venv
```

Good image-building practices:

- Pin important versions
- Remove temporary files
- Disable interactive prompts
- Clean package caches where appropriate
- Validate installed software
- Avoid embedding secrets
- Record image build metadata
- Test the resulting AMI

Example validation:

```bash
nginx -v
python3 --version
aws --version
systemctl is-enabled nginx
```

---

# 21. Non-interactive installation

Automation should not wait for a prompt.

For Debian-based systems:

```bash
export DEBIAN_FRONTEND=noninteractive
sudo -E apt-get install -y tzdata
```

Use non-interactive mode carefully. Some packages require configuration decisions that should be explicitly managed.

For configuration-heavy packages, use:

- Preseed/debconf
- Ansible templates
- Environment variables
- Explicit config files
- Post-install validation

---

# 22. Package installation versus application installation

Installing a package:

```bash
sudo apt install nginx
```

usually provides a distribution-managed version.

Installing an application artifact may involve:

- Downloading a JAR
- Copying a Python virtual environment
- Extracting a release archive
- Installing a Go binary
- Pulling a container image
- Deploying a static frontend bundle

Do not confuse operating-system dependencies with application release artifacts.

Example:

```text
OS packages:
  Java runtime
  NGINX
  curl
  system libraries

Application artifact:
  salary-api.jar
  frontend build
  notification-api release
```

---

# 23. Why blindly installing `latest` is risky

Examples:

```bash
pip install package
npm install package
apt install package
```

Without version controls, builds may change over time.

Risks:

- Breaking changes
- Supply-chain risk
- Inconsistent environments
- Non-reproducible deployments
- Difficult incident reproduction

Use appropriate version controls:

- APT package versions
- `requirements.txt`
- `package-lock.json`
- `go.mod`
- Maven dependency management
- Terraform provider lock files
- Container image digests

---

# 24. Troubleshooting: Package not found

Check:

```bash
cat /etc/os-release
sudo apt update
apt-cache policy <package>
apt search <package>
```

Possible causes:

- Repository metadata is stale
- Package name is incorrect
- Distribution release is unsupported
- Required repository is disabled
- Architecture is unsupported
- Package exists under another name
- Package is not available in the configured repositories

Do not immediately download random binaries from the internet.

---

# 25. Troubleshooting: Broken dependencies

Check package state:

```bash
dpkg --audit
```

Attempt repair:

```bash
sudo apt-get -f install
```

Inspect package status:

```bash
dpkg -l | less
```

For RHEL-family systems:

```bash
sudo dnf check
```

A broken package database may be caused by:

- Interrupted installation
- Disk full
- Power loss
- Conflicting repositories
- Manual file replacement
- Incomplete upgrade

Take care when repairing production systems.

---

# 26. Troubleshooting: Package manager is locked

Example error:

```text
Could not get lock /var/lib/dpkg/lock-frontend
```

Find package-management processes:

```bash
ps -ef | grep -E '[a]pt|[d]pkg|[u]nattended'
```

Check active locks:

```bash
sudo lsof /var/lib/dpkg/lock-frontend
sudo lsof /var/lib/dpkg/lock
```

Do not delete lock files immediately.

Correct approach:

1. Determine whether another package operation is active.
2. Wait if it is legitimate.
3. Inspect the process if it appears stuck.
4. Repair package state only after confirming no package operation is running.

---

# 27. Troubleshooting: Package installed but command not found

Check package files:

```bash
dpkg -L <package>
```

Find executable:

```bash
command -v <command>
find /usr /opt -type f -name '<command>' 2>/dev/null
```

Check PATH:

```bash
echo "$PATH"
```

Possible causes:

- Binary is not in PATH
- Package installs a differently named executable
- Shell hash is stale
- Package installation failed
- Command is under `/usr/sbin`
- Service exists but client binary does not

Refresh shell command lookup:

```bash
hash -r
```

---

# 28. Troubleshooting: Service fails after package upgrade

Investigate:

```bash
systemctl status nginx
journalctl -u nginx -b --no-pager
nginx -t
```

Check package history:

```bash
grep -i nginx /var/log/apt/history.log
```

Possible causes:

- Configuration syntax changed
- Deprecated directive
- Library incompatibility
- Permission changes
- Port conflict
- Service unit changed
- Required module removed

Use staged upgrades and retain rollback options.

---

# 29. Package ownership and file conflicts

Find the package owning a file:

```bash
dpkg -S /usr/bin/curl
```

Check whether a file was modified:

```bash
dpkg-query -W -f='${Conffiles}\n' nginx
```

On RPM systems:

```bash
rpm -qf /usr/bin/curl
rpm -V nginx
```

File conflicts often indicate:

- Multiple installation methods
- Manual binaries mixed with packages
- Conflicting repositories
- Old software not fully removed

Choose one clear installation source for each production component where possible.

---

# 30. Multiple installation methods: avoid ambiguity

A server may contain:

- APT-installed Java
- SDKMAN-installed Java
- Manually extracted Java
- Snap-installed tools
- `/usr/local/bin` binaries
- Application-bundled runtimes

This can create version confusion.

Inspect:

```bash
which java
readlink -f "$(command -v java)"
java -version
echo "$PATH"
```

For Python:

```bash
which python3
python3 --version
python3 -m pip --version
```

For Node.js:

```bash
which node
node --version
```

Document the installation source and expected version.

---

# 31. Package updates and immutable infrastructure

There are two common models.

## Mutable server model

Packages are updated on an existing server:

```text
EC2 instance
  -> apt update
  -> apt upgrade
  -> restart services
```

## Immutable image model

A new image is built and deployed:

```text
Packer
  -> install tested packages
  -> validate
  -> create AMI
  -> launch new EC2
  -> drain old EC2
```

Immutable infrastructure often improves:

- Reproducibility
- Rollback
- Drift control
- Disaster recovery
- Auditability

Mutable patching may still be required for urgent security fixes.

---

# 32. Scenario: Ansible package task is not idempotent

Bad pattern:

```yaml
- name: Install nginx
  ansible.builtin.shell: apt-get install -y nginx
```

Better:

```yaml
- name: Ensure nginx is installed
  ansible.builtin.apt:
    name: nginx
    state: present
    update_cache: true
  become: true
```

The module understands package state and can report whether a change was required.

Use shell commands only when a package module cannot express the required operation.

---

# 33. Scenario: Package installation succeeds but application fails

Check:

```bash
java -version
python3 --version
ldd /path/to/binary
systemctl status myapp
journalctl -u myapp -n 100
```

Possible causes:

- Wrong runtime version
- Missing shared library
- Incorrect architecture
- Missing environment variable
- File permission issue
- Package installed in a different path
- Application expects a different dependency version

A successful package installation is not proof that the application is compatible.

---

# 34. Scenario: Disk fills during package installation

Check:

```bash
df -h
df -i
du -sh /var/cache/apt
du -sh /var/lib/dpkg
```

Possible causes:

- Package cache
- Large logs
- Incomplete downloads
- Insufficient inode count
- Temporary files
- Old kernels
- Large application artifacts

Do not delete package database files such as `/var/lib/dpkg/status`.

---

# 35. Interview questions

1. What is package management?
2. What is the difference between `apt`, `apt-get` and `dpkg`?
3. What does `apt update` do?
4. What is the difference between `apt upgrade` and `apt full-upgrade`?
5. How do you install a specific package version?
6. How do you hold a package version?
7. How do you identify which package owns a file?
8. How do you list files installed by a package?
9. How do you troubleshoot a package-not-found error?
10. How do you troubleshoot broken dependencies?
11. Why should lock files not be deleted blindly?
12. How do you install packages non-interactively?
13. How do you install packages through Ansible?
14. Why is using `shell: apt-get install` less desirable than `ansible.builtin.apt`?
15. How do package repositories affect reproducibility?
16. What are package signatures and why are they important?
17. What is the difference between mutable and immutable infrastructure?
18. How do you verify a package after installation?
19. How do you troubleshoot a service that fails after a package upgrade?
20. Why should you avoid mixing multiple installation methods?
21. How do you detect whether a reboot is required?
22. How would you install baseline software in a Packer image?
23. How do you handle urgent security patches?
24. What is the difference between an OS package and an application artifact?
25. Why is installing unpinned `latest` software risky?

---

# 36. Interview checklist

- [ ] Identify the Linux distribution
- [ ] Use `apt`, `apt-get`, `dpkg`
- [ ] Use `dnf` and `rpm`
- [ ] Search package repositories
- [ ] Install and remove packages
- [ ] Pin package versions
- [ ] Understand package holds
- [ ] Inspect package ownership
- [ ] Troubleshoot broken dependencies
- [ ] Handle package locks safely
- [ ] Install packages through Ansible
- [ ] Install packages in Packer
- [ ] Use non-interactive installation
- [ ] Understand repository trust
- [ ] Plan security patching
- [ ] Validate services after upgrades
- [ ] Distinguish OS packages from app artifacts
- [ ] Avoid installation-source ambiguity

---

# Key DevOps principle

**Software installation must be reproducible, version-controlled and validated.**

A server is not production-ready merely because a package command completed successfully.

A reliable DevOps workflow should define:

- Which repository is trusted
- Which package version is required
- Which dependencies are installed
- How installation is automated
- How the service is validated
- How rollback or replacement is performed
