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
   * [4.1 Essential Jenkins Plugins](#41-essential-jenkins-plugins)
   * [4.2 Tools Installed Outside Jenkins](#42-tools-installed-outside-jenkins)
5. [Access Jenkins Plugin Manager](#5-access-jenkins-plugin-manager)
6. [Jenkins Plugin Installation Methods](#6-jenkins-plugin-installation-methods)
   * [6.1 Plugin Installation via Jenkins UI](#61-plugin-installation-via-jenkins-ui)
   * [6.2 Plugin Installation Using Direct Download](#62-plugin-installation-using-direct-download)
   * [6.3 Plugin Installation Using Jenkins Plugin Manager Tool](#63-plugin-installation-using-jenkins-plugin-manager-tool)
8. [Plugin Verification](#7-plugin-verification)
9. [Jenkins Configuration Management Using Ansible](#8-jenkins-configuration-management-using-ansible)
10. [Best Practices](#9-best-practices)
11. [Frequently Asked Questions (FAQs)](#10-frequently-asked-questions-faqs)
12. [Summary](#11-summary)
13. [Contact Information](#12-contact-information)
14. [References](#13-references)

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

> [!NOTE]
> Although Jenkins plugins exist for tools such as Terraform, Ansible, SonarQube, OWASP Dependency-Check, and OWASP ZAP, it is generally recommended to install and execute these tools directly on Jenkins agents rather than relying on Jenkins-specific plugins.
>
> Benefits:
> - Reduces Jenkins plugin dependencies.
> - Simplifies Jenkins upgrades and maintenance.
> - Improves platform stability and reliability.
> - Minimizes security risks associated with excessive plugins.
> - Enables reuse across multiple CI/CD platforms.
> - Aligns with modern DevOps and Infrastructure-as-Code (IaC) practices.
  
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

### 6.2 Plugin Installation Using Direct Download

### Download AWS Credentials Plugin

```bash
cd /var/lib/jenkins/plugins

sudo wget https://updates.jenkins.io/latest/aws-credentials.hpi \
-O aws-credentials.jpi

sudo chown jenkins:jenkins aws-credentials.jpi

sudo systemctl restart jenkins
```

### Verify Plugin Installation

Navigate to:

```text
Manage Jenkins
 → Plugins
 → Installed Plugins
```

Search for:

```text
AWS Credentials Plugin
```

<details>
<summary>📸 <strong>Click to view Screenshot</strong></summary>

<img width="1685" height="894" alt="image" src="https://github.com/user-attachments/assets/3a5ed198-6c41-4de4-aa70-061a64c24cd4" />

</details>

---

### 6.3 Plugin Installation Using Jenkins Plugin Manager Tool

Create Working Directory
```bash
mkdir -p ~/jenkins-plugin-install
cd ~/jenkins-plugin-install
```
Create plugins.txt

```bash
cat > plugins.txt << EOF
git
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
EOF
```
Verify:

```bash
cat plugins.txt
```
Download Jenkins Plugin Manager Tool

```bash
wget https://github.com/jenkinsci/plugin-installation-manager-tool/releases/download/2.13.2/jenkins-plugin-manager-2.13.2.jar \
-O jenkins-plugin-manager.jar
```

Verify:

```bash
ls -lh jenkins-plugin-manager.jar
```
Create a temporary directory:

```bash
mkdir -p snaatak-P18-plugins
```
Display all plugin dependencies:

```bash
java -jar jenkins-plugin-manager.jar \
  --plugin-file plugins.txt \
  --plugin-download-directory snaatak-P18-plugins \
  --list
```
Backup Existing Jenkins Plugins

```bash
sudo tar -czvf \
/tmp/jenkins-plugins-backup-$(date +%F).tar.gz \
/var/lib/jenkins/plugins
```
Stop Jenkins

```bash
sudo systemctl stop jenkins
```
Install Plugins

```bash
sudo java -jar jenkins-plugin-manager.jar \
  --plugin-file plugins.txt \
  --plugin-download-directory /var/lib/jenkins/plugins
```
Fix Permissions

```bash
sudo chown -R jenkins:jenkins /var/lib/jenkins/plugins
```
Start Jenkins

```bash
sudo systemctl start jenkins
```

Verify Installed Plugins
```text
Manage Jenkins
 → Plugins
 → Installed Plugins
```
<details>
<summary>📸 <strong>Click to view Screenshot</strong></summary>
<img width="1446" height="317" alt="image" src="https://github.com/user-attachments/assets/6f405243-a74a-4d9f-9642-34e6e5c5667a" />
<img width="1612" height="891" alt="image" src="https://github.com/user-attachments/assets/7e568a43-ce46-4eee-933e-2e95aedbaa31" />
<img width="1597" height="198" alt="image" src="https://github.com/user-attachments/assets/f6efe3aa-b86d-4ce7-85b2-343842d34e6f" />

</details>


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

Jenkins plugins extend Jenkins capabilities for source control integration, pipeline execution, credential management, report publishing, and deployment automation.

---

### Q2. Which plugins are required for a modern CI/CD platform?

Recommended plugins:

* Git Plugin
* Pipeline Plugin
* Credentials Plugin
* Credentials Binding Plugin
* SSH Agent Plugin
* AWS Credentials Plugin
* Config File Provider Plugin
* JUnit Plugin
* HTML Publisher Plugin
* Workspace Cleanup Plugin

---

### Q3. Should Terraform Plugin be installed?

No.

Terraform should be installed directly on Jenkins agents and executed through pipeline shell commands.

---

### Q4. Should Ansible Plugin be installed?

No.

Ansible should be installed on Jenkins agents and executed through standard CLI commands from Jenkins pipelines.

---

### Q5. Should SonarQube Scanner Plugin be installed?

Optional.

The SonarQube Scanner CLI can be installed directly on Jenkins agents. The plugin is only required if Jenkins-to-SonarQube integration features are needed.

---

### Q6. Should OWASP Dependency-Check Plugin be installed?

Optional.

Dependency-Check CLI can be executed directly from Jenkins pipelines and reports can be published using HTML Publisher.

---

### Q7. Should OWASP ZAP Plugin be installed?

No.

OWASP ZAP should run as a standalone security testing tool and be invoked from Jenkins pipelines.

---

<a id="11-summary"></a>

## 11. Summary

This Jenkins plugin strategy focuses on keeping Jenkins lightweight and maintainable.

### Essential Jenkins Plugins

* Git Plugin
* Pipeline Plugin
* Credentials Plugin
* Credentials Binding Plugin
* SSH Agent Plugin
* AWS Credentials Plugin
* Config File Provider Plugin
* JUnit Plugin
* HTML Publisher Plugin
* Workspace Cleanup Plugin

### External Tools

* Terraform
* Ansible
* Maven
* Go
* NodeJS
* Python
* SonarQube Scanner
* OWASP Dependency-Check
* OWASP ZAP

This approach minimizes plugin sprawl, simplifies maintenance, improves security, and follows modern CI/CD best practices.


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
