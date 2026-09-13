# Linux Security for DevOps

## Scope

This guide covers Linux security from a **DevOps Engineer perspective**.

The goal is to secure application hosts, CI/CD agents, cloud instances, and production services without treating security as a separate activity performed only after deployment.

Topics include:

- Linux attack surface.
- Least privilege.
- SSH hardening.
- Users, groups, and service accounts.
- File permissions and secrets.
- Sudo and privilege escalation.
- Package and patch management.
- Firewalling.
- Process and service hardening.
- Audit evidence.
- Jenkins, Ansible, EC2, SSM, NGINX, and systemd security.
- Incident-response commands and scenarios.

---

# 1. DevSecOps Security Mindset

Security should be integrated into the delivery lifecycle:

```text
Code
  ↓
Dependency and secret scanning
  ↓
Build
  ↓
Artifact scanning
  ↓
Infrastructure provisioning
  ↓
Host hardening
  ↓
Secure deployment
  ↓
Runtime monitoring
  ↓
Incident response
```

## Core principles

1. Least privilege.
2. Defense in depth.
3. Secure defaults.
4. Minimize exposed services.
5. Patch known vulnerabilities.
6. Separate duties.
7. Never store secrets in source code.
8. Prefer short-lived credentials.
9. Log security-relevant actions.
10. Verify that security controls actually work.

Security is not only confidentiality. It also includes:

- **Confidentiality:** Prevent unauthorized access.
- **Integrity:** Prevent unauthorized modification.
- **Availability:** Keep services operational.
- **Authenticity:** Verify identities and sources.
- **Accountability:** Record who performed an action.

---

# 2. Linux Attack Surface

An attack surface includes everything that can be used to enter, control, or abuse a system.

## Common attack-surface areas

- Open network ports.
- SSH.
- Web servers.
- Application APIs.
- Outdated packages.
- Weak credentials.
- Excessive sudo permissions.
- World-writable files.
- Exposed secrets.
- Unsafe systemd services.
- CI/CD agents.
- Cloud instance metadata access.
- Unrestricted outbound traffic.
- Writable application directories.
- Untrusted deployment artifacts.

## Identify listening services

```bash
sudo ss -tulpen
```

List processes listening on TCP/UDP ports:

```bash
sudo lsof -i -P -n
```

List enabled services:

```bash
systemctl list-unit-files --type=service --state=enabled
```

List running services:

```bash
systemctl --type=service --state=running
```

Security questions:

- Is this service required?
- Who should access it?
- Is it bound to `0.0.0.0` unnecessarily?
- Is it protected by a firewall or security group?
- Does it run as root?
- Is it patched?
- Does it expose sensitive information?

---

# 3. Users, Groups, and Service Accounts

## 3.1 Inspect identity

```bash
id
whoami
who
w
last
```

Inspect a user:

```bash
id ubuntu
getent passwd ubuntu
getent group sudo
```

## 3.2 Service-account principles

A service should normally run as a dedicated non-login user.

Example:

```bash
sudo useradd \
  --system \
  --home-dir /var/lib/myapp \
  --create-home \
  --shell /usr/sbin/nologin \
  myapp
```

Verify:

```bash
getent passwd myapp
```

Why use a dedicated service account?

- Limits filesystem access.
- Makes ownership clear.
- Improves auditability.
- Reduces blast radius.
- Prevents one application from accessing another application's files.

## 3.3 Avoid shared accounts

Poor practice:

```text
Multiple engineers use root or one shared deployment account.
```

Better:

- Individual named accounts.
- SSH keys or centrally managed identity.
- Sudo for approved actions.
- Logged administrative activity.
- Access removed when no longer required.

## 3.4 Review privileged users

```bash
getent group sudo
getent group adm
getent group docker
```

On some distributions:

```bash
getent group wheel
```

Find users with login shells:

```bash
awk -F: '$7 !~ /(nologin|false)$/ {print $1, $7}' /etc/passwd
```

