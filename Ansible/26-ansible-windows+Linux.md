# Ansible Managing Windows and Linux Servers (Detailed Production Guide)

This guide explains **how a DevOps engineer configures multiple Linux and Windows servers using Ansible in a real production environment**.

Example scenario used in this document:

* 1 Ansible Control Node
* 5 Linux servers
* 5 Windows servers

The key idea is that **Ansible uses different connection protocols for different operating systems**.

| OS      | Protocol Used | Default Port |
| ------- | ------------- | ------------ |
| Linux   | SSH           | 22           |
| Windows | WinRM         | 5985 / 5986  |

---

# 1. Overall Architecture

In production, Ansible runs from a **control node** which connects to target machines.

```
                +-----------------------+
                |   Ansible Controller  |
                +-----------------------+
                   /               \
                  /                 \
               SSH                   WinRM
                |                     |
        +----------------+    +----------------+
        |  Linux Servers |    | Windows Servers|
        | (5 machines)   |    | (5 machines)   |
        +----------------+    +----------------+
```

Linux machines communicate using **SSH**, while Windows machines communicate using **WinRM (Windows Remote Management)**.

---

# 2. Real Production Repository Structure

In real DevOps environments, Ansible code is organized carefully.

```
ansible-infrastructure/

├── inventory/
│   └── production.ini
│
├── playbooks/
│   └── site.yml
│
├── roles/
│   ├── linux_common/
│   │   └── tasks/
│   │       └── main.yml
│   │
│   └── windows_common/
│       └── tasks/
│           └── main.yml
│
├── group_vars/
│   ├── linux.yml
│   └── windows.yml
│
└── ansible.cfg
```

Why this structure is used:

* Easy to maintain
* Supports large infrastructure
* Enables reuse through roles

---

# 3. Inventory Configuration

Inventory defines **which servers Ansible will manage**.

File:

```
inventory/production.ini
```

Example:

```
[linux]
linux1 ansible_host=10.0.1.10
linux2 ansible_host=10.0.1.11
linux3 ansible_host=10.0.1.12
linux4 ansible_host=10.0.1.13
linux5 ansible_host=10.0.1.14

[windows]
win1 ansible_host=10.0.2.10
win2 ansible_host=10.0.2.11
win3 ansible_host=10.0.2.12
win4 ansible_host=10.0.2.13
win5 ansible_host=10.0.2.14
```

Two host groups are created:

* **linux**
* **windows**

---

# 4. Linux Connection Configuration

Linux hosts use **SSH authentication**.

File:

```
group_vars/linux.yml
```

```
ansible_user: ubuntu
ansible_become: true
ansible_ssh_private_key_file: ~/.ssh/id_rsa
```

Explanation:

| Parameter      | Purpose               |
| -------------- | --------------------- |
| ansible_user   | SSH user              |
| ansible_become | Enable sudo           |
| ssh key        | Secure authentication |

---

# 5. Windows Connection Configuration

Windows requires **WinRM configuration**.

File:

```
group_vars/windows.yml
```

Example configuration:

```
ansible_user: Administrator
ansible_password: StrongPassword
ansible_connection: winrm
ansible_port: 5985
ansible_winrm_transport: ntlm
ansible_winrm_server_cert_validation: ignore
```

Explanation:

| Parameter          | Purpose               |
| ------------------ | --------------------- |
| ansible_connection | Use WinRM             |
| ansible_port       | WinRM port            |
| ntlm               | authentication method |

Default ports:

| Port | Protocol    |
| ---- | ----------- |
| 5985 | HTTP WinRM  |
| 5986 | HTTPS WinRM |

---

# 6. Preparing Windows Servers

WinRM must be enabled on each Windows server.

Run PowerShell:

```
winrm quickconfig
```

This command:

* Enables WinRM service
* Opens firewall port
* Creates listener

To enable basic authentication:

```
Set-Item WSMan:\localhost\Service\Auth\Basic $true
```

---

# 7. Linux Role Example

File:

```
roles/linux_common/tasks/main.yml
```

```
---

- name: Install nginx
  apt:
    name: nginx
    state: present

- name: Start nginx
  service:
    name: nginx
    state: started
```

This installs and starts **nginx on all Linux servers**.

---

# 8. Windows Role Example

File:

```
roles/windows_common/tasks/main.yml
```

```
---

- name: Install IIS
  win_feature:
    name: Web-Server
    state: present

- name: Start IIS
  win_service:
    name: W3SVC
    state: started
```

This installs the **IIS web server on Windows machines**.

---

# 9. Main Playbook

File:

```
playbooks/site.yml
```

```
---

- name: Configure Linux servers
  hosts: linux
  become: true

  roles:
    - linux_common


- name: Configure Windows servers
  hosts: windows

  roles:
    - windows_common
```

The playbook contains **two plays**.

One for Linux and one for Windows.

