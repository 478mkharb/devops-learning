# 🧪 Maven SNAPSHOT vs RELEASE

---

## 📌 What is a SNAPSHOT?

A **SNAPSHOT** version represents a **work-in-progress build**.
It is **mutable**, meaning the artifact **can change without changing the version number**.

```xml
<version>1.0-SNAPSHOT</version>
```

📌 SNAPSHOTs are meant for **development and integration testing**, not production.

---

## 📌 What is a RELEASE / FINAL Version?

A **RELEASE (final) version** is:

* Stable
* Immutable
* Production-ready

```xml
<version>1.0.0</version>
```

📌 Once deployed, a release version **must never be modified**.

---

## 🔍 SNAPSHOT vs RELEASE (INTERVIEW TABLE)

| Aspect       | SNAPSHOT            | RELEASE      |
| ------------ | ------------------- | ------------ |
| Mutability   | Mutable             | Immutable    |
| Purpose      | Ongoing development | Production   |
| CI frequency | Built many times    | Built once   |
| Repository   | snapshot repo       | release repo |
| Example      | `1.0-SNAPSHOT`      | `1.0.0`      |

---

## ❓ Why SNAPSHOT is Created? (KEY QUESTION 🔥)

SNAPSHOT exists to **support continuous development and CI/CD**.

Without SNAPSHOT:

* Every code change would require a new version
* Teams couldn’t test integration early
* CI pipelines would be slow and fragile

📌 SNAPSHOT allows teams to **share the latest build automatically**.

---

## ⚙️ How SNAPSHOT Works Internally

When you run:

```bash
mvn install
```

Maven generates a timestamped artifact:

```text
my-app-1.0-20260303.123456-1.jar
```

📌 Maven uses metadata to always fetch the **latest SNAPSHOT**.

---

## 🔁 SNAPSHOT FLOW (REAL DEVOPS PIPELINE)

```text
Developer commits code
        ↓
CI triggers build
        ↓
mvn install / deploy
        ↓
1.0-SNAPSHOT pushed to Nexus (snapshot repo)
        ↓
Other services pull latest SNAPSHOT
        ↓
Integration testing
```

📌 Very common in **microservices-based architectures**.

---

## 🧪 Real Microservices Use Case

* Order Service → `2.1-SNAPSHOT`
* Payment Service depends on Order Service SNAPSHOT

Teams validate integration **before cutting a release**.

---

## 🚨 When NOT to Use SNAPSHOT

❌ Never use SNAPSHOT in:

* Production environments
* Release pipelines

✅ Convert SNAPSHOT → RELEASE when:

* Features are complete
* All tests pass
* Code is stable

---

## 🧠 Interview One-Liner (MEMORIZE)

> “SNAPSHOT versions support continuous development by allowing frequently updated, mutable artifacts, while release versions are immutable and production-ready.”

---
