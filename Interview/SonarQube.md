# SonarQube Interview Notes

## Table of Contents

1. [Introduction](#1-introduction)
2. [What is SonarQube?](#2-what-is-sonarqube)
3. [Static Code Analysis](#3-static-code-analysis)
4. [SonarQube Analysis](#4-sonarqube-analysis)
5. [Bugs vs Code Smells](#5-bugs-vs-code-smells)
6. [Vulnerabilities and Security Hotspots](#6-vulnerabilities-and-security-hotspots)
7. [Rules, Quality Profiles, and Quality Gates](#7-rules-quality-profiles-and-quality-gates)
8. [Quality Gates](#8-quality-gates)
9. [New Code vs Overall Code](#9-new-code-vs-overall-code)
10. [SonarQube Properties Files](#10-sonarqube-properties-files)
11. [SonarScanners](#11-sonarscanners)
12. [Where is SonarScanner Located?](#12-where-is-sonarscanner-located)
13. [H2 Database vs Production Database](#13-h2-database-vs-production-database)
14. [SonarQube Server Directory Structure](#14-sonarqube-server-directory-structure)
15. [SonarQube Logs](#15-sonarqube-logs)
16. [SonarQube Default Port](#16-sonarqube-default-port)
17. [Static Analysis Tools in OT-Microservices](#17-static-analysis-tools-in-ot-microservices)
18. [SonarQube in CI/CD](#18-sonarqube-in-cicd)
19. [Common SonarQube Commands](#19-common-sonarqube-commands)
20. [Frequently Asked Interview Questions](#20-frequently-asked-interview-questions)
21. [One-Line Interview Answers](#21-one-line-interview-answers)

---

# 1. Introduction

**SonarQube** is a platform for continuous inspection of code quality and security.

It analyzes source code and reports issues such as:

* Bugs
* Vulnerabilities
* Code Smells
* Security Hotspots
* Code Duplications
* Code Coverage

SonarQube can be integrated into CI/CD pipelines so that code quality checks happen automatically during software delivery.

A simplified flow is:

```text
Developer
    │
    ▼
Git Repository
    │
    ▼
CI/CD Pipeline
    │
    ▼
SonarScanner
    │
    ▼
SonarQube
    │
    ├── Bugs
    ├── Vulnerabilities
    ├── Code Smells
    ├── Security Hotspots
    ├── Duplications
    └── Coverage
            │
            ▼
       Quality Gate
            │
       ┌────┴────┐
       ▼         ▼
     PASS       FAIL
```

---

# 2. What is SonarQube?

SonarQube is a **centralized code quality and security analysis platform**.

It performs static analysis without executing the application.

It can be used with multiple programming languages and can integrate with build tools and CI/CD platforms.

SonarQube is commonly used to:

* Detect bugs
* Detect vulnerabilities
* Identify code smells
* Identify security hotspots
* Measure code coverage
* Detect duplicated code
* Enforce coding standards
* Enforce Quality Gates
* Provide centralized code-quality visibility

---

# 3. Static Code Analysis

## What is Static Code Analysis?

**Static Code Analysis** means examining source code without executing the application.

Example:

```java
String password = "admin123";
```

A static analysis tool can identify that hardcoded credentials may be a security concern.

Another example:

```java
String name = null;

System.out.println(name.length());
```

The analyzer can identify a potential null-pointer problem.

The simplified flow is:

```text
Source Code
    │
    ▼
Static Analyzer
    │
    ├── Bugs
    ├── Vulnerabilities
    ├── Code Smells
    ├── Security Issues
    └── Duplications
```

---

# 4. SonarQube Analysis

SonarQube analysis is generally performed by a **scanner**.

The scanner analyzes the source code and sends the analysis information to the SonarQube server.

Simplified flow:

```text
Source Code
    │
    ▼
SonarScanner
    │
    ▼
SonarQube Server
    │
    ▼
Analysis
    │
    ▼
Quality Gate
```

SonarQube analysis can identify:

* Bugs
* Vulnerabilities
* Code Smells
* Security Hotspots
* Code Duplications
* Coverage-related metrics

---

# 5. Bugs vs Code Smells

This is a common SonarQube interview question.

| Feature | Code Smell | Bug |
|---------|------------|-----|
| Meaning | Poor coding practice | Programming defect |
| Main Impact | Maintainability / readability | Functional correctness |
| Can application run? | Usually yes | May fail or behave incorrectly |
| Priority | Improve the code | Fix promptly |

## Code Smell

A **Code Smell** is not necessarily a functional defect.

It indicates that the code can be improved to make it more:

* Readable
* Maintainable
* Understandable
* Extensible
* Efficient

### Example — Long Method

```java
public void processEmployee() {
    // 300 lines of code
}
```

A better design might split the logic into:

```text
validateEmployee()
calculateSalary()
saveEmployee()
sendNotification()
```

### Example — Duplicate Code

```java
if(user.isAdmin()){
    // logic
}

...

if(user.isAdmin()){
    // same logic
}
```

The common logic can be extracted into a reusable method.

### Example — Unused Variable

```java
int salary = 50000;
```

If `salary` is never used, this may be reported as a code-quality issue.

---

## Bug

A **Bug** is a programming defect that can cause incorrect results, exceptions, or application failures.

### Null Pointer

```java
String name = null;

System.out.println(name.length());
```

Possible result:

```text
NullPointerException
```

### Divide by Zero

```java
int result = 10 / 0;
```

Possible result:

```text
ArithmeticException
```

### Incorrect String Comparison

```java
if(password == "admin")
```

For Java string content comparison, use:

```java
if(password.equals("admin"))
```

The key distinction is:

```text
Bug
  → Functional defect

Code Smell
  → Maintainability / design problem
```

### Simple Analogy

**Bug:**

```text
Car brakes do not work
→ Unsafe
```

**Code Smell:**

```text
Car works
→ But it is messy and difficult to maintain
```

---

# 6. Vulnerabilities and Security Hotspots

SonarQube analysis also considers security.

## Vulnerability

A vulnerability is a security weakness that can potentially be exploited.

Examples may include:

* SQL injection
* Unsafe authentication logic
* Hardcoded secrets
* Insecure data handling

## Security Hotspot

A Security Hotspot indicates code that requires a security review because it may have security implications.

It is not necessarily a confirmed vulnerability.

The simplified distinction is:

```text
Vulnerability
    ↓
Known security weakness requiring remediation

Security Hotspot
    ↓
Security-sensitive code requiring review
```

---

# 7. Rules, Quality Profiles, and Quality Gates

The relationship is:

```text
Rules
   │
   ▼
Quality Profile
   │
   ▼
Project Analysis
   │
   ▼
Quality Gate
   │
   ▼
PASS / FAIL
```

## Quick Comparison

| Feature | Rules | Quality Profile | Quality Gate |
|--------|-------|-----------------|--------------|
| Definition | Individual coding checks | Collection of rules | Pass/fail conditions |
| Purpose | Detect problems | Decide which rules are applied | Decide whether quality standards are met |
| Applied To | Source code | Programming language | Project analysis result |
| Evaluation | During analysis | Determines analysis configuration | After analysis |
| Output | Issues | Rules to apply | Pass / Fail |

---

## 7.1 Rules

A **Rule** is an individual coding check.

Examples:

* Unused variables
* SQL injection
* Duplicate code
* Null-pointer risks
* Hardcoded credentials
* Empty catch blocks

Example:

```java
String password = "admin123";
```

A relevant rule may identify this as a security-sensitive coding practice.

### Interview Answer

> A Rule is an individual code-analysis check used by SonarQube to detect bugs, vulnerabilities, code smells, or other code-quality issues.

---

## 7.2 Quality Profile

A **Quality Profile** is a collection of rules assigned to a programming language.

Example:

```text
Java Quality Profile
        │
        ├── Rule 1
        ├── Rule 2
        ├── Rule 3
        ├── Rule 4
        └── Rule 5
```

When SonarQube analyzes Java code, the rules in the selected Java Quality Profile are applied.

Examples of language-specific profiles:

```text
Java Profile
Python Profile
Go Profile
```

### Types of Quality Profiles

| Type | Description |
|------|-------------|
| Built-in Quality Profile | Default profile provided by SonarQube, such as **Sonar way** |
| Custom Quality Profile | User-created or copied profile with customized rules |

### Interview Answer

> A Quality Profile is a collection of rules assigned to a programming language that defines which checks SonarQube applies during analysis.

---

## 7.3 Quality Gate

A **Quality Gate** is a set of conditions that determines whether the analyzed project passes or fails predefined quality standards.

Example:

```text
Coverage > 80%

No Blocker Bugs

No Critical Vulnerabilities

Duplicated Code < 3%
```

Possible result:

```text
Coverage = 85%
Blocker Bugs = 0
Critical Vulnerabilities = 0

→ PASS
```

Another example:

```text
Coverage = 52%
Critical Vulnerabilities = 2

→ FAIL
```

### Interview Answer

> A Quality Gate is a set of pass/fail conditions evaluated after SonarQube analysis to determine whether a project meets the required quality standards.

---

# 8. Quality Gates

## Main Components of a Quality Gate

A Quality Gate condition is based on:

```text
Metric
   +
Operator
   +
Threshold Value
   +
Scope
```

The scope can be based on:

* New Code
* Overall Code

Example:

```text
Coverage
   >
80%
```

---

## Common Metrics Used in Quality Gates

Common quality metrics include:

* Coverage
* Bugs
* Vulnerabilities
* Duplicated Code
* Security Rating
* Reliability Rating
* Maintainability Rating

Example enterprise-style gate:

```text
Coverage > 80%
Blocker Bugs = 0
Critical Vulnerabilities = 0
Duplicated Code < 3%
```

---

## Quality Gate Evaluation

Suppose the project has:

```text
Coverage = 85%
Blocker Bugs = 0
Critical Vulnerabilities = 0
Duplicated Code = 2%
```

Gate:

```text
Coverage > 80%
Blocker Bugs = 0
Critical Vulnerabilities = 0
Duplicated Code < 3%
```

All conditions pass:

```text
Quality Gate
     │
     ▼
   PASS
```

If any required condition fails:

```text
Quality Gate
     │
     ▼
   FAIL
```

---

# 9. New Code vs Overall Code

SonarQube can evaluate quality on:

## New Code

Only recently added or modified code.

```text
Existing Code
████████████████████

New Code
        ████
        ↑
      Analyze
```

This helps teams focus on preventing new quality problems while gradually improving legacy code.

## Overall Code

The entire project.

```text
Entire Codebase
████████████████████
       Analyze
```

### Comparison

| Scope | Meaning |
|------|---------|
| New Code | Recently added or changed code |
| Overall Code | Entire analyzed codebase |

---

# 10. SonarQube Properties Files

SonarQube uses properties files at different levels.

| File | Location | Purpose | Used By |
|------|----------|---------|---------|
| `sonar-project.properties` | Project root | Project-specific analysis configuration | Developers / CI |
| `sonar.properties` | `<SONARQUBE_HOME>/conf/` | SonarQube server configuration | SonarQube Administrator |
| `sonar-scanner.properties` | `<SONAR_SCANNER_HOME>/conf/` | Default SonarScanner CLI configuration | Scanner Administrator |
| `wrapper.conf` | `<SONARQUBE_HOME>/conf/` | Older service-wrapper configuration | Legacy installations |

---

## 10.1 sonar-project.properties

Project-level configuration.

Example:

```properties
sonar.projectKey=employee-api
sonar.projectName=Employee API
sonar.projectVersion=1.0
sonar.sources=src
sonar.tests=test
sonar.java.binaries=target/classes
sonar.host.url=http://localhost:9000
sonar.token=xxxxxxxx
```

### Use Case

* Stored with the project
* Can be version-controlled
* Used during project analysis
* Useful in CI/CD pipelines

### Interview Question

**Which properties file is used most frequently for project configuration?**

> `sonar-project.properties`, because it contains project-specific analysis settings.

---

## 10.2 sonar.properties

Located at:

```text
<SONARQUBE_HOME>/conf/sonar.properties
```

Example:

```properties
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
sonar.jdbc.username=sonar
sonar.jdbc.password=password
sonar.web.port=9000
```

### Use Case

Used to configure the SonarQube server, including server-level settings such as:

* Database connectivity
* Web server port
* Authentication-related settings
* Other server configuration

---

## 10.3 sonar-scanner.properties

Located at:

```text
<SONAR_SCANNER_HOME>/conf/sonar-scanner.properties
```

Example:

```properties
sonar.host.url=http://localhost:9000
sonar.sourceEncoding=UTF-8
```

### Use Case

Used for default/global SonarScanner CLI configuration.

---

# 11. SonarScanners

SonarQube uses scanners to perform code analysis.

Common scanner options include:

| Scanner | Purpose | Best For |
|---------|---------|----------|
| SonarScanner CLI | Generic scanner | Projects without a dedicated build integration |
| SonarScanner for Maven | Maven integration | Java Maven projects |
| SonarScanner for Gradle | Gradle integration | Gradle projects |
| SonarScanner for .NET | MSBuild/.NET integration | C# / VB.NET |
| SonarScanner CLI integrations | CI integrations | Jenkins, Azure DevOps, GitHub Actions |

## Which Scanner Should I Use?

| Project | Typical Scanner |
|---------|------------------|
| Java + Maven | SonarScanner for Maven |
| Java + Gradle | SonarScanner for Gradle |
| C# | SonarScanner for .NET |
| Go | SonarScanner CLI |
| Python | SonarScanner CLI |
| React / JavaScript | SonarScanner CLI |
| Jenkins | SonarScanner integrated into Jenkins pipeline |

The scanner choice depends on how the project is built and how SonarQube is integrated into the CI/CD environment.

---

# 12. Where is SonarScanner Located?

The location depends on the installation method.

## Linux Manual Installation

Typical locations:

```text
/opt/sonar-scanner/
```

or:

```text
/usr/local/sonar-scanner/
```

Typical structure:

```text
sonar-scanner/
├── bin/
├── conf/
├── jre/
└── lib/
```

Executable:

```text
/opt/sonar-scanner/bin/sonar-scanner
```

Check location:

```bash
which sonar-scanner
```

Another option:

```bash
find / -name sonar-scanner
```

---

## Jenkins

When managed through Jenkins:

```text
Manage Jenkins
      ↓
Global Tool Configuration
      ↓
SonarScanner
```

A Jenkins-managed installation may be under:

```text
/var/lib/jenkins/tools/
```

The exact path depends on Jenkins configuration.

---

# 13. H2 Database vs Production Database

## What is H2?

H2 is a lightweight embedded Java database that can be bundled with SonarQube for evaluation and testing scenarios.

It is **not intended for production use**.

## Characteristics

| Feature | H2 |
|---------|----|
| Embedded | ✅ |
| Lightweight | ✅ |
| Production Recommended | ❌ |
| External Installation | Not required |
| High Availability | ❌ |
| Scalability | Limited |

## Why Not H2 for Production?

The source notes identify these limitations:

* Data-loss risk
* No clustering
* No high availability
* Limited scalability
* Poor concurrent performance

## Production Database

The source material identifies **PostgreSQL** as the recommended production database and also lists enterprise database options such as Oracle and Microsoft SQL Server.

### Interview Answer

> H2 is intended for evaluation and testing. Production SonarQube deployments should use a supported external production database such as PostgreSQL.

---

# 14. SonarQube Server Directory Structure

A typical SonarQube installation contains:

```text
sonarqube/

├── bin/
├── conf/
├── data/
├── extensions/
├── lib/
├── logs/
├── temp/
└── web/
```

## Directory Explanation

| Directory | Purpose |
|-----------|----------|
| `bin/` | Startup and shutdown scripts |
| `conf/` | Server configuration such as `sonar.properties` |
| `data/` | Internal runtime/search-related data |
| `extensions/` | Plugins and downloaded extensions |
| `lib/` | SonarQube Java libraries |
| `logs/` | SonarQube log files |
| `temp/` | Temporary runtime files |
| `web/` | Web application resources |

A more detailed structure can look like:

```text
sonarqube/

├── bin/
│   ├── linux-x86-64/
│   ├── windows-x86-64/
│   └── sonar.sh
│
├── conf/
│   └── sonar.properties
│
├── data/
│   └── es8/
│
├── extensions/
│   ├── downloads/
│   └── plugins/
│
├── logs/
│   ├── sonar.log
│   ├── web.log
│   ├── ce.log
│   └── es.log
│
├── temp/
├── lib/
└── web/
```

---

# 15. SonarQube Logs

Important SonarQube log files include:

| Log File | Purpose |
|----------|----------|
| `sonar.log` | Main application log |
| `web.log` | Web-server-related activity |
| `ce.log` | Compute Engine processing |
| `es.log` | Elasticsearch-related activity |

These logs are useful when troubleshooting:

* Server startup problems
* Web UI issues
* Analysis processing
* Elasticsearch problems
* Background task failures

Typical location:

```text
<SONARQUBE_HOME>/logs/
```

---

# 16. SonarQube Default Port

The source material identifies:

```text
SonarQube Web Server → 9000
```

Access:

```text
http://<server-ip>:9000
```

Example:

```text
http://localhost:9000
```

The scanner communicates with the SonarQube server using the configured SonarQube URL.

## Can the Port Be Changed?

Yes.

In:

```text
<SONARQUBE_HOME>/conf/sonar.properties
```

Example:

```properties
sonar.web.port=8080
```

Restart SonarQube after changing the configuration.

## Verify Port

```bash
ss -tulpn | grep 9000
```

or:

```bash
netstat -tulpn | grep 9000
```

---

# 17. Static Analysis Tools in OT-Microservices

The OT-Microservices project uses language-specific static-analysis tools together with SonarQube.

| Microservice | Language | Static Analysis Tool | Purpose |
|--------------|----------|----------------------|---------|
| **Employee API** | Go | **golangci-lint** | Detect Go coding issues, unused code, style violations, and common bugs |
| **Attendance API** | Python | **Flake8** or **Pylint** | Detect syntax errors, PEP 8 violations, code-quality issues, and maintainability problems |
| **Salary API** | Java / Spring Boot | **SpotBugs**, **Checkstyle**, **PMD**, **SonarQube** | Detect bugs, coding-standard issues, duplicate code, security issues, and code smells |
| **Frontend** | React / JavaScript | **ESLint** | Detect JavaScript/React syntax issues, coding-standard violations, and best-practice issues |
| **Notification Worker** | Python | **Flake8** | Detect Python syntax errors, PEP 8 violations, and code-quality issues |
| **All Microservices** | Multiple languages | **SonarQube** | Centralized static analysis, security checks, code quality assessment, coverage, and Quality Gate enforcement |

## Why Use Both Language-Specific Tools and SonarQube?

Language-specific tools provide:

* Fast language-focused feedback
* Build-local checks
* Language-specific linting and static analysis

SonarQube provides:

* Centralized analysis
* Cross-project visibility
* Security analysis
* Code-quality metrics
* Quality Gates
* CI/CD quality enforcement

Simplified:

```text
Language-specific tools
        │
        ├── Go → golangci-lint
        ├── Python → Flake8 / Pylint
        ├── Java → SpotBugs / Checkstyle / PMD
        └── React → ESLint
                    │
                    ▼
                SonarQube
                    │
                    ▼
              Quality Gate
```

---

# 18. SonarQube in CI/CD

SonarQube becomes especially useful when integrated into a CI/CD pipeline.

A simplified pipeline can be:

```text
Developer Push
      │
      ▼
Git Repository
      │
      ▼
Jenkins / CI
      │
      ├── Build
      ├── Unit Tests
      ├── Static Analysis
      ├── SonarQube Analysis
      │
      ▼
   Quality Gate
      │
   ┌──┴───┐
   ▼      ▼
 PASS    FAIL
   │      │
   ▼      ▼
Continue  Stop / Fix Issues
```

The key idea is:

> SonarQube can become a quality control point in the delivery pipeline.

For example:

```text
Coverage > 80%
No Blocker Bugs
No Critical Vulnerabilities
Duplications < 3%

            ↓

        Quality Gate

        PASS → Continue
        FAIL → Stop
```

---

# 19. Common SonarQube Commands

## Check SonarScanner

```bash
sonar-scanner --version
```

## Locate SonarScanner

```bash
which sonar-scanner
```

## Run SonarScanner

```bash
sonar-scanner
```

If project settings are in `sonar-project.properties`, the scanner can use those settings.

## Check SonarQube Web Server

```bash
curl http://localhost:9000
```

## Check SonarQube Service

On a systemd-managed installation:

```bash
sudo systemctl status sonarqube
```

Start:

```bash
sudo systemctl start sonarqube
```

Restart:

```bash
sudo systemctl restart sonarqube
```

The exact service name depends on how SonarQube was installed and configured.

---

# 20. Frequently Asked Interview Questions

| Question | Answer |
|----------|--------|
| **What is SonarQube?** | SonarQube is a centralized platform for continuous inspection of code quality and security. |
| **What is static code analysis?** | Static code analysis examines source code without executing the application. |
| **What issues can SonarQube detect?** | Bugs, vulnerabilities, code smells, security hotspots, duplicate code, and quality-related metrics such as coverage. |
| **What is a Code Smell?** | A Code Smell is an indication of code that can be improved for readability, maintainability, or design quality; it is not necessarily a functional bug. |
| **What is a Bug?** | A Bug is a programming defect that can cause incorrect behavior, runtime errors, or application failure. |
| **What is a Vulnerability?** | A Vulnerability is a security weakness that can potentially be exploited. |
| **What is a Security Hotspot?** | A Security Hotspot identifies security-sensitive code that requires review; it is not necessarily a confirmed vulnerability. |
| **What is a Rule?** | A Rule is an individual coding check that can identify bugs, vulnerabilities, code smells, or other issues. |
| **What is a Quality Profile?** | A Quality Profile is a collection of rules assigned to a programming language. |
| **What is a Quality Gate?** | A Quality Gate is a collection of pass/fail conditions evaluated after analysis. |
| **Which component decides what rules are applied?** | The Quality Profile determines which rules are used during analysis. |
| **Which component decides PASS or FAIL?** | The Quality Gate. |
| **What is the relationship between Rules, Profiles, and Gates?** | Rules detect issues, Quality Profiles determine which rules are applied, and Quality Gates determine whether the resulting project quality meets defined standards. |
| **What is New Code?** | New Code is the recently added or modified portion of a project evaluated separately from the entire existing codebase. |
| **What is Overall Code?** | Overall Code refers to the entire analyzed project codebase. |
| **What is `sonar-project.properties`?** | A project-level configuration file containing SonarQube analysis settings. |
| **What is `sonar.properties`?** | The server configuration file located under the SonarQube `conf` directory. |
| **What is `sonar-scanner.properties`?** | A configuration file for default/global SonarScanner CLI settings. |
| **Which scanner should be used for Maven?** | SonarScanner for Maven. |
| **Which scanner should be used for Gradle?** | SonarScanner for Gradle. |
| **Which scanner is commonly used for Go and Python?** | SonarScanner CLI, unless a different integration is configured. |
| **Why is H2 not recommended for production?** | It is intended for evaluation/testing and does not provide the production scalability and high-availability characteristics required for an enterprise deployment. |
| **Which production database is identified in the notes for SonarQube?** | PostgreSQL is identified as the recommended production database. |
| **What is the default SonarQube port?** | 9000. |
| **Where is `sonar.properties` located?** | `<SONARQUBE_HOME>/conf/sonar.properties`. |
| **Where is the SonarQube web UI accessed?** | `http://<server-ip>:9000` by default. |
| **What is `web.log`?** | The SonarQube web-server log. |
| **What is `ce.log`?** | The Compute Engine log. |
| **What is `es.log`?** | The Elasticsearch-related log. |
| **What is `sonar.log`?** | The main SonarQube application log. |
| **Why use SonarQube in CI/CD?** | To automatically evaluate code quality and security and enforce quality standards before code progresses through the delivery pipeline. |
| **Why use language-specific tools along with SonarQube?** | Language-specific tools provide focused, fast feedback, while SonarQube provides centralized quality, security, and Quality Gate analysis. |

---

# 21. One-Line Interview Answers

| Component / Concept | One-Line Answer |
|---------------------|-----------------|
| **SonarQube** | A platform for continuous inspection of code quality and security. |
| **Static Code Analysis** | Analyzing source code without executing the application. |
| **Bug** | A programming defect that can cause incorrect behavior or failure. |
| **Code Smell** | An indication of code that can be improved for maintainability or design quality. |
| **Vulnerability** | A security weakness that can potentially be exploited. |
| **Security Hotspot** | Security-sensitive code that requires review. |
| **Rule** | An individual coding check used during static analysis. |
| **Quality Profile** | A collection of rules assigned to a programming language. |
| **Quality Gate** | A set of pass/fail conditions applied after analysis. |
| **New Code** | Recently added or modified code evaluated separately. |
| **Overall Code** | The complete analyzed codebase. |
| **SonarScanner** | The analysis component that scans source code and sends analysis information to SonarQube. |
| **`sonar-project.properties`** | Project-level SonarQube analysis configuration. |
| **`sonar.properties`** | SonarQube server configuration. |
| **H2** | Lightweight embedded database intended for evaluation/testing rather than production. |
| **Port 9000** | Default SonarQube web-server port in the source notes. |
| **Quality Gate in CI/CD** | A quality control point that can allow or block pipeline progression. |

---

## Core Interview Flow to Remember

```text
Source Code
    │
    ▼
Static Analysis
    │
    ▼
Rules
(What checks?)
    │
    ▼
Quality Profile
(Which rules apply?)
    │
    ▼
SonarQube Analysis
    │
    ├── Bugs
    ├── Vulnerabilities
    ├── Code Smells
    ├── Security Hotspots
    ├── Duplications
    └── Coverage
            │
            ▼
       Quality Gate
       (Pass / Fail)
            │
            ▼
         CI/CD
```

## OT-Microservices Summary

```text
Employee API        → Go       → golangci-lint
Attendance API      → Python   → Flake8 / Pylint
Salary API          → Java     → SpotBugs / Checkstyle / PMD
Frontend            → React    → ESLint
Notification Worker → Python   → Flake8

                    │
                    ▼

               SonarQube
                    │
                    ▼
              Quality Gate
                    │
             ┌──────┴──────┐
             ▼             ▼
           PASS           FAIL
             │             │
             ▼             ▼
         Continue      Fix Issues
```