---

# 4. Least Privilege and Sudo

## 4.1 Check sudo access

```bash
sudo -l
```

Review sudoers safely:

```bash
sudo visudo
```

List configuration files:

```bash
sudo ls -l /etc/sudoers.d/
```

Never edit `/etc/sudoers` using a normal text editor. Use:

```bash
sudo visudo
```

For a specific file:

```bash
sudo visudo -f /etc/sudoers.d/myapp
```

## 4.2 Dangerous sudo permissions

These may enable privilege escalation if granted broadly:

```text
sudo bash
sudo sh
sudo su
sudo vim
sudo less
sudo find
sudo python
sudo perl
sudo systemctl
sudo docker
```

The exact risk depends on arguments, environment, binary behavior, and configuration.

## 4.3 Safer sudo design

Prefer narrowly scoped commands:

```text
deploy ALL=(root) /bin/systemctl restart myapp
```

Even this should be reviewed carefully.

A safer design may use:

- A controlled wrapper script.
- Fixed arguments.
- Root-owned script.
- Non-writable parent directories.
- Restricted environment.
- Logging.
- No user-controlled command interpolation.

## 4.4 Sudo security checks

```bash
sudo grep -R --line-number -E 'NOPASSWD|ALL=\(ALL|ALL=\(root\)' \
  /etc/sudoers /etc/sudoers.d 2>/dev/null
```

Do not assume every `NOPASSWD` entry is unsafe, but review it carefully.

---

# 5. SSH Security

SSH is a major administrative entry point.

## 5.1 Inspect SSH configuration

```bash
sudo sshd -T
```

Check the effective configuration:

```bash
sudo sshd -T | grep -E \
  'permitrootlogin|passwordauthentication|pubkeyauthentication|x11forwarding|allowusers|allowgroups|maxauthtries'
```

Check syntax before reload:

```bash
sudo sshd -t
```

## 5.2 Recommended baseline

Typical production preferences:

```text
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
X11Forwarding no
MaxAuthTries 3
```

These are recommendations, not universal values. Confirm that key-based access and recovery access work before disabling passwords.

## 5.3 Apply SSH changes safely

1. Open a second SSH session.
2. Keep the existing session active.
3. Validate configuration:

```bash
sudo sshd -t
```

4. Reload SSH:

```bash
sudo systemctl reload ssh
```

5. Test a new connection.
6. Only then close the old session.

A syntax error or incorrect access rule can lock out administrators.

## 5.4 Review authorized keys

```bash
sudo find /home -path '*/.ssh/authorized_keys' -type f -ls
```

Inspect a user's keys:

```bash
cat ~/.ssh/authorized_keys
```

Security checks:

- Remove unused keys.
- Avoid shared private keys.
- Use correct ownership.
- Restrict permissions.
- Rotate keys.
- Record key ownership and purpose.

Typical permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

## 5.5 SSH key permissions

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

Private keys must not be world-readable.

## 5.6 SSH logs

Ubuntu/Debian:

```bash
sudo journalctl -u ssh --since "1 hour ago"
sudo grep -i ssh /var/log/auth.log | tail -50
```

RHEL-like systems may use:

```bash
sudo journalctl -u sshd
sudo tail -50 /var/log/secure
```

Look for:

- Repeated failed logins.
- Unknown users.
- Unexpected source addresses.
- Successful login outside maintenance windows.
- Key changes.
- Brute-force patterns.

---

# 6. File Permissions and Ownership

## 6.1 Inspect permissions

```bash
ls -l
stat file.txt
namei -l /path/to/file
```

`namei -l` is useful because access depends on every directory in the path.

## 6.2 Dangerous permission patterns

Find world-writable files:

```bash
sudo find / -xdev -type f -perm -0002 -ls
```

Find world-writable directories:

```bash
sudo find / -xdev -type d -perm -0002 -ls
```

Find SUID files:

