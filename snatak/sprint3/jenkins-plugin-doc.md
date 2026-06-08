# <h1 align="center">Jenkins Plugin Installation Guide</h1>

<div align="center">
<img width="80" alt="Jenkins" src="https://www.jenkins.io/images/logos/jenkins/jenkins.png" />
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
    <th align="center">Last Updated On</th>
    <th align="center">L0 Reviewer</th>
    <th align="center">L1 Reviewer</th>
    <th align="center">L2 Reviewer</th>
  </tr>

  <tr>
    <td align="center">Mukesh Kharb</td>
    <td align="center">28/05/2026</td>
    <td align="center">1.0</td>
    <td align="center">Mukesh Kharb</td>
    <td align="center">28/05/2026</td>
    <td align="center">Mohit Kumar</td>
    <td align="center">Faisal Khan</td>
    <td align="center">Mahesh Kumar</td>
  </tr>

</table>

</div>

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Purpose of Jenkins Plugins](#2-purpose-of-jenkins-plugins)
3. [Role of Jenkins Plugins in CI/CD Workflow](#3-role-of-jenkins-plugins-in-cicd-workflow)
4. [Different Categories of Jenkins Plugins](#4-different-categories-of-jenkins-plugins)
   * [4.1 Core Jenkins Plugins](#41-core-jenkins-plugins)
   * [4.2 Terraform Plugins](#42-terraform-plugins)
   * [4.3 Ansible Plugins](#43-ansible-plugins)
   * [4.4 SAST Plugins](#44-sast-plugins)
   * [4.5 DAST Plugins](#45-dast-plugins)
   * [4.6 Build Tool Plugins](#46-build-tool-plugins)
   * [4.7 Reporting Plugins](#47-reporting-plugins)
   * [4.8 AWS Plugins](#48-aws-plugins)
5. [Access Jenkins Plugin Manager](#5-access-jenkins-plugin-manager)
6. [Jenkins Plugin Installation Methods](#6-jenkins-plugin-installation-methods)
7. [Plugin Verification](#7-plugin-verification)
8. [Jenkins Configuration Management Using Ansible](#8-jenkins-configuration-management-using-ansible)
9. [Best Practices](#9-best-practices)
10. [Frequently Asked Questions (FAQs)](#10-frequently-asked-questions-faqs)
11. [Summary](#11-summary)
12. [Contact Information](#12-contact-information)
13. [References](#13-references)

---

<a id="1-introduction"></a>

## 1. Introduction

Jenkins plugins are software components that extend Jenkins functionality and enable integration with external tools, technologies, and automation platforms.

Plugins help Jenkins support CI/CD workflows, deployment automation, infrastructure provisioning, security testing, reporting, monitoring, and cloud integrations.

This documentation explains Jenkins plugin categories, installation methods, and enterprise CI/CD integration use cases.

---

<a id="2-purpose-of-jenkins-plugins"></a>

## 2. Purpose of Jenkins Plugins

Jenkins plugins are used to:

* Extend Jenkins core functionality
* Integrate CI/CD tools and services
* Automate infrastructure provisioning
* Enable deployment automation
* Perform security testing
* Generate build and test reports
* Support cloud and DevOps integrations

---

<a id="3-role-of-jenkins-plugins-in-cicd-workflow"></a>

## 3. Role of Jenkins Plugins in CI/CD Workflow

Jenkins plugins play a major role in CI/CD pipelines by enabling:

* Source code integration from SCM platforms
* Automated application builds
* Continuous testing workflows
* Infrastructure provisioning using Terraform
* Deployment automation using Ansible
* Security scanning using SAST and DAST tools
* Report publishing and monitoring
* Cloud integration and deployment orchestration

---

<a id="4-different-categories-of-jenkins-plugins"></a>

## 4. Jenkins Plugin Categories

For a modern CI/CD platform, Jenkins should remain an orchestration engine while build, testing, security scanning, infrastructure provisioning, and deployment tooling should be installed directly on the Jenkins agent or managed through automation tools such as Ansible.


<table>

<tr>
<td colspan="2" align="left">

<a id="41-core-jenkins-plugins"></a>

### 4.1 Essential Jenkins Plugins

</td>
</tr>

<tr>
<td valign="top">

| Plugin                      | Purpose                                          |
| --------------------------- | ------------------------------------------------ |
| Git Plugin                  | Source code integration with Git repositories    |
| Pipeline Plugin             | Jenkinsfile execution and pipeline orchestration |
| Credentials Plugin          | Secure storage of credentials and secrets        |
| Credentials Binding Plugin  | Inject credentials into pipeline stages          |
| SSH Agent Plugin            | SSH-based authentication for remote deployments  |
| AWS Credentials Plugin      | Secure access to AWS services                    |
| Config File Provider Plugin | Manage configuration files used by pipelines     |
| JUnit Plugin                | Publish unit test reports                        |
| HTML Publisher Plugin       | Publish SonarQube, OWASP and custom reports      |
| Workspace Cleanup Plugin    | Clean workspaces after pipeline execution        |


</td>
</tr>

</table>


<table>

<tr>
<td colspan="2" align="left">

<a id="42-terraform-plugins"></a>

### 4.2 Tools Installed Outside Jenkins
The following tools should be installed on Jenkins agents or target servers rather than through Jenkins plugins.
</td>
</tr>

<tr>
<td valign="top">

| Tool                   | Purpose                                 |
| ---------------------- | --------------------------------------- |
| Terraform              | Infrastructure provisioning             |
| Ansible                | Configuration management and deployment |
| Maven                  | Java application builds                 |
| Go                     | Employee API compilation                |
| NodeJS & npm           | Frontend builds                         |
| Python & Poetry        | Attendance API builds                   |
| SonarQube Scanner      | Static code analysis                    |
| OWASP Dependency-Check | Dependency vulnerability scanning       |
| OWASP ZAP              | Dynamic application security testing    |

</td>
</tr>

</table>

### Why Install These Tools Outside Jenkins?

* Reduces Jenkins plugin dependencies.
* Simplifies Jenkins upgrades.
* Improves platform stability.
* Reduces security risks.
* Enables reuse across different CI/CD platforms.
* Aligns with modern DevOps and Infrastructure-as-Code practices.
  
---

## 5. Access Jenkins Plugin Manager

Login to Jenkins Dashboard.

Navigate to:

```text
Manage Jenkins → Plugins
```

Select:

```text
Available Plugins
```

To install plugins:

1. Search plugin name
2. Select plugin checkbox
3. Click:

```text
Install without restart
```

or

```text
Download now and install after restart
```

---

<a id="6-jenkins-plugin-installation-methods"></a>

## 6. Jenkins Plugin Installation Methods

### 6.1 Plugin Installation via Jenkins UI

```text
Manage Jenkins → Plugins → Available Plugins
```

---

### 6.2 Plugin Installation Using Jenkins CLI

```bash
java -jar jenkins-cli.jar -s http://<jenkins-server>:8080/ install-plugin git
```

---

### 6.3 Plugin Installation Using Plugin Manager Tool

```bash
java -jar jenkins-plugin-manager.jar --plugin-file plugins.txt
```

---

### 6.4 Plugin Installation Using Docker

```dockerfile
FROM jenkins/jenkins:lts
RUN jenkins-plugin-cli --plugins git terraform ansible sonar zap
```

---

### 6.5 Plugin Installation Using JCasC

```yaml
plugins:
  required:
    - git
    - terraform
    - ansible
```

---

### 6.6 Plugin Installation Using Ansible

```yaml
- name: Install Jenkins Plugins
  jenkins_plugin:
    name: git
```

---

<a id="7-plugin-verification"></a>

## 7. Plugin Verification

Navigate to:

```text
Manage Jenkins → Plugins → Installed Plugins
```

Verify plugin status:

```text
Enabled
```

Verify pipeline support:

```text
Dashboard → New Item → Pipeline
```

---

<a id="8-jenkins-configuration-management-using-ansible"></a>

## 8. Jenkins Configuration Management Using Ansible

Ansible can automate Jenkins installation, plugin management, configuration backup, and pipeline provisioning.

### Plugin Installation Playbook

```yaml
- name: Install Jenkins Plugins
  hosts: jenkins

  tasks:
    - name: Install plugins
      jenkins_plugin:
        name: "{{ item }}"
      loop:
        - git
        - terraform
        - ansible
```
---

<a id="9-best-practices"></a>

## 9. Best Practices

- Install only required Jenkins plugins to avoid unnecessary resource usage and security risks.
- Regularly update plugins to stable versions and monitor plugin compatibility.
- Remove unused or deprecated plugins from Jenkins environments.
- Use trusted plugins from the official Jenkins Plugin Repository only.
- Manage plugin installation and configuration using automation tools like Ansible or JCasC.

---

<a id="10-frequently-asked-questions-faqs"></a>

## 10. Frequently Asked Questions (FAQs)

### Q1. Why are Jenkins plugins required?

Jenkins plugins extend Jenkins functionality and enable integrations with Terraform, Ansible, SonarQube, OWASP ZAP, AWS, and build tools.

---

### Q2. Which plugins are mandatory for enterprise CI/CD pipelines?

Commonly used plugins include Git Plugin, Pipeline Plugin, Terraform Plugin, Ansible Plugin, SonarQube Scanner Plugin, and OWASP ZAP Plugin.

---

### Q3. Why is SonarQube Scanner Plugin required?

It enables Static Application Security Testing (SAST) and code quality analysis.

---

### Q4. Why is OWASP ZAP Plugin required?

It performs Dynamic Application Security Testing (DAST) after deployment.

---

### Q5. Can Jenkins plugins be installed without restart?

Yes. Jenkins supports:

```text
Install without restart
```

---

<a id="11-summary"></a>

## 11. Summary

This Jenkins plugin installation setup provides:

* CI/CD automation
* Infrastructure provisioning support
* Deployment automation
* SAST integration
* DAST integration
* Multi-language build support
* AWS deployment integration

---

<a id="12-contact-information"></a>

## 12. Contact Information

| Name         | Contact                                                                           |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

<a id="13-references"></a>

## 13. References

| S.No | Description                    | Click to View                                                                                                                                                   |
| ---- | ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | Jenkins Official Documentation | [![Jenkins](https://img.shields.io/badge/Jenkins-DOCUMENTATION-D24939?style=flat-square\&logo=jenkins\&logoColor=white)](https://www.jenkins.io/doc/)           |
| 2    | Terraform Jenkins Plugin       | [![Terraform](https://img.shields.io/badge/Terraform-PLUGIN-623CE4?style=flat-square\&logo=terraform\&logoColor=white)](https://plugins.jenkins.io/terraform/)  |
| 3    | Ansible Jenkins Plugin         | [![Ansible](https://img.shields.io/badge/Ansible-PLUGIN-EE0000?style=flat-square\&logo=ansible\&logoColor=white)](https://plugins.jenkins.io/ansible/)          |
| 4    | SonarQube Jenkins Plugin       | [![SonarQube](https://img.shields.io/badge/SonarQube-PLUGIN-4E9BCD?style=flat-square\&logo=sonarqube\&logoColor=white)](https://plugins.jenkins.io/sonar/)      |
| 5    | OWASP ZAP Plugin               | [![OWASP ZAP](https://img.shields.io/badge/OWASP_ZAP-PLUGIN-000000?style=flat-square\&logo=owasp\&logoColor=white)](https://plugins.jenkins.io/zap/)            |
| 6    | Jenkins Pipeline Documentation | [![Pipeline](https://img.shields.io/badge/Jenkins-Pipeline-2C5263?style=flat-square\&logo=jenkins\&logoColor=white)](https://www.jenkins.io/doc/book/pipeline/) |
