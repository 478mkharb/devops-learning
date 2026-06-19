# <h1 align="center">Python Declarative Pipeline for Unit Testing</h1>

<div align="center">

<img width="80" src="https://www.jenkins.io/images/logos/jenkins/jenkins.svg" />

<img width="180" src="https://www.python.org/static/community_logos/python-logo-master-v3-TM.png" />

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
4. [Repository Analysis](#4-repository-analysis)
5. [Jenkins Pipeline Implementation](#5-jenkins-pipeline-implementation)
6. [Jenkinsfile Structure](#6-jenkinsfile-structure)
7. [Jenkins Job Configuration](#7-jenkins-job-configuration)
8. [Pipeline Execution](#8-pipeline-execution)
9. [Artifacts Generated](#9-artifacts-generated)
10. [Challenges Encountered](#10-challenges-encountered)
11. [Common Issues and Troubleshooting](#11-common-issues-and-troubleshooting)
12. [Outcome](#12-outcome)
13. [Contact Information](#13-contact-information)
    
---

<a id="1-introduction"></a>
## 1. Introduction

This POC demonstrates implementation of a Jenkins Declarative Pipeline for Unit Testing of Python microservices.

The implementation covers:

- Commit Sign-Off Validation
- Parallel Repository Checkout
- Parallel Unit Test Execution
- JUnit XML Report Publishing
- Artifact Archiving
- Jenkins Test Dashboard Integration
- Build Status Management
- Error Handling using UNSTABLE State

Repositories used:

- Attendance_API
- Notification

The pipeline is designed to continue execution even when tests fail and generate reports for review.

---

<a id="2-pre-requisites"></a>
## 2. Pre-requisites

| Component | Version |
|------------|------------|
| Ubuntu Server | 24.04 LTS |
| Jenkins | 2.528+ |
| Python | 3.12 |
| Poetry | 1.8.2 |
| Pytest | Installed |
| Git | Installed |
| GitHub Credentials | Configured |

Install required packages on Jenkins Node:

```bash
sudo apt update
sudo apt install python3-dev -y
sudo apt install libpq-dev -y
sudo apt install gcc -y
```

Verify installation:

```bash
python3 --version
poetry --version
pytest --version
```

---

<a id="3-branch-creation"></a>
## 3. Branch Creation

```bash
cd Jenkins

git checkout main

git pull origin main

git checkout -b SCRUM-270-mukesh
```


<details>
<summary>📸 <strong>Click to view Screenshot - Branch Creation</strong></summary>

<img width="1500" height="222" alt="image" src="https://github.com/user-attachments/assets/58057bbf-3e0f-4053-ad16-23d2f618df6a" />

</details>

---

<a id="4-repository-analysis"></a>
## 4. Repository Analysis

### Attendance_API

Repository contains:

- Poetry based dependency management
- Pytest unit tests
- PostgreSQL dependency
- Redis dependency

Observed test files:

- client/tests/test_postgres_conn.py
- client/tests/test_redis_conn.py
- router/tests/*
- models/tests/*
- utils/tests/*

### Notification

Repository contains:

- Python notification service
- No unit test files available

Pipeline handles this condition gracefully.

---

<a id="5-jenkins-pipeline-implementation"></a>

## 5. Jenkins Pipeline Implementation

Pipeline Location:

```text
Declarative_Pipeline/
└── Python/
    └── Unit_Testing
```

Implemented stages:

1. Verify Tools
2. Checkout Repositories
3. Commit Sign-Off Validation
4. Attendance_API Unit Tests
5. Notification Unit Tests
6. Report Publishing
7. Artifact Archiving

---

<a id="6-jenkinsfile-structure"></a>
## 6. Jenkinsfile Structure

Major Features:

## Verify Tools

```bash
python3 --version
poetry --version
pytest --version
```

### Parallel Repository Checkout

Repositories:

- Attendance_API
- Notification

### Commit Sign-Off Validation

Checks latest commit for:

```text
Signed-off-by:
```

Generated Artifact:

```text
commit-signoff-report.xml
```

### Attendance_API Unit Testing

Steps:

- Configure Poetry
- Install dependencies
- Configure PYTHONPATH
- Execute pytest
- Generate JUnit XML

Command:

```bash
poetry run pytest --junitxml=attendance-test-report.xml
```

Generated Report:

```text
attendance-test-report.xml
```

### Notification Unit Testing

Checks whether tests exist:

```text
test_*.py
*_test.py
```
Generated Report:

```text
notification-test-report.xml
```

---

<a id="7-jenkins-job-configuration"></a>
## 7. Jenkins Job Configuration

Navigate:

```text
CI Implementation
    └── Declarative Pipeline
          └── Python
                └── Unit Testing
```

Configuration:

```text
Definition:
Pipeline script from SCM
```

Repository:

```text
https://github.com/Snaatak-Infra-Titans/Jenkins.git
```

Branch:

```text
*/SCRUM-270-mukesh
```

Script Path:

```text
Declarative_Pipeline/Python/Unit_Testing
```
<details>
<summary>📸 <strong>Click to view Screenshot - Jenkins Job Configuration</strong></summary>

<img width="1797" height="823" alt="image" src="https://github.com/user-attachments/assets/15cd6422-6c08-4b36-978e-b53d26ea342a" />


</details>

---

<a id="8-pipeline-execution"></a>
## 8. Pipeline Execution

Build Flow:

1. Verify Tools
2. Checkout Attendance_API
3. Checkout Notification
4. Validate Commit Sign-Off
5. Execute Unit Tests
6. Publish Reports
7. Archive Artifacts

Expected Results:

| Stage | Result |
|---------|---------|
| Verify Tools | Success |
| Checkout | Success |
| Commit Sign-Off | Success |
| Attendance_API Tests | Unstable |
| Notification Tests | Success |
| Post Actions | Success |



Save configuration.
<details>
<summary>📸 <strong>Click to view Screenshot - Pipeline Execution</strong></summary>
<img width="1815" height="872" alt="image" src="https://github.com/user-attachments/assets/0b41348f-c38e-4e12-a71f-d598e9751277" />

</details>

---

<a id="9-artifacts-generated"></a>
## 9. Artifacts Generated

The following artifacts are generated and archived by Jenkins:

| Artifact | Description |
|-----------|-------------|
| commit-signoff-report.xml | Commit Sign-Off Validation Report |
| attendance-test-report.xml | Attendance_API Unit Test Report |
| notification-test-report.xml | Notification Unit Test Report |

Artifacts can be accessed from:

Build → Artifacts
<details>
<summary>📸 <strong>Click to view Screenshot - Commit Sign-Off Validation</strong></summary>
<img width="1436" height="603" alt="image" src="https://github.com/user-attachments/assets/847bcbc8-5d4e-47a4-80c6-5cf57cfa310c" />

</details>
<details>
<summary>📸 <strong>Click to view Screenshot - Attendance_API Unit Test Result</strong></summary>
<img width="1853" height="743" alt="image" src="https://github.com/user-attachments/assets/04088a97-2666-43f3-8b84-ba09bebc89f2" />

</details>
<details>
<summary>📸 <strong>Click to view Screenshot - Notification Unit Test</strong></summary>

<img width="1853" height="743" alt="image" src="https://github.com/user-attachments/assets/1ae846c0-c9d3-4a34-bcfd-e80beb3ae90c" />


</details>
<details>
<summary>📸 <strong>Click to view Screenshot - Final Build Result</strong></summary>
<img width="1831" height="983" alt="image" src="https://github.com/user-attachments/assets/5119738e-1639-4a9d-8bc5-aaff5bfeb7a7" />

</details>

---

<a id="10-challenges-encountered"></a>
## 10. Challenges Encountered

## Poetry Installation Issues

Issue:

Poetry attempted package installation in system Python environment.

Error:

```text
externally-managed-environment
```

Resolution:

Enabled Poetry virtual environments.

```bash
poetry config virtualenvs.create true
poetry config virtualenvs.in-project true
```

---

### PostgreSQL Build Dependencies

Issue:

```text
pg_config executable not found
```

Resolution:

```bash
sudo apt install libpq-dev
sudo apt install gcc
sudo apt install python3-dev
```

---

### PyYAML Compatibility

Issue:

PyYAML build failures with Python 3.12.

Resolution:

Updated lock file and regenerated Poetry dependencies.

```bash
poetry lock
```

---

### Attendance_API Test Failure

Issue:

Attendance_API attempts PostgreSQL connection during test collection.

Error:

```text
psycopg2.OperationalError
connection to server at "127.0.0.1", port 5432 failed
```

Impact:

Build becomes UNSTABLE.

Pipeline continues execution.

Reports are still published.

---

<a id="11-common-issues-and-troubleshooting"></a>
## 11. Common Issues and Troubleshooting

| Issue | Cause | Resolution |
|---------|---------|---------|
| Poetry Install Failure | Virtualenv Disabled | Enable Poetry Virtualenv |
| pg_config Missing | PostgreSQL Dev Package Missing | Install libpq-dev |
| JUnit Report Missing | Tests Not Executed | Verify pytest execution |
| Git Checkout Failure | Invalid Credentials | Verify github-creds |
| Notification Tests Missing | No Test Files | Generate Dummy XML |
| Attendance_API Failure | PostgreSQL Not Running | Review Application Dependency |

---

<a id="12-outcome"></a>
## 12. Outcome

The implementation successfully demonstrates:

- Jenkins Declarative Pipeline
- Commit Sign-Off Validation
- Python Unit Testing
- Parallel Stage Execution
- JUnit XML Report Publishing
- Artifact Archiving
- Build Stability using UNSTABLE Status
- Handling Repositories Without Test Cases

Current build behaviour:

```text
Attendance_API Tests -> UNSTABLE
Notification Tests -> SUCCESS
Overall Build -> UNSTABLE
```

This ensures test results remain visible while avoiding complete pipeline failure.

---

<a id="13-contact-information"></a>
## 13. Contact Information

| Name | Contact |
|--------|---------|
| Mukesh Kharb | mukesh.Kharb.snaatak@mygurukulam.co |

---
