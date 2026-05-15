# <h1 align="center">POC - Java CI | Dependency Scanning using OWASP Dependency Check </h1>

<div align="center">
<img width="140" alt="Java" src="https://www.vectorlogo.zone/logos/java/java-ar21.svg" />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="150" alt="OWASP Dependency Check" src="https://owasp.org/www-project-dependency-check/assets/images/logo.png" />
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
2. [Clone Salary API Repository](#2-clone-salary-api-repository)
3. [Install Required Packages](#3-install-required-packages)
4. [Verify Installed JAVA and Maven](#4-verify-installed-java-and-maven)
5. [Add OWASP Dependency Check Plugin](#5-add-owasp-dependency-check-plugin)
6. [Build Java Application](#6-build-java-application)
7. [Run OWASP Dependency Scan](#7-run-owasp-dependency-scan)
8. [Generate HTML Report](#8-generate-html-report)
9. [Dependency Scan Findings](#9-dependency-scan-findings)
10. [Conclusion](#10-conclusion)
11. [Contact Information](#11-contact-information)
12. [References](#12-references)

---

## 1. Introduction

This POC demonstrates Dependency Scanning for the Java-based `salary-api` application using OWASP Dependency Check.

The scan validates:

* Vulnerable Maven dependencies
* Known CVEs in Spring Boot packages
* Dependency version risks
* Supply chain security issues
* Vulnerable third-party libraries

The POC uses the OT-Microservices `salary-api` Spring Boot project and integrates OWASP Dependency Check through Maven.

---

## 2. Clone Salary API Repository

Clone the repository:

```bash
git clone https://github.com/OT-MICROSERVICES/salary-api.git
cd salary-api
```

Verify project structure:

```bash
tree .
```

Expected project structure:

```text
salary-api/
├── pom.xml
├── src/
├── Dockerfile
├── migration/
├── static/
├── mvnw
└── README.md
```

<details>
<summary>📸 <strong>Click to view Screenshot - Salary API Repository</strong></summary>
<img width="1835" height="890" alt="image" src="https://github.com/user-attachments/assets/salary-api-repo.png" />
</details>

---

## 3. Install Required Packages

Update package index:

```bash
sudo apt update
```

Install JAVA and Maven:

```bash
sudo apt install -y openjdk-17-jdk maven wget curl unzip
```

Verify installation:

```bash
java -version
mvn -version
```

Expected Output:

```text
openjdk version "17"
Apache Maven 3.x
```

<details>
<summary>📸 <strong>Click to view Screenshot - JAVA and Maven Installation</strong></summary>
<img width="1835" height="890" alt="image" src="https://github.com/user-attachments/assets/java-maven-installation.png" />
</details>

---

## 4. Verify Installed JAVA and Maven

Check JAVA_HOME:

```bash
echo $JAVA_HOME
```

Verify Maven uses JAVA 17:

```bash
mvn -version
```

List installed JDKs:

```bash
sudo update-alternatives --config java
```

Expected JAVA:

```text
/usr/lib/jvm/java-17-openjdk-amd64
```

<details>
<summary>📸 <strong>Click to view Screenshot - JAVA Verification</strong></summary>
<img width="1820" height="880" alt="image" src="https://github.com/user-attachments/assets/java-verification.png" />
</details>

---

## 5. Add OWASP Dependency Check Plugin

Open `pom.xml`:

```bash
nano pom.xml
```

Add the following plugin inside the `<plugins>` section:

```xml
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
```

Also update Lombok dependency:

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.32</version>
    <optional>true</optional>
</dependency>
```

Lombok version update is required to ensure compatibility with newer Maven compiler and JDK environments. During build execution, the older Lombok version caused `java.lang.NoSuchFieldError` compilation issues related to internal Javac APIs.

Updating Lombok resolves:

* JDK compatibility issues
* Maven compiler plugin compatibility
* Javac internal API mismatch errors
* Build instability during dependency scanning execution

Save and exit.

<details>
<summary>📸 <strong>Click to view Screenshot - pom.xml Configuration</strong></summary>
<img width="1835" height="890" alt="image" src="https://github.com/user-attachments/assets/pom-xml-owasp.png" />
</details>

---

## 6. Build Java Application

Build the application while skipping tests:

```bash
mvn clean install -DskipTests
```

> [!NOTE]
> Tests are skipped because the `salary-api` application requires ScyllaDB/Cassandra connectivity during Spring Boot test execution.

Expected Output:

```text
BUILD SUCCESS
```

Generated artifact:

```text
target/salary-0.1.0-RELEASE.jar
```

<details>
<summary>📸 <strong>Click to view Screenshot - Maven Build Success</strong></summary>
<img width="1835" height="890" alt="image" src="https://github.com/user-attachments/assets/maven-build-success.png" />
</details>

---

## 7. Run OWASP Dependency Scan

Execute OWASP Dependency Check:

```bash
mvn org.owasp:dependency-check-maven:check -DskipTests
```

During first execution:

```text
Checking for updates
Downloaded 10000/351088
```

This occurs because OWASP downloads the NVD/CVE vulnerability database.

The scan validates:

* Vulnerable dependencies
* CVE mappings
* Dependency risk scoring
* Known Spring Boot vulnerabilities
* Third-party library security issues

Expected Output:

```text
BUILD SUCCESS
```

If high severity vulnerabilities are detected:

```text
BUILD FAILURE
```

<details>
<summary>📸 <strong>Click to view Screenshot - OWASP Dependency Scan</strong></summary>
<img width="1835" height="890" alt="image" src="https://github.com/user-attachments/assets/owasp-dependency-scan.png" />
</details>

---

## 8. Generate HTML Report

Verify report generation:

```bash
ls target/dependency-check-report.html
```

Open report:

```bash
xdg-open target/dependency-check-report.html
```

The report contains:

* CVE IDs
* Vulnerable packages
* CVSS severity ratings
* Recommended fixed versions
* Dependency paths
* Risk analysis summary

<details>
<summary>📸 <strong>Click to view Screenshot - Dependency Check HTML Report</strong></summary>
<img width="1835" height="890" alt="image" src="https://github.com/user-attachments/assets/dependency-check-report.png" />
</details>

---

## 9. Dependency Scan Findings

During OWASP Dependency Check execution, the `salary-api` application dependencies were analyzed against the NVD/CVE vulnerability database.

The scan successfully identified multiple vulnerable dependencies and automatically failed the build because the configured policy blocks vulnerabilities having CVSS score greater than or equal to `7.0`.

<details>
<summary>📸 <strong>Click to view Screenshot - OWASP Build Failure Output</strong></summary>
<img width="1835" height="890" alt="image" src="https://github.com/user-attachments/assets/owasp-build-failure.png" />
</details>text
BUILD FAILURE
One or more dependencies were identified with vulnerabilities
having a CVSS score greater than or equal to '7.0'
```

Example vulnerable dependencies identified during the scan:

| Dependency                       | CVE            | Severity |
| -------------------------------- | -------------- | -------- |
| snakeyaml-1.33.jar               | CVE-2022-1471  | Critical |
| spring-core-6.0.10.jar           | CVE-2024-22259 | High     |
| spring-web-6.0.10.jar            | CVE-2024-22259 | High     |
| tomcat-embed-core-10.1.10.jar    | Multiple CVEs  | Critical |
| netty-transport-4.1.94.Final.jar | Multiple CVEs  | High     |

This demonstrates:

* Automated dependency vulnerability detection
* CVE validation against NVD database
* Build policy enforcement using CVSS threshold
* Supply chain security analysis
* Early security issue identification during CI validation

<details>
<summary>📸 <strong>Click to view Screenshot - OWASP Vulnerability Findings</strong></summary>
<img width="1835" height="890" alt="image" src="https://github.com/user-attachments/assets/owasp-vulnerability-findings.png" />
</details>

---

## 10. Conclusion

This POC demonstrates manual dependency vulnerability scanning for the `salary-api` Spring Boot application using OWASP Dependency Check.

The implementation successfully validates third-party Maven dependencies against known CVEs from the NVD database and automatically enforces security policies using CVSS-based build failure conditions.

The scan identified multiple vulnerable dependencies including Spring Framework, SnakeYAML, Tomcat Embedded Core, and Netty libraries, demonstrating how dependency scanning helps strengthen software supply chain security during the CI validation process.

Generated HTML reports provide detailed vulnerability analysis, CVSS severity ratings, affected packages, and remediation recommendations for secure dependency management.

---

## 11. Contact Information

| Name         | Contact                                                                           |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

## 12. References

| S.No | Description                | Click to View                                                                                                                                                                |
| ---- | -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | OWASP Dependency Check     | [![OWASP](https://img.shields.io/badge/OWASP-DEPENDENCY_CHECK-2F2F2F?style=flat-square\&logo=owasp\&logoColor=white)](https://owasp.org/www-project-dependency-check/)       |
| 2    | Maven Documentation        | [![MAVEN](https://img.shields.io/badge/MAVEN-DOCUMENTATION-2F2F2F?style=flat-square\&logo=apachemaven\&logoColor=white)](https://maven.apache.org/guides/)                   |
| 3    | Spring Boot Documentation  | [![SPRING\_BOOT](https://img.shields.io/badge/SPRING_BOOT-DOCUMENTATION-2F2F2F?style=flat-square\&logo=springboot\&logoColor=white)](https://spring.io/projects/spring-boot) |
| 4    | NVD Vulnerability Database | [![NVD](https://img.shields.io/badge/NVD-DATABASE-2F2F2F?style=flat-square\&logoColor=white)](https://nvd.nist.gov/)                                                         |
| 5    | CVE Database               | [![CVE](https://img.shields.io/badge/CVE-DATABASE-2F2F2F?style=flat-square\&logoColor=white)](https://www.cve.org/)                                                          |
