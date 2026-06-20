# <h1 align="center">GoLang Declarative Pipeline for Dependency Scanning using Trivy</h1>

<div align="center">
 <img width="200"  alt="trivy-horizontal-featured-image" src="https://github.com/user-attachments/assets/ba1ba1e0-2f01-4847-8deb-3f6d75ff5e9a" />

</div>

<br/>

---

<div align="center">

| Author       | Created On | Version | Last Updated By | Last Edited On | Pre Reviewer | L0 Reviewer | L1 Reviewer | L2 Reviewer  |
| ------------ | ---------- | ------- | --------------- | -------------- | ------------ | ----------- | ----------- | ------------ |
| Mukesh Kharb | 20/06/2026 | 1.0     | Mukesh Kharb    | 20/06/2026     | Team         | Mohit Kumar | Faisal Khan | Mahesh Kumar |

</div>

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Pre-requisites](#2-pre-requisites)
3. [Branch Creation](#3-branch-creation)
4. [Trivy Installation](#4-trivy-installation)
5. [Create Jenkins Pipeline](#5-create-jenkins-pipeline)
6. [Configure Jenkins Pipeline Job](#6-configure-jenkins-pipeline-job)
7. [Execute Pipeline](#7-execute-pipeline)
8. [Review Trivy Reports](#8-review-trivy-reports)
9. [Slack & Email Notifications](#9-slack--email-notifications)
10. [Common Issues and Troubleshooting](#10-common-issues-and-troubleshooting)
11. [Contact Information](#11-contact-information)

---

<a id="1-introduction"></a>

## 1. Introduction

This document demonstrates the implementation of a Jenkins Declarative Pipeline for Dependency Vulnerability Scanning of the GoLang Employee_API application using Trivy.


---

<a id="2-pre-requisites"></a>

## 2. Pre-requisites

| Component              | Version    |
| ---------------------- | ---------- |
| Ubuntu Server          | 24.04 LTS  |
| Jenkins                | 2.528+     |
| Go                     | Installed  |
| Trivy                  | 0.71.2     |
| HTML Publisher Plugin  | Installed  |
| Slack Plugin           | Installed  |
| Email Extension Plugin | Installed  |
| Shared Library         | Configured |

---

<a id="3-branch-creation"></a>

## 3. Branch Creation

Create a feature branch using the Jira ticket number.

```bash
cd Jenkins

git checkout main

git pull origin main

git checkout -b SCRUM-290-mukesh
```

Verify:

```bash
git branch
```

Expected Output:

```text
* SCRUM-290-mukesh
```

<details>
<summary>📸 <strong>Click to view Screenshot - Branch Creation</strong></summary>

<img width="1368" height="316" alt="image" src="https://github.com/user-attachments/assets/542f6897-94a4-47c9-982f-50f5e3988024" />


</details>

---

<a id="4-trivy-installation"></a>

## 4. Trivy Installation

Install Trivy on Jenkins Server.

```bash
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | \
gpg --dearmor | \
sudo tee /usr/share/keyrings/trivy.gpg > /dev/null

echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb noble main" | \
sudo tee /etc/apt/sources.list.d/trivy.list

sudo apt-get update

sudo apt-get install trivy -y
```

Verify Installation:

```bash
trivy --version
```

Expected Output:

```text
Version: 0.71.2
```

<details>
<summary>📸 <strong>Click to view Screenshot - Trivy Installation</strong></summary>

<img width="960" height="272" alt="image" src="https://github.com/user-attachments/assets/c49006a4-65af-4020-853c-25f3af560882" />


</details>

---

<a id="5-create-jenkins-pipeline"></a>

## 5. Create Jenkins Pipeline

Create Pipeline File:

```text
Declarative_Pipeline/
    └── GoLang/
          └── Dependency_Scanning
```

Push Jenkinsfile:

```bash
git add .

git commit -s -m "SCRUM-290: Implement Go dependency scanning using Trivy"

git push origin SCRUM-290-mukesh
```

Expected Sign-Off:

```text
Signed-off-by: Mukesh Kharb <mukesh.Kharb.snaatak@mygurukulam.co>
```

<details>
<summary>📸 <strong>Click to view Screenshot - Jenkinsfile Commit</strong></summary>

<img width="1499" height="549" alt="image" src="https://github.com/user-attachments/assets/d4038e08-7ef8-4abe-97e8-4a31bf92b920" />

</details>

---

<a id="6-configure-jenkins-pipeline-job"></a>

## 6. Configure Jenkins Pipeline Job

Navigate:

```text
CI Implementation
    └── Declarative-Pipeline
          └── GoLang
                └── Dependency_Scanning
```

Pipeline Configuration:

```text
Definition:
Pipeline Script from SCM
```

Repository URL:

```text
https://github.com/Snaatak-Infra-Titans/Jenkins.git
```

Credentials:

```text
github-creds
```

Branch:

```text
*/SCRUM-290-mukesh
```

Script Path:

```text
Declarative_Pipeline/GoLang/Dependency_Scanning
```

Save Configuration.

<details>
<summary>📸 <strong>Click to view Screenshot - Jenkins Job Configuration</strong></summary>

<img width="1830" height="986" alt="image" src="https://github.com/user-attachments/assets/b982b39e-ee39-4f4a-8abd-b37de37d159b" />

</details>

---

<a id="7-execute-pipeline"></a>

## 7. Execute Pipeline

Run Jenkins Job:

```text
Build Now
```

Pipeline Execution Flow:

1. Checkout Employee_API
2. Verify Go Environment
3. Download Dependencies
4. Download Trivy Template
5. Execute Dependency Scan
6. Generate Reports
7. Publish HTML Report
8. Send Notifications
9. Archive Artifacts

<details>
<summary>📸 <strong>Click to view Screenshot - Successful Pipeline Execution</strong></summary>

<img width="1830" height="986" alt="image" src="https://github.com/user-attachments/assets/114a87ac-067e-4f4e-b7e9-3994ee6b4ea5" />


</details>

---

<a id="8-review-trivy-reports"></a>

## 8. Review Trivy Reports

Generated Reports:

```text
trivy-report.txt
trivy-report.json
trivy-report.html
```

Published Report:

```text
Trivy Dependency Scan Report
```

Sample Findings:

| Severity | Count |
| -------- | ----- |
| Critical | 1     |
| High     | 2     |

Detected Vulnerabilities:

| Package             | CVE            | Severity |
| ------------------- | -------------- | -------- |
| golang.org/x/crypto | CVE-2024-45337 | Critical |
| golang.org/x/crypto | CVE-2025-22869 | High     |
| golang.org/x/net    | CVE-2023-39325 | High     |

<details>
<summary>📸 <strong>Click to view Screenshot - Trivy HTML Report</strong></summary>

<img width="1830" height="986" alt="image" src="https://github.com/user-attachments/assets/0ff54c3e-4541-40dc-92e7-312e20fed9ff" />

</details>


---

<a id="9-slack--email-notifications"></a>

## 9. Slack & Email Notifications

Shared Library:

```text
Shared_Library@SCRUM-228-deepak
```

Notification Function:

```groovy
notifyTeam()
```

Configured Slack Channel:

```text
#jenkins-alerts
```

Configured Email:

```text
mukesh.Kharb.snaatak@mygurukulam.co
```

Notifications Sent:

* Success Notification
* Failure Notification
* Email Report
* Build Information
* Direct Jenkins Build URL

<details>
<summary>📸 <strong>Click to view Screenshot - Slack Notification</strong></summary>
<img width="1839" height="908" alt="image" src="https://github.com/user-attachments/assets/7aafc420-5da3-437a-b803-e79fed152ef2" />

</details>

<details>
<summary>📸 <strong>Click to view Screenshot - Email Notification</strong></summary>
<img width="1839" height="908" alt="image" src="https://github.com/user-attachments/assets/e91e40e1-535f-49a6-93eb-1a270a35a07d" />

</details>

---

<a id="10-common-issues-and-troubleshooting"></a>

## 10. Common Issues and Troubleshooting

| Issue                     | Cause                         | Resolution                 |
| ------------------------- | ----------------------------- | -------------------------- |
| Trivy Not Found           | Trivy Not Installed           | Install Trivy              |
| HTML Report Missing       | HTML Publisher Plugin Missing | Install Plugin             |
| Git Checkout Failed       | Invalid Credentials           | Verify github-creds        |
| Slack Notification Failed | Invalid Slack Configuration   | Verify Slack Plugin        |
| Email Failed              | SMTP Configuration Issue      | Verify Email Configuration |
| HTML Report Not Rendering | Invalid Template Download     | Download Template Again    |
| Vulnerabilities Detected  | Outdated Dependencies         | Upgrade Dependency Version |

---

<a id="11-contact-information"></a>

## 11. Contact Information

| Name         | Contact                                                                           |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---
