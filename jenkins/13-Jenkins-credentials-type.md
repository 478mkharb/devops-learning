# 🔐 Jenkins Credentials Types — Quick Guide

---

## 🔹 1. Username & Password

* Used for: GitHub, GitLab, Docker, web services

```groovy
withCredentials([usernamePassword(credentialsId: 'git-creds',
    usernameVariable: 'USER',
    passwordVariable: 'PASS')]) {
    sh 'git clone https://$USER:$PASS@github.com/repo.git'
}
```

---

## 🔹 2. SSH Username with Private Key

* Used for: SSH access, Git over SSH

```groovy
withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key',
    keyFileVariable: 'KEY')]) {
    sh 'ssh -i $KEY user@server'
}
```

---

## 🔹 3. Secret Text

* Used for: API tokens, bearer tokens

```groovy
withCredentials([string(credentialsId: 'api-token',
    variable: 'TOKEN')]) {
    sh 'curl -H "Authorization: Bearer $TOKEN" api'
}
```

---

## 🔹 4. Secret File

* Used for: kubeconfig, JSON keys, certificates

```groovy
withCredentials([file(credentialsId: 'kubeconfig',
    variable: 'FILE')]) {
    sh 'kubectl --kubeconfig=$FILE get pods'
}
```

---

## 🔹 5. Certificate (PKCS#12)

* Used for: SSL authentication
* Stored as `.p12` file

---

## 🔹 6. AWS Credentials

* Used for: AWS CLI / SDK access

```groovy
withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
    credentialsId: 'aws-creds']]) {
    sh 'aws s3 ls'
}
```

---

## 🔹 7. Docker Credentials

* Used for: Docker Hub / private registry login

---

# 🔥 Summary Table

| Type               | Use Case              |
| ------------------ | --------------------- |
| Username/Password  | Git, apps             |
| SSH Key            | Server login, Git SSH |
| Secret Text        | API tokens            |
| Secret File        | Config files          |
| Certificate        | SSL auth              |
| AWS Credentials    | AWS access            |
| Docker Credentials | Container registry    |

---

# 🔥 One-Liner

> Jenkins credentials securely store sensitive data and allow controlled access to external systems.

---
