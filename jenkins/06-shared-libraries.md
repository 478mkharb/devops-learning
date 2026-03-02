## 📚 What are Shared Libraries in Jenkins?

### 🧠 Definition

**Jenkins Shared Libraries** are **reusable Groovy code modules** that allow you to **share common pipeline logic across multiple Jenkins pipelines**.

👉 Instead of copying the same pipeline code into every `Jenkinsfile`, you write it **once** and reuse it everywhere.

---

## ❓ Why Shared Libraries are needed

Without shared libraries:

* ❌ Duplicate Jenkinsfile code
* ❌ Hard to maintain changes
* ❌ Inconsistent pipelines

With shared libraries:

* ✅ DRY (Don’t Repeat Yourself)
* ✅ Centralized pipeline logic
* ✅ Easy updates across teams
* ✅ Enterprise-scale CI/CD

---

## 🏗️ How Shared Libraries Work (High-Level)

```
Jenkinsfile
   |
   v
📚 Shared Library (Git Repo)
   |
   v
Reusable Pipeline Logic
```

Jenkins loads the library **at runtime** from Git.

---

## 📂 Standard Shared Library Structure

A Jenkins shared library **must follow this structure**:

```
shared-lib-repo/
 ├── vars/
 │    └── buildApp.groovy
 ├── src/
 │    └── com/company/utils.groovy
 └── resources/
      └── config.yaml
```

---

## 📁 Key Directories Explained

### 1️⃣ `vars/` – Global Pipeline Functions

* Each file becomes a **global pipeline step**
* Easy to call directly from Jenkinsfile

Example:

```
buildApp()
```

---

### 2️⃣ `src/` – Helper Classes (Advanced)

* Standard Groovy classes
* Used for complex logic

Example:

```
import com.company.Utils
Utils.deploy()
```

---

### 3️⃣ `resources/` – Static Files

* YAML / JSON / templates
* Loaded at runtime

---

## 📌 Example: Simple Shared Library

### 🔹 `vars/buildApp.groovy`

```
def call() {
  stage('Build') {
    sh 'mvn clean package'
  }
}
```

---

## 📌 Using Shared Library in Jenkinsfile

```
@Library('my-shared-lib') _

pipeline {
  agent any
  stages {
    stage('CI') {
      steps {
        buildApp()
      }
    }
  }
}
```

✔️ Pipeline stays clean & readable

---

## 🔄 Shared Library Loading Types

### 1️⃣ Implicit (Global) Shared Library ⭐

### 🧠 What it means

An **implicit shared library** is configured **globally in Jenkins** and is **automatically available** to all pipelines.

👉 You do **NOT** need to import it explicitly in the Jenkinsfile.

---

### 🧩 How it works internally

```
Jenkins Controller
   |
   v
Global Shared Library (Configured in UI)
   |
   v
Available to all Jenkinsfiles
```

Jenkins loads the library **before pipeline execution starts**.

---

### 📌 Example: Implicit Shared Library

**Library configured in Jenkins UI as:** `global-lib`

Jenkinsfile:

```
pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        buildApp()   // Directly usable
      }
    }
  }
}
```

✔️ No `@Library` annotation needed

---

### ✅ When to use Implicit Libraries

* Organization-wide standards
* Security, logging, notifications
* Mandatory pipeline behavior

---

### 2️⃣ Explicit Shared Library

### 🧠 What it means

An **explicit shared library** must be **declared inside the Jenkinsfile** using the `@Library` annotation.

👉 Jenkins loads it **only for that pipeline**.

---

### 📌 Example: Explicit Shared Library

```
@Library('my-shared-lib@v1.2') _

pipeline {
  agent any
  stages {
    stage('CI') {
      steps {
        buildApp()
      }
    }
  }
}
```

✔️ Version-controlled usage
✔️ Pipeline-specific behavior

---

### ✅ When to use Explicit Libraries

* Team-specific pipelines
* Version-pinned logic
* Experimental features

---

### 🆚 Implicit vs Explicit Shared Libraries

| Feature            | Implicit               | Explicit          |
| ------------------ | ---------------------- | ----------------- |
| Jenkinsfile import | ❌ Not required         | ✅ Required        |
| Scope              | Global                 | Pipeline-specific |
| Version pinning    | ❌ Not possible         | ✅ Yes             |
| Governance         | Centralized            | Flexible          |
| Risk               | Higher (global impact) | Lower             |

---

## 🌿 Versioning Shared Libraries

You can pin library versions:

```
@Library('my-shared-lib@v1.2') _
```

Options:

* Branch
* Tag
* Commit SHA

---

## 🧩 Real-World Example (Enterprise)

### Scenario

Company has **50 microservices**.

Shared logic:

* Build Docker image
* Push to registry
* Deploy to Kubernetes

Instead of repeating code:

```
buildDocker()
pushImage()
deployK8s()
```

All logic lives in **one shared library**.

---

## 🆚 Shared Libraries vs Copy-Paste

| Feature          | Copy-Paste | Shared Library |
| ---------------- | ---------- | -------------- |
| Reusability      | ❌          | ✅              |
| Maintainability  | ❌          | ✅              |
| Version control  | ❌          | ✅              |
| Enterprise-ready | ❌          | ✅              |

---

## ⚠️ Best Practices

* ✅ Keep libraries small & focused
* ✅ Version libraries
* ❌ Avoid heavy logic in Jenkinsfile
* ❌ Don’t hardcode credentials

---

## 🎤 Interview Golden Lines

* 📚 *Shared libraries enable reusable, versioned Jenkins pipeline logic*
* 🧩 *They help scale CI/CD across teams*
* 🔄 *Changes propagate without touching Jenkinsfiles*

---

## 💡 Pro Interview Tip

> “Shared libraries are essential when Jenkins pipelines need to scale across multiple teams and services.”

---

Want next?

* Advanced shared library patterns
* Library testing strategies
* Shared libraries with Kubernetes & Helm
* Common mistakes in shared libraries

Just say 👍
