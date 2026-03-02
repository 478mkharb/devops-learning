# Kubernetes ConfigMap – Concept & Ways to Inject into a Pod

## What is a ConfigMap?

A **ConfigMap** is a Kubernetes object used to store **non-sensitive configuration data** such as:

* Application configuration files
* Environment variables
* Command-line arguments
* Feature flags

ConfigMaps help **decouple configuration from application code**, making containers portable and easier to manage across environments (dev, test, prod).

> ⚠️ ConfigMaps are **not encrypted**. Do NOT store secrets (passwords, tokens). Use **Secrets** instead.

---

## Why Do We Need ConfigMaps?

* Avoid rebuilding images for config changes
* Centralized configuration management
* Same image, different configs per environment
* Dynamic updates without redeploying code

---

## Creating a ConfigMap

### 1. From Literal Values

```bash
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=APP_DEBUG=false
```

### 2. From a File

```bash
kubectl create configmap nginx-config --from-file=nginx.conf
```

### 3. Using YAML (Recommended)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "production"
  APP_DEBUG: "false"
  application.properties: |
    server.port=8080
    logging.level=INFO
```

---

## Ways to Inject ConfigMap into a Pod

There are **4 main ways** to inject ConfigMap data into Pods.

---

## 1️⃣ Inject ConfigMap as Environment Variables (Individual Keys)

### Pod YAML Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-demo
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "env | grep APP"]
    env:
    - name: APP_ENV
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_ENV
```

### Result inside Pod

```bash
APP_ENV=production
```

✅ Use when only **specific keys** are needed

---

## 2️⃣ Inject Entire ConfigMap as Environment Variables

### Pod YAML Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envfrom-demo
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "env | grep APP"]
    envFrom:
    - configMapRef:
        name: app-config
```

### Result

```bash
APP_ENV=production
APP_DEBUG=false
```

✅ Simple and clean
⚠️ All keys become env vars

---

## 3️⃣ Inject ConfigMap as Files Using Volume Mount

Each key becomes a **file**, and value becomes file content.

### Pod YAML Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-demo
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "cat /config/application.properties"]
    volumeMounts:
    - name: config-volume
      mountPath: /config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

### Files Created in Container

```bash
/config/APP_ENV
/config/APP_DEBUG
/config/application.properties
```

✅ Best for **config files** (nginx.conf, application.yml)

---

## 4️⃣ Inject Specific Files Using `items`

Mount only selected keys with custom filenames.

### Pod YAML Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: selective-volume-demo
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "cat /etc/config/app.properties"]
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
      items:
      - key: application.properties
        path: app.properties
```

✅ Full control over filenames

---

## Bonus: Read ConfigMap via Command-Line Args

```yaml
args:
- "--env=$(APP_ENV)"
```

Used together with environment variables.

---

## ConfigMap Update Behavior

| Injection Method | Pod Restart Needed? |
| ---------------- | ------------------- |
| Env Variables    | ✅ Yes               |
| Volume Mount     | ❌ No (auto-updates) |

> Volume-mounted ConfigMaps update automatically (with small delay).

---

## When to Use What?

| Use Case            | Recommended Method |
| ------------------- | ------------------ |
| App flags           | Env vars           |
| Config files        | Volume mount       |
| Partial config      | items              |
| Externalized config | envFrom            |

---

## ConfigMap vs Secret

| ConfigMap          | Secret           |
| ------------------ | ---------------- |
| Plain text         | Base64 encoded   |
| Non-sensitive data | Sensitive data   |
| No encryption      | Can be encrypted |

---

## Key Takeaway

> **ConfigMap = externalized configuration for Pods**

It keeps images clean, environments flexible, and deployments scalable.

---

If you want:

* Same content converted into **interview Q&A**
* **Real-world Nginx / Spring Boot example**
* **Short notes version**

Tell me 👍
