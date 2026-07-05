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

# One-Line Interview Answer

> **Software Composition Analysis (SCA) scans third-party dependencies for known vulnerabilities (CVEs) and license compliance, whereas Static Code Analysis (SAST) scans the application's own source code to detect security vulnerabilities, coding errors, bugs, and code quality issues without executing the program.**
