# <h1 align="center">Jenkins Configuration Management using Ansible Role</h1>

<div align="center">
<img width="120" alt="Jenkins" src="https://www.jenkins.io/images/logos/jenkins/jenkins.svg" />
</div>

<br/>

---

<div align="center">

| Author       | Created On | Version | Last Updated By | Last Edited On | Pre Reviewer | L0 Reviewer | L1 Reviewer | L2 Reviewer  |
| ------------ | ---------- | ------- | --------------- | -------------- | ------------ | ----------- | ----------- | ------------ |
| Mukesh Kharb | 16/06/2026 | 1.0     | Mukesh Kharb    | 16/06/2026     | Team         | Mohit Kumar | Faisal Khan | Mahesh Kumar |

</div>

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Pre-Requisites](#2-pre-requisites)
3. [Objective](#3-objective)
4. [Solution Overview](#4-solution-overview)
5. [Create Jenkins Ansible Role Structure](#5-create-jenkins-ansible-role-structure)
6. [Configure Plugin Management](#6-configure-plugin-management)
7. [Configure-Jenkins-Variables](#7-configure-jenkins-variables)
8. [Create JCasC Template](#8-create-jcasc-template)
9. [Configure Jenkins Configuration Management](#9-configure-jenkins-configuration-management)
10. [Execute Ansible Role](#10-execute-ansible-role)
11. [Validate Jenkins Configuration](#11-validate-jenkins-configuration)
12. [Common Issues and Troubleshooting](#12-common-issues-and-troubleshooting)
13. [Conclusion](#13-conclusion)
14. [Contact Information](#14-contact-information)
15. [References](#15-references)

---

<a id="1-introduction"></a>

## 1. Introduction

This POC demonstrates how Jenkins configuration can be managed using Ansible Role and Jenkins Configuration as Code (JCasC).

The implementation focuses on automating Jenkins administration activities through Infrastructure as Code (IaC) practices.

The scope of this POC includes:

* Jenkins Plugin Management
* Jenkins User Management
* Jenkins Credentials Management
* Jenkins Security Configuration
* Jenkins Global Configuration Management

Jenkins installation is assumed to be completed before executing this POC.

---

<a id="2-pre-requisites"></a>

# 2. Pre-Requisites

| Component     | Version                 |
| ------------- | ----------------------- |
| Ubuntu Server | 22.04 LTS               |
| Jenkins       | LTS (Already Installed) |
| Ansible       | 2.15+                   |


Verify Jenkins Service:

```bash
systemctl status jenkins
```

Verify Ansible Installation:

```bash
ansible --version
```
<details>
<summary>📸 <strong>Click to view Screenshot</strong></summary>
<img width="1704" height="319" alt="image" src="https://github.com/user-attachments/assets/6b96c14b-57d7-4920-944e-a4f88b568555" />
</details>



Verify Jenkins Access:

```bash
curl http://192.168.8.210:8080/login
```

---

<a id="3-objective"></a>

# 3. Objective

The objective of this POC is to automate Jenkins configuration management using Ansible.

The Ansible Role should be capable of:

* Installing required Jenkins plugins
* Managing Jenkins users
* Managing Jenkins credentials
* Managing Jenkins security settings
* Managing Jenkins global configuration

---

<a id="4-solution-overview"></a>

# 4. Solution Overview

```text
                    ┌─────────────────┐
                    │  Ansible Role   │
                    └────────┬────────┘
                             │
              ┌──────────────▼──────────────┐
              │                             │
              ▼                             ▼
      Plugin Install                      JCasC
              │                             │
              └──────────────|──────────────┘
                             ▼
                 Jenkins Configuration
                             │
                             ▼
                    Restart Jenkins
```

---

<a id="5-create-jenkins-ansible-role-structure"></a>

# 5. Create Jenkins Ansible Role Structure

Create the Jenkins Role:

```bash
ansible-galaxy init roles/jenkins
```

Verify Structure:

```bash
tree roles/jenkins
```

<details>
<summary>📸 <strong>Click to view Screenshot</strong></summary>
<img width="1187" height="740" alt="image" src="https://github.com/user-attachments/assets/9073986a-29ec-46a7-905b-7c36bce28a7c" />
</details>

---

<a id="6-configure-plugin-management"></a>

# 6. Configure Plugin Management

Create Plugin List:

```bash
nano roles/jenkins/files/plugins.txt
```

Add Plugins:

```text
configuration-as-code
git
matrix-auth
role-strategy
workflow-aggregator
credentials
credentials-binding
ssh-agent
aws-credentials
config-file-provider
junit
htmlpublisher
ws-cleanup
sonar
pipeline-stage-view
```

Create Plugin Task:

```bash
nano roles/jenkins/tasks/plugins.yml
```

```yaml
- name: Copy plugin list
  copy:
    src: plugins.txt
    dest: /tmp/plugins.txt

- name: Create plugin manager directory
  file:
    path: /opt/jenkins
    state: directory
    mode: '0755'

- name: Download Jenkins Plugin Manager Tool
  get_url:
    url: https://repo.jenkins-ci.org/releases/io/jenkins/plugin-management/plugin-management-cli/2.13.2/plugin-management-cli-2.13.2.jar
    dest: /opt/jenkins/jenkins-plugin-manager.jar
    mode: '0755'

- name: Install Jenkins Plugins
  shell: |
    java -jar /opt/jenkins/jenkins-plugin-manager.jar \
      --plugin-file /tmp/plugins.txt \
      --plugin-download-directory /var/lib/jenkins/plugins
  notify:
    - Restart Jenkins
```

<details>
<summary>📸 <strong>Click to view Screenshot - Plugin Installation</strong></summary>
<img width="1510" height="651" alt="image" src="https://github.com/user-attachments/assets/39cb105a-4898-4841-a49c-4dd20d935a58" />
</details>

---

<a id="7-configure-jenkins-variables"></a>

# 7. Configure Jenkins Variables

Create:

```bash
roles/jenkins/vars/main.yml
```

Add Vault Variables:

```yaml
admin_user: Snatak
admin_password: Snatak@123

github_username: mukesh130478
github_token: ghp_xxxxx
```

<details>
<summary>📸 <strong>Click to view Screenshot - Ansible Vault Configuration</strong></summary>
<img width="1345" height="315" alt="image" src="https://github.com/user-attachments/assets/dc5e908f-ffbf-471b-a365-2b5e4d5ea798" />
</details>

---

<a id="8-create-jcasc-template"></a>

# 8. Create JCasC Template

Create Template:

```bash
nano roles/jenkins/templates/casc.yaml.j2
```

Add Configuration:

```yaml
jenkins:

  systemMessage: "Managed by Ansible"

  numExecutors: 2

  mode: NORMAL

  securityRealm:
    local:
      allowsSignup: false
      users:
        - id: "{{ admin_user }}"
          password: "{{ admin_password }}"

  authorizationStrategy:
    loggedInUsersCanDoAnything:
      allowAnonymousRead: false

credentials:
  system:
    domainCredentials:
      - credentials:
          - usernamePassword:
              scope: GLOBAL
              id: github-creds
              username: "{{ github_username }}"
              password: "{{ github_token }}"
              description: GitHub Credentials
```

<details>
<summary>📸 <strong>Click to view Screenshot - JCasC Template</strong></summary>
<img width="1497" height="775" alt="image" src="https://github.com/user-attachments/assets/6acd5b1f-31e5-444e-a566-3b6cdad4d94e" />
</details>

---

<a id="9-configure-jenkins-configuration-management"></a>

# 9. Configure Jenkins Configuration Management

Create Configuration Task:

```bash
nano roles/jenkins/tasks/casc.yml
```

```yaml
- name: Create JCasC directory
  file:
    path: /var/lib/jenkins/casc_configs
    state: directory
    owner: jenkins
    group: jenkins
    mode: '0755'

- name: Deploy JCasC configuration
  template:
    src: casc.yaml.j2
    dest: /var/lib/jenkins/casc_configs/casc.yaml

- name: Configure JCasC path
  lineinfile:
    path: /etc/default/jenkins
    regexp: '^JAVA_ARGS='
    line: 'JAVA_ARGS="-Dcasc.jenkins.config=/var/lib/jenkins/casc_configs/"'
  notify:
    - Restart Jenkins
```
Import tasks:

```bash
nano roles/jenkins/tasks/main.yml
```
```bash
#SPDX-License-Identifier: MIT-0
---
# tasks file for roles/jenkins
- import_tasks: plugins.yml
- import_tasks: casc.yml

```
Create Handler:

```bash
nano roles/jenkins/handlers/main.yml
```

```yaml
- name: Restart Jenkins
  service:
    name: jenkins
    state: restarted
```

<details>
<summary>📸 <strong>Click to view Screenshot - JCasC Deployment</strong></summary>
<img width="1451" height="801" alt="image" src="https://github.com/user-attachments/assets/fddfdfcf-590e-41fe-b621-ceec0a175788" />
</details>

---

<a id="10-execute-ansible-role"></a>

# 10. Execute Ansible Role

Create Playbook:

```bash
nano site.yml
```

```yaml
---
- hosts: jenkins
  become: true
  roles:
    - jenkins
```

Create Inventory:

```bash
vi inventory
```

```ini
[jenkins]
jenkins-server ansible_host=192.168.8.210 ansible_user=harshwardhan ansible_python_interpreter=/usr/bin/python3
```

Execute Playbook:

```bash
ansible-playbook -i inventory site.yml
```


<details>
<summary>📸 <strong>Click to view Screenshot - Playbook Execution</strong></summary>
<img width="1639" height="890" alt="image" src="https://github.com/user-attachments/assets/2e973d74-09c3-444f-9c19-a9906ab4c2fc" />
</details>

---

<a id="11-validate-jenkins-configuration"></a>

# 11. Validate Jenkins Configuration

Verify Jenkins Service:

```bash
systemctl status jenkins
```
Open:

```text
Manage Jenkins
→ Users → Snaatak → Credentials
```
<details>
<summary>📸 <strong>Click to view Screenshot - Validation</strong></summary>
<img width="1852" height="997" alt="image" src="https://github.com/user-attachments/assets/0f02c5c8-0224-4f16-a2a4-8a65f263cad7" />
</details>

---

<a id="12-common-issues-and-troubleshooting"></a>

# 12. Common Issues and Troubleshooting

| Issue                          | Cause                        | Resolution                            |
| ------------------------------ | ---------------------------- | ------------------------------------- |
| Plugin installation failed     | Jenkins service unavailable  | Verify Jenkins service status         |
| JCasC configuration not loaded | Incorrect configuration path | Verify JAVA_ARGS value                |
| Authentication failure         | Incorrect  vars/main.yml     | Validate variables                    |
| Template rendering failure     | Missing variables            | Verify vars_files configuration       |
| Jenkins restart failure        | Permission issue             | Run playbook with elevated privileges |
| Credentials not visible        | Invalid JCasC syntax         | Validate YAML configuration           |

---

<a id="13-conclusion"></a>

# 13. Conclusion

This POC demonstrates how Jenkins configuration can be managed using Ansible Roles and Jenkins Configuration as Code (JCasC).

The solution automates plugin installation, user management, credentials management, security configuration, and global Jenkins settings.

This approach enables idempotent, secure, and version-controlled Jenkins administration aligned with Infrastructure as Code (IaC) best practices.

---

<a id="14-contact-information"></a>

# 14. Contact Information

| Name         | Contact                                                                           |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

<a id="15-references"></a>

# 15. References

| S.No | Description                              | Reference                                                                                                                                                                                       |
| ---- | ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | Jenkins Documentation                    | [![Jenkins](https://img.shields.io/badge/Jenkins-Documentation-D33833?style=flat-square\&logo=jenkins\&logoColor=white)](https://www.jenkins.io/doc/)                                           |
| 2    | Jenkins Configuration as Code            | [![JCasC](https://img.shields.io/badge/JCasC-Documentation-D33833?style=flat-square\&logo=jenkins\&logoColor=white)](https://www.jenkins.io/projects/jcasc/)                                    |
| 3    | Ansible Documentation                    | [![Ansible](https://img.shields.io/badge/Ansible-Documentation-EE0000?style=flat-square\&logo=ansible\&logoColor=white)](https://docs.ansible.com/)                                             |
| 4    | Ansible Vault Documentation              | [![Vault](https://img.shields.io/badge/Ansible-Vault-EE0000?style=flat-square\&logo=ansible\&logoColor=white)](https://docs.ansible.com/ansible/latest/vault_guide/index.html)                  |
| 5    | Jenkins Plugin Installation Manager Tool | [![Plugin Manager](https://img.shields.io/badge/Jenkins-Plugin_Manager-D33833?style=flat-square\&logo=jenkins\&logoColor=white)](https://github.com/jenkinsci/plugin-installation-manager-tool) |

---
