## ⏱️ What are Build Triggers in Jenkins?

### 🧠 Definition

**Build Triggers** in Jenkins define **when and how a Jenkins job or pipeline should start automatically**.

👉 Instead of clicking **“Build Now”**, triggers allow Jenkins to **react to events or schedules**.

---

## 🎯 Why Build Triggers are Important

* 🔄 Enable automation (CI/CD)
* ⚡ Faster feedback on code changes
* 🤖 Remove manual intervention
* 🔗 Integrate Jenkins with Git, other jobs, and external systems

---

## 🧩 High-Level Trigger Flow

```
Event (Time / Code / Job / API)
        ↓
Jenkins Build Trigger
        ↓
Job / Pipeline Starts
```

---

## 🛠️ Types of Build Triggers Available in Jenkins UI

These are the **standard triggers you see in the Jenkins job configuration UI**.

---

## 1️⃣ Build Periodically (Cron Trigger)

### 🧠 What it does

Runs a job on a **fixed schedule**, regardless of code changes.

### 📌 UI Label

**Build periodically**

### 🕒 Example

```
H 2 * * *
```

➡️ Runs every day around 2 AM

### ✅ Use cases

* Nightly builds
* Periodic health checks
* Cleanup jobs

---

## 2️⃣ Poll SCM

### 🧠 What it does

Jenkins **polls the source code repository** at intervals to check for changes.

If changes are detected → build starts.

### 📌 UI Label

**Poll SCM**

### 🕒 Example

```
H/5 * * * *
```

➡️ Check Git every 5 minutes

### ⚠️ Note

* Inefficient for large repos
* Largely replaced by webhooks

---

## 3️⃣ GitHub / GitLab Webhook Trigger

### 🧠 What it does

Jenkins is triggered **instantly** when a code push happens.

### 📌 UI Label

**GitHub hook trigger for GITScm polling**

### 🔄 Flow

```
Git Push
   ↓
Webhook
   ↓
Jenkins
```

### ✅ Best practice for CI

---

## 4️⃣ Trigger Builds Remotely (API Trigger)

### 🧠 What it does

Allows a job to be triggered via a **URL/API call**.

### 📌 UI Label

**Trigger builds remotely (e.g., from scripts)**

### 📌 Example

```
http://jenkins/job/my-job/build?token=TOKEN
```

### ✅ Use cases

* Trigger from scripts
* Trigger from other tools

---

## 5️⃣ Build After Other Projects Are Built

### 🧠 What it does

Triggers a job **after another job completes**.

### 📌 UI Label

**Build after other projects are built**

### 🔄 Flow

```
Job A
  ↓
Job B (Triggered)
```

### ✅ Use cases

* Job chaining
* Legacy pipelines

---

## 6️⃣ Quiet Period (Delay Trigger)

### 🧠 What it does

Delays job execution for a defined time after being triggered.

### 📌 UI Label

**Quiet period**

### ✅ Use cases

* Avoid duplicate builds
* Batch multiple commits

---

## 7️⃣ Parameterized Trigger (User Input)

### 🧠 What it does

Allows users to **pass parameters** when triggering a build.

### 📌 UI Label

**This project is parameterized**

### 📌 Example Parameters

* ENV=prod
* VERSION=1.2.3

---

## 🔁 Pipeline-Specific Triggers (Jenkinsfile)

Some triggers are defined **as code** instead of UI:

```
triggers {
  cron('H 1 * * *')
}
```

Common pipeline triggers:

* `cron`
* `pollSCM`
* `upstream`

---

## 🆚 Trigger Comparison Table

| Trigger Type       | Event-Based | Scheduled | Recommended |
| ------------------ | ----------- | --------- | ----------- |
| Build periodically | ❌           | ✅         | ⚠️ Limited  |
| Poll SCM           | ❌           | ✅         | ❌ Avoid     |
| Webhook            | ✅           | ❌         | ✅ Best      |
| Remote trigger     | ✅           | ❌         | ✅           |
| Upstream job       | ✅           | ❌         | ⚠️ Legacy   |

---

## 🎤 Interview Golden Lines

* ⏱️ *Build triggers define when a Jenkins job starts*
* 🔗 *Webhooks are preferred over Poll SCM*
* 📜 *Triggers can be defined in UI or Jenkinsfile*

---

## 💡 Pro Interview Tip

> “In modern CI, Jenkins jobs should be triggered by **webhooks**, not by polling.”

---