```bash
sudo find / -xdev -perm -4000 -type f -ls
```

Find SGID files:

```bash
sudo find / -xdev -perm -2000 -type f -ls
```

These commands may produce many legitimate results. Review rather than blindly delete or change files.

## 6.3 Application directory ownership

A common secure layout:

```text
/opt/myapp/                  root:root
/opt/myapp/releases/         root:root
/opt/myapp/current           root:root or controlled symlink
/var/lib/myapp/              myapp:myapp
/var/log/myapp/              myapp:adm
/etc/myapp/                  root:root
/etc/myapp/secrets.env       root:myapp, restrictive mode
```

The exact ownership depends on the application and deployment model.

## 6.4 Avoid broad permissions

Do not use as a routine fix:

```bash
chmod -R 777 /var/www
chmod -R 777 /opt/myapp
chown -R ubuntu:ubuntu /
```

These can create serious security and operational problems.

---

# 7. Secrets Management

## 7.1 Never commit secrets

Do not commit:

- Passwords.
- API keys.
- Private keys.
- Cloud access keys.
- Database connection strings with passwords.
- SMTP credentials.
- JWT signing keys.
- `.env` files containing production secrets.

Search a repository:

```bash
grep -RniE \
  'password=|secret=|api[_-]?key|access[_-]?key|private[_-]?key' \
  --exclude-dir=.git .
```

This is a basic heuristic, not a complete secret scanner.

Use dedicated tools such as:

- Gitleaks.
- TruffleHog.
- CI secret scanning.
- Cloud provider secret scanners.

## 7.2 File-based secrets

If a service must read a secret file:

```bash
sudo chown root:myapp /etc/myapp/secrets.env
sudo chmod 640 /etc/myapp/secrets.env
```

Ensure:

- Parent directories are not writable by untrusted users.
- The service account belongs to the intended group.
- Logs do not print secret values.
- Backups are encrypted and access-controlled.

## 7.3 Environment variables are not automatically secure

Environment variables can be exposed through:

```bash
/proc/<pid>/environ
```

Depending on permissions and process context.

Check:

```bash
tr '\0' '\n' < /proc/1234/environ
```

Do not place highly sensitive secrets in environments without understanding exposure risks.

Prefer:

- AWS Secrets Manager.
- AWS Systems Manager Parameter Store.
- Vault.
- Kubernetes secrets with appropriate controls.
- Short-lived credentials.
- Application-level secret retrieval.

## 7.4 Rotate exposed secrets

If a secret is exposed:

1. Revoke or rotate it.
2. Check access logs.
3. Remove it from source and history where appropriate.
4. Update the consuming service.
5. Redeploy safely.
6. Check for reuse elsewhere.
7. Document the incident.

Deleting a secret from the latest commit does not remove it from Git history.

---

# 8. Package and Patch Management

## 8.1 List installed packages

Ubuntu/Debian:

```bash
dpkg -l
apt list --upgradable
```

RHEL-like systems:

```bash
rpm -qa
dnf check-update
```

## 8.2 Apply updates

Ubuntu:

```bash
sudo apt update
sudo apt upgrade
```

For security-sensitive production systems:

- Test updates first.
- Use approved repositories.
- Pin versions where required.
- Schedule maintenance.
- Verify service behavior.
- Have rollback or recovery plans.

## 8.3 Identify reboot requirements

Ubuntu:

```bash
test -f /var/run/reboot-required && echo "Reboot required"
```

Check kernel:

```bash
uname -r
```

A package update may not fully take effect until a service or host is restarted.

## 8.4 Minimize installed software

Every installed package can add:

- Vulnerability exposure.
- Maintenance work.
- Dependencies.
- Background services.
- Configuration complexity.

Use minimal images and remove unnecessary packages, but do not remove dependencies blindly.

---

# 9. Firewall and Network Exposure

Linux firewalling is one layer. In cloud environments, also use:

