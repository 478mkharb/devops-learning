# <h1 align="center">React Declarative Pipeline for Static Code Analysis using SonarQube</h1>

<div align="center">
 <img width="200" height="auto" alt="DV-SonarQube" src="https://github.com/user-attachments/assets/36f08d50-e4ca-4704-9020-f0c5c8cb18dc" />
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
    <td align="center">19/06/2026</td>
    <td align="center">1.0</td>
    <td align="center">Mukesh Kharb</td>
    <td align="center">19/06/2026</td>
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
4. [SonarQube Configuration](#4-sonarqube-configuration)
5. [Create Jenkins Pipeline](#5-create-jenkins-pipeline)
6. [Configure SonarQube Credentials](#6-configure-sonarqube-credentials)
7. [Configure Jenkins Pipeline Job](#7-configure-jenkins-pipeline-job)
8. [Execute Pipeline](#8-execute-pipeline)
9. [Review SonarQube Report](#9-review-sonarqube-report)
10. [Common Issues and Troubleshooting](#10-common-issues-and-troubleshooting)
11. [Contact Information](#11-contact-information)

---

<a id="1-introduction"></a>

## 1. Introduction

This POC demonstrates how to implement a Jenkins Declarative Pipeline for performing Static Code Analysis on the Frontend React application using SonarQube.

The pipeline automatically:

* Checks out source code from GitHub
* Validates Commit Sign-Off compliance
* Verifies NodeJS and NPM installation
* Installs application dependencies
* Performs Static Code Analysis using SonarQube
* Publishes quality metrics and reports

This implementation helps teams:

* Improve code quality
* Detect bugs and vulnerabilities early
* Monitor code maintainability
* Reduce code duplication
* Maintain coding standards across repositories

---

<a id="2-pre-requisites"></a>

## 2. Pre-requisites

| Component          | Version                   |
| ------------------ | ------------------------- |
| Ubuntu Server      | 24.04 LTS                 |
| Jenkins            | 2.528+                    |
| NodeJS             | Installed                 |
| NPM                | Installed                 |
| SonarQube Server   | Configured                |
| Sonar Scanner      | Installed                 |
| GitHub Account     | Required                  |
| GitHub Credentials | Configured                |
| Sonar Token        | Configured as Secret Text |

---

<a id="3-branch-creation"></a>

## 3. Branch Creation

Create a feature branch using the SCRUM ticket number.

```bash
cd Jenkins
git checkout main
git pull origin main

git checkout -b SCRUM-250-mukesh

```

Verify current branch:

```bash
git branch
```

Expected Output:

```text
* SCRUM-250-mukesh
```

<details>
<summary>📸 <strong>Click to view Screenshot - Branch Creation</strong></summary>

<img width="1415" height="512" alt="image" src="https://github.com/user-attachments/assets/716f7159-c127-429d-9fe1-765c5f920eb4" />

</details>

---

<a id="4-sonarqube-configuration"></a>

## 4. SonarQube Configuration

Verify SonarQube server configuration in Jenkins.

Navigate to:

```text
Manage Jenkins
    └── System
          └── SonarQube Servers
```

Configured SonarQube Server:

```text
Name: SonarQube
URL : http://192.168.8.17/:9000
```

Verify Sonar Scanner Tool:

```text
Manage Jenkins
    └── Tools
          └── SonarQube Scanner Installations
```

Configured Scanner:

```text
Name: sonar-scanner
```

<details>
<summary>📸 <strong>Click to view Screenshot - SonarQube Configuration</strong></summary>
<img width="1722" height="890" alt="image" src="https://github.com/user-attachments/assets/54dbf7a4-587d-4cc9-8593-54f7294988fc" />
<img width="1722" height="890" alt="image" src="https://github.com/user-attachments/assets/3eb850c1-5874-42c6-979d-7603f968a7c6" />
</details>

---

<a id="5-create-jenkins-pipeline"></a>

## 5. Create Jenkins Pipeline

Create pipeline file:

```text
Declarative_pipeline/
    └── React/
          └── Static_Code_analysis
```

Pipeline Stages:

```bash
nano Declarative_pipeline/React/Static_Code_analysis
```
The following Declarative Pipeline was implemented for Static Code Analysis.

```groovy
pipeline {

    agent any

    options {
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/Snaatak-Infra-Titans/Frontend.git'
            }
        }

        stage('Commit Sign-Off Validation') {
            steps {
                script {
                    def signoff = sh(
                        script: "git log -1 --pretty=%B | grep 'Signed-off-by:'",
                        returnStatus: true
                    )

                    if (signoff != 0) {
                        echo 'WARNING: Latest commit does not contain Signed-off-by.'
                    } else {
                        echo 'Commit Sign-Off validation passed.'
                    }
                }
            }
        }

        stage('Verify Node Environment') {
            steps {
                sh '''
                    node --version
                    npm --version
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Static Code Analysis') {
            steps {
                script {

                    def scannerHome = tool 'sonar-scanner'

                    withSonarQubeEnv('SonarQube') {

                        withCredentials([
                            string(
                                credentialsId: 'sonar-token',
                                variable: 'SONAR_TOKEN'
                            )
                        ]) {

                            sh """
                                ${scannerHome}/bin/sonar-scanner \
                                -Dsonar.projectKey=Frontend \
                                -Dsonar.projectName=Frontend \
                                -Dsonar.sources=src \
                                -Dsonar.sourceEncoding=UTF-8 \
                                -Dsonar.token=${SONAR_TOKEN}
                            """
                        }
                    }
                }
            }
        }
    }

    post {

        success {
            echo 'Static Code Analysis completed successfully.'
        }

        failure {
            echo 'Static Code Analysis failed. Check console logs.'
        }

        always {
            cleanWs()
        }
    }
}
```
Push Jenkinsfile:

```bash
git add .

git commit -s -m "SCRUM-250: Implement React static code analysis with SonarQube"

git push origin SCRUM-250-mukesh
```
---

<a id="6-configure-sonarqube-credentials"></a>

## 6. Configure SonarQube Credentials

Generate SonarQube Token:

```text
SonarQube
    └── My Account
          └── Security
                └── Generate Token
```

Add Jenkins Credential:

```text
Manage Jenkins
    └── Credentials
          └── System
                └── Global Credentials
```

Credential Configuration:

```text
Kind: Secret Text

ID: sonar-token

Secret: <Generated SonarQube Token>
```

<details>
<summary>📸 <strong>Click to view Screenshot - SonarQube Token Configuration</strong></summary>
<img width="1722" height="890" alt="image" src="https://github.com/user-attachments/assets/ca0b0c35-12aa-40e5-b476-7017ea6f6b2b" />
<img width="1722" height="890" alt="image" src="https://github.com/user-attachments/assets/9856ebea-70a9-41c3-9922-57218f340463" />

</details>

---

<a id="7-configure-jenkins-pipeline-job"></a>

## 7. Configure Jenkins Pipeline Job

Navigate to:

```text
CI Implementation
    └── Declarative-Pipeline
          └── React
                └── Static_Code_Analysis
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
*/SCRUM-250-mukesh
```

Script Path:

```text
Declarative_pipeline/React/Static_Code_analysis
```

Save configuration.

<details>
<summary>📸 <strong>Click to view Screenshot - Jenkins Job Configuration</strong></summary>
<img width="1831" height="883" alt="image" src="https://github.com/user-attachments/assets/8c3a4a9a-ba48-4092-b08e-19e8866536d7" />
<img width="1831" height="882" alt="image" src="https://github.com/user-attachments/assets/041f9e8e-45b9-4589-8c1e-1bbf98133da1" />

</details>

---

<a id="8-execute-pipeline"></a>

## 8. Execute Pipeline

Run the Jenkins Job:

```text
Build Now
```

Pipeline Execution Flow:

1. Checkout Frontend Repository
2. Validate Commit Sign-Off
3. Verify NodeJS Environment
4. Install Dependencies
5. Execute SonarQube Analysis
6. Publish Analysis Results

<details>
<summary>📸 <strong>Click to view Screenshot - Successful Pipeline Execution</strong></summary>

<img width="1820" height="882" alt="image" src="https://github.com/user-attachments/assets/4b8d0bd1-ffe7-4a75-84bc-387fd13c7bb1" />


</details>

---

<a id="9-review-sonarqube-report"></a>

## 9. Review SonarQube Report

Access SonarQube Dashboard and review generated metrics.

Observed Results:

| Metric            | Result |
| ----------------- | ------ |
| Quality Gate      | Passed |
| Bugs              | 0      |
| Vulnerabilities   | 0      |
| Security Hotspots | 0      |
| Reliability       | A      |
| Security          | A      |
| Maintainability   | A      |
| Code Smells       | 34     |
| Duplications      | 3.5%   |
| Coverage          | 0%     |

Coverage remains 0% because Unit Testing and Coverage Reporting are outside the scope of this Static Code Analysis implementation.

<details>
<summary>📸 <strong>Click to view Screenshot - SonarQube Dashboard</strong></summary>
<img width="1831" height="883" alt="image" src="https://github.com/user-attachments/assets/af07c842-9d5a-4604-9972-14ea3145ce1a" />

</details>

---

<a id="10-common-issues-and-troubleshooting"></a>

## 10. Common Issues and Troubleshooting

| Issue                           | Cause                | Resolution                       |
| ------------------------------- | -------------------- | -------------------------------- |
| SonarQube Authentication Failed | Missing Token        | Configure Sonar Token Credential |
| Scanner Not Found               | Scanner Tool Missing | Configure Sonar Scanner Tool     |
| Node Command Not Found          | NodeJS Not Installed | Install NodeJS                   |
| NPM Install Failure             | Dependency Issue     | Review npm logs                  |
| Quality Gate Failed             | Code Quality Issues  | Review SonarQube Findings        |
| Git Checkout Failed             | Invalid Credentials  | Verify GitHub Credentials        |

---

<a id="11-contact-information"></a>

## 11. Contact Information

| Name         | ✉️ Contact                                                                        |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

