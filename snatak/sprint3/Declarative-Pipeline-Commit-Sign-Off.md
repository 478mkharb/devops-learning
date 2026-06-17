# <h1 align="center">Declarative Pipeline for Commit Signoff Validation</h1>

<div align="center">
<img width="120" alt="Jenkins" src="https://www.jenkins.io/images/logos/jenkins/jenkins.png" />
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
    <td align="center">17/06/2026</td>
    <td align="center">1.0</td>
    <td align="center">Mukesh Kharb</td>
    <td align="center">17/06/2026</td>
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
4. [Create Jenkinsfile](#4-create-jenkinsfile)
5. [Commit Signoff Validation Logic](#5-commit-signoff-validation-logic)
6. [Configure Jenkins Pipeline Job](#6-configure-jenkins-pipeline-job)
7. [Execute Pipeline](#7-execute-pipeline)
8. [Validate Success Scenario](#8-validate-success-scenario)
9. [Validate Failure Scenario](#9-validate-failure-scenario)
10. [Common Errors and Troubleshooting](#10-common-errors-and-troubleshooting)
11. [Conclusion](#11-conclusion)
12. [Contact Information](#12-contact-information)
13. [References](#13-references)

---

<a id="1-introduction"></a>

## 1. Introduction

This POC demonstrates how to implement a Jenkins Declarative Pipeline to validate Git Commit Signoff compliance using the Developer Certificate of Origin (DCO) standard.

The pipeline automatically verifies whether the latest commit contains a valid:

```text
Signed-off-by: Developer Name <developer@email.com>
```

entry in the commit message.

This validation helps teams:

- Enforce commit ownership
- Improve code traceability
- Maintain audit compliance
- Follow Open Source contribution standards
- Prevent unsigned code from entering repositories

---

<a id="2-pre-requisites"></a>

## 2. Pre-requisites

| Component | Version |
|------------|------------|
| Ubuntu Server | 24.04 LTS |
| Jenkins | 2.528+ |
| Git | 2.43+ |
| GitHub Account | Required |
| GitHub PAT | Required |
| Jenkins Git Plugin | Installed |

---

<a id="3-branch-creation"></a>

## 3. Branch Creation

Create a feature branch using the SCRUM ticket number.

```bash
git checkout main
git pull origin main

git checkout -b SCRUM-219-mukesh
```

Verify current branch:

```bash
git branch
```

Expected Output:

```text
* SCRUM-219-mukesh
```

<details>
<summary>📸 <strong>Click to view Screenshot - Branch Creation</strong></summary>

<img width="1501" height="641" alt="image" src="https://github.com/user-attachments/assets/84a8ca37-316d-42bb-848a-ee6d8b6a2d67" />


</details>

---

<a id="4-create-jenkinsfile"></a>

## 4. Create Jenkinsfile

Create the Jenkins pipeline file:

```bash
touch SCRUM-219-Jenkinsfile
```

Add the following Jenkins Pipeline:

```groovy
pipeline {
    agent any

    options {
        timestamps()
    }

    stages {

        stage('Commit Signoff Validation') {
            steps {
                sh '''
                    
                    echo "Latest Commit Details"

                    git log -1

                    echo ""
                    
                    echo "Validating Signed-off-by Footer"
                    
                    git log -1 --pretty=%B | grep -i "Signed-off-by:"
                '''
            }
        }
    }

    post {

        success {
            echo 'Commit signoff validation passed'
        }

        failure {
            echo 'Commit signoff validation failed'
        }

        always {
            cleanWs()
        }
    }
}
```

---

<a id="5-commit-signoff-validation-logic"></a>

## 5. Commit Signoff Validation Logic

The validation is performed using:

```bash
git log -1 --pretty=%B | grep -i "Signed-off-by:"
```

### Logic Flow

1. Fetch latest commit message
2. Search for `Signed-off-by:`
3. If found → Pipeline Passes
4. If missing → Pipeline Fails

Example Valid Commit:

```text
SCRUM-219 Updated README

Signed-off-by: Mukesh Kharb <mukesh.Kharb.snaatak@mygurukulam.co>
```

Example Invalid Commit:

```text
SCRUM-219 Updated README
```

---

<a id="6-configure-jenkins-pipeline-job"></a>

## 6. Configure Jenkins Pipeline Job

Navigate to Folder:

```text
CI_Implementation
    └── Declarative_Pipeline
          
```

Configure:

### General

```text
Pipeline Name:
Commit-Signoff-Validation
```

### Pipeline

```text
Definition:
Pipeline Script from SCM
```

### SCM

```text
Git
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
*/SCRUM-219-mukesh
```

Script Path:

```text
SCRUM-219-Jenkinsfile
```

Save configuration.

<details>
<summary>📸 <strong>Click to view Screenshot - Jenkins Job Configuration</strong></summary>
<img width="1826" height="981" alt="image" src="https://github.com/user-attachments/assets/bc81933f-4ab9-4b29-ac8a-e8cb60665d0f" />
</details>

---

<a id="7-execute-pipeline"></a>

## 7. Execute Pipeline

Run the job:

```text
Build Now
```

Jenkins will:

1. Checkout repository
2. Read Jenkinsfile
3. Validate latest commit
4. Mark build status

Expected Build Stages:

```text
Checkout Jenkins Repository
Commit Signoff Validation
```

<details>
<summary>📸 <strong>Click to view Screenshot - Pipeline Execution</strong></summary>

<img width="1826" height="981" alt="image" src="https://github.com/user-attachments/assets/0bd13a1a-7304-4e16-b5b3-9a5277cb7ad3" />

</details>

---

<a id="12-contact-information"></a>

## 12. Contact Information

| Name | ✉️ Contact |
|------------|----------------------------------------------------------------------------------|
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

<a id="13-references"></a>

## 13. References

| S.No | Description | Click to View |
| ---- | ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1 | Jenkins Pipeline Documentation | [![Jenkins](https://img.shields.io/badge/JENKINS-DOCUMENTATION-2F2F2F?style=flat-square&logo=jenkins&logoColor=white)](https://www.jenkins.io/doc/book/pipeline/) |
| 2 | Git Commit Documentation | [![Git](https://img.shields.io/badge/GIT-DOCUMENTATION-3A3A3A?style=flat-square&logo=git&logoColor=white)](https://git-scm.com/docs/git-commit) |
| 3 | Developer Certificate of Origin (DCO) | [![DCO](https://img.shields.io/badge/DCO-DOCUMENTATION-1F1F1F?style=flat-square)](https://developercertificate.org/) |
| 4 | Jenkins Git Plugin | [![Jenkins Git Plugin](https://img.shields.io/badge/JENKINS-GIT_PLUGIN-404040?style=flat-square&logo=jenkins&logoColor=white)](https://plugins.jenkins.io/git/) |

---