- VPC security groups.
- Network ACLs.
- Load balancer controls.
- Private subnets.
- Routing controls.
- WAF.
- Identity-aware access.

## 9.1 UFW status

Ubuntu:

```bash
sudo ufw status verbose
```

Example rules:

```bash
sudo ufw allow from 10.0.0.0/16 to any port 22 proto tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw deny 8080/tcp
```

Do not enable a firewall remotely until you have verified that your management path remains allowed.

## 9.2 nftables and iptables

Inspect nftables:

```bash
sudo nft list ruleset
```

Inspect iptables where applicable:

```bash
sudo iptables -S
sudo iptables -L -n -v
```

## 9.3 DevOps firewall design

Example:

```text
Internet
   ↓
Load Balancer: 443
   ↓
Application EC2: 8080 only from Load Balancer
   ↓
Database: 5432 only from Application EC2
```

Do not expose internal application ports directly to the internet.

---

# 10. Process and Service Hardening

## 10.1 Run services as non-root

Check:

```bash
systemctl show myapp -p User -p Group
ps -o user,group,pid,cmd -C myapp
```

## 10.2 systemd security settings

Useful options include:

```ini
[Service]
User=myapp
Group=myapp
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/var/lib/myapp /var/log/myapp
CapabilityBoundingSet=
RestrictAddressFamilies=AF_UNIX AF_INET AF_INET6
```

These settings must be tested against the application's requirements.

## 10.3 Review service security

```bash
systemd-analyze security myapp.service
```

This provides a security-exposure assessment of the unit configuration.

It is not a guarantee that the application is secure.

## 10.4 Inspect service files

```bash
systemctl cat myapp
systemctl show myapp
```

Look for:

- Root execution.
- Writable executable paths.
- Unrestricted environment files.
- Weak file permissions.
- Dangerous capabilities.
- Excessive filesystem access.
- Unnecessary network access.
- Unsafe temporary directories.

---

# 11. Linux Capabilities

Linux capabilities split some root privileges into individual permissions.

Inspect a process:

```bash
getpcaps 1234
```

Inspect a binary:

```bash
getcap /usr/bin/ping
```

Find files with capabilities:

```bash
sudo getcap -r / 2>/dev/null
```

Capabilities can be useful, but excessive capabilities increase risk.

Examples include:

- `CAP_NET_BIND_SERVICE`: Bind to privileged ports.
- `CAP_NET_ADMIN`: Network administration.
- `CAP_SYS_ADMIN`: Broad and powerful; often high risk.
- `CAP_SYS_PTRACE`: Trace other processes.

Prefer the smallest required capability set.

---

# 12. Auditing and Security Evidence

## 12.1 Authentication history

```bash
last
lastb
who
w
```

`lastb` may require appropriate permissions and a configured failed-login database.

## 12.2 Journal review

```bash
sudo journalctl --since "1 hour ago"
sudo journalctl -p warning..alert
sudo journalctl -k
```

## 12.3 Auditd

If installed:

```bash
sudo systemctl status auditd
sudo ausearch -m USER_LOGIN
sudo ausearch -m USER_CMD
```

Audit rules should be designed according to security requirements. Excessive auditing can create performance and storage overhead.

## 12.4 File integrity

Useful controls include:

- AIDE.
- Package verification.
- Immutable infrastructure.
- Image signing.
- Artifact checksums.
- Configuration drift detection.

For package verification on RPM-based systems:

```bash
rpm -Va
```

Review output carefully; not every difference is malicious.

---

# 13. Security in Jenkins and CI/CD

CI/CD systems are high-value targets because they may have:

- Source-code access.
- Cloud credentials.
- Deployment permissions.
- Artifact signing keys.
- Production network access.

## Security practices

