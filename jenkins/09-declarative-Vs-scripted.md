## 🆚 Difference Between Declarative and Scripted Jenkins Pipeline

Jenkins supports **two pipeline styles**:

* **Declarative Pipeline** (recommended)
* **Scripted Pipeline** (advanced / legacy)

Both use Groovy, but they differ **in structure, flexibility, and use cases**.

---

## 1️⃣ Declarative Pipeline

### 🧠 What it is

A **Declarative Pipeline** is a **structured, opinionated way** to define pipelines using a **predefined syntax**.

👉 Jenkins enforces rules → pipelines are **clean, readable, and consistent**.

---

### 🧩 Key Characteristics

* ✅ Simple & readable
* ✅ Clear stages & steps
* ✅ Built-in error handling
* ❌ Less flexible than scripted

---

### 📌 Declarative Pipeline Example

```
pipeline {
  agent any

  stages {
    stage('Build') {
      steps {
        sh 'mvn clean package'
      }
    }

    stage('Test') {
      steps {
        sh 'mvn test'
      }
    }
  }

  post {
    success {
      echo 'Build successful'
    }
    failure {
      echo 'Build failed'
    }
  }
}
```

---

### 🧠 Declarative Execution Model

```
Pipeline
  ↓
Agent
  ↓
Stages
  ↓
Steps
```

---

### ✅ When to use Declarative Pipeline

* CI/CD pipelines
* Team-wide standards
* Production Jenkins setups
* Beginners & interviews

---

## 2️⃣ Scripted Pipeline

### 🧠 What it is

A **Scripted Pipeline** is written as **pure Groovy code** using Jenkins Pipeline APIs.

👉 You control **everything**, but Jenkins does **not enforce structure**.

---

### 🧩 Key Characteristics

* ✅ Extremely flexible
* ✅ Full Groovy power
* ❌ Harder to read
* ❌ Easier to break

---

### 📌 Scripted Pipeline Example

```
node {
  stage('Build') {
    sh 'mvn clean package'
  }

  stage('Test') {
    sh 'mvn test'
  }

  if (env.BRANCH_NAME == 'main') {
    stage('Deploy') {
      sh 'deploy.sh'
    }
  }
}
```

---

### 🧠 Scripted Execution Model

```
Groovy Script
  ↓
node {}
  ↓
stage {}
  ↓
steps
```

---

## 🆚 Declarative vs Scripted – Side-by-Side Comparison (Detailed)

| Aspect                 | Declarative Pipeline                    | Scripted Pipeline                  |
| ---------------------- | --------------------------------------- | ---------------------------------- |
| Purpose                | Opinionated, standard CI/CD pipelines   | Maximum flexibility & custom logic |
| Syntax Style           | Structured, rule-based                  | Free-form Groovy code              |
| Entry Point            | `pipeline {}` block is mandatory        | `node {}` block is used            |
| Readability            | ⭐ Very high (easy to read)              | ⭐⭐ Medium to low (logic-heavy)     |
| Learning Curve         | Easy for beginners                      | Steep, requires Groovy knowledge   |
| Validation Time        | ✅ Validated at **parse time**           | ❌ Errors appear at **runtime**     |
| Error Handling         | Built-in (`post { success / failure }`) | Manual (`try/catch`)               |
| Stage Enforcement      | Mandatory `stages {}`                   | Optional, not enforced             |
| Parallel Execution     | Simple & declarative                    | Manual & verbose                   |
| Conditional Logic      | Limited (`when {}`)                     | Fully flexible (`if/else`, loops)  |
| Loops & Iterations     | ❌ Not directly supported                | ✅ Fully supported                  |
| Reusability            | High with shared libraries              | High but harder to maintain        |
| Maintainability        | ⭐ Very high                             | ⭐ Medium                           |
| Governance & Standards | Excellent for org-wide standards        | Hard to enforce standards          |
| Debugging              | Easier (structured logs)                | Harder (runtime failures)          |
| Resume After Restart   | ✅ Supported                             | ✅ Supported                        |
| CPS Complexity         | Abstracted away                         | Exposed to user                    |
| Plugin Compatibility   | Best supported                          | Supported but risky                |
| Typical Usage          | 90% real-world pipelines                | Edge cases & legacy pipelines      |
| Jenkins Recommendation | ✅ Strongly recommended                  | ⚠️ Use only when required          |

---

## 🧠 Internal Behavior Difference (Interview Critical)

### Declarative Pipeline

* Jenkins builds the execution model **before running**
* Invalid syntax fails immediately
* Predictable execution flow

### Scripted Pipeline

* Jenkins executes Groovy line-by-line
* Errors discovered during execution
* Execution flow can change dynamically

---

## 📌 Real-World Decision Guide

### Choose Declarative Pipeline when:

* You want clean, readable pipelines
* Multiple teams maintain Jenkinsfiles
* CI/CD standards must be enforced
* Production pipelines

### Choose Scripted Pipeline when:

* You need complex loops or dynamic stages
* Pipeline logic depends heavily on runtime data
* Migrating old freestyle jobs

---

## 🎤 Interview Killer Summary

> “Declarative pipelines provide structure, validation, and maintainability, while scripted pipelines provide flexibility at the cost of complexity. Most modern Jenkins setups prefer declarative pipelines with small scripted blocks when needed.”

---

## ⚙️ Validation & Error Handling (Important Difference)

### Declarative

* Jenkins validates pipeline **before execution**
* Syntax errors fail fast

### Scripted

* Errors appear **during execution**
* Harder to debug

---

## 🎤 Interview One-Liners

* 📜 *Declarative pipeline is structured and recommended*
* 📜 *Jenkins validates Declarative pipeline **before execution** whereas Errors appear in Scripted pipeline **during execution***
* 🧠 *Scripted pipeline offers maximum flexibility*
* ⚖️ *Declarative for standard CI/CD, Scripted for complex logic*

---

## 💡 Pro Interview Tip

> “Most teams use **Declarative pipelines with scripted blocks** when flexibility is required.”

Example:

```
script {
  if (env.BRANCH_NAME == 'main') {
    sh 'deploy.sh'
  }
}
```

---

## 🏁 Final Recommendation

✅ Use **Declarative Pipeline** by default
⚠️ Use **Scripted Pipeline** only for edge cases

---
