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
2. [Pre-requisites](#2-pre-requisites)
3. [Clone Salary API Repository](#3-clone-salary-api-repository)
4. [Install and Configure JAVA 17 and Maven](#4-install-and-configure-java-17-and-maven)
5. [Add OWASP Dependency Check Plugin](#5-add-owasp-dependency-check-plugin)
6. [Build Java Application](#6-build-java-application)
7. [Run OWASP Dependency Scan](#7-run-owasp-dependency-scan)
8. [Generate HTML Report](#8-generate-html-report)
9. [Dependency Scan Findings](#9-dependency-scan-findings)
10. [Conclusion](#10-conclusion)
11. [FAQs](#11-faqs)
12. [Contact Information](#12-contact-information)
13. [References](#13-references)
---

## 1. Introduction

This POC demonstrates Dependency Scanning for the Java-based `salary-api` application using OWASP Dependency Check.

---
## 2. Pre-requisites

Before performing Java dependency scanning using OWASP Dependency Check, ensure the following prerequisites are available:

| Requirement | Version/Specification |
|---|---|
| Ubuntu/Linux VM | Ubuntu 22.04 LTS | 
| JAVA | OpenJDK 17 |
| Maven | 3.8+ |
| RAM | Minimum 2 GB |
| Disk Space | Minimum 5 GB Free |

---

## 3. Clone Salary API Repository

Clone the repository:

```bash
git clone https://github.com/OT-MICROSERVICES/salary-api.git
cd salary-api
```
<details>
<summary>📸 <strong>Click to view Screenshot - Salary API Repository</strong></summary>
<img width="1325" height="902" alt="image" src="https://github.com/user-attachments/assets/9c1595f4-7445-490e-90e8-1d3d6163edbc" />
</details>

---

## 4. Install and Configure JAVA 17 and Maven

Install JAVA 17 and Maven:

```bash
sudo apt update && sudo apt install -y openjdk-17-jdk maven wget curl unzip
```
Verify installed JAVA and Maven:

```bash
java -version
mvn -version
```
If another JAVA version is active, switch to JAVA 17:

```bash
sudo update-alternatives --config java
sudo update-alternatives --config javac
```

<details>
<summary>📸 <strong>Click to view Screenshot - JAVA 17 and Maven Configuration</strong></summary>
<img width="1248" height="350" alt="image" src="https://github.com/user-attachments/assets/82953f54-4401-4c45-9661-2063325996b7" />
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

> [!NOTE]
> Lombok version update is required to ensure compatibility with newer Maven compiler and JDK environments. During build execution, the older Lombok version caused `java.lang.NoSuchFieldError` compilation issues related to internal Javac APIs.

<details>
<summary>📄 <strong>Click to view Full Corrected pom.xml</strong></summary>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.1.1</version>
        <relativePath/>
    </parent>

    <groupId>com.opstree.microservice</groupId>
    <artifactId>salary</artifactId>
    <version>0.1.0-RELEASE</version>

    <name>salary</name>

    <description>
        Java microservice to handle all salary related data
    </description>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>

        <!-- Spring Boot Actuator -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.32</version>
            <optional>true</optional>
        </dependency>

        <!-- Logstash Logging -->
        <dependency>
            <groupId>net.logstash.logback</groupId>
            <artifactId>logstash-logback-encoder</artifactId>
            <version>6.6</version>
        </dependency>

        <!-- OpenAPI -->
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
            <version>2.0.3</version>
        </dependency>

        <!-- Spring Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Cassandra -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-cassandra</artifactId>
        </dependency>

        <!-- Redis -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

        <!-- Prometheus Metrics -->
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-registry-prometheus</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <build>

        <plugins>

            <!-- Spring Boot Plugin -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>

                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>

            <!-- Unit Testing -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>

                <configuration>
                    <excludes>
                        <exclude>
                            com/opstree/microservice/salary/controller/**
                        </exclude>
                        <exclude>
                            com/opstree/microservice/salary/service/**
                        </exclude>
                        <exclude>
                            com/opstree/microservice/salary/repository/**
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>

            <!-- JaCoCo Coverage -->
            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>0.8.7</version>

                <executions>

                    <execution>
                        <goals>
                            <goal>prepare-agent</goal>
                        </goals>
                    </execution>

                    <execution>
                        <id>report</id>
                        <phase>test</phase>

                        <goals>
                            <goal>report</goal>
                        </goals>
                    </execution>

                </executions>

                <configuration>
                    <excludes>
                        <exclude>
                            com/opstree/microservice/salary/controller/**
                        </exclude>
                        <exclude>
                            com/opstree/microservice/salary/service/**
                        </exclude>
                        <exclude>
                            com/opstree/microservice/salary/repository/**
                        </exclude>
                    </excludes>
                </configuration>

            </plugin>

            <!-- OWASP Dependency Check -->
            <plugin>
                <groupId>org.owasp</groupId>
                <artifactId>dependency-check-maven</artifactId>
                <version>10.0.4</version>

                <configuration>
                    <format>HTML</format>
                    <failBuildOnCVSS>7</failBuildOnCVSS>
                </configuration>

            </plugin>

        </plugins>

    </build>

</project>
```

</details>

---

## 6. Build Java Application

Build the application while skipping tests:

```bash
mvn clean install -DskipTests
```

> [!NOTE]
> Tests are skipped because the `salary-api` application requires ScyllaDB/Cassandra connectivity during Spring Boot test execution.

Generated artifact:

```text
target/salary-0.1.0-RELEASE.jar
```

<details>
<summary>📸 <strong>Click to view Screenshot - Maven Build Success</strong></summary>
<img width="1485" height="903" alt="image" src="https://github.com/user-attachments/assets/9ed3a597-c170-4bc2-87f6-ad08a99c5b6d" />
<img width="1485" height="464" alt="image" src="https://github.com/user-attachments/assets/72eab5ca-768c-457a-ac8b-943f8f5ea9b3" />
<img width="1485" height="118" alt="image" src="https://github.com/user-attachments/assets/a9f2293f-90d5-469c-8842-5088f738426e" />

</details>

---

## 7. Run OWASP Dependency Scan

Execute OWASP Dependency Check:

```bash
mvn org.owasp:dependency-check-maven:check -DskipTests
```

> [!IMPORTANT]
> During the first execution, OWASP Dependency Check downloads the complete NVD/CVE vulnerability database from NIST.
>
> ```text
> Checking for updates
> Downloaded 10000/351088
> ```
>
> This process may take significant time depending on internet speed because vulnerability metadata and CVE feeds are downloaded and cached locally for future scans.
>
> Subsequent scans will be considerably faster since the database is reused from the local cache.

<details>
<summary>📸 <strong>Click to view Screenshot - OWASP Dependency Scan</strong></summary>
<img width="1502" height="907" alt="image" src="https://github.com/user-attachments/assets/3287e5d2-0731-4a40-af1c-75e48f036dc1" />
<img width="1502" height="907" alt="image" src="https://github.com/user-attachments/assets/5d9419bc-61e8-4cb8-9830-19edb6b62e9c" />

</details>

---

## 8. Generate HTML Report

Verify report generation:

```bash
ls target/dependency-check-report.html
```
Open report:

```bash
cd ~/salary-api/target
python3 -m http.server 4000
```

<details>
<summary>📸 <strong>Click to view Screenshot - Dependency Check HTML Report</strong></summary>
<img width="1497" height="279" alt="image" src="https://github.com/user-attachments/assets/ee30b35a-30e0-4a7c-9446-ac1ba6a6c40e" />
<img width="1497" height="279" alt="image" src="https://github.com/user-attachments/assets/60f2ac69-56aa-4fec-8264-b025edc7b2b0" />
<img width="1832" height="989" alt="Screenshot from 2026-05-15 21-32-11" src="https://github.com/user-attachments/assets/999934af-cb83-4189-b308-c54e45aee899" />
<img width="1832" height="884" alt="Screenshot from 2026-05-15 21-32-27" src="https://github.com/user-attachments/assets/a3878c95-b70a-4d59-a646-29009f810f35" />

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

---

## 10. Conclusion

This POC demonstrates manual dependency vulnerability scanning for the `salary-api` Spring Boot application using OWASP Dependency Check.

The implementation successfully validates third-party Maven dependencies against known CVEs from the NVD database and automatically enforces security policies using CVSS-based build failure conditions.

The scan identified multiple vulnerable dependencies including Spring Framework, SnakeYAML, Tomcat Embedded Core, and Netty libraries, demonstrating how dependency scanning helps strengthen software supply chain security during the CI validation process.

Generated HTML reports provide detailed vulnerability analysis, CVSS severity ratings, affected packages, and remediation recommendations for secure dependency management.

---
## 11. FAQs

### 1. What is OWASP Dependency Check?

> OWASP Dependency Check is a Software Composition Analysis (SCA) tool used to identify vulnerable third-party dependencies in applications.

---

### 2. Why is dependency scanning important in CI pipelines?

> Dependency scanning helps identify known vulnerabilities in external libraries before application deployment.

---

### 3. What does `failBuildOnCVSS=7` mean?

> It automatically fails the build if any detected vulnerability has a CVSS score greater than or equal to 7.

---

### 4. Why was `-DskipTests` used during Maven build?

> Tests were skipped because the application required ScyllaDB/Cassandra connectivity during test execution.

---

### 5. Why was the Lombok version updated?

> The Lombok version was updated to resolve JDK and Maven compiler compatibility issues.

---

### 6. Why was the OWASP plugin execution block removed from `pom.xml`?

> The execution block was removed to prevent OWASP scans from running automatically during every Maven build and to allow manual dependency scanning execution.

---

## 12. Contact Information

| Name         | Contact                                                                           |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

## 13. References

| S.No | Description                | Click to View                                                                                                                                                                |
| ---- | -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | OWASP Dependency Check     | [![OWASP](https://img.shields.io/badge/OWASP-DEPENDENCY_CHECK-2F2F2F?style=flat-square\&logo=owasp\&logoColor=white)](https://owasp.org/www-project-dependency-check/)       |
| 2    | Maven Documentation        | [![MAVEN](https://img.shields.io/badge/MAVEN-DOCUMENTATION-2F2F2F?style=flat-square\&logo=apachemaven\&logoColor=white)](https://maven.apache.org/guides/)                   |
| 3    | Spring Boot Documentation  | [![SPRING\_BOOT](https://img.shields.io/badge/SPRING_BOOT-DOCUMENTATION-2F2F2F?style=flat-square\&logo=springboot\&logoColor=white)](https://spring.io/projects/spring-boot) |
| 4    | NVD Vulnerability Database | [![NVD](https://img.shields.io/badge/NVD-DATABASE-2F2F2F?style=flat-square\&logoColor=white)](https://nvd.nist.gov/)                                                         |
| 5    | CVE Database               | [![CVE](https://img.shields.io/badge/CVE-DATABASE-2F2F2F?style=flat-square\&logoColor=white)](https://www.cve.org/)                                                          |
