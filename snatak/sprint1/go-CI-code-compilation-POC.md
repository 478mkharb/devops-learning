<h1 align="center">POC - GO Lang CI | Code Compilation </h1>

<div align="center">
<img width="120" alt="GoLang" src="https://go.dev/blog/go-brand/Go-Logo/PNG/Go-Logo_Blue.png" />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="120" alt="Jenkins" src="https://www.jenkins.io/images/logos/jenkins/jenkins.png" />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="120" alt="GitHub" src="https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png" />
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
    <td align="center">14/05/2026</td>
    <td align="center">1.0</td>
    <td align="center">Mukesh Kharb</td>
    <td align="center">14/05/2026</td>
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
3. [Install Required Packages](#3-install-required-packages)
4. [Install GO Language](#4-install-go-language)
5. [Configure GO Environment](#5-configure-go-environment)
6. [Configure Jenkins Pipeline](#6-configure-jenkins-pipeline)
7. [Jenkinsfile](#7-jenkinsfile)
8. [Pipeline Execution](#8-pipeline-execution)
9. [Verify Build Status](#9-verify-build-status)
10. [FAQs](#10-faqs)
11. [Contact Information](#11-contact-information)
12. [References](#12-references)

---

<a id="1-introduction"></a>

## 1. Introduction

This POC demonstrates GO Lang CI code compilation using Jenkins and GitHub for the Employee API service in OT-Microservices.

The pipeline validates:

* GO dependency installation
* GO application compilation
* Jenkins CI execution
* Build success or failure status

---
<a id="2-pre-requisites"></a>

## 2. Pre-Requisites

| Component | Version | Purpose |
|---|---|---|
| Ubuntu Server | 22.04 LTS | Jenkins CI Server |
| GO Language | 1.22.3 | GO application compilation |
| Jenkins | 2.555.2 | CI pipeline execution |
| OpenJDK | 21 | Jenkins runtime dependency |
| Git | 2.34.1 | Source code management |
| GitHub Repository | Latest | Employee API source repository |

---
<a id="3-install-required-packages"></a>

## 3. Install Required Packages

```bash
sudo apt update &&\
sudo apt install -y curl wget git nano net-tools build-essential
```

<details>
<summary>📸 <strong>Click to view Screenshot - Required Packages Installation</strong></summary>

<div align="center">
<img src="https://github.com/user-attachments/assets/02bd2fb6-074c-4133-ae13-8a4cdac75378"width="1000" />
</div>

</details>

---

<a id="4-install-go-language"></a>

## 4. Install GO Language

Install and verify GO language:

```bash
sudo apt install -y golang-go && go version
```

<details>
<summary>📸 <strong>Click to view Screenshot - GO Version Verification</strong></summary>

<div align="center">
  <img width="1000" height="auto" alt="image" src="https://github.com/user-attachments/assets/7ddd9a92-361b-4c16-afa0-bf451633ec43" />

</div>

</details>

---

<a id="5-configure-go-environment"></a>

## 5. Configure GO Environment

Configure GO environment variables:

```bash
echo 'export GOPATH=$HOME/go' >> ~/.bashrc && \
echo 'export PATH=$PATH:$GOPATH/bin' >> ~/.bashrc && \
source ~/.bashrc
```

Verify GO environment:

```bash
echo $GOPATH && which go
```
<details>
<summary>📸 <strong>Click to view Screenshot - Configure GO Environment</strong></summary>

<div align="center">
<img width="1000" height="auto" alt="image" src="https://github.com/user-attachments/assets/3024b47e-98e6-4281-a1dd-60f32a86e02d" />
</div>

</details>

---

<a id="6-configure-jenkins-pipeline"></a>

## 6. Configure Jenkins Pipeline

* Login to Jenkins Dashboard
* Click `New Item`
* Enter pipeline name
* Select `Pipeline`
* Click `OK`

Under Pipeline section:

* Select `Pipeline script from SCM`
* SCM → `Git`
* Enter Employee API repository URL
* Select repository branch
* Save pipeline configuration

<details>
<summary>📸 <strong>Click to view Screenshot - Jenkins Pipeline Configuration</strong></summary>

<div align="center">
  <img src="https://github.com/user-attachments/assets/sample-jenkins-pipeline" width="700"/>
</div>

</details>

---

<a id="7-jenkinsfile"></a>

## 7. Jenkinsfile

Create `Jenkinsfile` inside repository root:

```groovy
pipeline {
    agent any

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/OT-MICROSERVICES/employee-api.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'go mod tidy'
            }
        }

        stage('GO Compilation Check') {
            steps {
                sh 'go build -o employee-api'
            }
        }

    }

    post {

        success {
            echo 'GO Build Successful'
        }

        failure {
            echo 'GO Build Failed'
        }

    }
}
```

---

<a id="8-pipeline-execution"></a>

## 8. Pipeline Execution

Trigger Jenkins pipeline:

* Open Jenkins Pipeline
* Click `Build Now`
* Monitor pipeline console logs

Pipeline workflow:

```text
Developer Push
       │
       ▼
GitHub Repository
       │
       ▼
Jenkins Pipeline Triggered
       │
       ▼
Checkout Source Code
       │
       ▼
Install GO Dependencies
(go mod tidy)
       │
       ▼
GO Compilation Check
(go build)
       │
       ▼
Compilation Success / Failure
       │
       ▼
Jenkins Build Status
```

<details>
<summary>📸 <strong>Click to view Screenshot - Jenkins Pipeline Execution</strong></summary>

<div align="center">
  <img src="https://github.com/user-attachments/assets/sample-pipeline-execution" width="700"/>
</div>

</details>

---

<a id="9-verify-build-status"></a>

## 9. Verify Build Status

Successful build output:

<details>
<summary>📸 <strong>Click to view Screenshot - Successful Jenkins Build</strong></summary>

<div align="center">
  <img src="https://github.com/user-attachments/assets/sample-success-build" width="700"/>
</div>

</details>

Failed build output:

<details>
<summary>📸 <strong>Click to view Screenshot - Failed Jenkins Build</strong></summary>

<div align="center">
  <img src="https://github.com/user-attachments/assets/sample-failed-build" width="700"/>
</div>

</details>

Verify generated binary:

```bash
ls -l employee-api
```

<details>
<summary>📸 <strong>Click to view Screenshot - Build Status</strong></summary>

<div align="center">
  <img src="https://github.com/user-attachments/assets/sample-build-status" width="700"/>
</div>

</details>

---

<a id="10-faqs"></a>

## 10. FAQs

### Q1. Why is GO language required?

**Answer:**
GO language is required because the Employee API microservice is developed using Golang.

---

### Q2. What does `go mod tidy` do?

**Answer:**
It downloads and validates all dependencies defined in the `go.mod` file.

---

### Q3. What does `go build` do?

**Answer:**
It compiles the GO source code and generates executable binary.

---

### Q4. Why is Jenkins used in this POC?

**Answer:**
Jenkins automates GO code compilation checks whenever developers push code to GitHub.

---

### Q5. How do we verify successful compilation?

**Answer:**
Successful Jenkins pipeline execution and generated binary confirm successful compilation.

---

### Q6. What happens if compilation fails?

**Answer:**
Jenkins marks the pipeline as FAILED and stops further execution.

---

<a id="11-contact-information"></a>

## 11. Contact Information

| Name         | ✉️ Contact                                                                        |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

<a id="12-references"></a>

## 12. References

| S.No | Description               | Click to View                                                                                                                                                                    |
| ---- | ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | GO Official Documentation | [![GO Docs](https://img.shields.io/badge/GO-DOCUMENTATION-2F2F2F?style=flat-square\&logo=go\&logoColor=white)](https://go.dev/doc/)                                              |
| 2    | GO Modules Documentation  | [![GO Modules](https://img.shields.io/badge/GO-MODULES-3A3A3A?style=flat-square\&logo=go\&logoColor=white)](https://go.dev/ref/mod)                                              |
| 3    | Jenkins Documentation     | [![Jenkins](https://img.shields.io/badge/JENKINS-DOCUMENTATION-2B2B2B?style=flat-square\&logo=jenkins\&logoColor=white)](https://www.jenkins.io/doc/)                            |
| 4    | GitHub Documentation      | [![GitHub](https://img.shields.io/badge/GITHUB-DOCUMENTATION-1F1F1F?style=flat-square\&logo=github\&logoColor=white)](https://docs.github.com/)                                  |
| 5    | Employee API Repository   | [![Employee API](https://img.shields.io/badge/EMPLOYEE_API-GO_SERVICE-404040?style=flat-square\&logo=github\&logoColor=white)](https://github.com/OT-MICROSERVICES/employee-api) |
