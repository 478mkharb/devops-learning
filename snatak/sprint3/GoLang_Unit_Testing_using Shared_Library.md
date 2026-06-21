# <h1 align="center">GoLang Shared Library for Unit Testing</h1>

<div align="center">
<img width="100" alt="GoLang" src="https://go.dev/blog/go-brand/Go-Logo/PNG/Go-Logo_Blue.png" />
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
    <td align="center">21/06/2026</td>
    <td align="center">1.0</td>
    <td align="center">Mukesh Kharb</td>
    <td align="center">21/06/2026</td>
    <td align="center">Team</td>
    <td align="center">Mohit Kumar</td>
    <td align="center">Faisal Khan</td>
    <td align="center">Mahesh Kumar</td>
  </tr>

</table>

</div>

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Pre-requisites](#2-pre-requisites)
3. [Branch Creation](#3-branch-creation)
4. [Create Shared Library Function](#4-create-shared-library-function)
5. [Create Jenkinsfile](#5-create-jenkinsfile)
6. [Configure Jenkins Pipeline Job](#6-configure-jenkins-pipeline-job)
7. [Execute Pipeline](#7-execute-pipeline)
8. [Review Coverage Report](#8-review-coverage-report)
9. [Review Notifications](#9-review-notifications)
10. [Common Issues and Troubleshooting](#10-common-issues-and-troubleshooting)
11. [Contact Information](#11-contact-information)

---

<a id="1-introduction"></a>

## 1. Introduction

This POC demonstrates the implementation of a Jenkins Shared Library for GoLang Unit Testing.

The implementation uses a reusable Shared Library function that executes unit tests for the Employee API and generates test coverage reports.

---

<a id="2-pre-requisites"></a>

## 2. Pre-requisites

| Component               | Requirement              |
| ----------------------- | ------------------------ |
| Jenkins                 | Installed and Configured |
| Shared Library          | Configured in Jenkins    |
| GoLang                  | Installed                |
| GitHub Credentials      | Configured               |
| Slack Integration       | Configured               |
| Email Notification      | Configured               |
| Employee API Repository | Accessible               |

---

<a id="3-branch-creation"></a>

## 3. Branch Creation

```bash
cd Shared_Library

git checkout main

git pull origin main

git checkout -b SCRUM-300-mukesh origin/SCRUM-228-deepak
```
The Shared Library branch for this implementation was created from the existing notification shared library branch.
The SCRUM-228 implementation already contained the notification framework used for Slack and Email notifications.

Verify current branch:

```bash
git branch
```

Expected Output:

```text
* SCRUM-300-mukesh
```

<details>
<summary>📸 <strong>Click to view Screenshot - Branch Creation</strong></summary>
<img width="1315" height="270" alt="image" src="https://github.com/user-attachments/assets/c524036a-f065-4ba8-b18d-d761b34c85f9" />
<img width="1288" height="77" alt="image" src="https://github.com/user-attachments/assets/a5eb2e27-dfc0-4fda-a8f0-098cbc5528a5" />

</details>

---

<a id="4-create-shared-library-function"></a>

## 4. Create Shared Library Function

Create the Shared Library function:

```text
vars/
 └── goUnitTesting.groovy
```

Create the file:

```bash
nano vars/goUnitTesting.groovy
```

Function Purpose:

* Verify Go environment
* Execute Go unit tests
* Generate coverage.out
* Generate coverage.html

Verify file creation:

```bash
cat vars/goUnitTesting.groovy
```

<details>
<summary>📸 <strong>Click to view Screenshot - Shared Library Function</strong></summary>
<img width="1329" height="390" alt="image" src="https://github.com/user-attachments/assets/80866832-89b2-4271-a805-75f34bbe9ae5" />

</details>

---

<a id="5-create-jenkins-pipeline"></a>

## 5. Create Jenkinsfile

Create Pipeline Script:

```text
Declarative_Pipeline/
 └── GoLang/
      └── Unit_Testing_using_Shared_Lib
```

Push Changes:

```bash
git add .

git commit -s -m "SCRUM-300: Implement GoLang shared library for unit testing"

git push origin SCRUM-300-mukesh
```

<details>
<summary>📸 <strong>Click to view Screenshot - Jenkinsfile Creation</strong></summary>
<img width="1386" height="722" alt="image" src="https://github.com/user-attachments/assets/98725910-8ad1-4110-bdd0-c117c20f3579" />

</details>

---

<a id="6-configure-jenkins-pipeline-job"></a>

## 6. Configure Jenkins Pipeline Job

Navigate to:

```text
CI Implementation
    └── Shared-Library
          └── GoLang
                └── Unit_Testing
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
*/SCRUM-300-mukesh
```

Script Path:

```text
Declarative_Pipeline/GoLang/Unit_Testing_using_Shared_Lib
```

Save Configuration.

<details>
<summary>📸 <strong>Click to view Screenshot - Jenkins Job Configuration</strong></summary>

<img width="1842" height="879" alt="image" src="https://github.com/user-attachments/assets/f20c9f06-925e-4f05-a1dc-aa5db6e7b942" />


</details>

---

<a id="7-execute-pipeline"></a>

## 7. Execute Pipeline

Run the Jenkins Job:

```text
Build Now
```

Pipeline Execution Flow:

1. Checkout SCM
2. Checkout Employee API
3. Execute Shared Library Function
4. Run GoLang Unit Tests
5. Generate Coverage Report
6. Archive Artifacts
7. Send Notifications

Build Result:

```text
SUCCESS
```

<details>
<summary>📸 <strong>Click to view Screenshot - Successful Pipeline Execution</strong></summary>

<img width="1842" height="879" alt="image" src="https://github.com/user-attachments/assets/c5f0c3a2-52eb-48e1-be07-2a3c54e9860f" />


</details>

---

<a id="8-review-coverage-report"></a>

## 8. Review Coverage Report

Coverage artifacts generated:

```text
coverage.out
coverage.html
```

Access Coverage Report:

```text
Unit_Testing
    └── GoLang Coverage Report
```

Generated Reports:

| Report        | Purpose                     |
| ------------- | --------------------------- |
| coverage.out  | Raw coverage data           |
| coverage.html | HTML coverage visualization |

<details>
<summary>📸 <strong>Click to view Screenshot - Coverage Report</strong></summary>

<img width="1852" height="985" alt="image" src="https://github.com/user-attachments/assets/48b6fb85-ff9c-4b76-ac5d-2735060a4534" />
<img width="1852" height="985" alt="image" src="https://github.com/user-attachments/assets/78e5aa74-2f22-4bab-bc7e-5efab8afc0bb" />


</details>

---

<a id="9-review-notifications"></a>

## 9. Review Notifications

Notifications are sent after pipeline execution.

Notification Channels:

* Slack
* Email

Success Notification Includes:

* Job Name
* Build Number
* Build Status
* Build URL

<details>
<summary>📸 <strong>Click to view Screenshot - Slack Notification</strong></summary>
<img width="1849" height="592" alt="image" src="https://github.com/user-attachments/assets/9b689c3b-4b1e-4695-92b7-0ed22c82f360" />

</details>

<br>

<details>
<summary>📸 <strong>Click to view Screenshot - Email Notification</strong></summary>

<img width="1111" height="668" alt="image" src="https://github.com/user-attachments/assets/c0dcafd9-d40f-4c9d-b14a-2a9449efc883" />


</details>

---

<a id="10-common-issues-and-troubleshooting"></a>

## 10. Common Issues and Troubleshooting

| Issue                     | Cause                      | Resolution                          |
| ------------------------- | -------------------------- | ----------------------------------- |
| Shared Library Not Found  | Incorrect Library Name     | Verify Shared Library Configuration |
| Git Checkout Failed       | Invalid Credentials        | Verify GitHub Credentials           |
| go Command Not Found      | GoLang Missing             | Install GoLang                      |
| Unit Tests Failed         | Test Case Failure          | Review Console Logs                 |
| Coverage Report Missing   | Coverage Generation Failed | Verify go test Command              |
| Slack Notification Failed | Slack Configuration Issue  | Verify Slack Plugin Configuration   |
| Email Notification Failed | SMTP Configuration Issue   | Verify Email Settings               |

---

<a id="11-contact-information"></a>

## 11. Contact Information

| Name         | ✉️ Contact                                                                        |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

**Documentation Location**

```text
Documentation/
└── CI_Implementation/
    └── GoLang_Shared_Library/
            └── Unit_Testing/
                └── README.md
```

---

**SCRUM Ticket:** SCRUM-300
**Module:** GoLang Shared Library – Unit Testing
**Repository:** Shared_Library + Jenkins + Employee_API
