## ⚙️ How Jenkins Pipeline Execution Works Internally

This explains **what actually happens inside Jenkins** when a pipeline runs — step by step — from Git push to execution on agents.

---

## 🧠 High-Level Idea

A Jenkins pipeline is **not executed line-by-line like a shell script**.

Instead, Jenkins:

* Parses the pipeline
* Converts it into an **execution graph**
* Runs it using a **state machine (CPS engine)**
* Persists state so it can **resume after failures/restarts**

---

## 1️⃣ Pipeline Trigger Phase

### 🔔 How execution starts

A pipeline can be triggered by:

* Git webhook
* SCM polling
* Manual trigger
* Upstream job

```
Git Push
   ↓
Webhook
   ↓
Jenkins Controller
```

👉 Jenkins controller receives an event and schedules a build.

---

## 2️⃣ Jenkinsfile Retrieval & Parsing

### 📂 What Jenkins does next

* Jenkins checks out the repository
* Looks for a **Jenkinsfile**
* Parses the Groovy-based pipeline syntax

```
repo/
 ├── app-code/
 └── Jenkinsfile  👈
```

If parsing fails → pipeline **fails immediately** ❌

---

## 3️⃣ Pipeline Compilation (CPS Transformation)

### 🧠 CPS (Continuation Passing Style)

Jenkins transforms pipeline code into a **CPS execution model**:

* Each step becomes a resumable unit
* Execution state is stored after every step

Why?

* Jenkins can restart safely
* Pipelines can pause (input, sleep)

👉 This is why pipelines survive Jenkins restarts 🔁

---

## 4️⃣ Execution Graph Creation

Jenkins creates a **Directed Acyclic Graph (DAG)**:

```
Pipeline
 ├── Stage: Build
 ├── Stage: Test
 └── Stage: Deploy
```

* Stages = nodes
* Dependencies define order
* Parallel stages branch in graph

---

## 5️⃣ Executor & Agent Allocation

### 🧑‍🏭 How Jenkins runs work

* Jenkins controller **does NOT run jobs**
* It assigns work to **agents (nodes)**

```
Controller
   ↓ assigns work
Agent / Executor
```

If no executor is free → pipeline waits ⏳

---

## 6️⃣ Workspace Allocation

On the agent:

* Jenkins creates a **workspace directory**
* Source code is checked out

```
/var/jenkins/workspace/my-pipeline/
```

Each build has an isolated workspace.

---

## 7️⃣ Step-by-Step Execution

Each pipeline step:

* Runs on agent
* Reports status back to controller
* Saves execution state

Example:

```
sh 'mvn clean test'
```

If Jenkins crashes here → execution can resume after restart.

---

## 8️⃣ Parallel & Conditional Execution

### ⚡ Parallel stages

```
parallel {
  stage('Unit Test')
  stage('Security Scan')
}
```

* Jenkins forks execution paths
* Runs them on multiple executors

---

### 🔀 Conditional execution

```
when { branch 'main' }
```

* Evaluated at runtime
* Controls execution path

---

## 9️⃣ Pause, Input & Resume

Some steps intentionally pause:

* `input` (manual approval)
* `sleep`

Jenkins:

* Saves pipeline state
* Waits
* Resumes from exact step

---

## 🔟 Post Actions & Cleanup

After main stages:

```
post {
  success { notify() }
  failure { rollback() }
}
```

Runs regardless of pipeline outcome.

---

## 🧩 End-to-End Internal Flow (Visual)

```
Trigger
  ↓
Checkout Jenkinsfile
  ↓
Parse & CPS Transform
  ↓
Build Execution Graph
  ↓
Allocate Agent & Executor
  ↓
Create Workspace
  ↓
Execute Steps (Persist State)
  ↓
Post Actions
```

---

## 📌 Example: Simple Pipeline Execution

Pipeline:

```
stage('Build')
stage('Test')
stage('Deploy')
```

Internal reality:

* Each stage → graph node
* Each step → CPS checkpoint
* Controller coordinates
* Agent executes

---

## 🆚 Why Pipelines are Better than Freestyle Internally

| Capability           | Freestyle | Pipeline |
| -------------------- | --------- | -------- |
| Resume after restart | ❌         | ✅        |
| Execution graph      | ❌         | ✅        |
| Parallelism          | ❌         | ✅        |
| State persistence    | ❌         | ✅        |

---

## 🎤 Interview Golden Lines

* ⚙️ *Jenkins pipelines run on agents but are orchestrated by the controller*
* 🔁 *CPS allows pipelines to pause, persist state, and resume*
* 📊 *Pipeline execution is graph-based, not linear*

---

## 💡 Pro Interview Tip

> “The biggest internal difference is that pipelines are **resumable workflows**, not simple scripts.”

---

Want to go deeper?

* CPS limitations & best practices
* Jenkins pipeline performance tuning
* Jenkins controller vs agent internals
* Pipeline failure & recovery scenarios

Just say 👍
