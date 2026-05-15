# <h1 align="center">POC - Java CI | Dependency Scanning </h1>

<div align="center">
<img width="140" alt="Java" src="https://www.vectorlogo.zone/logos/java/java-ar21.svg" />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="80" alt="Jenkins" src="https://www.jenkins.io/images/logos/jenkins/jenkins.png" />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="200" alt="OWASP Dependency Check" src="https://owasp.org/www-project-dependency-check/assets/images/logo.png" />
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
    <td align="center">15/05/2026</td>
    <td align="center">1.0</td>
    <td align="center">Mukesh Kharb</td>
    <td align="center">15/05/2026</td>
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
2. [Clone Java Repository](#2-clone-java-repository)
3. [Install Required Packages](#3-install-required-packages)
4. [Verify Java and Maven](#4-verify-java-and-maven)
5. [Build Java Application](#5-build-java-application)
6. [Configure OWASP Dependency Check](#6-configure-owasp-dependency-check)
7. [Run Dependency Scanning](#7-run-dependency-scanning)
8. [Generate HTML Report](#8-generate-html-report)
9. [Jenkins Pipeline Integration](#9-jenkins-pipeline-integration)
10. [Dependency Scanning Findings](#10-dependency-scanning-findings)
11. [Contact Information](#11-contact-information)
12. [References](#12-references)

---

## 1. Introduction

This POC demonstrates Dependency Scanning for Java applications in CI pipelines using OWASP Dependency Check.

The scan validates:

* Vulnerable third-party dependencies
* Known CVEs in Maven packages
* Dependency version risks
* Software supply chain security
* Build policy enforcement in CI pipelines

The POC integrates Java, Maven, Jenkins, and OWASP Dependency Check to automate dependency vulnerability analysis during CI execution.

---

## 2. Jenkinsfile for Java Dependency Scanning

Create the following `Jenkinsfile` inside the Java application repository:

```groovy
pipeline {
    agent any

    tools {
        jdk 'JDK17'
        maven 'Maven3'
    }

    environment {
        MAVEN_OPTS = '-Xmx1024m'
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                git branch: 'main', url: 'https://github.com/OT-MICROSERVICES/salary-api.git'
            }
        }

        stage('Verify Java and Maven') {
            steps {
                sh 'java -version'
                sh 'mvn -version'
            }
        }

        stage('Resolve Dependencies') {
            steps {
                sh 'mvn dependency:tree'
            }
        }

        stage('Build Application') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Dependency Scanning') {
            steps {
                sh '''
                mvn org.owasp:dependency-check-maven:check \
                -Dformat=HTML \
                -DfailBuildOnCVSS=7
                '''
            }
        }

        stage('Publish Dependency Report') {
            steps {
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'target',
                    reportFiles: 'dependency-check-report.html',
                    reportName: 'OWASP Dependency Check Report'
                ])
            }
        }
    }

    post {

        success {
            echo 'Dependency scanning completed successfully.'
        }

        failure {
            echo 'Critical vulnerabilities detected or build failed.'
        }

        always {
            archiveArtifacts artifacts: 'target/dependency-check-report.html', fingerprint: true
        }
    }
}
```

---

## 3. Configure OWASP Dependency Check Plugin

Add the OWASP Dependency Check plugin inside `pom.xml`:

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.owasp</groupId>
      <artifactId>dependency-check-maven</artifactId>
      <version>10.0.4</version>
      <configuration>
        <format>HTML</format>
        <failBuildOnCVSS>7</failBuildOnCVSS>
      </configuration>
      <executions>
        <execution>
          <goals>
            <goal>check</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

This configuration enables:

* Automated CVE scanning
* NVD vulnerability database integration
* HTML report generation
* Build failure on high-severity vulnerabilities
* CI-integrated dependency analysis

<details>
<summary>📸 <strong>Click to view Screenshot - pom.xml Configuration</strong></summary>
<img width="1830" height="880" alt="image" src="https://github.com/user-attachments/assets/pom-owasp-config.png" />
</details>

---

## 4. Jenkins Pipeline Workflow

```text
Developer Push
       │
       ▼
GitHub Repository
       │
       ▼
Jenkins Pipeline Trigger
       │
       ▼
Checkout Source Code
       │
       ▼
Maven Dependency Resolution
       │
       ▼
Application Build
       │
       ▼
OWASP Dependency Check Scan
       │
 ┌───────────────┐
 │ CVE Database  │
 │ NVD Database  │
 │ GH Advisories │
 └───────────────┘
       │
       ▼
Vulnerability Analysis
       │
       ▼
HTML Report Generation
       │
       ▼
Publish Report in Jenkins
       │
       ▼
Build Pass / Fail Decision
```

<details>
<summary>📸 <strong>Click to view Screenshot - Jenkins Dependency Scanning Pipeline</strong></summary>
<img width="1838" height="887" alt="image" src="https://github.com/user-attachments/assets/jenkins-dependency-pipeline.png" />
</details>

---

## 5. Generated Dependency Scan Report

Generated report:

```text
target/dependency-check-report.html
```

The report includes:

* Vulnerable dependencies
* CVE identifiers
* CVSS severity scores
* Dependency paths
* Recommended package versions
* Risk classification

Example build result:

```text
BUILD SUCCESS
```

If high-severity vulnerabilities are detected:

```text
BUILD FAILURE
```

<details>
<summary>📸 <strong>Click to view Screenshot - Dependency Check Report</strong></summary>
<img width="1836" height="882" alt="image" src="https://github.com/user-attachments/assets/dependency-check-report.png" />
</details>

---

## 10. Dependency Scanning Findings

| Finding                      | Risk Level  | Description                                |
| ---------------------------- | ----------- | ------------------------------------------ |
| Vulnerable Spring Dependency | 🔴 High     | Detected vulnerable Spring library version |
| Outdated Log4j Package       | 🔴 Critical | Known RCE vulnerability present            |
| Weak Transitive Dependency   | 🟠 Medium   | Vulnerable indirect package dependency     |
| Deprecated Package Usage     | 🟡 Low      | Unsupported dependency version detected    |
| Missing Dependency Updates   | 🟠 Medium   | Recommended package upgrades available     |

Example CVE Findings:

| CVE ID         | Severity | Package          |
| -------------- | -------- | ---------------- |
| CVE-2021-44228 | Critical | log4j-core       |
| CVE-2022-22965 | High     | spring-framework |
| CVE-2023-20873 | Medium   | spring-security  |

---

## 11. Contact Information

| Name         | Email                                                                             |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

## 12. References

| S.No | Resource                             | Link                                                                                               |
| ---- | ------------------------------------ | -------------------------------------------------------------------------------------------------- |
| 1    | OWASP Dependency Check Documentation | [https://owasp.org/www-project-dependency-check/](https://owasp.org/www-project-dependency-check/) |
| 2    | Maven Documentation                  | [https://maven.apache.org/guides/](https://maven.apache.org/guides/)                               |
| 3    | Jenkins Documentation                | [https://www.jenkins.io/doc/](https://www.jenkins.io/doc/)                                         |
| 4    | NVD Vulnerability Database           | [https://nvd.nist.gov/](https://nvd.nist.gov/)                                                     |
| 5    | GitHub Security Advisories           | [https://github.com/advisories](https://github.com/advisories)                                     |

---

## Conclusion

This POC demonstrates how Java CI Dependency Scanning can be integrated into Jenkins pipelines using OWASP Dependency Check to identify vulnerable dependencies before deployment. Automated dependency analysis improves software supply chain security, reduces exposure to known CVEs, and enables proactive remediation during the CI lifecycle.
