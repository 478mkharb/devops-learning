# 🔐 Kyverno — Deep Dive DevOps Canvas

## 📌 Overview

Kyverno is a **Kubernetes-native policy engine** that allows you to **validate, mutate, and generate resources using YAML** (no Rego).

👉 It runs as an **admission controller** and enforces rules before resources are created.

---

## 🧠 Core Architecture

```
kubectl apply → API Server → Kyverno → (validate / mutate / generate) → Resource Created
```

---

## 🔥 Core Capabilities

| Capability    | Purpose                     |
| ------------- | --------------------------- |
| Validate      | Block bad configurations    |
| Mutate        | Auto-fix missing configs    |
| Generate      | Create additional resources |
| Verify Images | Enforce signed images       |

---

# 🧩 1. VALIDATE POLICIES (Block Misconfigurations)

## ✅ Enforce Non-Root Containers

```
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-non-root
spec:
  rules:
  - name: check-non-root
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "Containers must run as non-root"
      pattern:
        spec:
          containers:
          - securityContext:
              runAsNonRoot: true
```

---

## 🚫 Block Latest Image Tag

```
validate:
  message: "latest tag not allowed"
  pattern:
    spec:
      containers:
      - image: "!*:latest"
```

---

## 🏷️ Enforce Labels

```
validate:
  message: "Labels required"
  pattern:
    metadata:
      labels:
        app: "?*"
        env: "?*"
```

---

## 🔒 Block Privileged Containers

```
validate:
  message: "Privileged not allowed"
  pattern:
    spec:
      containers:
      - securityContext:
          privileged: false
```

---

## ⚡ Enforce Resource Limits

```
validate:
  message: "CPU/Memory limits required"
  pattern:
    spec:
      containers:
      - resources:
          limits:
            cpu: "?*"
            memory: "?*"
```

---

## 🌐 Restrict Registry

```
validate:
  message: "Only company registry allowed"
  pattern:
    spec:
      containers:
      - image: "mycompany.com/*"
```

---

# 🔄 2. MUTATE POLICIES (Auto-Fix)

## ✅ Add Non-Root Automatically

```
mutate:
  patchStrategicMerge:
    spec:
      containers:
      - (name): "*"
        securityContext:
          runAsNonRoot: true
```

---

## 🏷️ Auto Add Labels

```
mutate:
  patchStrategicMerge:
    metadata:
      labels:
        env: production
```

---

## 🔐 Add Default Security Context

```
mutate:
  patchStrategicMerge:
    spec:
      containers:
      - (name): "*"
        securityContext:
          allowPrivilegeEscalation: false
```

---

# 🏗️ 3. GENERATE POLICIES (Auto Create Resources)

## 🌐 Generate NetworkPolicy

```
generate:
  kind: NetworkPolicy
  name: default-deny
  namespace: "{{request.object.metadata.name}}"
```

---

## 📜 Generate ConfigMap

```
generate:
  kind: ConfigMap
  name: default-config
```

---

# 🔐 4. IMAGE VERIFICATION (ADVANCED)

👉 Ensure only signed images are used

```
verifyImages:
- image: "mycompany.com/*"
  key: "cosign.pub"
```

---

# ⚙️ POLICY TYPES

| Type          | Scope            |
| ------------- | ---------------- |
| Policy        | Namespace scoped |
| ClusterPolicy | Cluster-wide     |

---

# 🚀 REAL DEVOPS USE CASES

## 🔐 Security

* Non-root enforcement
* Block privileged containers
* Restrict capabilities

## ⚡ Governance

* Label enforcement
* Naming standards

## 📜 Compliance

* CIS benchmarks
* SOC2 / PCI

## 🏢 Platform Engineering

* Standardized deployments

---

# ⚠️ COMMON ISSUES

## ❗ Pod rejected

* Missing securityContext

## ❗ Image blocked

* Not from trusted registry

## ❗ Resource rejected

* Missing limits

---

# 🔄 KYVERNO VS OPA GATEKEEPER

| Feature  | Kyverno | OPA     |
| -------- | ------- | ------- |
| Language | YAML    | Rego    |
| Ease     | Easy    | Hard    |
| Mutation | Yes     | Limited |
| Generate | Yes     | No      |

---

# 🧠 BEST PRACTICES

* Always use ClusterPolicy for org-wide rules
* Combine validate + mutate
* Keep policies modular
* Test policies in audit mode first

---

# 🔥 PRODUCTION FLOW

```
Developer → CI/CD → Kyverno → Kubernetes
```

👉 Kyverno ensures only compliant resources are deployed

---

# 🧾 FINAL SUMMARY

✔️ Kyverno is a YAML-based policy engine
✔️ Enforces security, governance, compliance
✔️ Supports validate, mutate, generate
✔️ Easier than OPA for most teams

---

# 💡 INTERVIEW ANSWER

"Kyverno is a Kubernetes-native policy engine that enforces security and governance using YAML-based policies. It can validate, mutate, and generate resources, making it simpler than OPA Gatekeeper for implementing DevOps and security best practices."
