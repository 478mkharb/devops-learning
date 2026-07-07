# CI Pipeline Stages

This document describes the recommended Continuous Integration (CI) pipeline for each technology stack used in the OT-Microservices project.

---

# 1. React (Frontend) CI Pipeline

| Stage | CI Step | Recommended Tool / Command | Purpose |
|:----:|--------------------------|--------------------------------------------|--------------------------------------------|
| **1** | Checkout | `git clone` | Fetch source code from SCM |
| **2** | Secret Scanning | `gitleaks detect .` | Detect hardcoded secrets like API keys and passwords |
| **3** | Install Dependencies | `npm install` `npm ci` | Install dependencies from `package-lock.json` |
| **4** | Code Formatting | `npx prettier --check .` | Validate code formatting |
| **5** | Linting | `npm run lint` | Perform static code analysis using ESLint |
| **6** | Dependency Scan (SCA) | `npm audit --audit-level=high` | Detect vulnerable npm packages |
| **7** | Unit Testing | `CI=true npm test -- --watchAll=false` | Execute Jest unit tests |
| **8** | Code Coverage | `npm test -- --coverage --watchAll=false` | Generate code coverage report |
| **9** | Build/Code Compilation | `CI=false npm run build` | Generate optimized production build |
| **10** | SonarQube Analysis | `sonar-scanner` | Analyze code quality and security |
| **11** | Quality Gate | SonarQube | Validate project quality before deployment |
| **12** | Build Artifact | `build/` | Archive production build |
| **13** | Deploy | Nginx | Deploy static React application |
| **14** | DAST | OWASP ZAP | Perform Dynamic Application Security Testing |

### Notes

- `npm ci` is recommended over `npm install` in CI/CD because it performs a **clean, reproducible installation** using `package-lock.json`.
- `npm ci` removes the existing `node_modules` directory before installing dependencies.
- `CI=true` prevents React tests from entering watch mode inside Jenkins.
- `CI=false` avoids build failures caused by React warnings during production builds.

---

# 2. Java CI Pipeline

| Sr. No. | Stage | Recommended Command | Purpose |
|:------:|--------------------------|--------------------------------------------|--------------------------------------------|
| **1** | Checkout | `git clone` | Fetch source code from the Git repository. |
| **2** | Secret Scanning | `gitleaks detect .` | Detect hardcoded secrets (passwords, API keys, tokens) before the build starts. |
| **3** | Validate | `mvn validate` | Validate project structure and `pom.xml`. Ensures the project configuration is correct before compilation. |
| **4** | Dependency Resolution | `mvn dependency:resolve` | **Maven Dependency Plugin** downloads all project dependencies into the local Maven repository (`~/.m2/repository`). |
| **5** | Code Formatting | `mvn spotless:check` | **Spotless Maven Plugin** verifies that the source code follows the configured formatting rules. |
| **6** | Linting | `mvn checkstyle:check` | **Maven Checkstyle Plugin** validates Java coding standards and detects style violations. |
| **7** | Dependency Scan (SCA) | `mvn org.owasp:dependency-check:check` | **OWASP Dependency-Check Plugin** scans third-party libraries for known CVEs and vulnerable dependencies. |
| **8** | Code & Test Compilation | `mvn compile`<br>`mvn test-compile` | **Maven Compiler Plugin** compiles application source code into `target/classes` and test source code into `target/test-classes`. |
| **9** | Unit Testing | `mvn test` | **Maven Surefire Plugin** executes JUnit/TestNG unit tests. **JaCoCo Maven Plugin** attaches its Java Agent to record code coverage and generates `jacoco.exec`. |
| **10** | Package | `mvn package -DskipTests` | **Maven JAR Plugin / Spring Boot Maven Plugin** packages compiled classes into an executable JAR (or WAR). |
| **11** | Verify | `mvn verify` | **JaCoCo Maven Plugin** generates HTML/XML coverage reports from `jacoco.exec` and executes additional verification tasks configured for the project. |
| **12** | SonarQube Analysis | `mvn sonar:sonar` | **Sonar Maven Plugin** uploads source code, test results, and JaCoCo coverage reports to SonarQube for static code analysis. |
| **13** | Quality Gate | `waitForQualityGate()` | Jenkins waits for SonarQube to evaluate Quality Gate conditions (coverage, bugs, vulnerabilities, code smells, etc.). |
| **14** | Install | `mvn install` | **Maven Install Plugin** installs the packaged artifact into the local Maven repository (`~/.m2/repository`) for use by other local projects. |
| **15** | Publish Artifact | `mvn deploy` | **Maven Deploy Plugin** uploads the packaged artifact to a remote repository such as Nexus or Artifactory for sharing and deployment. |
| **16** | Deploy Application | `java -jar app.jar` *(or Ansible/Systemd/Kubernetes)* | Deploy the application to the target environment (VM, container, or Kubernetes cluster). |
| **17** | DAST | `zap-baseline.py -t http://<app-url>` | **OWASP ZAP** performs Dynamic Application Security Testing against the running application to detect runtime vulnerabilities. |

## 📝 Notes

