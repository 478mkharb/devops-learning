# SELinux, AppArmor and Security Contexts for DevOps

## Scope

This guide explains mandatory access control from a **DevOps Engineer perspective**.

It focuses on how SELinux and AppArmor affect:

- EC2 instances.
- NGINX.
- Jenkins agents.
- Ansible deployments.
- systemd services.
- Application directories.
- Log collection.
- File uploads.
- Reverse proxies.
- Production troubleshooting.

The goal is to understand why an application can receive:

```text
Permission denied
```

even when normal Linux ownership and permissions appear correct.

---

# 1. Discretionary Access Control vs Mandatory Access Control

Linux normally uses **Discretionary Access Control (DAC)**.

DAC is based on:

- User ID.
- Group ID.
- File owner.
- File permissions.
- ACLs.

Example:

```bash
ls -l /opt/myapp/config.yaml
```

A file may show:

```text
-rw-r----- 1 root myapp config.yaml
```

The application must have the correct user or group permissions.

Mandatory Access Control (MAC) adds another security layer.

Even if DAC allows access, MAC may deny it.

```text
Application request
    ↓
DAC check: owner/group/mode
    ↓
MAC check: SELinux or AppArmor policy
    ↓
Allow or deny
```

Therefore:

```text
Correct chmod/chown does not guarantee access.
```

---

# 2. SELinux Overview

SELinux is a label-based mandatory access-control system.

It is commonly used on:

- RHEL.
- CentOS Stream.
- Rocky Linux.
- AlmaLinux.
- Fedora.

Ubuntu usually uses AppArmor by default, but SELinux can also be configured.

## 2.1 Check SELinux status

```bash
getenforce
```

Possible outputs:

```text
Enforcing
Permissive
Disabled
```

Detailed status:

```bash
sestatus
```

Configuration:

```bash
cat /etc/selinux/config
```

## 2.2 SELinux modes

| Mode | Behavior |
|---|---|
| Enforcing | Policy violations are blocked and logged |
| Permissive | Violations are logged but generally not blocked |
| Disabled | SELinux is not active |

`Permissive` is useful for controlled diagnosis, not as a permanent production fix.

## 2.3 Important warning

Do not disable SELinux simply because an application fails.

Poor response:

```bash
sudo setenforce 0
```

and then leaving the host permissive.

Better response:

1. Confirm the denial.
2. Identify the source process.
3. Inspect file contexts.
4. Fix labels or policy.
5. Restore enforcing mode.
6. Verify application behavior.

---

# 3. SELinux Security Contexts

Inspect a file context:

```bash
ls -Z /var/www/html/index.html
```

Inspect a process context:

```bash
ps -eZ | grep nginx
```

Inspect a directory recursively:

```bash
ls -Zd /var/www/html
ls -Z /var/www/html
```

A context commonly contains:

```text
user:role:type:level
```

Example:

```text
system_u:object_r:httpd_sys_content_t:s0
```

The most operationally important field for many web-server issues is the **type**.

---

# 4. Common SELinux File Types

Typical web-server-related types include:

| Type | Typical purpose |
|---|---|
| `httpd_sys_content_t` | Web content readable by HTTP services |
| `httpd_sys_rw_content_t` | Web content that HTTP services may write when policy permits |
| `httpd_log_t` | Web-server log files |
| `httpd_config_t` | Web-server configuration |
| `httpd_sys_script_exec_t` | Certain executable web scripts |
| `var_t` | Generic variable-data context; not automatically correct for every application |

The exact policy depends on the distribution and installed SELinux policy packages.

## 4.1 Find contexts

```bash
ls -Z /var/www/html
find /var/www -exec ls -Zd {} \;
```

## 4.2 Search by context

```bash
sudo semanage fcontext -l | grep '/var/www'
```

If `semanage` is missing, install the distribution's SELinux management package.

---

# 5. Persistent Contexts with `semanage fcontext`

A direct `chcon` change may not survive a relabel or restore operation.

Preferred approach:

