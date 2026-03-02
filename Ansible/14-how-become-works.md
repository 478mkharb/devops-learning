# How Does `become` Work Internally in Ansible?

`become` is Ansible’s **privilege escalation mechanism**. It allows Ansible to **run tasks as another user (usually root)** after connecting to a host.

> 🧠 Think of `become` as: *"Log in as a normal user, then temporarily switch identity to do privileged work."*

---

## 🔑 What Problem Does `become` Solve?

* SSH best practice: **don’t log in as root**
* Many tasks require **root privileges**
* `become` bridges this gap safely

---

## 🧩 High-Level Internal Flow

```
Ansible Control Node
        │
        │ SSH (normal user)
        ▼
Remote Host (e.g. ubuntu)
        │
        │ become (sudo / su / doas)
        ▼
Target User (root)
```

---

## ⚙️ Step-by-Step: How `become` Works Internally

1. Ansible connects to the host via SSH as `ansible_user`
2. Task is marked with `become: true`
3. Ansible wraps the command with a **privilege escalation command**
4. Command is executed as the **become user** (default: root)
5. Output is returned to Ansible

---

## 🧪 Basic Example

```yaml
- name: Install nginx
  apt:
    name: nginx
    state: present
  become: true
```

### What Ansible Executes Internally

```bash
sudo -H -S -n apt-get install nginx
```
(Not shown to user, but conceptually accurate)
### 🔹 `-H` → Set HOME for Target User

### What it does

* Sets the `HOME` environment variable to the **target user’s home directory**
* Usually `/root` when becoming root

### Why Ansible uses it

* Prevents tools from reading config files from the **original user’s home**
* Avoids permission and config conflicts

### Example

Without `-H`:

```bash
HOME=/home/ansible
```

With `-H`:

```bash
HOME=/root
```

✅ Important for package managers, git, pip, etc.

---

### 🔹 `-S` → Read Password from STDIN

### What it does

* Tells `sudo` to **read the password from standard input (STDIN)**
* Instead of prompting interactively

### Why Ansible uses it

* Ansible is **non-interactive**
* Password (if required) is piped securely via STDIN

⚠️ Ansible does this internally and **never stores the password on disk**.

---

### 🔹 `-n` → Non-Interactive Mode

### What it does

* Disables password prompts completely
* If sudo needs a password → command **fails immediately**

### Why Ansible uses it

* Prevents playbooks from **hanging forever**
* Ensures predictable automation behavior
---

## 👤 `become_user`

Run task as a specific user:

```yaml
- name: Run as postgres user
  command: whoami
  become: true
  become_user: postgres
```

---

## 🔄 `become_method`

Defines **how** privilege escalation happens.

Common methods:

| Method | Used On         |
| ------ | --------------- |
| sudo   | Linux (default) |
| su     | Legacy systems  |
| doas   | OpenBSD         |
| pbrun  | PowerBroker     |
| dzdo   | Centrify        |

Example:

```yaml
become: true
become_method: su
```

---

## 🔐 Password Handling (`become_pass`)

If sudo requires a password:

```bash
ansible-playbook site.yml --ask-become-pass
```

Internally:

* Password is sent via **STDIN**
* Never written to disk

---

## 🧠 `become` vs `remote_user`

| Feature               | remote_user | become |
| --------------------- | ----------- | ------ |
| Used for SSH login    | ✅ Yes       | ❌ No   |
| Changes user mid-task | ❌ No        | ✅ Yes  |
| Typical user          | ansible     | root   |

---

## 🧯 Common Failure Scenarios

### ❌ "sudo: a password is required"

Cause:

* User not in sudoers
* Missing `--ask-become-pass`

### ❌ "become_user does not exist"

Cause:

* Target user missing on system

---

## ⚠️ Security Best Practices

* Use **passwordless sudo** for Ansible user
* Limit sudo permissions
* Avoid `become: true` at play level unless needed

---

## 🧠 One-Line Summary (Interview Gold)

> **Ansible connects as a normal user and uses `become` to wrap commands with sudo (or similar) to run them as another user.**

---

## 🎯 Interview Follow-Ups

Common questions:

* `become` vs `sudo`
* `become` with `delegate_to`
* Why root login is discouraged

If you want, I can also add:

* Execution flow diagram
* `become` with roles
* Real-world sudoers example
