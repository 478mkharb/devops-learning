# SonarQube Interview Notes

---

# 1. Types of Sonar Properties Files and Their Use Cases

SonarQube uses a **properties file** to configure analysis. The most commonly used properties files are:

| File                              | Location                     | Purpose                                    | Used By                  |
| --------------------------------- | ---------------------------- | ------------------------------------------ | ------------------------ |
| `sonar-project.properties`        | Project root                 | Defines project-specific analysis settings | Developers / CI Pipeline |
| `sonar.properties`                | `<SONARQUBE_HOME>/conf/`     | Configures the SonarQube server            | SonarQube Administrator  |
| `wrapper.conf` *(older versions)* | `<SONARQUBE_HOME>/conf/`     | Java service wrapper configuration         | Legacy installations     |
| `sonar-scanner.properties`        | `<SONAR_SCANNER_HOME>/conf/` | Default configuration for SonarScanner CLI | Scanner Administrator    |

---

## 1. sonar-project.properties

Project-level configuration.

Example

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

**Use Case**

- Every project has its own configuration.
- Stored inside the Git repository.
- Used during CI/CD analysis.

---

## 2. sonar.properties

Located inside

```text
<SONARQUBE_HOME>/conf/
```

Example

```properties
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube

sonar.jdbc.username=sonar

sonar.jdbc.password=password

sonar.web.port=9000
```

**Use Case**

- Configure SonarQube Server
- Database connection
- Web server port
- Elasticsearch settings
- Authentication

---

## 3. sonar-scanner.properties

Located inside

```text
<SONAR_SCANNER_HOME>/conf/
```

Example

```properties
sonar.host.url=http://localhost:9000

sonar.sourceEncoding=UTF-8
```

**Use Case**

Global scanner configuration.

---

# Interview Question

### Which properties file is used most frequently?

**Answer**

> **sonar-project.properties**, because every project requires its own analysis configuration.

---

# 2. Types of Sonar Scanners

SonarQube provides different scanners depending on the build tool or project type.

| Scanner | Purpose | Best For |
|----------|---------|----------|
| **SonarScanner CLI** | Generic scanner | Any language |
| **SonarScanner for Maven** | Maven integration | Java Maven Projects |
| **SonarScanner for Gradle** | Gradle integration | Java/Kotlin Projects |
| **SonarScanner for .NET (MSBuild)** | MSBuild integration | C#, VB.NET |
| **SonarScanner for NPM (CLI Wrapper)** | Node.js integration | JavaScript/TypeScript |
| **Jenkins SonarScanner Plugin** | CI/CD integration | Jenkins Pipelines |
| **Azure DevOps Extension** | Azure DevOps integration | Azure Pipelines |
| **GitHub Actions Integration** | GitHub CI | GitHub Actions |

---

## Most Common Scanner

| Scanner | Interview Importance |
|----------|---------------------|
| SonarScanner CLI | ⭐⭐⭐⭐⭐ |
| SonarScanner for Maven | ⭐⭐⭐⭐⭐ |
| SonarScanner for Gradle | ⭐⭐⭐⭐ |
| SonarScanner for .NET | ⭐⭐⭐ |
| Jenkins Integration | ⭐⭐⭐⭐⭐ |

---

# Interview Question

### Which SonarScanner should I use?

| Project | Scanner |
|----------|----------|
| Java + Maven | SonarScanner for Maven |
| Java + Gradle | SonarScanner for Gradle |
| C# | SonarScanner for .NET |
| Go | SonarScanner CLI |
| Python | SonarScanner CLI |
| React | SonarScanner CLI |
| Node.js | SonarScanner CLI |

---

# 3. Where is SonarScanner Located?

It depends on how it was installed.

## Linux (Manual Installation)

```text
/opt/sonar-scanner/
```

or

```text
/usr/local/sonar-scanner/
```

Important directories

```text
sonar-scanner/

├── bin/

├── conf/

├── jre/

└── lib/
```

Executable

```text
/opt/sonar-scanner/bin/sonar-scanner
```

Check location

```bash
which sonar-scanner
```

or

```bash
find / -name sonar-scanner
```

---

## Jenkins

Managed through

```text
Manage Jenkins

↓

Global Tool Configuration

↓

SonarScanner
```

Usually installed under

```text
/var/lib/jenkins/tools/
```

---

# 4. What is the H2 Database in SonarQube?

## Definition

H2 is a lightweight, embedded Java database that comes bundled with SonarQube.

It is intended **only for evaluation, learning, or testing**.

---

## Characteristics

| Feature | H2 |
|----------|----|
| Embedded | ✅ |
| Lightweight | ✅ |
| Production Ready | ❌ |
| External Installation | Not Required |
| Performance | Low |
| HA Support | ❌ |