1. Use dedicated build agents.
2. Do not run builds as root.
3. Avoid sharing workspaces between trust boundaries.
4. Clean sensitive workspaces safely.
5. Restrict Jenkins credentials.
6. Mask secrets in logs.
7. Avoid printing environment variables.
8. Pin trusted build tools.
9. Scan dependencies and images.
10. Restrict outbound access where practical.
11. Separate build and deployment permissions.
12. Review Jenkins plugins.
13. Protect the Jenkins controller.
14. Use ephemeral agents when possible.
15. Require approval for production changes.

## Dangerous pipeline pattern

```groovy
sh "echo ${SECRET}"
```

This may expose secrets in logs or command arguments.

Prefer credential bindings and avoid echoing secret values.

## Workspace permissions

```bash
find "$WORKSPACE" -maxdepth 2 -ls
```

Ensure that one job cannot read another job's secrets or artifacts.

---

# 14. Security in Ansible

## Recommended practices

- Use SSH keys or SSM-based access.
- Avoid plaintext passwords.
- Use Ansible Vault for encrypted variables.
- Limit become privileges.
- Pin collection versions.
- Validate inventory sources.
- Review dynamic inventory permissions.
- Avoid shell commands when a module is sufficient.
- Do not disable host-key checking casually.
- Restrict playbook execution permissions.

Example:

```bash
ansible-vault encrypt group_vars/prod/secrets.yml
```

Run with:

```bash
ansible-playbook site.yml --ask-vault-pass
```

Never commit the vault password into Git.

---

# 15. Security in AWS EC2 and SSM

## EC2 security controls

- Use IAM instance profiles instead of static access keys.
- Prefer private subnets for internal services.
- Restrict security-group ingress.
- Use SSM instead of open SSH where practical.
- Patch AMIs and instances.
- Encrypt EBS volumes.
- Enable centralized logging.
- Restrict metadata access.
- Use least-privilege IAM policies.
- Monitor CloudTrail and GuardDuty where available.

## SSM security benefits

SSM can reduce the need for:

- Public IP addresses.
- Bastion hosts.
- Open inbound SSH.
- Shared administrative keys.

However, SSM access is still powerful. Control it through:

- IAM permissions.
- Session logging.
- Approval workflows.
- Role separation.
- Network endpoints.
- Audit trails.

## Check instance identity

```bash
aws sts get-caller-identity
```

Do not expose credentials in shell history or logs.

---

# 16. Security Scanning Commands

## Find suspicious permissions

```bash
sudo find / -xdev -type f -perm -0002 -ls
sudo find / -xdev -perm -4000 -type f -ls
sudo find / -xdev -perm -2000 -type f -ls
```

## Find private keys

```bash
sudo find /home /opt /etc -type f \
  \( -name 'id_rsa' -o -name 'id_ed25519' -o -name '*.pem' \) \
  -ls 2>/dev/null
```

Review results carefully. Some keys may be legitimate.

## Check outdated services

```bash
systemctl --failed
systemctl list-unit-files --type=service --state=enabled
sudo ss -tulpen
```

## Check suspicious processes

```bash
ps auxf
ps -eo pid,ppid,user,%cpu,%mem,lstart,cmd --sort=-%cpu | head -30
```

## Check cron jobs

```bash
crontab -l
sudo ls -la /etc/cron.d
sudo ls -la /etc/cron.daily
sudo grep -Rni '' /var/spool/cron 2>/dev/null
```

Review unexpected scheduled tasks.

---

# 17. Incident Response: Suspected Compromise

If compromise is suspected, do not immediately destroy evidence.

## Initial steps

1. Confirm the alert and scope.
2. Record time and affected hosts.
3. Preserve logs and relevant evidence.
4. Restrict access according to the incident plan.
5. Notify the responsible security or incident team.
6. Rotate potentially exposed credentials.
7. Isolate or replace the host when authorized.
8. Rebuild from a trusted image if integrity is uncertain.
9. Review persistence mechanisms.
10. Document the timeline.

## Useful evidence

```bash
date
hostname
uptime
who
last
ps auxf
ss -tulpen
ip addr
ip route
systemctl --failed
journalctl --since "24 hours ago"
```

