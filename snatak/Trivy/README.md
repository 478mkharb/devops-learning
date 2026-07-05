# 🔐 Trivy – Complete DevSecOps Deep Dive Guide

---

## 🧭 1. What is Trivy?

**Trivy** is an open-source vulnerability scanner used in DevSecOps pipelines to detect:

* Vulnerabilities (CVEs)
* Misconfigurations
* Secrets
* License risks

👉 Developed by Aqua Security

---

## 🧠 2. Core Concept

```
Shift Left Security

Fix security issues early → before production
```

👉 Trivy integrates at multiple stages of pipeline

---

## 🧩 3. What Trivy Scans (Detailed)

### 🐳 3.1 Container Images

```
trivy image nginx:latest
```

#### Scans:

* OS packages (Ubuntu, Alpine)
* Installed binaries
* Libraries

#### Example Output:

```
CRITICAL: 2
HIGH: 5
```

#### Real Example:

* OpenSSL outdated → CVE detected

---

### 📦 3.2 Filesystem (Code Scan)

```
trivy fs .
```

#### Scans:

* Node (package.json)
* Python (requirements.txt)
* Java (pom.xml)

#### Detects:

* Vulnerable dependencies
* Secrets

---

### 🔐 3.3 Secret Scanning

#### Detects:

* AWS keys
* API tokens
* Passwords

#### Example:

```
AWS_SECRET_ACCESS_KEY detected
```

---

### 🏗️ 3.4 Infrastructure as Code (IaC)

```
trivy config .
```

#### Scans:

* Terraform
* Kubernetes YAML
* Dockerfile

#### Detects:

* Open security groups
* Public S3 buckets
* Privileged containers

---

### ☸️ 3.5 Kubernetes Cluster

```
trivy k8s cluster
```

#### Scans:

* Pods
* RBAC
* Services

#### Detects:

* Privileged pods
* Insecure configs

---

### 📜 3.6 License Scanning

Detects risky licenses like GPL

---

## 🚨 4. Types of Issues Detected

| Type      | Description                |
| --------- | -------------------------- |
| CVE       | Known vulnerabilities      |
| Misconfig | Security misconfigurations |
| Secrets   | Hardcoded credentials      |
| License   | Compliance issues          |

---

## 🚀 5. DevOps Use Cases (Most Important)

---

### 🔥 5.1 Docker Image Scanning

```
docker build -t myapp .
trivy image myapp
```

👉 Prevent vulnerable images

---

### 🔥 5.2 CI/CD Pipeline Integration

#### Jenkins Example:

```
trivy image myapp --exit-code 1 --severity HIGH,CRITICAL
```

👉 Pipeline fails if vulnerabilities exist

---

### 🔥 5.3 IaC Security (Terraform)

```
trivy config .
```

#### Example Issues:

* 0.0.0.0/0 open
* No encryption

---

### 🔥 5.4 Kubernetes Security

```
trivy k8s cluster
```

---

### 🔥 5.5 Secret Detection in Code

```
trivy fs .
```

---

### 🔥 5.6 Developer Local Scan

```
trivy fs .
```

---

## 🔄 6. DevOps Pipeline Flow

```
Code → Build → Scan → Deploy
              ↑
            Trivy
```

---

## ⚙️ 7. Advanced Usage

### Filter severity

```
trivy image myapp --severity HIGH,CRITICAL
```

### Fail pipeline

```
trivy image myapp --exit-code 1
```

### JSON output

```
trivy image myapp -f json
```

---

## 🧠 8. Real Production Practices

* Block deployments with CRITICAL issues
* Use Trivy in CI/CD
* Scan IaC before apply
* Scan Kubernetes clusters regularly

---

## ⚔️ 9. Trivy vs Other Tools

| Tool    | Use                |
| ------- | ------------------ |
| Trivy   | All-in-one scanner |
| Snyk    | Developer-focused  |
| Clair   | Container scanning |
| Anchore | Enterprise         |

---

## ⚠️ 10. Common Mistakes

* Ignoring HIGH vulnerabilities
* Not scanning IaC
* Hardcoding secrets

---

## 🧹 11. Best Practices

* Always scan before deploy
* Use minimal base images
* Fix vulnerabilities immediately

---

## ✅ 12. Final Summary

✔ Trivy = DevSecOps scanning tool
✔ Scans containers, code, IaC, Kubernetes
✔ Detects vulnerabilities, secrets, misconfigurations
✔ Used in CI/CD pipelines

---

## 🚀 13. Interview Questions

1. What is Trivy?
2. What does it scan?
3. How to integrate in CI/CD?
4. Difference between vulnerability and misconfiguration?
5. What is shift-left security?

---

**This is a complete real-world DevSecOps security workflow using Trivy.**