```bash
sudo semanage fcontext -a -t httpd_sys_content_t '/srv/myapp(/.*)?'
sudo restorecon -Rv /srv/myapp
```

Verify:

```bash
ls -Zd /srv/myapp
ls -Z /srv/myapp
```

## 5.1 Read-only web content

```bash
sudo semanage fcontext -a -t httpd_sys_content_t '/opt/myapp/public(/.*)?'
sudo restorecon -Rv /opt/myapp/public
```

## 5.2 Writable upload directory

Only if the application genuinely needs to write there:

```bash
sudo semanage fcontext -a -t httpd_sys_rw_content_t '/opt/myapp/uploads(/.*)?'
sudo restorecon -Rv /opt/myapp/uploads
```

Do not label the entire application directory as writable unless required.

## 5.3 `chcon` versus `semanage`

| Command | Behavior |
|---|---|
| `chcon` | Changes current label directly |
| `semanage fcontext` | Defines persistent labeling policy |
| `restorecon` | Applies the expected label |

Recommended production pattern:

```bash
semanage fcontext
restorecon
```

---

# 6. Diagnose SELinux Denials

## 6.1 Search audit logs

```bash
sudo ausearch -m AVC -ts recent
```

Search recent denials:

```bash
sudo ausearch -m AVC -ts today
```

Kernel or journal view:

```bash
sudo journalctl -k | grep -i avc
```

## 6.2 Use `audit2why`

```bash
sudo ausearch -m AVC -ts recent | audit2why
```

This helps explain why a denial occurred.

## 6.3 Use `audit2allow` carefully

```bash
sudo ausearch -m AVC -ts recent | audit2allow -w
```

Generating a policy module:

```bash
sudo ausearch -m AVC -ts recent | audit2allow -M myapp_local
```

Install only after careful review:

```bash
sudo semodule -i myapp_local.pp
```

Do not blindly run `audit2allow` for every denial.

Why?

- It may create overly broad permissions.
- It can hide a bad file label.
- It may allow an unsafe behavior.
- It may turn an application bug into a permanent policy exception.

Always determine whether the correct fix is:

- File relabeling.
- Directory ownership.
- Service configuration.
- Boolean adjustment.
- A narrowly scoped policy module.

---

# 7. SELinux Booleans

Booleans enable or disable predefined policy behavior.

List booleans:

```bash
getsebool -a
```

Search web-server-related booleans:

```bash
getsebool -a | grep httpd
```

Example:

```bash
getsebool httpd_can_network_connect
```

Enable persistently:

```bash
sudo setsebool -P httpd_can_network_connect on
```

This may be needed when an HTTP service must connect to an upstream service, but enabling it increases the service's permitted behavior.

Use the narrowest applicable boolean and document why it is required.

---

# 8. SELinux and NGINX

## 8.1 Common symptom

NGINX returns:

```text
403 Forbidden
```

Normal permissions appear correct.

Investigate:

```bash
ls -lZ /var/www/html/index.html
namei -l /var/www/html/index.html
sudo ausearch -m AVC -ts recent
```

Possible cause:

- Incorrect SELinux file type.
- Parent directory label.
- NGINX policy restriction.
- Missing read permission.

## 8.2 Correct a web directory

```bash
sudo semanage fcontext -a -t httpd_sys_content_t '/srv/site(/.*)?'
sudo restorecon -Rv /srv/site
```

Then verify:

```bash
ls -Zd /srv/site
```

## 8.3 NGINX proxy connection denied

If NGINX cannot connect to an upstream service:

```bash
sudo ausearch -m AVC -ts recent
getsebool httpd_can_network_connect
```

If policy requires it and the risk is accepted:

```bash
sudo setsebool -P httpd_can_network_connect on
```

Do not enable it without understanding the network access implications.

---

# 9. SELinux and systemd Services

A systemd service may fail because:

- Its executable has the wrong label.
- Its configuration file has the wrong label.
- Its data directory has the wrong label.
- The service accesses a forbidden path.
- The service attempts a restricted network operation.