Do not upload sensitive evidence to public services.

## Persistence checks

Review:

```bash
crontab -l
sudo systemctl list-unit-files --state=enabled
sudo find /etc/systemd /usr/lib/systemd -type f -mtime -7 -ls
sudo find /tmp /var/tmp -type f -mtime -3 -ls
```

These are starting points, not complete forensic procedures.

---

# 18. Common Security Misconfigurations

## Misconfiguration 1: SSH open to the entire internet

Risk:

- Brute-force attempts.
- Credential attacks.
- Larger attack surface.

Better:

- Restrict source IPs.
- Use SSM or VPN.
- Disable password login.
- Use MFA-backed access workflows where available.

## Misconfiguration 2: Application runs as root

Risk:

- Application compromise becomes host compromise.

Better:

- Dedicated service account.
- systemd hardening.
- Minimal filesystem permissions.

## Misconfiguration 3: Secrets in Git

Risk:

- Permanent exposure in history and forks.

Better:

- Secret manager.
- Secret scanning.
- Rotation.
- Access review.

## Misconfiguration 4: World-writable application directory

Risk:

- Unauthorized code or configuration modification.

Better:

- Root-owned release directories.
- Controlled deployment user.
- Restricted write paths.

## Misconfiguration 5: Jenkins agent has production credentials

Risk:

- Compromised build can access production.

Better:

- Separate roles.
- Short-lived credentials.
- Environment-specific permissions.
- Approval gates.

## Misconfiguration 6: Internal database port exposed

Risk:

- Direct attacks against the database.

Better:

- Private subnet.
- Security-group references.
- Application-only access.

---

# 19. Security Troubleshooting Scenarios

## Scenario 1: SSH suddenly stops working after hardening

Check:

```bash
sudo sshd -t
sudo sshd -T
sudo journalctl -u ssh -n 100 --no-pager
```

Likely causes:

- Invalid syntax.
- Wrong `AllowUsers` or `AllowGroups`.
- Incorrect key permissions.
- Firewall rule.
- Security-group change.
- User shell disabled.

Use an existing session or out-of-band access to recover.

---

## Scenario 2: Jenkins deployment fails with permission denied

Check:

```bash
id
namei -l /opt/myapp/current
ls -ld /opt/myapp /opt/myapp/releases
systemctl status myapp
```

Likely causes:

- Deployment user lacks directory access.
- Service account owns runtime directory.
- Release directory is root-owned.
- SELinux/AppArmor policy.
- Incorrect systemd permissions.

Do not solve by setting `777`.

---

## Scenario 3: Application cannot read its secret file

Check:

```bash
namei -l /etc/myapp/secrets.env
ls -l /etc/myapp/secrets.env
systemctl show myapp -p User -p Group
```

Verify:

- Service user.
- Group membership.
- Parent directory traversal.
- File mode.
- Secret file path.
- Mandatory access-control policy.

---

## Scenario 4: Unexpected process listening on a port

Check:

```bash
sudo ss -tulpen
sudo lsof -i :PORT
ps -fp PID
readlink -f /proc/PID/exe
systemctl status SERVICE
```

Then:

- Identify owner and package.
- Check process start time.
- Review logs.
- Compare against approved inventory.
- Escalate if unauthorized.

Do not kill it before collecting evidence unless there is an immediate threat and the incident procedure authorizes it.

---

# 20. Interview Questions and Answers

## Q1. What is least privilege?

Granting a user, process, or service only the permissions required to perform its task, for the minimum necessary duration.

## Q2. Why should applications not run as root?

A vulnerability in a root-running application can provide broad control over the host. A dedicated service account reduces blast radius.

## Q3. How do you harden SSH?

Use key authentication, disable root login, restrict users or groups, limit authentication attempts, restrict network access, validate configuration, and monitor authentication logs.

## Q4. Why is `chmod 777` dangerous?

