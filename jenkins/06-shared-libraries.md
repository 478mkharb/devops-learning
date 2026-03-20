## 📚 What are Shared Libraries in Jenkins?

## 🧠 Definition

**Jenkins Shared Libraries** are **reusable Groovy code modules** that allow you to **share common pipeline logic across multiple Jenkins pipelines**.

👉 Instead of copying the same pipeline code into every `Jenkinsfile`, you write it **once** and reuse it everywhere.

---
## 🎯 Objective

Understand how **resources, src, and vars work together** in a Jenkins Shared Library using a clean, production-style example.

---

# 🏗️ 1. Project Structure

```
(shared-library)
│
├── vars/
│   └── deployApp.groovy
│
├── src/
│   └── com/company/devops/ConfigDeployer.groovy
│
└── resources/
    └── config/app.yaml
```

---

# 📄 2. `resources/` → Configuration Layer

## 🔹 Purpose

* Stores **static files** (YAML, JSON, templates)
* No logic
* Used via `libraryResource()`

## ✅ Example: `resources/config/app.yaml`

```yaml
appName: my-app
environment: dev
dockerImage: my-app:v1
replicas: 2
```

## 🧠 Key Points

* Acts as **data layer**
* Easily replaceable per environment
* Keeps pipelines flexible

---

# 🧠 3. `src/` → Logic Layer (Core Engine)

## 🔹 Purpose

* Contains **Groovy classes**
* Implements **business logic**
* Uses config from `resources/`

## ✅ Example: `ConfigDeployer.groovy`

```groovy
package com.company.devops

import groovy.yaml.YamlSlurper

class ConfigDeployer implements Serializable {
    def steps

    ConfigDeployer(steps) {
        this.steps = steps
    }

    def deploy() {
        // Load config from resources
        def yamlText = steps.libraryResource('config/app.yaml')

        // Parse YAML
        def config = new YamlSlurper().parseText(yamlText)

        // Use config values
        steps.echo "Deploying ${config.appName} to ${config.environment}"

        steps.sh """
            docker run -d --name ${config.appName} \\
            --replicas=${config.replicas} \\
            ${config.dockerImage}
        """
    }
}
```

## 🧠 Key Concepts

### ✅ Class-Based Design

* Structured and reusable

### ✅ Uses Pipeline Context

```groovy
new ConfigDeployer(this)
```

### ✅ Serializable Required

* Supports Jenkins pipeline pause/resume (CPS)

### ❌ Cannot Use Directly

```groovy
sh "ls"  // ❌ Not allowed
```

### ✅ Correct Way

```groovy
steps.sh "ls"
```

---

# 🚀 4. `vars/` → Entry Point Layer

## 🔹 Purpose

* Provides **global functions**
* Acts as **bridge between pipeline and src/**

## ✅ Example: `deployApp.groovy`

```groovy
def call() {
    def deployer = new com.company.devops.ConfigDeployer(this)
    deployer.deploy()
}
```

## 🧠 Key Points

* Automatically available in pipeline
* No import needed
* Should stay **thin (no heavy logic)**

---

# ⚙️ 5. Jenkins Pipeline Usage

```groovy
@Library('my-shared-lib') _

pipeline {
    agent any

    stages {
        stage('Deploy') {
            steps {
                deployApp()
            }
        }
    }
}
```

---

# 🔄 6. End-to-End Flow

```
Pipeline
   ↓
vars/deployApp.groovy   (entry point)
   ↓
src/ConfigDeployer      (logic layer)
   ↓
resources/app.yaml      (configuration)
```

---

# 🧠 7. Architecture Mapping

| Layer      | Role             | Responsibility      |
| ---------- | ---------------- | ------------------- |
| resources/ | Data Layer       | Store configuration |
| src/       | Service Layer    | Process logic       |
| vars/      | Controller Layer | Trigger execution   |

---

# 🔥 8. Real DevOps Use Cases

## 🚀 Multi-Environment Deployment

```
config/dev.yaml
config/prod.yaml
```

## ☁️ Cloud Automation

* AWS / GCP logic in `src/`
* Config-driven deployments

## 🐳 Docker Pipelines

* Image name, tags from YAML

## ☸️ Kubernetes Deployments

* Use config for replicas, namespace

---

# ⚠️ 9. Common Mistakes

### ❌ Wrong: Reading file directly

```groovy
readFile('resources/config/app.yaml')
```

### ✅ Correct

```groovy
libraryResource('config/app.yaml')
```

---

### ❌ Heavy logic in `vars/`

* Makes pipeline messy

### ✅ Keep logic in `src/`

---

### ❌ No Serializable

* Causes runtime failure

---

# 🎯 10. Key Takeaways

* `resources/` → stores configuration (YAML/JSON)
* `src/` → contains reusable Groovy classes (logic)
* `vars/` → exposes simple functions to pipeline

---

# 🧠 Interview Summary

> A Jenkins Shared Library is structured into three layers: `resources/` for configuration, `src/` for reusable logic implemented as Groovy classes, and `vars/` for exposing entry points to pipelines. The `src/` layer consumes configuration using `libraryResource` and executes pipeline steps via injected context, enabling scalable and maintainable CI/CD design.

---

# ✅ Final Mental Model

```
resources → WHAT to do (data)
src       → HOW to do (logic)
vars      → WHEN to do (trigger)
```