- The CI/CD stages shown above represent a **logical enterprise pipeline**. Depending on the project and organization, some stages may be combined, skipped, or reordered.
- **Maven lifecycle phases are cumulative.** Executing a later phase automatically runs all preceding phases.
- **`mvn dependency:resolve`** is optional because Maven automatically downloads missing dependencies during the build. It is mainly used to fail fast if dependency resolution fails.
- **Always execute unit tests before packaging** the application.
- Running `mvn package` executes unit tests by default.
- In CI/CD, a common optimization is:

  ```bash
  mvn test
  mvn package -DskipTests
  ```

  This prevents unit tests from running twice while ensuring they have already passed.

- **JaCoCo** is the de facto Java code coverage tool. During `mvn test`, it records execution into `jacoco.exec`; during `mvn verify`, it generates HTML/XML coverage reports, which are then consumed by SonarQube.
- **Quality Gate** is not a Maven phase. It is a Jenkins stage that waits for SonarQube to validate quality conditions (coverage, bugs, vulnerabilities, code smells, etc.) before allowing the pipeline to continue.
- **`mvn install`** installs the artifact into the **local Maven repository** (`~/.m2/repository`), whereas **`mvn deploy`** publishes it to a **remote artifact repository** (e.g., Nexus or Artifactory).
- **DAST** is always performed **after deployment** because it tests the running application for runtime security vulnerabilities.
- In production CI/CD pipelines, teams often execute **`mvn verify sonar:sonar`** instead of individual Maven phases because it automatically performs all required lifecycle phases up to `verify` before running SonarQube analysis.

---

# 3. Python (Flask) CI Pipeline

| Stage | CI Step | Recommended Tool / Command | Purpose |
|:----:|--------------------------|--------------------------------------------|--------------------------------------------|
| **1** | Checkout | `git clone` | Fetch source code |
| **2** | Secret Scanning | `gitleaks detect .` | Detect secrets in repository |
| **3** | Install Dependencies | `poetry install` | Install project dependencies |
| **4** | Code Formatting | `black --check .` | Validate formatting |
| **5** | Linting | `pylint .` | Static code analysis |
| **6** | Syntax Validation + Bytecode Code Compliation | `python -m py_compile *.py` | Validate Python syntax |
| **7** | Dependency Scan (SCA) | `pip-audit` | Detect vulnerable Python packages |
| **8** | Unit Testing | `pytest` | Execute unit tests |
| **9** | Code Coverage | `pytest --cov=. --cov-report=xml` | Generate coverage report |
| **10** | SonarQube Analysis | `sonar-scanner` | Analyze code quality |
| **11** | Quality Gate | SonarQube | Validate quality gate |
| **12** | Build Artifact | Python Wheel *(Optional)* | Package Python application |
| **13** | Deploy | Gunicorn + Systemd | Deploy Flask application |
| **14** | DAST | OWASP ZAP | Dynamic security testing |

### Notes

- If the project uses **Poetry**, use `poetry install` for dependency management.
- If Poetry is not used, install dependencies with:
  ```bash
  pip install -r requirements.txt
  ```
- `black` is the standard formatter for Python projects.
- `ruff` is a modern, high-performance linter and is becoming popular, but `pylint` remains widely accepted in enterprise projects.
- `pytest` is the de facto standard framework for Python testing.

---

# 4. Go (Gin) CI Pipeline

| Stage | CI Step | Recommended Tool / Command | Purpose |
|:----:|--------------------------|--------------------------------------------|--------------------------------------------|
| **1** | Checkout | `git clone` | Fetch source code |
| **2** | Secret Scanning | `gitleaks detect .` | Detect hardcoded secrets |
| **3** | Install Dependencies | `go mod download` | Download Go modules |
| **4** | Code Formatting | `gofmt -l .` | Validate Go code formatting |
| **5** | Static Analysis | `go vet ./...` | Detect suspicious constructs |
| **6** | Linting | `golangci-lint run` | Run multiple Go linters |
| **7** | Dependency Scan (SCA) | `govulncheck ./...` | Detect known Go vulnerabilities |
| **8** | Unit Testing | `go test ./...` | Execute Go test cases |
| **9** | Code Coverage | `go test -coverprofile=coverage.out ./...` | Generate coverage report |
| **10** | Build/Code Compilation | `go build -o employee-api .` | Build Go executable |
| **11** | SonarQube Analysis | `sonar-scanner` | Analyze code quality |
| **12** | Quality Gate | SonarQube | Validate quality gate |
| **13** | Build Artifact | Go Binary | Archive executable |
| **14** | Deploy | Systemd | Deploy Go application |
| **15** | DAST | OWASP ZAP | Dynamic security testing |

### Notes

- `go mod download` ensures all dependencies are downloaded before compilation.
- `gofmt` is the official Go formatter and should always be executed before linting.
- `go vet` identifies potential coding mistakes that the compiler may not detect.
- `golangci-lint` combines multiple Go linters into a single command and is the industry-standard linting tool.
- `govulncheck` is maintained by the Go team and is recommended for vulnerability scanning of Go modules.
- Go binaries are statically compiled, making deployment simple and portable.
---

# SonarQube Analysis Breakdown

```mermaid
flowchart LR

A[Source Code]
B[Coverage Report]

A --> C[SonarQube Scanner]
B --> C

C --> D[SAST]
C --> E[Bug Analysis]
C --> F[Code Smells]
C --> G[Code Duplication]
C --> H[Maintainability]
C --> I[Reliability]

D --> J[Quality Gate]
E --> J
F --> J
G --> J
H --> J
I --> J
```