It grants read, write, and execute permissions to everyone and may allow unauthorized modification or code execution.

## Q5. How do you find world-writable files?

```bash
sudo find / -xdev -type f -perm -0002 -ls
```

## Q6. What is the difference between authentication and authorization?

Authentication verifies who someone is. Authorization determines what that identity is allowed to do.

## Q7. Why are secrets in Git dangerous even after deletion?

Git history, forks, clones, caches, and artifacts may retain the secret.

## Q8. How should CI/CD credentials be protected?

Use least-privilege roles, short-lived credentials, secret stores, masking, separated environments, and restricted build-agent permissions.

## Q9. What is the purpose of `systemd-analyze security`?

It evaluates the security exposure of a systemd service based on its unit settings. It is a guide, not proof of complete security.

## Q10. Why are cloud security groups not a replacement for host firewalls?

They provide network-level filtering, while host controls protect against local processes, lateral movement, and traffic that reaches the instance through allowed paths.

## Q11. What should you do if a secret is exposed?

Revoke or rotate it immediately, investigate usage, remove it from source and artifacts, update consumers, and document the incident.

## Q12. Why is SSM often preferable to public SSH for EC2 administration?

It can remove the need for public IPs, bastion hosts, and inbound SSH exposure, while providing IAM-based access and session auditing when configured correctly.

## Q13. What is defense in depth?

Using multiple independent security controls so that failure of one control does not automatically result in compromise.

## Q14. Why should production hosts be rebuilt from trusted images after compromise?

An attacker may establish persistence or modify binaries and configurations. Rebuilding provides a more trustworthy recovery path than attempting to clean every artifact manually.

## Q15. What is the DevOps Engineer's role in security?

Integrate secure defaults, patching, identity controls, secret handling, infrastructure security, CI/CD protection, monitoring, and incident response into delivery and operations.

---

# 21. Security Review Checklist

## Identity

- [ ] Individual user accounts exist.
- [ ] Shared accounts are avoided.
- [ ] Unused users and keys are removed.
- [ ] Privileged access is reviewed.
- [ ] Service accounts use non-login shells where appropriate.

## SSH

- [ ] Root login is disabled or tightly controlled.
- [ ] Password login is disabled where practical.
- [ ] Keys have restrictive permissions.
- [ ] SSH source access is restricted.
- [ ] Authentication logs are monitored.

## Files

- [ ] Application directories are not world-writable.
- [ ] Secrets have restrictive ownership and modes.
- [ ] SUID/SGID files are reviewed.
- [ ] Release directories are protected.
- [ ] Log files do not contain secrets.

## Services

- [ ] Services run as non-root users.
- [ ] Unnecessary services are disabled.
- [ ] systemd hardening is evaluated.
- [ ] Capabilities are minimized.
- [ ] Listening ports are documented.

## CI/CD

- [ ] Build agents are isolated.
- [ ] Credentials are least-privilege.
- [ ] Secrets are masked.
- [ ] Dependencies are scanned.
- [ ] Production deployment requires appropriate controls.

## Cloud

- [ ] Security groups are restrictive.
- [ ] Internal services use private networking.
- [ ] IAM roles replace static keys.
- [ ] EBS volumes are encrypted.
- [ ] SSM access is audited.
- [ ] Metadata access is restricted.

---

# 22. Final DevOps Principles

1. Security starts before deployment.
2. Least privilege applies to users, processes, services, IAM roles, and pipelines.
3. A secure host still needs secure applications.
4. A secure application still needs secure infrastructure.
5. Never solve permission problems with `777`.
6. Never solve credential problems by hardcoding secrets.
7. Prefer short-lived identity over long-lived keys.
8. Treat CI/CD as production-grade infrastructure.
9. Preserve evidence during security incidents.
10. Rebuild compromised systems from trusted sources.
11. Validate security controls continuously.
12. Make secure behavior the easiest operational path.
