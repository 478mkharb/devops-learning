# <h1 align="center">POC - Java CI | Dependency Scanning using Trivy</h1>

<div align="center">

<img width="140" alt="Java" src="https://www.vectorlogo.zone/logos/java/java-ar21.svg" />      <img width="140" alt="Trivy" src="https://trivy.dev/latest/assets/images/logo.png" />

</div>

<br/>

---

<div align="center">

| Author       | Created On | Version | Last Updated By | Last Edited On | Pre Reviewer | L0 Reviewer | L1 Reviewer | L2 Reviewer  |
| ------------ | ---------- | ------- | --------------- | -------------- | ------------ | ----------- | ----------- | ------------ |
| Mukesh Kharb | 09/06/2026 | 1.0     | Mukesh Kharb    | 09/06/2026     | Team         | Mohit Kumar | Faisal Khan | Mahesh Kumar |

</div>

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Pre-requisites](#2-pre-requisites)
3. [Clone Salary API Repository](#3-clone-salary-api-repository)
4. [Install and Configure JAVA 17 and Maven](#4-install-and-configure-java-17-and-maven)
5. [Install Trivy](#5-install-trivy)
6. [Build Java Application](#6-build-java-application)
7. [Run Dependency Scan using Trivy](#7-run-dependency-scan-using-trivy)
8. [Generate JSON and HTML Reports](#8-generate-json-and-html-reports)
9. [Dependency Scan Findings](#9-dependency-scan-findings)
10. [Fail Build Based on Vulnerability Severity](#10-fail-build-based-on-vulnerability-severity)
11. [Conclusion](#11-conclusion)
12. [FAQs](#12-faqs)
13. [Contact Information](#13-contact-information)
14. [References](#14-references)

---

## 1. Introduction

This POC demonstrates dependency vulnerability scanning for the Java-based `salary-api` application using Trivy.

Trivy is an open-source vulnerability scanner developed by Aqua Security that helps identify vulnerabilities in application dependencies, operating system packages, container images, Infrastructure as Code (IaC), and secrets.

In this POC, Trivy performs Software Composition Analysis (SCA) on Maven dependencies used by the `salary-api` application and generates vulnerability reports in both JSON and HTML formats.

---

## 2. Pre-requisites

Before performing dependency scanning using Trivy, ensure the following prerequisites are available:

| Requirement           | Version / Specification                     |
| --------------------- | ------------------------------------------- |
| Ubuntu/Linux VM       | Ubuntu 22.04 LTS                            |
| JAVA                  | OpenJDK 17                                  |
| Maven                 | 3.8+                                        |
| Trivy                 | Latest Stable Version                       |
| RAM                   | Minimum 2 GB                                |
| Disk Space            | Minimum 5 GB Free                           |
| Internet Connectivity | Required for vulnerability database updates |

---

## 3. Clone Salary API Repository

Clone the repository:

```bash
git clone https://github.com/OT-MICROSERVICES/salary-api.git

cd salary-api
```

<details>
<summary>📸 <strong>Click to view Screenshot - Salary API Repository</strong></summary>

<img width="1325" height="902" alt="image" src="https://github.com/user-attachments/assets/45289164-241b-4f43-9c9c-ec7ad3b8ff23" />

</details>

---

## 4. Install and Configure JAVA 17 and Maven

Install JAVA 17 and Maven:

```bash
sudo apt update

sudo apt install -y openjdk-17-jdk maven
```

Verify installation:

```bash
java -version

mvn -version
```

If another JAVA version is active:

```bash
sudo update-alternatives --config java

sudo update-alternatives --config javac
```

<details>
<summary>📸 <strong>Click to view Screenshot - JAVA 17 and Maven Installation</strong></summary>

<img width="1248" height="350" alt="image" src="https://github.com/user-attachments/assets/b01f9f1b-af4b-43e0-9234-d6d1b9d93ac8" />


</details>

---

## 5. Install Trivy

Install required packages:

```bash
sudo apt-get update

sudo apt-get install -y wget apt-transport-https gnupg lsb-release
```

Add Trivy GPG Key:

```bash
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key \
| sudo gpg --dearmor -o /usr/share/keyrings/trivy.gpg
```

Add Trivy Repository:

```bash
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] \
https://aquasecurity.github.io/trivy-repo/deb \
$(lsb_release -sc) main" \
| sudo tee /etc/apt/sources.list.d/trivy.list
```

Install Trivy:

```bash
sudo apt update

sudo apt install -y trivy
```

Verify installation:

```bash
trivy --version
```

Download vulnerability database:

```bash
trivy image --download-db-only
```

> [!IMPORTANT]
> During the first execution, Trivy downloads vulnerability databases from Aqua Security repositories. This process may take several minutes depending on internet speed.

<details>
<summary>📸 <strong>Click to view Screenshot - Trivy Installation</strong></summary>

<img width="1707" height="296" alt="image" src="https://github.com/user-attachments/assets/775f4c62-fc1a-4789-a0e3-d0512cbde395" />


</details>

---

## 6. Build Java Application

Build the application:

```bash
mvn clean install -DskipTests
```

> [!NOTE]
> Tests are skipped because the application may require external backend service connectivity during Spring Boot test execution.

Verify build success:

```bash
ls target/
```

<details>
<summary>📸 <strong>Click to view Screenshot - Maven Build Success</strong></summary>

<img width="1450" height="118" alt="image" src="https://github.com/user-attachments/assets/5224893b-2e9a-4c53-865b-80dfd51526b4" />

</details>

---

## 7. Run Dependency Scan using Trivy

Run filesystem vulnerability scan:

```bash
trivy fs .
```

Trivy automatically:

* Detects Maven dependencies from `pom.xml`
* Resolves dependency tree
* Downloads vulnerability metadata
* Maps dependencies against CVE databases
* Reports detected vulnerabilities

<details>
<summary>📸 <strong>Click to view Screenshot - Trivy Dependency Scan</strong></summary>

<img width="1769" height="748" alt="image" src="https://github.com/user-attachments/assets/90ed01ac-0e97-4c50-ba89-55dfa5e32acb" />

</details>

---

## 8. Generate JSON and HTML Reports

### Generate JSON Report

```bash
trivy fs \
--format json \
--output trivy-report.json \
.
```

Verify report:

```bash
ls -lh trivy-report.json
```

---

### Generate HTML Report

Download Trivy HTML Template:

```bash
curl -L \
https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl \
-o html.tpl
```

Generate HTML report:

```bash
trivy fs \
--format template \
--template "@html.tpl" \
--output trivy-report.html \
.
```

Verify report:

```bash
ls -lh trivy-report.html
```

Open report locally:

```bash
python3 -m http.server 4000
```

Browse:

```text
http://localhost:4000/trivy-report.html
```

<details>
<summary>📸 <strong>Click to view Screenshot - Trivy Reports</strong></summary>

<img width="1852" height="971" alt="image" src="https://github.com/user-attachments/assets/d8caeffc-829c-4fc2-9f2a-09d84f1d31eb" />


</details>

---

## 9. Dependency Scan Findings

The scan successfully identified multiple vulnerable dependencies and automatically failed the build because the configured policy blocks vulnerabilities having CVSS score greater than or equal to `7.0`.

Vulnerable dependencies identified during the scan:

| Dependency | Description | Highest Severity | CVE Count |
|---|---|---|---|
| commons-lang3-3.12.0.jar | Apache Commons utility library for Java | 🟡 Medium | 1 |
| jackson-databind-2.15.2.jar | JSON serialization and deserialization library | 🟡 Medium | 1 |
| logstash-logback-encoder-6.6.jar | Structured JSON logging encoder for Logback | 🟡 Medium | 1 |
| spring-boot-3.1.1.jar | Core Spring Boot framework dependency | 🟡 Medium | 1 |
| spring-boot-starter-web-3.1.1.jar | Spring Boot starter for REST web applications | 🟡 Medium | 1 |
| log4j-api-2.20.0.jar | Logging API for Java applications | 🟠 High | 5 |
| logback-core-1.4.8.jar | Core logging framework for Spring Boot | 🟠 High | 1 |
| micrometer-registry-prometheus-1.11.1.jar | Prometheus metrics exporter integration | 🟠 High | 1 |
| netty-transport-4.1.94.Final.jar | Asynchronous event-driven network framework | 🟠 High | 12 |
| simpleclient-0.16.0.jar | Prometheus Java metrics client library | 🟠 High | 1 |
| spring-core-6.0.10.jar | Core utilities and framework classes for Spring | 🟠 High | 3 |
| spring-data-cassandra-4.1.1.jar | Spring Data integration for Cassandra database | 🟠 High | 6 |
| spring-web-6.0.10.jar | Spring Web MVC and REST framework | 🟠 High | 3 |
| snakeyaml-1.33.jar | YAML parsing library for Java applications | 🔴 Critical | 1 |
| swagger-ui-4.17.1.jar | Interactive API documentation user interface | 🔴 Critical | 10 |
| tomcat-embed-core-10.1.10.jar | Embedded Apache Tomcat web server | 🔴 Critical | 39 |


> [!NOTE]
> Actual findings may vary depending on dependency versions and the vulnerability database version used during scanning.

---

## 10. Fail Build Based on Vulnerability Severity

Trivy can be configured to fail the build if HIGH or CRITICAL vulnerabilities are detected.

Execute:

```bash
trivy fs \
--severity HIGH,CRITICAL \
--exit-code 1 \
.
```

Behavior:

| Condition                              | Result        |
| -------------------------------------- | ------------- |
| No HIGH/CRITICAL vulnerabilities       | Exit Code = 0 |
| HIGH/CRITICAL vulnerabilities detected | Exit Code = 1 |

This feature is useful for Jenkins CI pipelines where deployments should be blocked when severe vulnerabilities exist.

Example Jenkins stage:

```groovy
stage('Dependency Scan') {
    steps {
        sh '''
        trivy fs \
        --severity HIGH,CRITICAL \
        --exit-code 1 \
        .
        '''
    }
}
```

---

## 11. Conclusion

This POC demonstrates dependency vulnerability scanning for Java applications using Trivy.

The implementation successfully validates Maven dependencies against known vulnerabilities and generates both machine-readable JSON reports and human-readable HTML reports.

Trivy provides a lightweight and efficient Software Composition Analysis (SCA) solution that can be integrated into Jenkins CI pipelines to automate dependency security validation before deployment.

The generated reports help identify vulnerable third-party components and support proactive vulnerability remediation throughout the software development lifecycle.

---

## 12. FAQs

### 1. What is Trivy?

> Trivy is an open-source vulnerability scanner developed by Aqua Security that supports dependency scanning, container image scanning, filesystem scanning, secret detection, and Infrastructure as Code security analysis.

---

### 2. Why use Trivy for dependency scanning?

> Trivy automatically identifies vulnerable third-party libraries and recommends secure versions for remediation.

---

### 3. Does Trivy support Maven projects?

> Yes. Trivy automatically detects and scans Maven dependencies defined in `pom.xml`.

---

### 4. Why generate JSON reports?

> JSON reports allow integration with Jenkins, SIEM tools, dashboards, Elasticsearch, and custom automation workflows.

---

### 5. Why generate HTML reports?

> HTML reports provide an easy-to-read vulnerability summary for audits, reviews, and security assessments.

---

### 6. Can Trivy fail a CI build?

> Yes. Using the `--exit-code` option, Trivy can automatically fail CI jobs when specified severity thresholds are exceeded.

---

### 7. Can Trivy scan Docker images?

> Yes. Trivy supports vulnerability scanning for container images in addition to dependency scanning.

---

### 8. Can Trivy be integrated with Jenkins?

> Yes. Trivy integrates easily into Jenkins pipelines and supports artifact generation for security reporting.

---

## 13. Contact Information

| Name         | Contact                                                                           |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

## 14. References

| S.No | Description                 | Click to View                   |
| ---- | --------------------------- | ------------------------------- |
| 1    | Trivy Documentation         | https://trivy.dev               |
| 2    | Aqua Security Documentation | https://www.aquasec.com         |
| 3    | Maven Documentation         | https://maven.apache.org        |
| 4    | Java Documentation          | https://docs.oracle.com/en/java |
| 5    | CVE Database                | https://www.cve.org             |
| 6    | NVD Vulnerability Database  | https://nvd.nist.gov            |
