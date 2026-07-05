# SCA (Software Composition Analysis) vs Static Code Analysis (SAST)

> **Interview Tip:** This is one of the most common DevOps and DevSecOps interview questions. Many people confuse these two because both analyze code before deployment, but they focus on different things.

| Feature | SCA (Software Composition Analysis) | Static Code Analysis (SAST) |
|---------|--------------------------------------|-----------------------------|
| **Definition** | Analyzes third-party/open-source dependencies used in an application. | Analyzes the application's own source code for coding errors and security vulnerabilities. |
| **Focus** | External libraries and packages. | Developer-written source code. |
| **Scans** | Dependencies (Maven, npm, pip, Go modules, etc.) | Java, Python, Go, C#, JavaScript source code. |
| **Checks** | Known vulnerabilities (CVEs), licenses, outdated packages. | Security flaws, coding mistakes, code quality, bugs. |
| **Database Used** | CVE/NVD databases, OSS databases. | Source code analysis rules and security patterns. |
| **Detects** | Vulnerable libraries like Log4j, Spring, OpenSSL, etc. | SQL Injection, XSS, Hardcoded Passwords, Null Pointer, Resource Leaks, etc. |
| **License Scanning** | ✅ Yes | ❌ No |
| **Code Quality** | ❌ No | ✅ Yes |
| **OWASP Coverage** | Dependency-related risks | Application code vulnerabilities |
| **Runs During** | Dependency installation/build stage | Source code analysis stage |
| **Output** | Vulnerable packages and CVEs | Bugs, vulnerabilities, code smells |
| **Examples** | Snyk, Trivy, OWASP Dependency-Check, Black Duck, Mend | SonarQube, Checkmarx, Fortify, Semgrep, PMD, SpotBugs |

---

# SCA (Software Composition Analysis)

## What is SCA?

SCA scans **third-party libraries and dependencies** used in your application.

It checks for:

- Vulnerable dependencies
- CVEs
- Outdated packages
- License compliance

### Example

Java Project

```text
pom.xml

Spring Boot
Log4j
JUnit
Lombok
```

SCA scans these dependencies and reports:

```text
Log4j

CVE-2021-44228

Critical Vulnerability
```

It **does not** scan your Java code.

---

# Static Code Analysis (SAST)

## What is Static Code Analysis?

Static Code Analysis analyzes **your own source code** without executing it.

It checks for:

- SQL Injection
- Cross-Site Scripting (XSS)
- Hardcoded credentials
- Memory leaks
- Null pointer exceptions
- Code smells
- Duplicate code
- Coding standard violations

### Example

```java
String query =
"SELECT * FROM users WHERE id='" + userInput + "'";
```

SAST reports:

```text
SQL Injection Risk
```

---

# Real CI/CD Pipeline

```text
Developer
     │
     ▼
Git Push
     │
     ▼
Build
     │
     ▼
Dependency Installation
     │
     ▼
SCA
     │
     ▼
Static Code Analysis (SAST)
     │
     ▼
Unit Testing
     │
     ▼
Build Artifact
     │
     ▼
Container Scan
     │
     ▼
Deploy
```

---

# Example

Suppose your Java application uses:

```
Spring Boot

Log4j

Apache Commons
```

### SCA Finds

```
Log4j

Critical CVE

Upgrade Required
```

---

Your own Java code:

```java
String sql =
"SELECT * FROM employee WHERE id=" + input;
```

### Static Code Analysis Finds

```
SQL Injection

High Severity
```

---

# Common Tools

## SCA Tools

| Tool | Purpose |
|------|---------|
| Snyk | Dependency & License Scanning |
| Trivy | Dependency & Container Scanning |
| OWASP Dependency-Check | CVE Detection |
| Black Duck | Enterprise SCA |
| Mend (WhiteSource) | Enterprise SCA |
| Grype | Dependency Vulnerability Scanning |

---

## Static Code Analysis (SAST) Tools

| Tool | Purpose |
|------|---------|
| SonarQube | Code Quality + Security |
| Checkmarx | Enterprise SAST |
| Fortify | Enterprise SAST |
| Semgrep | Fast Static Analysis |
| PMD | Java Code Quality |
| SpotBugs | Java Bug Detection |
| ESLint | JavaScript Static Analysis |
| Pylint | Python Static Analysis |

---

# Key Differences

| SCA | Static Code Analysis |
|-----|----------------------|
| Scans dependencies | Scans source code |
| Finds vulnerable libraries | Finds insecure coding practices |
| Uses CVE databases | Uses coding rules |
| Detects outdated packages | Detects code defects |
| Checks licenses | Does not check licenses |

---

# Interview Questions

| Question | Answer |
|----------|--------|
| What is SCA? | Software Composition Analysis scans third-party dependencies for vulnerabilities and license compliance. |
| What is Static Code Analysis? | Static Code Analysis (SAST) scans source code for security vulnerabilities, bugs, and code quality issues without executing the application. |
| Does SCA scan source code? | No. It scans third-party dependencies. |
| Does Static Code Analysis scan dependencies? | No. It scans developer-written source code. |
| Does SCA detect CVEs? | Yes. |
| Does Static Code Analysis detect SQL Injection? | Yes. |
| Does SCA perform License Scanning? | Yes. |
| Is SonarQube an SCA tool? | No. It is primarily a Static Code Analysis (SAST) tool. |
| Is OWASP Dependency-Check an SCA tool? | Yes. |
| Is Trivy an SCA tool? | Yes, it performs dependency scanning and can also scan container images. |

