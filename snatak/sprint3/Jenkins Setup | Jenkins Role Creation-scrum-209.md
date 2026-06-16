# <h1 align="center">POC - Jenkins Configuration Management using Ansible Role</h1>

<div align="center">
<img width="180" alt="Jenkins" src="https://www.jenkins.io/images/logos/jenkins/jenkins.svg" />
</div>

<br/>

---

<div align="center">

| Author       | Created On | Version | Last Updated By | Last Edited On | Pre Reviewer | L0 Reviewer | L1 Reviewer | L2 Reviewer  |
| ------------ | ---------- | ------- | --------------- | -------------- | ------------ | ----------- | ----------- | ------------ |
| Mukesh Kharb | 16/06/2026 | 1.0     | Mukesh Kharb    | 16/06/2026     | Team         | Mohit Kumar | Faisal Khan | Mahesh Kumar |

</div>

---

# Table of Contents

1. [Introduction](#1-introduction)
2. [Pre-Requisites](#2-pre-requisites)
3. [Objective](#3-objective)
4. [Solution Overview](#4-solution-overview)
5. [Create Jenkins Ansible Role Structure](#5-create-jenkins-ansible-role-structure)
6. [Configure Plugin Management](#6-configure-plugin-management)
7. [Configure Ansible Vault for Users and Credentials](#7-configure-ansible-vault-for-users-and-credentials)
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

# 1. Introduction

This Proof of Concept (POC) demonstrates how Jenkins configuration can be managed using Ansible Roles and Jenkins Configuration as Code (JCasC).

The implementation focuses on automating Jenkins administration activities through Infrastructure as Code (IaC) practices.

The scope of this POC includes:

* Jenkins Plugin Management
* Jenkins User Management
* Jenkins Credentials Management
* Jenkins Security Configuration
* Jenkins Global Configuration Management
* Secret Management using Ansible Vault

Jenkins installation is assumed to be completed before executing this POC.

---

<a id="2-pre-requisites"></a>

# 2. Pre-Requisites

| Component     | Version                 |
| ------------- | ----------------------- |
| Ubuntu Server | 22.04 LTS               |
| Jenkins       | LTS (Already Installed) |
| Ansible       | 2.15+                   |
| Git           | Latest                  |

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
* Securing secrets using Ansible Vault

---

<a id="4-solution-overview"></a>

# 4. Solution Overview

```text
                    ┌─────────────────┐
                    │  Ansible Role   │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
      Plugin Install     Ansible Vault     JCasC
              │              │              │
              └──────────────┼──────────────┘
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

Create additional directories:

```bash
mkdir -p roles/jenkins/files
mkdir -p roles/jenkins/templates
mkdir -p roles/jenkins/vault
```

Verify Structure:

```bash
tree roles/jenkins
```

<details>
<summary>📸 <strong>Click to view Screenshot</strong></summary>

<img width="1500" height="804" alt="image" src="https://github.com/user-attachments/assets/8450f49a-c278-4a58-89a8-32c46de9a4a3" />

</details>

---

<a id="6-configure-plugin-management"></a>

# 6. Configure Plugin Management

Create Plugin List:

```bash
vi roles/jenkins/files/plugins.txt
```

Add Plugins:

```text
configuration-as-code
git
credentials
matrix-auth
role-strategy
```

Create Plugin Task:

```bash
vi roles/jenkins/tasks/plugins.yml
```

```yaml
- name: Copy plugin list
  copy:
    src: plugins.txt
    dest: /tmp/plugins.txt

- name: Install Jenkins plugins
  command:
    jenkins-plugin-cli --plugin-file /tmp/plugins.txt
  notify:
    - Restart Jenkins
```

Verify Plugins:

```bash
ls /var/lib/jenkins/plugins
```

<details>
<summary>📸 <strong>Click to view Screenshot - Plugin Installation</strong></summary>

Add Screenshot Here

</details>

---

<a id="7-configure-ansible-vault-for-users-and-credentials"></a>

# 7. Configure Ansible Vault for Users and Credentials

Create Vault File:

```bash
ansible-vault create roles/jenkins/vault/secrets.yml
```

Add Vault Variables:

```yaml
vault_admin_user: admin

vault_admin_password: Admin@123

vault_github_username: github-user

vault_github_token: ghp_xxxxxxxxxxxxxx
```

Verify Encryption:

```bash
cat roles/jenkins/vault/secrets.yml
```

Expected Output:

```text
$ANSIBLE_VAULT;1.1;AES256
6133363739376630...
```

<details>
<summary>📸 <strong>Click to view Screenshot - Ansible Vault Configuration</strong></summary>

Add Screenshot Here

</details>

---

<a id="8-create-jcasc-template"></a>

# 8. Create JCasC Template

Create Template:

```bash
vi roles/jenkins/templates/casc.yaml.j2
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
        - id: "{{ vault_admin_user }}"
          password: "{{ vault_admin_password }}"

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
              username: "{{ vault_github_username }}"
              password: "{{ vault_github_token }}"
              description: GitHub Credentials
```

<details>
<summary>📸 <strong>Click to view Screenshot - JCasC Template</strong></summary>

Add Screenshot Here

</details>

---

<a id="9-configure-jenkins-configuration-management"></a>

# 9. Configure Jenkins Configuration Management

Create Configuration Task:

```bash
vi roles/jenkins/tasks/casc.yml
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

Create Handler:

```bash
vi roles/jenkins/handlers/main.yml
```

```yaml
- name: Restart Jenkins
  service:
    name: jenkins
    state: restarted
```

<details>
<summary>📸 <strong>Click to view Screenshot - JCasC Deployment</strong></summary>

Add Screenshot Here

</details>

---

<a id="10-execute-ansible-role"></a>

# 10. Execute Ansible Role

Create Playbook:

```bash
vi site.yml
```

```yaml
---
- hosts: jenkins

  become: true

  vars_files:
    - roles/jenkins/vault/secrets.yml

  roles:
    - jenkins
```

Create Inventory:

```bash
vi inventory
```

```ini
[jenkins]
localhost ansible_connection=local
```

Execute Playbook:

```bash
ansible-playbook \
-i inventory \
site.yml \
--ask-vault-pass
```

Expected Output:

```text
PLAY RECAP
localhost : ok=10 changed=5 failed=0
```

<details>
<summary>📸 <strong>Click to view Screenshot - Playbook Execution</strong></summary>

Add Screenshot Here

</details>

---

<a id="11-validate-jenkins-configuration"></a>

# 11. Validate Jenkins Configuration

Verify Jenkins Service:

```bash
systemctl status jenkins
```

Verify Plugins:

```bash
ls /var/lib/jenkins/plugins
```

Expected Plugins:

```text
configuration-as-code
git
credentials
matrix-auth
role-strategy
```

Verify JCasC Configuration:

```bash
curl http://localhost:8080/configuration-as-code/viewExport
```

Verify User Login:

```text
Username : admin
Password : ********
```

Verify Credentials:

```text
Manage Jenkins
 → Credentials
 → System
```

Expected Credential:

```text
github-creds
```

<details>
<summary>📸 <strong>Click to view Screenshot - Validation</strong></summary>

Add Screenshot Here

</details>

---

<a id="12-common-issues-and-troubleshooting"></a>

# 12. Common Issues and Troubleshooting

| Issue                          | Cause                        | Resolution                            |
| ------------------------------ | ---------------------------- | ------------------------------------- |
| Plugin installation failed     | Jenkins service unavailable  | Verify Jenkins service status         |
| JCasC configuration not loaded | Incorrect configuration path | Verify JAVA_ARGS value                |
| Authentication failure         | Incorrect vault values       | Validate vault secrets                |
| Template rendering failure     | Missing variables            | Verify vars_files configuration       |
| Jenkins restart failure        | Permission issue             | Run playbook with elevated privileges |
| Credentials not visible        | Invalid JCasC syntax         | Validate YAML configuration           |

---

<a id="13-conclusion"></a>

# 13. Conclusion

This POC demonstrates how Jenkins configuration can be managed using Ansible Roles and Jenkins Configuration as Code (JCasC).

The solution automates plugin installation, user management, credentials management, security configuration, and global Jenkins settings while securely managing secrets through Ansible Vault.

This approach enables repeatable, secure, and version-controlled Jenkins administration aligned with Infrastructure as Code (IaC) best practices.

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
