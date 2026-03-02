# ✅ Ansible Fresh Worker Node Checklist

This is a **ready-to-copy checklist** to prepare fresh worker/managed nodes for running Ansible playbooks from a control node.

Use this in your **README, playbook prep guide, or onboarding document**.

---

## 1. Python Installation

Ansible requires Python to execute modules.

```bash
# Check if Python is installed
python3 --version

# Install Python if missing
# RHEL/CentOS
sudo yum install python3 -y

# Ubuntu/Debian
sudo apt-get update && sudo apt-get install python3 -y
```

---

## 2. SSH Access from Control Node

Set up passwordless SSH for the user that will run playbooks.

```bash
# Generate SSH key on control node
ssh-keygen -t rsa -b 4096

# Copy SSH key to worker node
ssh-copy-id user@worker-node

# Test connectivity
ssh user@worker-node uptime
```

---

## 3. User Privileges / Sudo

Ensure the SSH user has required privileges.

```bash
# Check sudo access
sudo -l

# If sudo is required, add user to sudoers file
sudo usermod -aG sudo user  # Ubuntu/Debian
sudo usermod -aG wheel user  # RHEL/CentOS
```

In your playbooks, use:

```yaml
become: yes
become_user: root
```

---

## 4. Required System Packages (Optional but Recommended)

Install minimal packages often used by playbooks.

```bash
# RHEL/CentOS
sudo yum install sudo tar unzip curl git -y

# Ubuntu/Debian
sudo apt-get install sudo tar unzip curl git -y
```

---

## 5. Firewall / Network

* Ensure SSH port (default 22) is open and reachable from the control node.
* Optional: allow Ansible control node IP in firewall rules.

```bash
# Check connectivity
nc -zv control-node-ip 22
```

---

## 6. Optional Tools for Advanced Playbooks

* Python pip (for Python package installation)
* Git (if playbooks clone repositories)
* System tools for gathering facts (df, uname, lsb_release, etc.)

```bash
sudo apt-get install python3-pip git -y   # Ubuntu
sudo yum install python3-pip git -y       # RHEL/CentOS
```

---

## ✅ Summary Table

| Requirement       | Command / Check                                |
| ----------------- | ---------------------------------------------- |
| Python installed  | `python3 --version`                            |
| SSH access        | `ssh user@worker-node`                         |
| User privileges   | `sudo -l` + add to sudoers if needed           |
| Required packages | `sudo yum/apt install sudo tar unzip curl git` |
| Firewall          | SSH port open & reachable                      |
| Optional tools    | pip, git, system utilities                     |

---

Once all items in this checklist are completed, the worker node is **ready to run Ansible playbooks safely** from the control node.