Inspect:

```bash
systemctl status myapp
systemctl cat myapp
ls -Z /usr/local/bin/myapp
ls -Zd /var/lib/myapp
sudo ausearch -m AVC -ts recent
```

A useful troubleshooting sequence:

```text
systemd status
    ↓
Application logs
    ↓
Normal DAC permissions
    ↓
SELinux AVC denial
    ↓
Correct label or policy
```

---

# 10. AppArmor Overview

AppArmor is a path-based mandatory access-control system.

It is commonly enabled on:

- Ubuntu.
- Debian-based systems.
- Ubuntu cloud images.

AppArmor profiles define what an application may access.

## 10.1 Check status

```bash
sudo aa-status
```

Alternative:

```bash
sudo apparmor_status
```

List loaded profiles:

```bash
sudo aa-status --profiled
```

## 10.2 AppArmor modes

| Mode | Behavior |
|---|---|
| Enforce | Violations are blocked and logged |
| Complain | Violations are logged but generally not blocked |
| Disabled/unloaded | Profile is not active |

Complain mode is useful for controlled discovery, not as a permanent substitute for a properly designed profile.

---

# 11. AppArmor Profiles

Profiles are commonly stored under:

```text
/etc/apparmor.d/
```

List profiles:

```bash
sudo ls -l /etc/apparmor.d/
```

Inspect a profile:

```bash
sudo cat /etc/apparmor.d/usr.sbin.nginx
```

Profiles may define:

- File read access.
- File write access.
- Directory traversal.
- Network access.
- Capability use.
- Signal permissions.
- Execution transitions.

## 11.1 Reload a profile

```bash
sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.nginx
```

Or reload the service:

```bash
sudo systemctl reload apparmor
```

Validate syntax before loading:

```bash
sudo apparmor_parser -p /etc/apparmor.d/usr.sbin.nginx
```

---

# 12. Diagnose AppArmor Denials

Check kernel messages:

```bash
sudo journalctl -k | grep -i apparmor
```

Search audit messages:

```bash
sudo ausearch -m APPARMOR -ts recent
```

Common log pattern:

```text
apparmor="DENIED"
```

Inspect recent denials:

```bash
sudo journalctl --since "30 minutes ago" | grep -i 'apparmor.*denied'
```

Investigation questions:

1. Which process was denied?
2. Which path was accessed?
3. Was the operation read, write, execute, or network-related?
4. Is the access expected?
5. Should the profile be updated?
6. Is the application using an incorrect path?

---

# 13. AppArmor with NGINX

A common issue is moving web content from:

```text
/var/www/html
```

to:

```text
/srv/site
```

Normal permissions may be correct, but AppArmor may not allow NGINX to read the new path.

Investigate:

```bash
sudo aa-status
sudo journalctl -k | grep -i apparmor
sudo grep -Rni '/var/www\|/srv' /etc/apparmor.d/
```

A profile may need rules similar to:

```text
/srv/site/ r,
/srv/site/** r,
```

The exact profile syntax and required permissions must be reviewed for the application.

Reload after changes:

```bash
sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.nginx
```

Then test:

```bash
curl -I http://127.0.0.1/
```

---

# 14. SELinux vs AppArmor

| Feature | SELinux | AppArmor |
|---|---|---|
| Main model | Labels and types | Paths and profiles |
| Common default | RHEL-family | Ubuntu |
| Policy style | Type enforcement and labels | Profile rules |
| File inspection | `ls -Z` | Profile files |
| Status | `getenforce`, `sestatus` | `aa-status` |
| Denial logs | AVC/audit logs | AppArmor/audit logs |
| Persistent file labeling | `semanage fcontext` + `restorecon` | Profile path rules |
| Typical fix | Correct context, boolean, policy | Correct profile rule |
| Temporary diagnosis | Permissive | Complain |
| Main risk | Broad policy exceptions | Overly broad path permissions |

Neither system replaces:

