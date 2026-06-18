# <h1 align="center">Jenkins Shared Library | Commit Sign-Off Validation</h1>

<div align="center">
<img width="100" alt="Jenkins" src="https://www.jenkins.io/images/logos/jenkins/jenkins.png" />
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
    <td align="center">18/06/2026</td>
    <td align="center">1.0</td>
    <td align="center">Mukesh Kharb</td>
    <td align="center">18/06/2026</td>
    <td align="center">Team</td>
    <td align="center">Mohit Kumar</td>
    <td align="center">Faisal Khan</td>
    <td align="center">Mahesh Kumar</td>
  </tr>

</table>

</div>

---

# Table of Contents

1. [Introduction](#1-introduction)
2. [Objective](#2-objective)
3. [Prerequisites](#3-prerequisites)
4. [Shared Library Structure](#4-shared-library-structure)
5. [Create Feature Branch](#5-create-feature-branch)
6. [Implement Commit Sign-Off Validation](#6-implement-commit-sign-off-validation)
7. [Commit Changes with Sign-Off](#7-commit-changes-with-sign-off)
8. [Push Changes to GitHub](#8-push-changes-to-github)
9. [Configure Jenkins Shared Library](#9-configure-jenkins-shared-library)
10. [Pipeline Usage Example](#10-pipeline-usage-example)
11. [Benefits](#11-benefits)
12. [Conclusion](#12-conclusion)
13. [Contact Information](#13-contact-information)

---

<a id="1-introduction"></a>

## 1. Introduction

Commit Sign-Off validation is a CI governance mechanism used to ensure that all code contributions include a valid Developer Certificate of Origin (DCO) sign-off.

To standardize this validation across OT-Microservices repositories, a reusable Jenkins Shared Library function has been implemented. This approach centralizes commit validation logic and eliminates duplication across multiple Jenkins pipelines.

The implementation verifies whether the latest Git commit contains a valid `Signed-off-by:` entry before allowing the pipeline to continue.

---

<a id="2-objective"></a>

## 2. Objective

The objectives of this implementation are:

* Enforce commit sign-off validation during CI execution.
* Ensure contribution traceability and ownership.
* Centralize validation logic using Jenkins Shared Libraries.
* Improve governance and compliance standards.
* Enable reusability across multiple repositories.
* Prevent unsigned commits from progressing through CI pipelines.

---

<a id="3-prerequisites"></a>

## 3. Prerequisites

| Component                      | Version                   |
| ------------------------------ | ------------------------- |
| Jenkins                        | 2.x or later              |
| Git                            | Latest                    |
| GitHub Repository              | Shared Library Repository |
| Jenkins Shared Library Support | Enabled                   |
| Pipeline Job                   | Configured                |

---

<a id="4-shared-library-structure"></a>

## 4. Shared Library Structure

Clone the repository:

```bash
git clone https://github.com/Snaatak-Infra-Titans/Shared_Library.git
cd Shared_Library
```
Create the shared library function:

```bash
mkdir -p vars

touch vars/commitSignOffCheck.groovy
```
Repository Structure:

```text
Shared_Library/
├── vars/
│   └── commitSignOffCheck.groovy
└── README.md
```

The `vars` directory stores reusable pipeline functions that can be directly invoked from Jenkins pipelines.

<details>
<summary>📸 <strong>Click to view Screenshot - Shared Library Repository Structure</strong></summary>

<img width="1531" height="237" alt="image" src="https://github.com/user-attachments/assets/76bb0c32-783d-4679-a831-7129ec076ce2" />
<img width="1531" height="237" alt="image" src="https://github.com/user-attachments/assets/88c8ae49-9ff8-4872-b31d-fd5ac692d8c7" />


</details>

---

<a id="5-create-feature-branch"></a>

## 5. Create Feature Branch

Create Jira feature branch:

```bash
git checkout -b SCRUM-230-mukesh
```

Verify current branch:

```bash
git branch
```

Expected Output:

```text
* SCRUM-230-mukesh
main
```

<details>
<summary>📸 <strong>Click to view Screenshot - Create Feature Branch</strong></summary>
<img width="1515" height="146" alt="image" src="https://github.com/user-attachments/assets/6dc41675-fc5f-4eb0-9382-03df4aef0fc3" />

</details>

---

<a id="6-implement-commit-sign-off-validation"></a>

## 6. Implement Commit Sign-Off Validation

Implementation:

```groovy
def call() {

    echo "Checking Commit Sign-Off"

    def commitMessage = sh(
        script: "git log -1 --pretty=%B",
        returnStdout: true
    ).trim()

    if (!commitMessage.contains("Signed-off-by:")) {
        error("Commit Sign-Off Missing")
    }

    echo "Commit Sign-Off Validation Passed"
}
```

The function performs the following actions:

* Retrieves the latest Git commit message.
* Searches for the `Signed-off-by:` entry.
* Fails the Jenkins build if the sign-off is missing.
* Allows pipeline execution when validation succeeds.

<details>
<summary>📸 <strong>Click to view Screenshot - Commit Sign-Off Shared Library Function</strong></summary>
<img width="1523" height="498" alt="image" src="https://github.com/user-attachments/assets/fa0e9447-c229-4d79-937c-bda8cc4d1cb0" />

Add Screenshot Here

</details>

---

<a id="7-commit-changes-with-sign-off"></a>

## 7. Commit Changes with Sign-Off

Stage the implementation:

```bash
git add .
```

Create a signed commit:

```bash
git commit -s -m "SCRUM-230 Add commit signoff shared library function"
```

Verify the commit:

```bash
git log -1
```

Expected Output:

```text
SCRUM-230 Add commit signoff shared library function

Signed-off-by: Mukesh Kharb <mukesh.Kharb.snaatak@mygurukulam.co>
```

<details>
<summary>📸 <strong>Click to view Screenshot - Signed Commit Verification</strong></summary>
<img width="1508" height="315" alt="image" src="https://github.com/user-attachments/assets/f6d20a35-fb29-4580-9de5-ced70fe3d9d2" />
</details>

---

<a id="8-push-changes-to-github"></a>

## 8. Push Changes to GitHub

Push the feature branch:

```bash
git push -u origin SCRUM-230-mukesh
```

Verify the branch is visible in GitHub.

Create a Pull Request:

```text
Source Branch:
SCRUM-230-mukesh

Target Branch:
main
```

Pull Request Title:

```text
SCRUM-230 Add commit signoff shared library function
```

<details>
<summary>📸 <strong>Click to view Screenshot - GitHub Pull Request</strong></summary>

<img width="1512" height="438" alt="image" src="https://github.com/user-attachments/assets/0955abda-2cb4-4ea1-bff5-2d3a43e43146" />


</details>

---

<a id="9-configure-jenkins-shared-library"></a>

<a id="9-configure-jenkins-shared-library"></a>

## 9. Configure Jenkins Shared Library

After implementing the shared library function, Jenkins must be configured to load the Shared Library repository.

Navigate to:

```text
Manage Jenkins
→ System
→ Global Trusted Pipeline Libraries
```

Click **Add** and provide the following details:

| Field                                          | Value          |
| ---------------------------------------------- | -------------- |
| Name                                           | Shared_Library |
| Default Version                                | main           |
| Allow default version to be overridden         | Enabled        |
| Include @Library changes in job recent changes | Enabled        |
| Retrieval Method                               | Modern SCM     |

Select:

```text
Modern SCM
```

Git Configuration:

| Field                  | Value                                                      |
| ---------------------- | ---------------------------------------------------------- |
| Source Code Management | Git                                                        |
| Project Repository     | https://github.com/Snaatak-Infra-Titans/Shared_Library.git |
| Credentials            | github-creds                                               |

Click:

```text
Save
```

Once configured, Jenkins automatically loads all reusable functions available under:

```text
vars/
```

Example:

```text
vars/
└── commitSignOffCheck.groovy
```

becomes available as:

```groovy
commitSignOffCheck()
```

inside Jenkins Pipelines.

<details>
<summary>📸 <strong>Click to view Screenshot - Jenkins Shared Library Configuration</strong></summary>

<img width="1739" height="813" alt="image" src="https://github.com/user-attachments/assets/63fc4045-cf84-4da8-b8ab-dd83516e40dc" />

</details>

---

<a id="10-pipeline-usage-example"></a>

## 10. Pipeline Usage Example

After configuring the Shared Library, import it inside the Jenkins Pipeline.

```groovy
@Library('Shared_Library@SCRUM-230-mukesh') _
```

Complete Pipeline Example:

```groovy
@Library('Shared_Library@SCRUM-230-mukesh') _

pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'SCRUM-230-mukesh',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/Snaatak-Infra-Titans/Shared_Library.git'
            }
        }

        stage('Shared Library Validation') {
            steps {
                commitSignOffCheck()
            }
        }
    }
}
```

### Successful Validation

Create a signed commit:

```bash
git commit -s -m "SCRUM-230 Add commit signoff validation"
```

Pipeline Output:

```text
Checking Commit Sign-Off

Commit Sign-Off Validation Passed
```

Result:

```text
SUCCESS
```

---

### Failed Validation

Create a commit without sign-off:

```bash
git commit -m "SCRUM-230 Add commit signoff validation"
```

Pipeline Output:

```text
Checking Commit Sign-Off

Commit Sign-Off Missing
```

Result:

```text
FAILED
```

The pipeline immediately stops execution and prevents unsigned commits from progressing further in the CI workflow.

<details>
<summary>📸 <strong>Click to view Screenshot - Jenkins Pipeline Execution</strong></summary>

<img width="1813" height="879" alt="image" src="https://github.com/user-attachments/assets/1ed1bf6f-510b-4c26-8222-e7d8cf69abbe" />

</details>

---

<a id="11-benefits"></a>

## 11. Benefits

This implementation provides the following benefits:

* Centralized commit validation logic.
* Reusable Jenkins Shared Library function.
* Consistent governance across repositories.
* Improved compliance with DCO requirements.
* Reduced pipeline code duplication.
* Easier maintenance and future enhancements.
* Standardized CI validation process.

---

<a id="12-conclusion"></a>

## 12. Conclusion

The Commit Sign-Off Validation Shared Library implementation provides a reusable and centralized mechanism for enforcing signed commits within Jenkins CI pipelines.

By implementing this functionality as a Shared Library, the validation can be reused across multiple repositories without duplicating code in individual Jenkinsfiles. This improves maintainability, governance, compliance, and consistency across OT-Microservices projects.

---

<a id="13-contact-information"></a>

## 13. Contact Information

| Name         | ✉️ Contact                                                                        |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---
