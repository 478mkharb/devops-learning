# Ansible Ad-Hoc Commands – Complete Guide with Multiple Examples

This document provides a **detailed, practical, and interview-ready guide** to **Ansible ad-hoc commands** with **multiple real-world examples**.

Ad-hoc commands are used for **quick, one-time operations** without writing a playbook.

---

## 1. What Are Ansible Ad-Hoc Commands?

An **ad-hoc command** is a **single-line Ansible command** executed from the CLI to perform a **simple task** on managed nodes.

They are best suited for:

* Quick checks
* Troubleshooting
* One-time changes
* Emergency operations

---
## 2. General Syntax

```bash
ansible <host-pattern> -m <module> -a "module arguments" [options]
```

### Components Explained

* `ansible` → CLI tool for ad-hoc commands
* `<host-pattern>` → Inventory group or host
* `-m` → Module name
* `-a` → Module arguments
* `-i` → Inventory file (optional)
* `-b` → Become (sudo)

---

## 3. Verify Connectivity (Ping Module)

```bash
ansible all -m ping
```

Checks:

* SSH connectivity
* Python availability

---

## 4. Execute Shell / Command Modules

### Run a Simple Command

```bash
ansible web -m command -a "uptime"
```

### Using Shell (supports pipes, redirects)

```bash
ansible web -m shell -a "df -h | grep /dev"
```

---

## 5. File Operations

### Create a File

```bash
ansible web -m file -a "path=/tmp/demo.txt state=touch"
```

### Remove a File

```bash
ansible web -m file -a "path=/tmp/demo.txt state=absent"
```

---

## 6. Copy Files to Remote Hosts

```bash
ansible web -m copy -a "src=./index.html dest=/var/www/html/index.html"
```

---

## 7. Manage Packages

### Install Package (YUM)

```bash
ansible web -m yum -a "name=httpd state=present" -b
```

### Install Package (APT)

```bash
ansible ubuntu -m apt -a "name=nginx state=present update_cache=yes" -b
```

---

## 8. Manage Services

### Start a Service

```bash
ansible web -m service -a "name=httpd state=started" -b
```

### Restart a Service

```bash
ansible web -m service -a "name=httpd state=restarted" -b
```

---

## 9. User Management

### Create a User

```bash
ansible all -m user -a "name=deploy state=present" -b
```

### Delete a User

```bash
ansible all -m user -a "name=deploy state=absent" -b
```

---

## 10. Disk and System Information

### Check Disk Usage

```bash
ansible all -m shell -a "df -h"
```

### Check Memory

```bash
ansible all -m shell -a "free -m"
```

---

## 11. Fetch Files from Remote Hosts

```bash
ansible web -m fetch -a "src=/var/log/messages dest=./logs flat=yes"
```

---

## 12. Manage Permissions

```bash
ansible web -m file -a "path=/var/www/html owner=apache group=apache mode=0644" -b
```

---

## 13. Using Variables in Ad-Hoc Commands

```bash
ansible web -m shell -a "echo {{ inventory_hostname }}"
```

---

## 14. Using Custom Inventory

```bash
ansible web -i inventory.ini -m ping
```

---

## 15. Become (sudo) Usage

```bash
ansible web -m yum -a "name=git state=present" -b
```

---

## 16. Limit Hosts

```bash
ansible all --limit web1 -m ping
```

---

## 17. Run Commands in Parallel

```bash
ansible all -m shell -a "uptime" -f 10
```

---

## 18. Dry Run (Check Mode)

```bash
ansible web -m copy -a "src=a.txt dest=/tmp/a.txt" --check
```

---

## 19. Common Mistakes

❌ Using `shell` instead of `command` unnecessarily

❌ Forgetting `-b` for privileged tasks

❌ Using ad-hoc commands for complex workflows

---

## 20. Best Practices

* Use ad-hoc commands for **simple tasks only**
* Prefer **idempotent modules** over shell
* Move repeated tasks to playbooks
* Use inventory groups wisely

---

## 21. Interview One-Line Answers

* **Ad-hoc command**: One-time Ansible command without a playbook
* Best module for checks: `ping`
* Supports sudo: Yes, using `-b`

---

## Final Summary

* Ad-hoc commands are fast and powerful
* Ideal for troubleshooting and quick fixes
* Not a replacement for playbooks

Mastering ad-hoc commands makes you **much faster and more effective with Ansible**.

---

Happy Automating with Ansible 🚀