---

## Why is H2 Not Recommended for Production?

- Data loss risk
- No clustering
- No high availability
- Limited scalability
- Poor concurrent performance

---

## Production Databases Supported

| Database | Recommended |
|----------|-------------|
| PostgreSQL | ✅ Yes |
| Oracle | Enterprise |
| Microsoft SQL Server | Enterprise |

> **Note:** Modern SonarQube versions support **PostgreSQL** as the primary production database. H2 is only for evaluation.

---

# Interview Question

### Why shouldn't H2 be used in Production?

**Answer**

Because H2 is an embedded database intended for testing and evaluation. It does not provide scalability, high availability, or production-grade reliability.

---

# 5. SonarQube Server Directory Structure

Typical installation

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

---

## Directory Explanation

| Directory | Purpose |
|------------|----------|
| **bin/** | Startup and shutdown scripts |
| **conf/** | Server configuration (`sonar.properties`) |
| **data/** | Elasticsearch indexes and internal data |
| **extensions/** | Plugins installed in SonarQube |
| **lib/** | SonarQube Java libraries |
| **logs/** | Application logs |
| **temp/** | Temporary runtime files |
| **web/** | Web application resources |

---

## Detailed Structure

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

# Important Log Files

| Log File | Purpose |
|-----------|----------|
| **sonar.log** | Main application log |
| **web.log** | Web server log |
| **ce.log** | Compute Engine log |
| **es.log** | Elasticsearch log |

---
# 6. SonarQube Default Ports

| Component | Default Port | Protocol | Purpose |
|-----------|-------------:|----------|---------|
| **SonarQube Web Server** | **9000** | HTTP | Access the SonarQube Web UI and REST APIs. |
| **SonarScanner → SonarQube** | **9000** | HTTP/HTTPS | Used by SonarScanner to send analysis reports to the SonarQube server. |

---

## Access SonarQube

```text
http://<server-ip>:9000
```

Example:

```text
http://localhost:9000
```

or

```text
http://192.168.1.100:9000
```

---

## Can the Port Be Changed?

**Yes.**

The default port **9000** can be changed in the `sonar.properties` file.

Location:

```text
<SONARQUBE_HOME>/conf/sonar.properties
```

Example:

```properties
sonar.web.port=8080
```

Restart SonarQube after changing the port.

---

## Verify the Port

### Linux

```bash
ss -tulpn | grep 9000
```

or

```bash
netstat -tulpn | grep 9000
```

---

# Frequently Asked Interview Questions

| Question | Answer |
|----------|--------|
| Which properties file is used for project configuration? | `sonar-project.properties` |
| Which properties file configures the SonarQube server? | `sonar.properties` |
| Which properties file configures the SonarScanner? | `sonar-scanner.properties` |
| How many major SonarScanners are there? | CLI, Maven, Gradle, .NET, Jenkins Integration, Azure DevOps, GitHub Actions (depending on integration method). |
| Which scanner is used for Go and Python? | SonarScanner CLI |
| Where is SonarScanner installed? | Typically `/opt/sonar-scanner/` or `/usr/local/sonar-scanner/` |
| What is H2 Database? | An embedded database used only for testing and evaluation. |
| Which database is recommended for production? | PostgreSQL |
| Where are plugins stored? | `extensions/plugins/` |
| Where are logs stored? | `logs/` |
| Which log shows Elasticsearch issues? | `es.log` |
| Which directory contains the server configuration? | `conf/` |
| Which directory stores installed plugins? | `extensions/` |
| What is the default port of SonarQube? | **9000** |
| Which file is used to change the SonarQube port? | `conf/sonar.properties` |
| Which property changes the port? | `sonar.web.port` |
| Which port does SonarScanner use to communicate with SonarQube? | By default, **9000** (or whatever port the SonarQube server is configured to use). |

---

# One-Line Interview Answers

| Topic | Answer |
|-------|--------|
| **sonar-project.properties** | Project-specific SonarQube analysis configuration file. |
| **sonar.properties** | Server configuration file for SonarQube. |
| **sonar-scanner.properties** | Global configuration file for the SonarScanner CLI. |
| **SonarScanner CLI** | Generic scanner used for most languages such as Go, Python, JavaScript, and React. |
| **H2 Database** | Embedded database used only for evaluation and testing, not for production. |
| **Production Database** | PostgreSQL is the recommended production database for SonarQube. |
| **extensions/** | Stores plugins and language analyzers. |
| **logs/** | Contains SonarQube, Web, Compute Engine, and Elasticsearch logs. |
>**The default port of SonarQube is 9000, configured using the `sonar.web.port` property in the `conf/sonar.properties` file.**
