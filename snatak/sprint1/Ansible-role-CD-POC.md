# <h1 align="center">POC - Ansible Role CD using Ansible Role </h1>

<div align="center">
<img width="100" alt="Ansible" src="https://upload.wikimedia.org/wikipedia/commons/2/24/Ansible_logo.svg" />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
</div>

<br/>

---

<div align="center">

<table>
  <tr>
    <th align="center">Author</th>
    <th align="center">Created On</th>
    <th align="center">Version</th>
    <th align="center">Last Updated By</th>
    <th align="center">Last Edited On</th>
    <th align="center">Pre Reviewer</th>
    <th align="center">L0 Reviewer</th>
    <th align="center">L1 Reviewer</th>
    <th align="center">L2 Reviewer</th>
  </tr>

  <tr>
    <td align="center">Mukesh Kharb</td>
    <td align="center">17/05/2026</td>
    <td align="center">1.0</td>
    <td align="center">Mukesh Kharb</td>
    <td align="center">17/05/2026</td>
    <td align="center">Team</td>
    <td align="center">Prince Batra</td>
    <td align="center">Nikita Joshi</td>
    <td align="center">Piyush Upadhyay</td>
  </tr>
</table>

</div>

---

# Table of Contents

1. [Introduction](#1-introduction)
2. [Environment Setup](#2-environment-setup)
3. [Install Required Packages](#3-install-required-packages)
4. [Create Ansible Role Structure](#4-create-ansible-role-structure)
5. [Configure Inventory and Playbook](#5-configure-inventory-and-playbook)
6. [Validate Ansible Role](#6-validate-ansible-role)
7. [Execute Role Deployment](#7-execute-role-deployment)
8. [Verify Deployment](#8-verify-deployment)
9. [Troubleshooting](#9-troubleshooting)
10. [Conclusion](#10-conclusion)
11. [Contact Information](#11-contact-information)
12. [References](#12-references)

---

## 1. Introduction

This POC demonstrates Continuous Deployment (CD) using Ansible Roles and Playbooks.

The deployment process automates:

* Infrastructure configuration
* Package installation
* Service deployment
* Configuration management
* Post-deployment verification

This approach ensures consistent and repeatable infrastructure deployments across multiple environments.

---

## 2. Environment Setup

| Component          | Details              |
| ------------------ | -------------------- |
| OS                 | Ubuntu 22.04         |
| Automation Tool    | Ansible              |
| Version Control    | GitHub               |
| Runtime            | Python 3             |
| Deployment Method  | SSH                  |
| Target Environment | Ubuntu Managed Nodes |

---

## 3. Install Required Packages

Update system packages:

```bash
sudo apt update
```

Install required dependencies:

```bash
sudo apt install -y python3 python3-pip git openssh-client sshpass
```

Install Ansible:

```bash
sudo apt install -y ansible
```

Verify installation:

```bash
ansible --version
```

Install ansible-lint:

```bash
pip3 install ansible-lint
```

Verify ansible-lint:

```bash
ansible-lint --version
```

<details>
<summary>📸 <strong>Click to view Screenshot - Installation of Required Packages</strong></summary>

<img width="1536" height="1024" alt="ChatGPT Image May 17, 2026, 10_53_55 PM" src="https://github.com/user-attachments/assets/d417f464-246e-47e0-87e3-c79d782a2e7d" />


</details>

---

## 4. Create Ansible Role Structure

Create role directory:

```bash
ansible-galaxy init roles/webserver
```

Generated role structure:

```text
roles/
└── webserver/
    ├── defaults/
    ├── files/
    ├── handlers/
    ├── meta/
    ├── tasks/
    ├── templates/
    ├── tests/
    ├── vars/
    └── README.md
```

Navigate to task directory:

```bash
cd roles/webserver/tasks
```

Create main task file:

```bash
nano main.yml
```

task:

```yaml
---
- name: Install NGINX
  apt:
    name: nginx
    state: present
    update_cache: yes

- name: Start NGINX
  service:
    name: nginx
    state: started
    enabled: yes
```

<details>
<summary>📸 <strong>Click to view Screenshot - Ansible Role Structure</strong></summary>

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/c4a96a83-2a9e-4f50-b3cc-b64ec2bfe761" />

</details>

---

## 5. Configure Inventory and Playbook

Create inventory directory:

```bash
mkdir inventory
```

Create production inventory:

```bash
nano inventory/prod
```

Example inventory:

```ini
[web]
192.168.1.10 ansible_user=ubuntu
```

Create playbook:

```bash
nano site.yml
```

Example playbook:

```yaml
---
- name: Deploy Webserver Role
  hosts: web
  become: yes

  roles:
    - webserver
```

Verify inventory:

```bash
ansible-inventory -i inventory/prod --list
```

Test SSH connectivity:

```bash
ansible all -i inventory/prod -m ping
```

<details>
<summary>📸 <strong>Click to view Screenshot - Inventory and Playbook Configuration</strong></summary>

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/c757f549-f7c7-4c6e-b983-b68a002807b5" />


</details>

---

## 6. Validate Ansible Role

Run syntax validation:

```bash
ansible-playbook site.yml -i inventory/prod --syntax-check
```

Run ansible-lint:

```bash
ansible-lint
```

Run dry-run deployment:

```bash
ansible-playbook -i inventory/prod site.yml --check
```

<details>
<summary>📸 <strong>Click to view Screenshot - Validation and Linting</strong></summary>

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/3a97cb20-a10f-40c2-9b19-004c78b9746e" />


</details>

---

## 7. Execute Role Deployment

Deploy Ansible Role:

```bash
ansible-playbook -i inventory/prod site.yml
```


<details>
<summary>📸 <strong>Click to view Screenshot - Role Deployment</strong></summary>

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/ef9a8092-6987-4f4b-9c60-675ad7d7057a" />


</details>

---

## 8. Verify Deployment

Check NGINX service status:

```bash
systemctl status nginx
```

Verify web server response:

```bash
curl http://localhost
```

Verify deployed packages:

```bash
dpkg -l | grep nginx
```

<details>
<summary>📸 <strong>Click to view Screenshot - Deployment Verification</strong></summary>

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/97e189f8-5ede-44f6-b980-982ce984f8a2" />


</details>

---

## 9. Troubleshooting

| Issue                   | Cause                          | Solution                     |
| ----------------------- | ------------------------------ | ---------------------------- |
| SSH Connection Failed   | SSH access issue               | Verify SSH keys and firewall |
| ansible-lint Failed     | YAML syntax issue              | Correct linting violations   |
| Inventory Not Reachable | Incorrect inventory IP         | Verify managed node IP       |
| Role Execution Failed   | Missing package or permissions | Verify sudo privileges       |
| Service Not Starting    | Package misconfiguration       | Check service logs           |

---

## 10. Conclusion

This POC successfully demonstrates automated Continuous Deployment (CD) using Ansible Roles and Playbooks. The deployment process validates infrastructure configurations, performs automated deployments, and ensures consistent infrastructure management across environments.

---

## 11. Contact Information

| Name         | Contact                                                                           |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

## 12. References

| S.No | Description                | Reference Link |
| ---- | -------------------------- | -------------- |
| 1 | Ansible Documentation | [![Ansible Docs](https://img.shields.io/badge/ANSIBLE-DOCUMENTATION-2F2F2F?style=flat-square&logo=ansible&logoColor=white)](https://docs.ansible.com/) |
| 2 | Ansible Galaxy | [![Ansible Galaxy](https://img.shields.io/badge/ANSIBLE-GALAXY-3A3A3A?style=flat-square&logo=ansible&logoColor=white)](https://galaxy.ansible.com/) |
| 3 | Ansible Lint Documentation | [![Ansible Lint](https://img.shields.io/badge/ANSIBLE-LINT-404040?style=flat-square&logo=ansible&logoColor=white)](https://ansible.readthedocs.io/projects/lint/) |
| 4 | GitHub Documentation | [![GitHub Docs](https://img.shields.io/badge/GITHUB-DOCUMENTATION-1F1F1F?style=flat-square&logo=github&logoColor=white)](https://docs.github.com/) |
| 5 | YAML Documentation | [![YAML Docs](https://img.shields.io/badge/YAML-DOCUMENTATION-4A4A4A?style=flat-square&logo=yaml&logoColor=white)](https://yaml.org/spec/) |
| 6 | OpenSSH Documentation | [![OpenSSH Docs](https://img.shields.io/badge/OPENSSH-DOCUMENTATION-2B2B2B?style=flat-square&logo=gnubash&logoColor=white)](https://www.openssh.com/manual.html) |