---
# DevSecOps CI Checks Classification

| Check / Activity | Category | What it Checks | Common Tools |
|------------------|----------|----------------|--------------|
| **Syntax Validation** | Separate Stage (Build Validation) | Programming language syntax, compilation errors | `javac`, `go build`, `python -m py_compile`, `tsc`, `gcc` |
| **Linting** | SAST (Static Code Analysis - Code Quality) | Coding standards, formatting, best practices, style violations | ESLint, Pylint, Flake8, Checkstyle, GolangCI-Lint |
| **Code Quality Analysis** | SAST | Code smells, duplicate code, maintainability, complexity | SonarQube, PMD, SpotBugs |
| **Security Static Analysis** | SAST | SQL Injection, XSS, hardcoded passwords, insecure APIs, buffer overflows | SonarQube, Checkmarx, Fortify, Semgrep, Veracode |
| **Dependency Scanning** | SCA | Vulnerable third-party libraries and dependencies (CVEs) | OWASP Dependency-Check, Snyk, Trivy, Grype |
| **License Scanning** | SCA | Open-source license compliance (MIT, GPL, Apache, BSD, etc.) | Black Duck, Mend, FOSSA, Trivy, Snyk |
| **SBOM Generation** | SCA | Software Bill of Materials (complete dependency inventory) | Syft, CycloneDX, SPDX |
| **Secret/Credential Scanning** | Separate Stage | Hardcoded passwords, API keys, tokens, SSH keys, certificates | Gitleaks, TruffleHog, GitGuardian, detect-secrets |
| **Unit Testing** | Separate Stage | Functional correctness of individual units/components | JUnit, pytest, Jest, Go Test |
| **Code Coverage** | Separate Stage | Percentage of code covered by unit tests | JaCoCo, Cobertura, Istanbul, Coverage.py |
| **Build/Compilation** | Separate Stage | Generates executable/binary/artifact | Maven, Gradle, Go Build, npm, Poetry |
| **Container Image Scanning** | Separate Stage | Vulnerabilities in Docker images and OS packages | Trivy, Grype, Docker Scout, Clair |
| **IaC Scanning** | Separate Stage | Security issues in Terraform, CloudFormation, Kubernetes YAML | Checkov, tfsec, Terrascan, KICS |
| **DAST (Dynamic Analysis)** | Separate Stage | Runtime security testing of a running application | OWASP ZAP, Burp Suite |
| **Performance Testing** | Separate Stage | Load, stress, and scalability testing | JMeter, Gatling, k6 |
| **Infrastructure Compliance** | Separate Stage | Cloud security and compliance validation | AWS Config, Prowler, ScoutSuite |

---

# What Comes Under SAST?

| Included in SAST | Purpose |
|------------------|---------|
| Linting | Coding standards and style |
| Static Security Analysis | Finds vulnerabilities in source code |
| Code Quality Analysis | Code smells, maintainability, duplication |
| Bug Detection | Finds coding defects before execution |

---

# What Comes Under SCA?

| Included in SCA | Purpose |
|-----------------|---------|
| Dependency Scanning | Finds vulnerable third-party libraries |
| License Scanning | Identifies OSS licenses and policy violations |
| SBOM Generation | Generates inventory of all dependencies |
| Dependency Version Analysis | Detects outdated packages |

---

# What Does NOT Belong to SAST or SCA?

| Separate Stage | Purpose |
|----------------|---------|
| Syntax Validation | Checks language syntax and compilation |
| Secret Scanning | Finds exposed credentials and secrets |
| Unit Testing | Verifies application functionality |
| Code Coverage | Measures test coverage |
| Build/Compilation | Produces executable artifacts |
| Container Image Scanning | Scans Docker images |
| IaC Scanning | Scans Infrastructure as Code |
| DAST | Runtime application security testing |
| Performance Testing | Measures performance under load |

---

# Easy Interview Memory Trick

| Think About... | Category |
|----------------|----------|
| **My Source Code** | **SAST** |
| **Third-Party Dependencies** | **SCA** |
| **Secrets (Passwords, Tokens)** | **Secret Scanning** |
| **Code Compiles?** | **Syntax Validation** |
| **Application Works?** | **Unit Testing** |
| **Docker Image Secure?** | **Container Scanning** |
| **Terraform/Kubernetes Secure?** | **IaC Scanning** |
| **Running Application Secure?** | **DAST** |

---

# One-Line Rule

| Category | One-Line Definition |
|----------|---------------------|
| **SAST** | Scans **your own source code** for bugs, vulnerabilities, and code quality issues. |
| **SCA** | Scans **third-party dependencies** for vulnerabilities, licenses, and outdated packages. |
| **Secret Scanning** | Scans repositories for hardcoded credentials and sensitive information. |
| **Syntax Validation** | Verifies that the code is syntactically correct and can compile. |
# One-Line Interview Answer

> **Software Composition Analysis (SCA) scans third-party dependencies for known vulnerabilities (CVEs) and license compliance, whereas Static Code Analysis (SAST) scans the application's own source code to detect security vulnerabilities, coding errors, bugs, and code quality issues without executing the program.**
