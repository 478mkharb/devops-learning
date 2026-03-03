# Maven
---

## 🗂️ 1️⃣ `.m2` DIRECTORY 

### 📌 What is `.m2`?

`.m2` is Maven’s **local repository** used to store:

* 📦 Dependencies (JARs)
* 🔌 Maven plugins
* 🧾 Metadata

Default location:

```bash
~/.m2/
```

---

### 🕒 When & How is `.m2` CREATED?

🟢 `.m2` is created **automatically** the **first time any Maven command runs** on a system.

📌 There is **NO dedicated command** to create `.m2`.

#### Commands that trigger `.m2` creation (first run):

```bash
mvn -version
mvn validate
mvn compile
mvn test
mvn package
mvn install
mvn deploy
```

---

### 🧪 What Maven Does Internally (First Run)

1. Checks for `~/.m2`
2. Creates `.m2/repository`
3. Downloads core plugins
4. Downloads project dependencies

---

### 📁 `.m2` Structure

```text
.m2/
 ├── repository/
 │   └── groupId/artifactId/version/*.jar
 └── settings.xml
```

---

### 🎯 Why `.m2` Matters in DevOps

* ⚡ Faster builds (dependency caching)
* 🌐 Enables offline builds
* 📉 Reduces load on Nexus/Maven Central

---

### 🚀 `.m2` in CI/CD

**Problem:** Fresh agents → slow builds

**Solution:** Cache `.m2`

```yaml
# GitHub Actions example
path: ~/.m2
```

📌 Can reduce build time by **60–80%**.

---

### ⚠️ Common `.m2` Issues

* Corrupted dependency → delete specific folder
* SNAPSHOT conflicts
* Disk space exhaustion on agents

---

## 🔄 2️⃣ MAVEN BUILD LIFECYCLE

### 🔁 What is a Maven Lifecycle?

A lifecycle is an **ordered sequence of phases** Maven executes to build and deliver software.

📌 Running one phase automatically runs **all previous phases**.

---

### 🧠 Maven Lifecycles (High Level)

| Lifecycle | Purpose          |
| --------- | ---------------- |
| default   | Build & deploy   |
| clean     | Remove old build |
| site      | Documentation    |

---

### 📜 DEFAULT LIFECYCLE — COMPLETE FLOW

```text
validate → compile → test → package → verify → install → deploy
```

---

### 🔍 Phase-by-Phase Explanation (WITH EXAMPLES)

#### 1️⃣ validate 🟡

* Checks project structure
* Validates `pom.xml`
* Ensures mandatory values exist

📌 No code compilation

👉 *Interview meaning:* “Is this project buildable?”

---

#### 2️⃣ compile ⚙️

* Compiles source code (`.java → .class`)

---

#### 3️⃣ test 🧪

* Runs unit tests
* Uses Surefire plugin

---

#### 4️⃣ package 📦

* Creates artifact (JAR/WAR)
* Output stored in `target/`

---

#### 5️⃣ verify 🔴

* Runs post-package checks
* Integration tests, coverage, quality gates

👉 *Interview meaning:* “Is this artifact release-ready?”

---

#### 🔍 validate vs verify (MOST ASKED)

| Aspect             | validate          | verify            |
| ------------------ | ----------------- | ----------------- |
| Lifecycle position | Start             | Near end          |
| Focus              | Project structure | Artifact quality  |
| Code compiled      | ❌                 | ✅                 |
| DevOps meaning     | Build readiness   | Release readiness |

🧠 Memory tip:

* **validate → project ready?**
* **verify → artifact ready?**

---

#### 6️⃣ install 📥

* Installs artifact into local `.m2`

---

#### 7️⃣ deploy 🚀

* Uploads artifact to Nexus / Artifactory

---

### ▶️ Command Execution Examples

```bash
mvn verify   # runs validate → compile → test → package → verify
mvn install  # runs validate → compile → test → package → verify → install
```

📌 Maven never skips earlier phases.

---

## 📄 3️⃣ `pom.xml` — SINGLE SOURCE OF TRUTH

### 📌 What is `pom.xml`?

Project Object Model — defines how Maven builds the project.

---

### 🔑 Important Sections (No Duplication)

#### 🧾 Project Coordinates

```xml
<groupId>com.company</groupId>
<artifactId>order-service</artifactId>
<version>1.0.0</version>
```

---

#### 📦 Dependencies

Defines external libraries.

---

#### 🧠 dependencyManagement

Central version control (multi-module projects).

---

#### 🔌 Plugins

Bind actions to lifecycle phases.

---

#### 🎭 Profiles

Environment-specific configuration (dev / qa / prod).

---

## ▶️ 4️⃣ COMMON `mvn` COMMANDS — WHAT THEY REALLY DO

| Command       | What it Does             |
| ------------- | ------------------------ |
| `mvn clean`   | Deletes `target/`        |
| `mvn compile` | Compiles code            |
| `mvn test`    | Runs unit tests          |
| `mvn package` | Creates artifact         |
| `mvn install` | Publishes to `.m2`       |
| `mvn deploy`  | Publishes to remote repo |

---