---

# 10. Execution Process

Command used:

```
ansible-playbook -i inventory/production.ini playbooks/site.yml
```

Execution flow:

```
Step 1
Ansible reads inventory

Step 2
Hosts grouped (linux/windows)

Step 3
Linux → SSH connection

Step 4
Windows → WinRM connection

Step 5
Roles executed
```

---

# 11. Internal Communication Flow

### Linux Communication

```
Ansible Controller
      |
      | SSH
      v
Linux Host
      |
      v
Execute module
```

### Windows Communication

```
Ansible Controller
      |
      | WinRM
      v
Windows Host
      |
      v
PowerShell module execution
```

---

# 12. Important Ansible Modules

### Linux Modules

| Module  | Purpose          |
| ------- | ---------------- |
| apt     | install packages |
| yum     | install packages |
| service | manage services  |
| copy    | copy files       |
| file    | manage files     |

### Windows Modules

| Module      | Purpose                  |
| ----------- | ------------------------ |
| win_feature | install Windows features |
| win_service | manage services          |
| win_copy    | copy files               |
| win_package | install software         |

---

# 13. Scaling in Real Production

Large companies often manage:

* 100+ Linux servers
* 100+ Windows servers

Maintaining static inventory files becomes difficult in cloud environments because servers are created and destroyed frequently.

To solve this, production environments use **Dynamic Inventory**.

Dynamic inventory automatically pulls server information from cloud platforms like:

* AWS
* Azure
* GCP
* VMware

Instead of manually writing IP addresses in inventory files, Ansible **queries the cloud API** and generates the host list automatically.

---

# 14. Dynamic Inventory Architecture

```
                +----------------------+
                |   Ansible Controller |
                +----------------------+
                          |
                          |
                    Cloud API Query
                          |
                          v
               +-----------------------+
               |  Dynamic Inventory    |
               |  Plugin / Script      |
               +-----------------------+
                    /             \
                   /               \
            Linux Instances     Windows Instances
```

Flow:

1. Ansible runs inventory plugin
2. Plugin queries cloud provider API
3. API returns instance list
4. Ansible groups hosts automatically

---

# 15. Example Dynamic Inventory (AWS)

Production environments commonly use the **AWS EC2 inventory plugin**.

Directory structure:

```
inventory/
  aws_ec2.yml
```

Example configuration:

```
plugin: amazon.aws.aws_ec2
regions:
  - ap-south-1

filters:
  instance-state-name: running

keyed_groups:
  - key: tags.OS
    prefix: os
```

Example EC2 tags:

| Instance | Tag        |
| -------- | ---------- |
| EC2-1    | OS=linux   |
| EC2-2    | OS=windows |

Inventory groups generated automatically:

```
os_linux
os_windows
```

---

# 16. Example Playbook with Dynamic Inventory

```
---

- name: Configure Linux servers
  hosts: os_linux
  become: true

  roles:
    - linux_common


- name: Configure Windows servers
  hosts: os_windows

  roles:
    - windows_common
```

Now whenever a new EC2 instance is launched with tag:

```
OS=linux
```

Ansible **automatically discovers it**.

No manual inventory update required.

---

# 17. Running Dynamic Inventory

Command:

```
ansible-inventory -i inventory/aws_ec2.yml --graph
```

Example output:

```
@all
 |--@os_linux
 |   |--10.0.1.10
 |   |--10.0.1.11
 |
 |--@os_windows
     |--10.0.2.10
     |--10.0.2.11
```

Run playbook:

```
ansible-playbook -i inventory/aws_ec2.yml playbooks/site.yml
```

---

# 18. Benefits of Dynamic Inventory

| Benefit                 | Explanation                        |
| ----------------------- | ---------------------------------- |
| Auto discovery          | New servers detected automatically |
| Cloud integration       | Works with AWS, Azure, GCP         |
| No manual IP management | Uses instance metadata             |
| Scales easily           | Works with thousands of servers    |

---

# 19. Testing Connectivity

Test Linux connectivity:

```
ansible os_linux -i inventory/aws_ec2.yml -m ping
```

Test Windows connectivity:

```
ansible os_windows -i inventory/aws_ec2.yml -m win_ping
```

---

Test Linux connectivity:

```
ansible linux -i inventory/production.ini -m ping
```

Test Windows connectivity:

```
ansible windows -i inventory/production.ini -m win_ping
```

---

# 15. DevOps Interview Answer

If asked:

**How do you manage both Windows and Linux servers using Ansible?**

A good answer is:

Ansible manages Linux servers using SSH and Windows servers using WinRM. We define separate host groups in the inventory and configure connection parameters using group variables. Linux systems use modules like apt or yum while Windows systems use modules such as win_feature and win_service. A single playbook can contain multiple plays targeting different host groups, enabling centralized automation of heterogeneous infrastructure.
