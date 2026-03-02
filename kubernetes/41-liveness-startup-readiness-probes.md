# Kubernetes Probes Explained (Liveness, Readiness, Startup)

Kubernetes probes are **health checks** used by kubelet to understand the state of a container and decide **when to restart it, when to send traffic to it, and when to wait during startup**.

---

## 1️⃣ Liveness Probe

### What is a Liveness Probe?

A **liveness probe** checks whether a container is **still alive**.

* If the liveness probe **fails repeatedly**, Kubernetes **restarts the container**.
* It is used to detect **deadlocks, infinite loops, or hung applications**.

### When to Use

* Application is running but **not responding**
* Needs a **restart** to recover

### What Happens on Failure?

➡️ Container is **killed and restarted**

### Example: HTTP Liveness Probe

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
```

### Example: Command-Based Liveness Probe

```yaml
livenessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  periodSeconds: 10
```

### Key Points

* Used for **self-healing**
* Failure = **restart container**

---

## 2️⃣ Readiness Probe

### What is a Readiness Probe?

A **readiness probe** checks whether a container is **ready to accept traffic**.

* If the readiness probe **fails**, Kubernetes **removes the Pod from Service endpoints**.
* The container is **NOT restarted**.

### When to Use

* App is running but **temporarily unavailable**
* App is waiting for:

  * Database connection
  * Cache warm-up
  * External API

### What Happens on Failure?

➡️ Pod is marked **NotReady**
➡️ Traffic **stops**

### Example: HTTP Readiness Probe

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

### Example: TCP Readiness Probe

```yaml
readinessProbe:
  tcpSocket:
    port: 3306
```

### Key Points

* Controls **traffic flow**
* Failure = **no traffic**, no restart

---

## 3️⃣ Startup Probe

### What is a Startup Probe?

A **startup probe** checks whether an application has **successfully started**.

* Used for **slow-starting applications**
* Disables liveness & readiness probes **until startup succeeds**

### Why It Exists

Without startup probe:

* Slow apps may fail liveness probe
* Kubernetes restarts container **before it finishes starting**

### What Happens on Failure?

➡️ Container is **restarted**

### Example: Startup Probe

```yaml
startupProbe:
  httpGet:
    path: /health
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

➡️ App gets **300 seconds (30×10)** to start

### Key Points

* Used **only during startup phase**
* Prevents premature restarts

---

## 🔁 How Probes Work Together

```text
Startup Phase → Startup Probe
Running Phase → Liveness Probe
Traffic Control → Readiness Probe
```

---

## 📊 Comparison Table

| Probe Type | Purpose                | Failure Result      | Restarts Container |
| ---------- | ---------------------- | ------------------- | ------------------ |
| Liveness   | Is app alive?          | Restart             | ✅ Yes              |
| Readiness  | Can app serve traffic? | Remove from Service | ❌ No               |
| Startup    | Has app started?       | Restart             | ✅ Yes              |

---

## ✅ Best Practices

* Use **startup probe** for slow apps (Java, Spring Boot)
* Keep **liveness probe simple**
* Use **readiness probe** for dependencies
* Never use readiness probe for fatal checks

---

## 🧠 One-Line Summary

* **Liveness** → Restart me
* **Readiness** → Send traffic to me
* **Startup** → Wait for me to start
