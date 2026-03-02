# 🛠️ API Deep Dive for DevOps — Interactive Guide (Part 3)

## Real DevOps APIs (Jenkins, Kubernetes, AWS, Terraform) — with `curl`

> This part connects **theory → real production systems**.
> If you understand this, you don’t *use* DevOps tools — you **control** them.

---

## 🧠 First Big Truth (Read This Slowly)

❗ Almost every DevOps tool is **just an API with a CLI wrapper**.

* Jenkins UI → Jenkins REST API
* kubectl → Kubernetes API Server
* Terraform → Cloud Provider APIs
* AWS CLI → AWS APIs

CLIs are convenience layers. **APIs do the real work.**

---

## 1️⃣ Jenkins API — CI/CD Is API-Driven

### What Jenkins Exposes

* Trigger jobs
* Fetch build status
* Download artifacts

---

### 🔹 Trigger a Jenkins Job (curl)

```bash
curl -X POST \
  http://jenkins.example.com/job/my-pipeline/build \
  --user user:api_token
```

👉 This is what GitHub Webhooks also do internally.

---

### 🔹 Get Build Status

```bash
curl -u user:api_token \
  http://jenkins.example.com/job/my-pipeline/lastBuild/api/json
```

🧠 Interactive Check:
❓ Is Jenkins REST or SOAP?

✅ REST (JSON, HTTP verbs)

---

## 2️⃣ Kubernetes API — The Heart of K8s

### Key Insight

> Kubernetes **is an API platform**, not a container platform.

Everything goes through:

```text
kube-apiserver
```

---

### kubectl = API Client

```bash
kubectl get pods
```

⬇️ What actually happens:

```http
GET /api/v1/namespaces/default/pods
```

---

### 🔹 Call Kubernetes API Directly

```bash
curl -k \
  -H "Authorization: Bearer <token>" \
  https://k8s-api.example.com/api/v1/namespaces/default/pods
```

🧠 Interactive Check:
❓ Why is Kubernetes API RESTful?

✅ Resources + verbs + stateless calls

---

## 3️⃣ AWS APIs — Cloud Is 100% API

### Important Mindset

> There is **NO AWS button**. Only APIs.

AWS Console = UI calling AWS APIs.

---

### Example: EC2 Creation Flow

```text
Terraform / AWS CLI → AWS EC2 API → Instance Created
```

---

### 🔹 AWS CLI = API Wrapper

```bash
aws ec2 describe-instances
```

This internally calls:

```text
DescribeInstances API
```

---

### DevOps Interview Insight

❓ Can AWS work without console?

✅ Yes — APIs only

---

## 4️⃣ Terraform — API Orchestrator

### What Terraform Really Does

* Reads `.tf` files
* Builds dependency graph
* Calls provider APIs

Terraform itself **creates nothing**.

---

### Terraform Apply (Behind the Scenes)

```text
terraform apply
  ↓
AWS API
Azure API
GCP API
```

---

### 🔹 Terraform Provider Example

```hcl
resource "aws_instance" "web" {
  ami = "ami-123"
  instance_type = "t2.micro"
}
```

👉 Provider translates this to AWS API calls.

---

## 5️⃣ Comparing DevOps APIs (Quick Table)

| Tool       | API Type             | Uses              |
| ---------- | -------------------- | ----------------- |
| Jenkins    | REST                 | CI/CD automation  |
| Kubernetes | REST                 | Cluster control   |
| AWS        | REST/Query           | Cloud infra       |
| Terraform  | REST (via providers) | IaC orchestration |

---

## 6️⃣ Real DevOps Scenarios (Interview Gold)

### Scenario 1

**Trigger CI on Git push**

➡️ GitHub Webhook → Jenkins API

---

### Scenario 2

**Auto-scale Pods**

➡️ HPA → Kubernetes API

---

### Scenario 3

**Infra as Code**

➡️ Terraform → Cloud APIs

---

## 🧠 Mental Model (Memorize This)

```text
UI / CLI
  ↓
API
  ↓
System Action
```

If API is down — nothing works.

---