- Linux permissions.
- Firewalls.
- IAM.
- Package patching.
- Application security.
- Secret management.
- Monitoring.

---

# 15. Troubleshooting Decision Tree

```text
Application gets Permission denied
        |
        v
Check user/group/mode
        |
        v
Check parent directory traversal
        |
        v
Check ACLs
        |
        v
Is SELinux enabled?
        |
       Yes
        |
        v
Check AVC denials and file context
        |
        v
Fix label, boolean, or policy
        |
        v
Is AppArmor active?
        |
       Yes
        |
        v
Check AppArmor denial and profile path rules
        |
        v
Reload profile and retest
```

Commands:

```bash
ls -l /path/to/file
namei -l /path/to/file
getfacl /path/to/file
getenforce
ls -Z /path/to/file
sudo ausearch -m AVC -ts recent
sudo aa-status
sudo journalctl -k | grep -i apparmor
```

---

# 16. DevOps Deployment Considerations

## 16.1 Directory changes

Changing application paths may require:

- SELinux context updates.
- AppArmor profile updates.
- systemd `ReadWritePaths`.
- NGINX configuration changes.
- Logrotate changes.
- Backup changes.
- Monitoring path changes.

Never assume that changing:

```text
/var/www/html
```

to:

```text
/opt/myapp/current
```

requires only an NGINX configuration update.

## 16.2 Immutable releases

For release directories:

```text
/opt/myapp/releases/2026-09-13-001
/opt/myapp/releases/2026-09-13-002
/opt/myapp/current -> /opt/myapp/releases/2026-09-13-002
```

Security requirements:

- Release directories should not be writable by the running application.
- The deployment process should be controlled.
- Contexts should be applied consistently.
- The symlink target must be permitted by MAC policy.
- Rollback should preserve valid labels and profiles.

## 16.3 Ansible

Ansible can manage SELinux contexts:

```yaml
- name: Apply SELinux context to application content
  ansible.posix.sefcontext:
    target: "/opt/myapp/public(/.*)?"
    setype: httpd_sys_content_t
    state: present

- name: Restore SELinux contexts
  ansible.builtin.command: restorecon -Rv /opt/myapp/public
  changed_when: false
```

For AppArmor:

- Deploy profile files.
- Validate syntax.
- Load or reload profiles.
- Test the application.
- Avoid silently disabling profiles.

## 16.4 Packer images

When building an image:

1. Install required policy packages.
2. Install application files.
3. Apply persistent labels.
4. Enable required profiles.
5. Validate service startup.
6. Test the application path.
7. Capture policy configuration in version control.
8. Verify the image after boot.

---

# 17. Production Safety Rules

Do not:

```bash
sudo setenforce 0
```

and forget to restore enforcing mode.

Do not:

```bash
sudo aa-disable /etc/apparmor.d/profile
```

as a permanent workaround.

Do not:

- Generate broad `audit2allow` rules without review.
- Label all files with a generic type.
- Add unrestricted write access.
- Disable mandatory access control across the host.
- Change policies directly on production without version control.
- Test policy changes only through a successful HTTP status code.

Always:

- Capture denial evidence.
- Make the smallest policy change.
- Validate after reload.
- Test read, write, execute, and network behavior.
- Document the reason for each exception.

---

# 18. Troubleshooting Scenarios

## Scenario 1: NGINX returns 403 after moving content

Commands:

```bash
ls -lZ /srv/site/index.html
namei -l /srv/site/index.html
sudo ausearch -m AVC -ts recent
```

Likely SELinux fix:

```bash
sudo semanage fcontext -a -t httpd_sys_content_t '/srv/site(/.*)?'
sudo restorecon -Rv /srv/site
```

If Ubuntu uses AppArmor, inspect:

```bash
sudo aa-status
sudo journalctl -k | grep -i apparmor
```

---

## Scenario 2: Application can read configuration but cannot write uploads

Check:

```bash
ls -ldZ /opt/myapp/uploads
sudo ausearch -m AVC -ts recent
```

Possible fixes:

