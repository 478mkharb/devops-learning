# <h1 align="center">Documentation - SonarQube Setup using Ansible Role on AWS EC2 </h1>

<div align="center">
<img width="180" alt="Ansible" src="https://upload.wikimedia.org/wikipedia/commons/2/24/Ansible_logo.svg" />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="120" alt="SonarQube" src="https://upload.wikimedia.org/wikipedia/commons/c/c5/Sonar-logo.svg" />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="120" alt="AWS EC2" src="https://upload.wikimedia.org/wikipedia/commons/9/93/Amazon_Web_Services_Logo.svg" />
</div>

<br/>

---

<div align="center">

| Author       | Created On | Version | Last Updated By | Last Edited On | Pre Reviewer | L0 Reviewer | L1 Reviewer | L2 Reviewer  |
| ------------ | ---------- | ------- | --------------- | -------------- | ------------ | ----------- | ----------- | ------------ |
| Mukesh Kharb | 28/05/2026 | 1.0     | Mukesh Kharb    | 28/05/2026     | Team         | Mohit Kumar | Faisal Khan | Mahesh Kumar |

</div>

---

# Table of Contents

1. [Introduction](#1-introduction)
2. [Purpose](#2-purpose)
3. [Pre-requisites and System Requirements](#3-pre-requisites-and-system-requirements)
4. [Architecture Diagram](#4-architecture-diagram)
5. [Directory Structure](#5-directory-structure)
6. [Ansible Role Components](#6-ansible-role-components)
7. [Dynamic Inventory Configuration](#7-dynamic-inventory-configuration)
8. [Complete Role Files](#8-complete-role-files)
9. [Step-by-Step SonarQube Installation](#9-step-by-step-sonarqube-installation)
10. [Quality Gate Setup](#10-quality-gate-setup)
11. [Authentication and Authorization Setup](#11-authentication-and-authorization-setup)
12. [Verification Steps](#12-verification-steps)
13. [Troubleshooting](#13-troubleshooting)
14. [Best Practices](#14-best-practices)
15. [Conclusion](#15-conclusion)
16. [FAQs](#16-faqs)
17. [Contact Information](#17-contact-information)
18. [References](#18-references)

---

<a id="1-introduction"></a>

# 1. Introduction

This documentation explains automated SonarQube deployment on dynamic AWS EC2 instances using reusable Ansible Roles. The setup automates installation, configuration, and service management for standardized DevOps deployment workflows.

---

<a id="2-purpose"></a>

# 2. Purpose

The purpose of this implementation is to:

* Automate SonarQube installation on AWS EC2
* Use reusable Ansible roles
* Support dynamic cloud infrastructure
* Reduce manual deployment effort
* Enable Infrastructure as Code practices
* Standardize DevOps deployment workflows
* Integrate SonarQube with CI/CD pipelines

---

<a id="3-pre-requisites-and-system-requirements"></a>

# 3. Pre-requisites and System Requirements

| Requirement          | Description                          |
| -------------------- | ------------------------------------ |
| Ansible Control Node | Linux machine with Ansible installed |
| Python3              | Required for Ansible and AWS SDK     |
| Java                 | OpenJDK 17                           |
| RAM                  | Minimum 4 GB                         |
| CPU                  | 2 vCPU                               |
| Storage              | Minimum 8 GB                         |
| PostgreSQL           | Version 14+                          |
| SonarQube            | Version 10.5.1                       |

> [!NOTE]
> PostgreSQL is required because SonarQube stores users, projects, analysis reports, quality gates, and code metrics inside a relational database backend.

---

<a id="4-architecture-diagram"></a>

# 4. Architecture Diagram

```text
                    +----------------------+
                    |   Ansible Control    |
                    |        Node          |
                    +----------+-----------+
                               |
                               |
                     Dynamic AWS Inventory
                               |
                     +---------+---------+
                     |                   |
                     |   EC2 Instance    |
                     |    SonarQube      |
                     |                   |
                     +-------------------+
```

---

<a id="5-directory-structure"></a>

# 5. Directory Structure

```text
sonarqube-ansible/
├── ansible.cfg
├── inventory/
│   └── aws_ec2.yml
├── playbook.yml
├── roles/
│   └── sonarqube/
│       ├── defaults/
│       │   └── main.yml
│       ├── files/
│       ├── handlers/
│       │   └── main.yml
│       ├── meta/
│       │   └── main.yml
│       ├── tasks/
│       │   └── main.yml
│       ├── templates/
│       │   ├── sonar.properties.j2
│       │   └── sonarqube.service.j2
│       └── vars/
│           └── main.yml
```

---

<a id="6-ansible-role-components"></a>

# 6. Ansible Role Components

| Component | Purpose                                  |
| --------- | ---------------------------------------- |
| defaults  | Stores default variables                 |
| vars      | Stores static variables                  |
| tasks     | Main automation execution                |
| handlers  | Service restart handlers                 |
| templates | Jinja2 configuration templates           |
| files     | Static files                             |
| meta      | Role metadata and dependency information |
| inventory | Dynamic EC2 inventory configuration      |

---

<a id="7-dynamic-inventory-configuration"></a>

# 7. Dynamic Inventory Configuration

## Install AWS Collection

```bash
ansible-galaxy collection install amazon.aws
pip install boto3 botocore
```

---

## inventory/aws_ec2.yml

```yaml
plugin: amazon.aws.aws_ec2

regions:
  - ap-south-1

filters:
  instance-state-name: running

hostnames:
  - public-ip-address

compose:
  ansible_host: public_ip_address
```

---

## Verify Inventory

```bash
ansible-inventory -i inventory/aws_ec2.yml --graph
```

---

<a id="8-complete-role-files"></a>

# 8. Complete Role Files

## 8.1 defaults/main.yml

```yaml
---
sonarqube_version: "10.5.1.90531"

sonarqube_download_url: "https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.5.1.90531.zip"

sonarqube_install_dir: "/opt/sonarqube"

sonarqube_user: "sonar"

sonarqube_group: "sonar"

sonarqube_port: 9000

postgresql_db: "sonarqube"

postgresql_user: "sonar"

postgresql_password: "password"
```

---

## 8.2 vars/main.yml

```yaml
---
java_package: "openjdk-17-jdk"

required_packages:
  - unzip
  - wget
  - curl
  - "{{ java_package }}"
  - postgresql
  - postgresql-contrib
```

---

## 8.3 handlers/main.yml

```yaml
---
- name: restart sonarqube
  systemd:
    name: sonarqube
    state: restarted
```

---

## 8.4 meta/main.yml

```yaml
---
galaxy_info:
  author: Mukesh Kharb
  description: SonarQube Setup Role
  company: OT-Microservices
  license: MIT
  min_ansible_version: 2.10

dependencies: []
```

---

## 8.5 tasks/main.yml

```yaml
---
- name: Update apt cache
  apt:
    update_cache: yes

- name: Install required packages
  apt:
    name: "{{ required_packages }}"
    state: present

- name: Create SonarQube group
  group:
    name: "{{ sonarqube_group }}"

- name: Create SonarQube user
  user:
    name: "{{ sonarqube_user }}"
    group: "{{ sonarqube_group }}"
    shell: /bin/bash
    create_home: yes

- name: Download SonarQube package
  get_url:
    url: "{{ sonarqube_download_url }}"
    dest: /tmp/sonarqube.zip

- name: Extract SonarQube
  unarchive:
    src: /tmp/sonarqube.zip
    dest: /opt/
    remote_src: yes

- name: Rename SonarQube directory
  command: mv /opt/sonarqube-{{ sonarqube_version }} {{ sonarqube_install_dir }}
  args:
    creates: "{{ sonarqube_install_dir }}"

- name: Set ownership
  file:
    path: "{{ sonarqube_install_dir }}"
    owner: "{{ sonarqube_user }}"
    group: "{{ sonarqube_group }}"
    recurse: yes

- name: Configure sonar.properties
  template:
    src: sonar.properties.j2
    dest: "{{ sonarqube_install_dir }}/conf/sonar.properties"
    owner: "{{ sonarqube_user }}"
    group: "{{ sonarqube_group }}"
  notify:
    - restart sonarqube

- name: Configure SonarQube service
  template:
    src: sonarqube.service.j2
    dest: /etc/systemd/system/sonarqube.service
  notify:
    - restart sonarqube

- name: Reload systemd
  systemd:
    daemon_reload: yes

- name: Enable SonarQube service
  systemd:
    name: sonarqube
    enabled: yes

- name: Start SonarQube service
  systemd:
    name: sonarqube
    state: started
```

---

## 8.6 templates/sonar.properties.j2

```properties
sonar.jdbc.username={{ postgresql_user }}

sonar.jdbc.password={{ postgresql_password }}

sonar.jdbc.url=jdbc:postgresql://localhost/{{ postgresql_db }}

sonar.web.host=0.0.0.0

sonar.web.port={{ sonarqube_port }}
```

---

## 8.7 templates/sonarqube.service.j2

```ini
[Unit]
Description=SonarQube Service
After=network.target

[Service]
Type=forking

ExecStart={{ sonarqube_install_dir }}/bin/linux-x86-64/sonar.sh start
ExecStop={{ sonarqube_install_dir }}/bin/linux-x86-64/sonar.sh stop

User={{ sonarqube_user }}
Group={{ sonarqube_group }}

Restart=always
LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
```

---

## 8.8 inventory/aws_ec2.yml

```yaml
plugin: amazon.aws.aws_ec2

regions:
  - ap-south-1

filters:
  instance-state-name: running

hostnames:
  - public-ip-address

compose:
  ansible_host: public_ip_address
```

---

## 8.9 playbook.yml

```yaml
---
- name: SonarQube Setup on EC2
  hosts: all
  become: yes

  roles:
    - sonarqube
```

---

## 8.10 ansible.cfg

```ini
[defaults]
inventory = inventory/aws_ec2.yml
host_key_checking = False
remote_user = ubuntu
private_key_file = ~/.ssh/sonarqube.pem

[inventory]
enable_plugins = amazon.aws.aws_ec2
```

---

<a id="9-step-by-step-sonarqube-installation"></a>

# 9. Step-by-Step SonarQube Installation

## Step 1: Install Ansible

```bash
sudo apt update
sudo apt install ansible -y
ansible --version
```

---

## Step 2: Install AWS Dependencies

```bash
pip install boto3 botocore
ansible-galaxy collection install amazon.aws
```

---

## Step 3: Configure AWS Credentials

```bash
aws configure
```

---

## Step 4: Create Project Directory

```bash
mkdir -p sonarqube-ansible
cd sonarqube-ansible
```

---

## Step 5: Create Role Structure

```bash
mkdir -p roles/sonarqube/{tasks,handlers,defaults,vars,templates,files,meta}
mkdir inventory
```

---

## Step 6: Add All Role Files

Add all files shown in Section 8.

---

## Step 7: Verify Inventory

```bash
ansible-inventory --graph
```

---

## Step 8: Run Playbook

```bash
ansible-playbook playbook.yml
```

---

<a id="10-quality-gate-setup"></a>

# 10. Quality Gate Setup

## Create Quality Gate

Navigate:

```text
Administration → Quality Gates
```

---

## Recommended Quality Gate Conditions

| Metric          | Condition        |
| --------------- | ---------------- |
| Bugs            | Must be 0        |
| Vulnerabilities | Must be 0        |
| Code Smells     | Less than 10     |
| Coverage        | Greater than 80% |
| Duplications    | Less than 3%     |

---

## Assign Quality Gate

```text
Projects → Project Settings → Quality Gate
```

Assign the created quality gate to the required project.

---

<a id="11-authentication-and-authorization-setup"></a>

# 11. Authentication and Authorization Setup

## Create Users

Navigate:

```text
Administration → Security → Users
```

---

## Create Groups

Navigate:

```text
Administration → Security → Groups
```

---

## Recommended Groups

| Group      | Purpose                    |
| ---------- | -------------------------- |
| admin      | Full administrative access |
| developers | Project scan access        |
| viewers    | Read-only dashboard access |

---

## Configure Permission Templates

Navigate:

```text
Administration → Security → Permission Templates
```

---

## Recommended Permissions

| Permission               | Recommended Group |
| ------------------------ | ----------------- |
| Administer System        | admin             |
| Execute Analysis         | developers        |
| Browse Projects          | viewers           |
| Administer Quality Gates | admin             |

---

## Generate Authentication Token

Navigate:

```text
User Profile → Security → Generate Tokens
```

These tokens can be integrated with Jenkins pipelines for automated code analysis.

---

<a id="12-verification-steps"></a>

# 12. Verification Steps

## Verify SonarQube Service

```bash
sudo systemctl status sonarqube
```

---

## Verify Port Listening

```bash
ss -tulnp | grep 9000
```

---

## Access SonarQube

```text
http://<EC2-PUBLIC-IP>:9000
```

---

## Default Credentials

```text
Username: admin
Password: admin
```

---

<a id="13-troubleshooting"></a>

# 13. Troubleshooting

| Issue                      | Cause                | Solution           |
| -------------------------- | -------------------- | ------------------ |
| SonarQube service failed   | Java missing         | Install OpenJDK 17 |
| Port 9000 inaccessible     | Firewall restriction | Open port 9000     |
| Database connection failed | PostgreSQL issue     | Verify credentials |
| Dynamic inventory empty    | AWS IAM issue        | Verify credentials |
| SonarQube not starting     | Low memory           | Increase EC2 RAM   |

---

<a id="14-best-practices"></a>

# 14. Best Practices

| Best Practice           | Description                       |
| ----------------------- | --------------------------------- |
| Use IAM Roles           | Avoid static AWS credentials      |
| Use Ansible Vault       | Secure sensitive variables        |
| Keep Roles Modular      | Improve reusability               |
| Use Dynamic Inventory   | Avoid static inventory management |
| Enable Database Backups | Protect SonarQube metadata        |
| Monitor Services        | Continuously monitor SonarQube    |

---

<a id="15-conclusion"></a>

# 15. Conclusion

Using reusable Ansible Roles with AWS Dynamic Inventory provides a scalable and maintainable approach for SonarQube deployment on EC2 infrastructure. This implementation standardizes deployment workflows, simplifies automation, and improves CI/CD integration.

---

<a id="16-faqs"></a>

# 16. FAQs

### Q1. Why is Ansible Role used for SonarQube deployment?

**Answer:**
Ansible Roles provide reusable and modular automation which simplifies SonarQube deployment and maintenance.

---

### Q2. Why is Dynamic Inventory required in AWS?

**Answer:**
Dynamic Inventory automatically discovers running EC2 instances without manually updating inventory files.

---

### Q3. Which Java version is recommended for SonarQube?

**Answer:**
OpenJDK 17 is recommended because modern SonarQube versions require Java 17.

---

### Q4. Why is PostgreSQL required for SonarQube?

**Answer:**
PostgreSQL stores projects, users, code analysis reports, quality gates, and permissions.

---

### Q5. What happens if SonarQube service fails after deployment?

**Answer:**
Logs should be verified using `journalctl` and `systemctl status sonarqube` for troubleshooting.

---

<a id="17-contact-information"></a>

# 17. Contact Information

| Name         | ✉️ Contact                                                                        |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

<a id="18-references"></a>

# 18. References

| S.No | Description              | Click to View                                                                                                                                                                                                               |
| ---- | ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | Ansible Documentation    | [![Ansible](https://img.shields.io/badge/ANSIBLE-DOCUMENTATION-2F4F4F?style=flat-square\&logo=ansible\&logoColor=white)](https://docs.ansible.com/)                                                                         |
| 2    | SonarQube Documentation  | [![SonarQube](https://img.shields.io/badge/SONARQUBE-DOCUMENTATION-3A3A3A?style=flat-square\&logo=sonarqube\&logoColor=white)](https://docs.sonarsource.com/)                                                               |
| 3    | AWS EC2 Documentation    | [![AWS EC2](https://img.shields.io/badge/AWS-EC2_DOCUMENTATION-404040?style=flat-square\&logo=amazonaws\&logoColor=white)](https://docs.aws.amazon.com/ec2/)                                                                |
| 4    | AWS Dynamic Inventory    | [![Dynamic Inventory](https://img.shields.io/badge/AWS-DYNAMIC_INVENTORY-2B2B2B?style=flat-square\&logo=amazonaws\&logoColor=white)](https://docs.ansible.com/ansible/latest/collections/amazon/aws/aws_ec2_inventory.html) |
| 5    | PostgreSQL Documentation | [![PostgreSQL](https://img.shields.io/badge/POSTGRESQL-DOCUMENTATION-1F1F1F?style=flat-square\&logo=postgresql\&logoColor=white)](https://www.postgresql.org/docs/)                                                         |
| 6    | OpenJDK Documentation    | [![OpenJDK](https://img.shields.io/badge/OPENJDK-DOCUMENTATION-353535?style=flat-square\&logo=openjdk\&logoColor=white)](https://openjdk.org/projects/jdk/)                                                                 |
