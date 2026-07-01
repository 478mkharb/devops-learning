# OT-Microservices CI Pipeline

## Table 1 – CI Pipeline Stages & Purpose

| Stage | Purpose | Open Source Tools |
|:-----:|---------|-------------------|
| **1. Checkout** | Download source code from SCM | Git, JGit |
| **2. Secret Scanning** | Detect hardcoded passwords, API keys and tokens | GitLeaks, TruffleHog, GitGuardian CLI |
| **3. Code Formatting** | Ensure consistent coding style | gofmt, Black, Spotless, Prettier |
| **4. Syntax Validation** | Validate syntax before compilation | Language Compilers / Interpreters |
| **5. Dependency & License Scan (SCA)** | Scan dependencies, CVEs, licenses and misconfigurations | Trivy, OWASP Dependency-Check, Grype, Snyk OSS |
| **6. Install Dependencies** | Download project dependencies | Go Modules, Poetry, Maven, npm |
| **7. Linting** | Detect coding issues and best-practice violations | golangci-lint, Pylint, Checkstyle, ESLint |
| **8. Build / Compile** | Generate executable or build output | Go, Maven, npm |
| **9. Unit Testing** | Execute unit tests | Go Test, PyTest, JUnit, Jest |
| **10. Code Coverage** | Measure unit test coverage | JaCoCo, Coverage.py, go cover, Jest Coverage |
| **11. SonarQube Analysis** | Perform SAST, Bug Analysis, Code Smells, Duplication and Maintainability Analysis | SonarQube CE |
| **12. Quality Gate** | Fail pipeline if quality thresholds are not met | SonarQube Quality Gate |
| **13. Build Artifact** | Package deployable output | Binary, JAR, React Build |
| **14. Deploy to Test** | Deploy application to Test/Staging | Systemd, Gunicorn, Spring Boot, Nginx |
| **15. DAST** | Scan running application for runtime vulnerabilities | OWASP ZAP, Nikto, Wapiti, Arachni |

---

## Table 2 – Commands Used in OT-Microservices

| Stage | Employee API (Go) | Attendance API (Python) | Salary API (Java) | Frontend (React) | Notification API (Python) |
|:-----:|-------------------|--------------------------|-------------------|------------------|---------------------------|
| **1. Checkout** | `git clone` | `git clone` | `git clone` | `git clone` | `git clone` |
| **2. Secret Scanning** | `gitleaks detect .` | `gitleaks detect .` | `gitleaks detect .` | `gitleaks detect .` | `gitleaks detect .` |
| **3. Code Formatting** | `gofmt -l .` | `black --check .` | `mvn spotless:check` | `prettier --check .` | `black --check .` |
| **4. Syntax Validation** | `go vet ./...` | `python -m py_compile app.py` | `mvn validate` | `npm run build` | `python -m py_compile notification_api.py` |
| **5. Dependency & License Scan (SCA)** | `trivy fs .` | `trivy fs .` | `trivy fs .` | `trivy fs .` | `trivy fs .` |
| **6. Install Dependencies** | `go mod download` | `poetry install` | `mvn dependency:resolve` | `npm install` | `poetry install` |
| **7. Linting** | `golangci-lint run` | `pylint .` | `mvn checkstyle:check` | `eslint .` | `pylint .` |
| **8. Build / Compile** | `go build` | *No Compilation* | `mvn clean compile` | `npm run build` | *No Compilation* |
| **9. Unit Testing** | `go test ./...` | `pytest` | `mvn test` | `npm test` | `pytest` |
| **10. Code Coverage** | `go test -cover` | `pytest --cov` | JaCoCo | `jest --coverage` | `pytest --cov` |
| **11. SonarQube Analysis** | `sonar-scanner` | `sonar-scanner` | `mvn sonar:sonar` | `sonar-scanner` | `sonar-scanner` |
| **12. Quality Gate** | SonarQube | SonarQube | SonarQube | SonarQube | SonarQube |
| **13. Build Artifact** | Binary | Python Package *(Optional)* | JAR | `build/` Folder | Python Package *(Optional)* |
| **14. Deploy to Test** | Systemd | Gunicorn | Spring Boot | Nginx | Gunicorn |
| **15. DAST** | OWASP ZAP | OWASP ZAP | OWASP ZAP | OWASP ZAP | OWASP ZAP |

---

# CI Pipeline Flow

```mermaid
flowchart TD

A["Developer Push"] --> B["Checkout"]
B --> C["Secret Scan"]
C --> D["Formatting"]
D --> E["Syntax Validation"]
E --> F["SCA"]
F --> G["Install Dependencies"]
G --> H["Linting"]
H --> I["Build"]
I --> J["Unit Tests"]
J --> K["Coverage"]
K --> L["SonarQube"]

L --> L1["SAST"]
L --> L2["Bug Analysis"]
L --> L3["Code Smells"]
L --> L4["Duplication"]
L --> L5["Maintainability"]
L --> L6["Reliability"]

L1 --> M["Quality Gate"]
L2 --> M
L3 --> M
L4 --> M
L5 --> M
L6 --> M

M --> N["Artifact"]
N --> O["Deploy"]
O --> P["DAST"]
```
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
