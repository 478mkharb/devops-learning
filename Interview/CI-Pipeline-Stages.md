# CI Pipeline Stages Interview Notes

## Table of Contents

1. [Introduction](#1-introduction)
2. [Common CI/CD Pipeline Flow](#2-common-cicd-pipeline-flow)
3. [React Frontend CI Pipeline](#3-react-frontend-ci-pipeline)
4. [Java CI Pipeline](#4-java-ci-pipeline)
5. [Python Flask CI Pipeline](#5-python-flask-ci-pipeline)
6. [Go Gin CI Pipeline](#6-go-gin-ci-pipeline)
7. [SAST vs SCA](#7-sast-vs-sca)
8. [DevSecOps CI Checks Classification](#8-devsecops-ci-checks-classification)
9. [SonarQube in CI](#9-sonarqube-in-ci)
10. [Java Maven Lifecycle in CI](#10-java-maven-lifecycle-in-ci)
11. [`package.json`, `package-lock.json`, `npm install`, and `npm ci`](#11-packagejson-package-lockjson-npm-install-and-npm-ci)
12. [Build, Artifact, Publish, and Deploy](#12-build-artifact-publish-and-deploy)
13. [DAST in CI/CD](#13-dast-in-cicd)
14. [Frequently Asked Interview Questions](#14-frequently-asked-interview-questions)
15. [One-Line Interview Answers](#15-one-line-interview-answers)

---

# 1. Introduction

A **CI (Continuous Integration) pipeline** automatically validates application code whenever changes are integrated into a shared source-code repository.

Typical CI activities include:

* Source checkout
* Secret scanning
* Dependency installation
* Formatting and linting
* Static code analysis
* Dependency vulnerability scanning
* Unit testing
* Code coverage
* Compilation / build
* SonarQube analysis
* Quality Gate validation
* Artifact creation

A broader CI/CD pipeline can continue with:

* Artifact publishing
* Deployment
* Dynamic security testing

The exact stages and their order depend on the application's technology stack and organizational requirements.

---

# 2. Common CI/CD Pipeline Flow

A generalized enterprise pipeline can be represented as:

```text
Developer
    │
    ▼
Git Repository
    │
    ▼
Checkout
    │
    ▼
Secret Scanning
    │
    ▼
Dependency Installation
    │
    ▼
Formatting / Linting / Static Analysis
    │
    ▼
SCA / Dependency Scan
    │
    ▼
Compile / Syntax Validation
    │
    ▼
Unit Testing
    │
    ▼
Code Coverage
    │
    ▼
Build / Package
    │
    ▼
SonarQube Analysis
    │
    ▼
Quality Gate
    │
    ▼
Artifact
    │
    ├── Publish
    │
    └── Deploy
          │
          ▼
        DAST
```

Not every project uses every stage.

Some stages may be:

* Combined
* Skipped
* Reordered
* Technology-specific

---

# 3. React Frontend CI Pipeline

| Stage | CI Step | Recommended Tool / Command | Purpose |
|:---:|---|---|---|
| **1** | Checkout | `git clone` | Fetch source code from SCM |
| **2** | Secret Scanning | `gitleaks detect .` | Detect hardcoded secrets |
| **3** | Install Dependencies | `npm install` / `npm ci` | Install project dependencies |
| **4** | Code Formatting | `npx prettier --check .` | Validate formatting |
| **5** | Linting | `npm run lint` | Run ESLint/static code checks |
| **6** | Dependency Scan | `npm audit --audit-level=high` / `trivy fs .` | Detect vulnerable npm packages |
| **7** | Unit Testing | `CI=true npm test -- --watchAll=false` | Execute Jest tests |
| **8** | Code Coverage | `npm test -- --coverage --watchAll=false` | Generate coverage |
| **9** | Build | `CI=false npm run build` | Generate optimized production build |
| **10** | SonarQube Analysis | `sonar-scanner` | Analyze code quality/security |
| **11** | Quality Gate | SonarQube | Validate quality conditions |
| **12** | Build Artifact | `build/` | Archive production build |
| **13** | Deploy | Nginx | Deploy static React application |
| **14** | DAST | OWASP ZAP | Test the running application |

## Why `npm ci` is Recommended in CI

```bash
npm ci
```

It performs a clean, reproducible dependency installation based on `package-lock.json`.

It removes the existing:

```text
node_modules/
```

before installing dependencies.

## Why `CI=true --watchAll=false`?

Example:

```bash
CI=true npm test -- --watchAll=false
```

* `CI=true` tells the test tooling that it is running in a CI environment.
* `--watchAll=false` ensures tests run once and exit instead of entering watch mode.

---

# 4. Java CI Pipeline

| Stage | Recommended Command | Purpose |
|:---:|---|---|
| **1. Checkout** | `git clone` | Fetch source code |
| **2. Secret Scanning** | `gitleaks detect .` | Detect passwords, keys, tokens |
| **3. Validate** | `mvn validate` | Validate project structure and `pom.xml` |
| **4. Dependency Resolution** | `mvn dependency:resolve` | Resolve project dependencies |
| **5. Code Formatting** | `mvn spotless:check` | Validate configured formatting |
| **6. Linting** | `mvn checkstyle:check` | Validate Java coding standards |
| **7. Dependency Scan / SCA** | `mvn org.owasp:dependency-check:check` | Detect vulnerable dependencies |
| **8. Compile** | `mvn compile` / `mvn test-compile` | Compile production and test code |
| **9. Unit Testing** | `mvn test` | Run JUnit/TestNG tests |
| **10. Package** | `mvn package -DskipTests` | Create JAR/WAR after tests have already run |
| **11. Verify** | `mvn verify` | Run verification and generate coverage reports |
| **12. SonarQube Analysis** | `mvn sonar:sonar` | Upload analysis information |
| **13. Quality Gate** | `waitForQualityGate()` | Wait for SonarQube gate result |
| **14. Install** | `mvn install` | Put artifact into local Maven repository |
| **15. Publish Artifact** | `mvn deploy` | Publish artifact to remote repository |
| **16. Deploy Application** | `java -jar app.jar` / Ansible / Systemd / Kubernetes | Deploy application |
| **17. DAST** | `zap-baseline.py -t http://<app-url>` | Dynamic security testing |

## Maven Pipeline Notes

Maven lifecycle phases are cumulative:

```text
validate
   ↓
compile
   ↓
test
   ↓
package
   ↓
verify
   ↓
install
   ↓
deploy
```

Therefore:

```bash
mvn verify
```

runs the earlier required lifecycle phases before verification.

### `mvn dependency:resolve`

This goal is optional because Maven normally resolves missing dependencies during normal builds.

It can be used to:

```text
Resolve dependencies early
        ↓
Fail fast if resolution fails
```

### Unit Tests Before Packaging

Normally:

```bash
mvn package
```

includes the test phase.

A pipeline may intentionally do:

```bash
mvn test
mvn package -DskipTests
```

to avoid running the same unit tests twice.

### JaCoCo

JaCoCo generates Java code-coverage data.

```text
mvn test
    │
    ▼
JaCoCo
    │
    ▼
Coverage Data
    │
    ▼
mvn verify
    │
    ▼
Coverage Reports
    │
    ▼
SonarQube
```

### Quality Gate and Maven

`waitForQualityGate()` is a Jenkins pipeline step, not a Maven phase.

```text
Maven
   └── validate / compile / test / package / verify

Jenkins
   └── waitForQualityGate()

SonarQube
   └── Quality Gate evaluation
```

### `mvn install` vs `mvn deploy`

```text
mvn install
    ↓
~/.m2/repository
```

`mvn install` places the artifact in the local Maven repository.

```text
mvn deploy
    ↓
Remote Artifact Repository
```

`mvn deploy` publishes the artifact to a remote repository such as Nexus or Artifactory.

### Common Combined Command

```bash
mvn verify sonar:sonar
```

The `verify` phase performs the Maven lifecycle work up to verification before the SonarQube goal runs.

---

# 5. Python Flask CI Pipeline

| Stage | CI Step | Recommended Tool / Command | Purpose |
|:---:|---|---|---|
| **1** | Checkout | `git clone` | Fetch source code |
| **2** | Secret Scanning | `gitleaks detect .` | Detect secrets |
| **3** | Install Dependencies | `poetry install` | Install dependencies |
| **4** | Code Formatting | `black --check .` | Validate formatting |
| **5** | Linting | `pylint .` | Static code analysis |
| **6** | Syntax Validation | `python -m py_compile *.py` | Validate Python syntax |
| **7** | Dependency Scan / SCA | `pip-audit` | Detect vulnerable packages |
| **8** | Unit Testing | `pytest` | Execute tests |
| **9** | Code Coverage | `pytest --cov=. --cov-report=xml` | Generate coverage report |
| **10** | SonarQube Analysis | `sonar-scanner` | Analyze code |
| **11** | Quality Gate | SonarQube | Validate quality |
| **12** | Build Artifact | Python Wheel *(optional)* | Package application |
| **13** | Deploy | Gunicorn + Systemd | Deploy Flask application |
| **14** | DAST | OWASP ZAP | Dynamic security testing |

## Python Dependency Installation

With Poetry:

```bash
poetry install
```

With `requirements.txt`:

```bash
pip install -r requirements.txt
```

## Python Tools

### Black

```bash
black --check .
```

Validates Python formatting.

### Pylint

```bash
pylint .
```

Performs Python static analysis and code-quality checks.

### Pytest

```bash
pytest
```

Runs Python tests.

---

# 6. Go Gin CI Pipeline

| Stage | CI Step | Recommended Tool / Command | Purpose |
|:---:|---|---|---|
| **1** | Checkout | `git clone` | Fetch source code |
| **2** | Secret Scanning | `gitleaks detect .` | Detect secrets |
| **3** | Install Dependencies | `go mod download` | Download Go modules |
| **4** | Formatting | `gofmt -l .` | Validate formatting |
| **5** | Static Analysis | `go vet ./...` | Detect suspicious constructs |
| **6** | Linting | `golangci-lint run` | Run multiple Go linters |
| **7** | Dependency Scan / SCA | `govulncheck ./...` | Detect known Go vulnerabilities |
| **8** | Unit Testing | `go test ./...` | Execute tests |
| **9** | Code Coverage | `go test -coverprofile=coverage.out ./...` | Generate coverage |
| **10** | Build / Compilation | `go build -o employee-api .` | Build executable |
| **11** | SonarQube Analysis | `sonar-scanner` | Analyze code |
| **12** | Quality Gate | SonarQube | Validate quality |
| **13** | Build Artifact | Go binary | Archive executable |
| **14** | Deploy | Systemd | Deploy application |
| **15** | DAST | OWASP ZAP | Dynamic security testing |

## Go Tooling

### Dependency Download

```bash
go mod download
```

### Formatting

```bash
gofmt -l .
```

### Static Analysis

```bash
go vet ./...
```

### Linting

```bash
golangci-lint run
```

### Vulnerability Check

```bash
govulncheck ./...
```

### Build

```bash
go build -o employee-api .
```

The source notes emphasize that Go binaries are statically compiled, making deployment relatively simple and portable.

---

# 7. SAST vs SCA

## SAST

**Static Application Security Testing** analyzes application source code without executing it.

Example:

```java
String query =
    "SELECT * FROM users WHERE id='" + userInput + "'";
```

A static analyzer may identify a possible SQL injection risk.

SAST can identify:

* SQL injection
* XSS
* Hardcoded credentials
* Coding defects
* Code smells
* Duplicate code
* Insecure APIs
* Resource problems

## SCA

**Software Composition Analysis** analyzes third-party dependencies.

Example:

```text
pom.xml
   │
   ├── Spring Boot
   ├── Log4j
   ├── JUnit
   └── Lombok
        │
        ▼
       SCA
        │
        ▼
   Vulnerable Dependency
```

SCA commonly checks:

* Known vulnerabilities
* CVEs
* Outdated packages
* License compliance
* Dependency inventory / SBOM

## Core Difference

```text
Application Source Code
        │
        ▼
       SAST
```

```text
Third-Party Dependencies
        │
        ▼
        SCA
```

| Feature | SAST | SCA |
|---|---|---|
| Primary focus | Application source code | Third-party dependencies |
| Security defects | ✅ | Dependency-focused |
| Code quality | ✅ | ❌ |
| CVE scanning | Not its main purpose | ✅ |
| License scanning | ❌ | ✅ |
| Typical tools | SonarQube, Semgrep, Checkmarx, Fortify | Snyk, Trivy, OWASP Dependency-Check, Black Duck, Mend |

---

# 8. DevSecOps CI Checks Classification

Not every CI check is SAST or SCA.

| Check / Activity | Category | What It Checks | Common Tools |
|---|---|---|---|
| Syntax Validation | Build Validation | Syntax / compilation errors | `javac`, `go build`, `python -m py_compile`, `tsc` |
| Linting | Static Code Quality | Style, standards, best practices | ESLint, Pylint, Flake8, Checkstyle, GolangCI-Lint |
| Code Quality Analysis | SAST / Static Analysis | Smells, duplication, maintainability | SonarQube, PMD, SpotBugs |
| Static Security Analysis | SAST | SQLi, XSS, insecure APIs, hardcoded credentials | SonarQube, Checkmarx, Fortify, Semgrep |
| Dependency Scanning | SCA | Vulnerable third-party libraries | OWASP Dependency-Check, Snyk, Trivy, Grype |
| License Scanning | SCA | Open-source license compliance | Black Duck, Mend, FOSSA, Trivy, Snyk |
| SBOM Generation | SCA | Dependency inventory | Syft, CycloneDX, SPDX |
| Secret Scanning | Security | Passwords, API keys, tokens, certificates | Gitleaks, TruffleHog, GitGuardian |
| Unit Testing | Testing | Functional correctness | JUnit, pytest, Jest, Go Test |
| Code Coverage | Testing | Percentage of tested code | JaCoCo, Coverage.py, Istanbul |
| Build / Compilation | Build | Executable/artifact generation | Maven, Gradle, Go Build, npm |
| Container Image Scanning | Container Security | Image/OS/package vulnerabilities | Trivy, Grype, Docker Scout, Clair |
| IaC Scanning | IaC Security | Terraform/Kubernetes configuration | Checkov, tfsec, Terrascan, KICS |
| DAST | Dynamic Security Testing | Runtime application vulnerabilities | OWASP ZAP, Burp Suite |
| Performance Testing | Performance | Load, stress, scalability | JMeter, Gatling, k6 |
| Infrastructure Compliance | Compliance | Cloud security/configuration compliance | AWS Config, Prowler, ScoutSuite |

---

# 9. SonarQube in CI

SonarQube provides centralized code-quality and security analysis.

A simplified flow is:

```text
Source Code
      │
      ├─────────────────┐
      │                 │
      ▼                 ▼
SonarQube Scanner   Coverage Report
      │                 │
      └────────┬────────┘
               ▼
          SonarQube
               │
       ┌───────┼────────┐
       ▼       ▼        ▼
     Bugs   Smells   Duplication
       │       │        │
       └───────┼────────┘
               ▼
         Quality Gate
```

Typical results include:

* Bugs
* Code smells
* Code duplication
* Maintainability
* Reliability
* Security analysis
* Test coverage

In a Jenkins pipeline:

```text
SonarQube Analysis
        ↓
waitForQualityGate()
        ↓
PASS → Continue
FAIL → Stop / Fix
```

`waitForQualityGate()` is a Jenkins step, not a Maven phase.

---

# 10. Java Maven Lifecycle in CI

Maven's default lifecycle is cumulative.

```text
validate
   ↓
compile
   ↓
test
   ↓
package
   ↓
verify
   ↓
install
   ↓
deploy
```

| Command | Purpose |
|---|---|
| `mvn validate` | Validate project configuration |
| `mvn compile` | Compile application code |
| `mvn test` | Run tests |
| `mvn package` | Package the application |
| `mvn verify` | Run earlier phases and verification |
| `mvn install` | Install artifact in `~/.m2/repository` |
| `mvn deploy` | Publish artifact to remote repository |

### `mvn package` and Tests

Normally:

```bash
mvn package
```

includes the test phase.

A pipeline may use:

```bash
mvn test
mvn package -DskipTests
```

to avoid running the same tests twice.

### `mvn verify` and Quality Gates

`mvn verify` is a Maven lifecycle phase.

SonarQube's Quality Gate is a separate quality evaluation.

```text
Maven
   └── verify

SonarQube
   └── Quality Gate

Jenkins
   └── waitForQualityGate()
```

### `mvn install` vs `mvn deploy`

```text
mvn install
    ↓
~/.m2/repository
```

```text
mvn deploy
    ↓
Remote Repository
```

### `mvn dependency:resolve`

Optional early dependency-resolution step:

```bash
mvn dependency:resolve
```

It can fail the pipeline early when dependency resolution is unavailable.

### `mvn -U clean package`

```bash
mvn -U clean package
```

Meaning:

```text
-U       → Force checks for updated SNAPSHOT dependencies / metadata
clean    → Remove previous build output
package  → Build, test, and package
```

`-U` is generally not necessary for every build because it can increase remote repository checks and build time.

### JaCoCo

```text
mvn test
    │
    ▼
JaCoCo
    │
    ▼
Coverage Data
    │
    ▼
mvn verify
    │
    ▼
Coverage Reports
```

A common combined analysis command is:

```bash
mvn verify sonar:sonar
```

---

# 11. `package.json`, `package-lock.json`, `npm install`, and `npm ci`

## `package.json`

Contains project metadata, scripts, and dependency requirements.

Example:

```json
{
  "dependencies": {
    "axios": "^1.8.2"
  }
}
```

It describes what the project requires.

## `package-lock.json`

Records the resolved dependency tree.

Example:

```text
Project
   │
   ▼
express
   │
   ├── dependency A
   ├── dependency B
   └── dependency C
```

It records exact resolved versions, including transitive dependencies, so CI installations can be reproduced consistently.

### Comparison

| Feature | `package.json` | `package-lock.json` |
|---|---|---|
| Main purpose | Declare project/dependency requirements | Record resolved dependency tree |
| Version specification | Can be flexible | Exact resolved versions |
| Transitive dependency detail | Limited | Detailed |
| CI reproducibility | Helps define requirements | Provides exact resolution |

---

## `npm install`

```bash
npm install
```

It:

* Reads project dependencies
* Resolves packages
* Installs dependencies
* Can update `package-lock.json` when dependency resolution changes

Typical use:

* Local development
* Adding dependencies
* Updating dependencies

## `npm ci`

```bash
npm ci
```

It:

* Requires `package-lock.json`
* Removes existing `node_modules`
* Installs the locked dependency tree
* Does not update `package-lock.json`

Typical use:

* Jenkins
* GitHub Actions
* GitLab CI
* Reproducible CI/CD builds

### Comparison

| Feature | `npm install` | `npm ci` |
|---|---|---|
| Reads `package.json` | ✅ | ✅ |
| Reads lock file | ✅ | ✅ Required |
| Can create/update lock file | ✅ | ❌ |
| Removes existing `node_modules` | ❌ | ✅ |
| Reproducible CI install | Less strict | ✅ |
| Typical use | Development | CI/CD |

### Why `package-lock.json` Matters

`package.json` might specify:

```text
Express 4.18.x
```

but Express itself has dependencies.

```text
Your Project
      │
      ▼
Express
      │
      ├── dependency A
      ├── dependency B
      └── dependency C
```

The lock file captures the resolved dependency tree, including transitive dependencies.

Mental model:

```text
package.json
    ↓
"Shopping List"

package-lock.json
    ↓
"Exact Shopping Bill"
```

---

# 12. Build, Artifact, Publish, and Deploy

These are different concepts.

## Build

Transforms source code into application output.

Examples:

```text
React → build/
Java → JAR/WAR
Go → Binary
Python → Wheel
```

## Artifact

The output produced by the build.

Examples:

```text
employee-api.jar
employee-api
frontend build/
python-package.whl
```

## Publish Artifact

Uploads the artifact to a repository.

```text
Build Artifact
      │
      ▼
Nexus / Artifactory
```

## Deploy

Installs/runs the artifact in a target environment.

```text
Artifact
   │
   ▼
Server / VM / Kubernetes
   │
   ▼
Running Application
```

Simplified lifecycle:

```text
Source
  ↓
Build
  ↓
Artifact
  ↓
Publish
  ↓
Deploy
```

---

# 13. DAST in CI/CD

**DAST** stands for **Dynamic Application Security Testing**.

DAST tests the application while it is running.

Therefore:

```text
Source Code
    │
    ▼
Build
    │
    ▼
Artifact
    │
    ▼
Deploy Application
    │
    ▼
Running Application
    │
    ▼
DAST
```

Example:

```bash
zap-baseline.py -t http://<app-url>
```

DAST tools can identify runtime security issues that cannot be fully evaluated from source code alone.

---

# 14. Frequently Asked Interview Questions

| Question | Answer |
|---|---|
| **What is a CI pipeline?** | An automated sequence that validates source-code changes through checks such as dependency installation, static analysis, testing, building, and quality validation. |
| **Why run Secret Scanning?** | To detect hardcoded credentials and sensitive information before the code progresses through the pipeline. |
| **What is SAST?** | Static Application Security Testing analyzes application source code without executing it. |
| **What is SCA?** | Software Composition Analysis scans third-party dependencies for vulnerabilities, licenses, and dependency inventory. |
| **What is the difference between SAST and SCA?** | SAST scans application source code; SCA scans third-party dependencies. |
| **What is the purpose of linting?** | To identify style violations, coding-standard issues, and other source-code quality problems. |
| **What is code coverage?** | A measure of how much code is exercised by tests. |
| **Why use `mvn package -DskipTests` after `mvn test`?** | To package the application without rerunning unit tests that already passed. |
| **Does `mvn package` normally run tests?** | Yes, reaching the package phase normally includes the test phase. |
| **What does `mvn verify` do?** | It runs the preceding Maven lifecycle phases and then performs verification configured for the project. |
| **What is the purpose of `mvn install`?** | It installs the artifact into the local Maven repository. |
| **What is the purpose of `mvn deploy`?** | It publishes the artifact to a remote Maven repository. |
| **What is a Quality Gate?** | A SonarQube evaluation that determines whether defined quality conditions pass or fail. |
| **Is Quality Gate a Maven phase?** | No. It is a SonarQube evaluation; Jenkins can wait for the result with `waitForQualityGate()`. |
| **What is JaCoCo?** | A Java code-coverage tool. |
| **What is DAST?** | Dynamic security testing against a running application. |
| **Why is DAST performed after deployment?** | The application must be running for DAST to test its runtime behavior. |
| **Why use `npm ci` in Jenkins?** | It performs a clean, reproducible installation from `package-lock.json`. |
| **Does `npm ci` update `package-lock.json`?** | No. It requires an existing lock file and does not update it. |
| **Why is `package-lock.json` important?** | It records the resolved dependency tree so installations can be reproduced consistently. |
| **What is `gofmt`?** | The official Go formatter. |
| **What is `go vet`?** | A Go tool that identifies suspicious constructs and potential coding mistakes. |
| **What is `golangci-lint`?** | A tool that runs multiple Go linters together. |
| **What is `govulncheck`?** | A Go vulnerability-checking tool for known vulnerabilities affecting Go code and dependencies. |
| **What is `py_compile` used for?** | It validates Python syntax and compiles source to bytecode. |
| **What is the role of `black`?** | Python code formatting. |
| **What is the role of `pylint`?** | Python static analysis and code-quality checking. |
| **What is `npm audit`?** | A mechanism for identifying known vulnerabilities in npm dependencies. |
| **What is Gitleaks?** | A secret-scanning tool used to detect credentials and sensitive information in repositories. |

---

# 15. One-Line Interview Answers

| Concept | One-Line Answer |
|---|---|
| **CI** | Automatically validate code changes before they are integrated or released. |
| **Secret Scanning** | Detect hardcoded credentials and sensitive information. |
| **SAST** | Scan application source code without executing it. |
| **SCA** | Scan third-party dependencies for vulnerabilities and licenses. |
| **Linting** | Detect coding-standard and source-quality issues. |
| **Unit Testing** | Verify application behavior at the unit level. |
| **Coverage** | Measure how much code is exercised by tests. |
| **SonarQube** | Centralized code-quality and security analysis. |
| **Quality Gate** | Pass/fail conditions that determine whether code meets quality standards. |
| **Artifact** | Build output that can be stored or deployed. |
| **Publish** | Upload an artifact to a repository. |
| **Deploy** | Install/run an artifact in a target environment. |
| **DAST** | Test a running application for runtime security issues. |
| **`npm install`** | Resolve and install dependencies and may update the lock file. |
| **`npm ci`** | Cleanly install the dependency tree recorded in the lock file. |
| **`mvn validate`** | Validate Maven project configuration. |
| **`mvn test`** | Compile and execute unit tests. |
| **`mvn package`** | Build and package the application. |
| **`mvn verify`** | Run lifecycle phases through verification. |
| **`mvn install`** | Install the artifact into the local Maven repository. |
| **`mvn deploy`** | Publish the artifact to a remote Maven repository. |
| **`gofmt`** | Format Go source code. |
| **`go vet`** | Find suspicious Go constructs. |
| **`py_compile`** | Validate and compile Python source into bytecode. |