- Correct owner/group.
- Correct DAC permissions.
- `httpd_sys_rw_content_t` for SELinux where appropriate.
- AppArmor write rule.
- systemd `ReadWritePaths`.
- Application path correction.

Do not make the entire application directory writable.

---

## Scenario 3: Service starts manually but fails under systemd

Possible causes:

- Different user.
- Different environment.
- Different working directory.
- SELinux context.
- AppArmor profile.
- systemd sandboxing.
- Missing permission to access files or sockets.

Compare:

```bash
systemctl show myapp -p User -p Group -p WorkingDirectory
systemctl cat myapp
ps -ef | grep myapp
sudo ausearch -m AVC -ts recent
sudo journalctl -k | grep -i apparmor
```

---

## Scenario 4: Ansible deployment succeeds but application cannot access files

Check:

```bash
ls -lZ /opt/myapp
namei -l /opt/myapp/config.yaml
getenforce
sudo aa-status
```

Likely cause:

- Deployment created files with incorrect labels.
- Application runs under a different user.
- MAC policy does not allow the new path.
- Symlink target has incorrect context.

Make the fix part of the deployment automation.

---

# 19. Interview Questions and Answers

## Q1. What is the difference between DAC and MAC?

DAC uses owner, group, permissions, and ACLs. MAC adds centrally enforced policy that can deny access even when DAC permits it.

## Q2. What is SELinux enforcing mode?

It blocks policy violations and records denials.

## Q3. What is the difference between permissive and enforcing?

Enforcing blocks and logs. Permissive logs but generally does not block.

## Q4. Why is `chcon` not always a permanent fix?

A relabel operation may restore the default context and remove the manual `chcon` change. Use `semanage fcontext` and `restorecon` for persistent configuration.

## Q5. How do you investigate an SELinux denial?

```bash
sudo ausearch -m AVC -ts recent
sudo ausearch -m AVC -ts recent | audit2why
ls -Z /path
```

Then determine whether the correct fix is a label, boolean, configuration change, or narrowly scoped policy.

## Q6. Why should `audit2allow` not be used blindly?

It may generate excessive permissions and hide an incorrect file context or application configuration.

## Q7. What is AppArmor?

A path-based mandatory access-control system that restricts applications using profiles.

## Q8. How do you check AppArmor status?

```bash
sudo aa-status
```

## Q9. How do you find AppArmor denials?

```bash
sudo journalctl -k | grep -i apparmor
sudo ausearch -m APPARMOR -ts recent
```

## Q10. Why can NGINX return 403 even when permissions are correct?

SELinux or AppArmor may deny access to the content path.

## Q11. What should be included when changing an application directory?

Filesystem permissions, MAC labels or profiles, systemd restrictions, reverse-proxy configuration, log paths, backups, monitoring, and deployment automation.

## Q12. How do SELinux and AppArmor fit into DevSecOps?

They enforce runtime least privilege and reduce the impact of application compromise.

## Q13. Should production systems run with SELinux disabled?

Not by default. Disabling it removes a security control and should require an explicit risk decision.

## Q14. What is the safest way to change a security policy?

Version-control the change, test it in a lower environment, apply the smallest rule, validate logs and behavior, and monitor after deployment.

## Q15. Does MAC replace file permissions?

No. DAC and MAC are separate layers and both must allow the operation.

---

# 20. Final Security Principles

1. Permission denied does not always mean `chmod` is wrong.
2. Always check DAC and MAC separately.
3. Use persistent SELinux labeling with `semanage fcontext` and `restorecon`.
4. Treat `audit2allow` as a reviewed exception mechanism, not a first response.
5. AppArmor profiles must reflect actual application paths.
6. Do not disable security controls to hide configuration problems.
7. Keep policy changes in version control.
8. Test policies during Packer and Ansible image creation.
9. Secure release directories and writable paths independently.
10. Include SELinux/AppArmor in deployment runbooks.
11. Prefer narrow permissions over broad exceptions.
12. Verify both application functionality and security logs after every policy change.
