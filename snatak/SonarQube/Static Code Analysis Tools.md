# Static Code Analysis Tools Used in OT-Microservices

| Microservice | Language | Static Code Analysis Tool | Purpose |
|--------------|----------|---------------------------|---------|
| **Employee API** | Go | **golangci-lint** | Detect Go coding issues, unused code, style violations, and common bugs before build. |
| **Attendance API** | Python | **Flake8** (or **Pylint**) | Detect syntax errors, PEP 8 violations, code quality issues, and maintainability problems. |
| **Salary API** | Java (Spring Boot) | **SpotBugs**, **Checkstyle**, **PMD** + **SonarQube** | Detect Java bugs, coding standard violations, duplicate code, security issues, and code smells. |
| **Frontend** | React (JavaScript) | **ESLint** | Detect JavaScript/React syntax errors, coding standard violations, and best practice issues. |
| **Notification Worker** | Python | **Flake8** | Detect Python syntax errors, PEP 8 violations, and code quality issues. |
| **All Microservices** | All Languages | **SonarQube** | Perform centralized static code analysis, detect bugs, vulnerabilities, security hotspots, code smells, duplicate code, measure code coverage, and enforce Quality Gates. |

---

## Interview Questions

| Question | Answer |
|----------|--------|
| Which Static Code Analysis tool is used for the Go Employee API? | **golangci-lint** |
| Which Static Code Analysis tool is used for the Python Attendance API? | **Flake8** (or **Pylint**) |
| Which Static Code Analysis tools are used for the Java Salary API? | **SpotBugs**, **Checkstyle**, **PMD**, and **SonarQube** |
| Which Static Code Analysis tool is used for the React Frontend? | **ESLint** |
| Which Static Code Analysis tool is used for the Notification Worker? | **Flake8** |
| Which tool provides centralized static code analysis across all microservices? | **SonarQube** |
| Why are both language-specific tools and SonarQube used? | Language-specific tools provide fast, language-focused feedback, while SonarQube performs comprehensive static analysis, security checks, code quality assessment, and Quality Gate enforcement across all projects. |

---

## One-Line Interview Answer

> **OT-Microservices uses language-specific static analysis tools such as golangci-lint (Go), Flake8/Pylint (Python), SpotBugs/Checkstyle/PMD (Java), and ESLint (React), while SonarQube acts as the centralized Static Application Security Testing (SAST) platform to analyze code quality, security vulnerabilities, code smells, and enforce Quality Gates across all microservices.**
