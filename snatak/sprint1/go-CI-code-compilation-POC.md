<h1 align="center">POC - GO Lang CI | Code Compilation </h1>

<div align="center">
<img width="120" alt="GoLang" src="https://go.dev/blog/go-brand/Go-Logo/PNG/Go-Logo_Blue.png" />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="80" alt="Jenkins" src="https://www.jenkins.io/images/logos/jenkins/jenkins.png" />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="70" alt="GitHub" src="https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png" />
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
8. [Verify Build Status](#8-verify-build-status)
9. [FAQs](#9-faqs)
10. [Contact Information](#10-contact-information)
11. [References](#11-references)

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
| OpenJDK | 21 | Jenkins runtime dependency |

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

<details>
<summary>📸 <strong>Click to view Screenshot - Jenkins Pipeline Configuration</strong></summary>

<div align="center">
 <img width="1000" height="auto" alt="image" src="https://github.com/user-attachments/assets/c217b9aa-98f5-4c5f-bfc7-4b4dd5ea6d68" />

</div>

</details>

---

<a id="7-jenkinsfile"></a>

## 7. Jenkinsfile

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

<a id="8-verify-build-status"></a>

## 8. Verify Build Status

Successful build output:

<details>
<summary>📸 <strong>Click to view Screenshot - Successful Jenkins Build</strong></summary>

<div align="center">
  <img width="1000" height="auto" alt="Screenshot from 2026-05-15 00-32-49" src="https://github.com/user-attachments/assets/8fa4e135-99c9-456a-a211-926dff5c148a" />

</div>

</details>

<details>
<summary>📸 <strong>Click to view Screenshot - Build Status</strong></summary>

<div align="center">
  <img width="1000" height="auto" alt="image" src="https://github.com/user-attachments/assets/c2b86bd4-5da1-46bc-a676-4853b07fcc9e" />

</div>

</details>

---

<a id="9-faqs"></a>

## 9. FAQs

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

<a id="10-contact-information"></a>

## 10. Contact Information

| Name         | ✉️ Contact                                                                        |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

<a id="11-references"></a>

## 11. References

| S.No | Description               | Click to View                                                                                                                                                                    |
| ---- | ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | GO Official Documentation | [![GO Docs](https://img.shields.io/badge/GO-DOCUMENTATION-2F2F2F?style=flat-square\&logo=go\&logoColor=white)](https://go.dev/doc/)                                              |
| 2    | GO Modules Documentation  | [![GO Modules](https://img.shields.io/badge/GO-MODULES-3A3A3A?style=flat-square\&logo=go\&logoColor=white)](https://go.dev/ref/mod)                                              |
| 3    | Jenkins Documentation     | [![Jenkins](https://img.shields.io/badge/JENKINS-DOCUMENTATION-2B2B2B?style=flat-square\&logo=jenkins\&logoColor=white)](https://www.jenkins.io/doc/)                            |
| 4    | GitHub Documentation      | [![GitHub](https://img.shields.io/badge/GITHUB-DOCUMENTATION-1F1F1F?style=flat-square\&logo=github\&logoColor=white)](https://docs.github.com/)                                  |
| 5    | Employee API Repository   | [![Employee API](https://img.shields.io/badge/EMPLOYEE_API-GO_SERVICE-404040?style=flat-square\&logo=github\&logoColor=white)](https://github.com/OT-MICROSERVICES/employee-api) |
