# OCI (Open Container Initiative) — Complete Mental Model

OCI is the **standard that makes containers portable and interoperable**. It defines *what a container image is* and *how a container must be run*, independent of Docker, Kubernetes, or any vendor.

---

## 1. Why OCI exists (real-world problem)

Before OCI:

* Docker had its own image & runtime formats
* Other runtimes behaved differently
* Vendor lock-in and incompatibility

**Goal of OCI:**

> One image, one runtime contract, works everywhere.

---

## 2. What is OCI?

**OCI = Open Container Initiative** (Linux Foundation project)

OCI does **not** run containers.
OCI defines **specifications**.

Think of OCI as:

> *The constitution of containers*

---

## 3. The 3 OCI Specifications (must-know)

### 1️⃣ OCI Image Specification

Defines:

* Image layers
* Image manifest
* Image config (CMD, ENTRYPOINT, ENV)

Example image:

```
nginx:latest
```

If an image follows OCI Image Spec, **any OCI runtime can run it**.

---

### 2️⃣ OCI Runtime Specification (MOST IMPORTANT)

Defines:

* How to create a container
* Namespaces to use
* cgroups to apply
* Mounts, capabilities, rootfs

Key file:

```
config.json
```

This file tells the runtime *exactly* how to start the container.

---

### 3️⃣ OCI Distribution Specification

Defines:

* How images are pushed/pulled
* Registry API behavior

Used by:

* Docker Hub
* ECR
* GCR
* Harbor

---

## 4. Where `runc` fits into OCI

* `runc` is the **reference implementation** of the OCI Runtime Spec
* It reads `config.json`
* It creates namespaces + cgroups
* It starts PID 1

```
OCI Runtime Spec
      ↓
    runc
      ↓
 Linux kernel
```

---

## 5. OCI bundle (very important concept)

An **OCI bundle** is a directory with:

```
bundle/
 ├── rootfs/
 └── config.json
```

* `rootfs` → container filesystem
* `config.json` → runtime instructions

containerd/Docker create this bundle.
`runc` consumes it.

---

## 6. Full container creation flow (OCI-based)

```
Image (OCI Image Spec)
   ↓
Unpacked to rootfs
   ↓
OCI bundle created
   ↓
config.json generated (OCI Runtime Spec)
   ↓
runc create
   ↓
(namespaces + cgroups)
   ↓
runc start
   ↓
PID 1 running
```

---

## 7. OCI vs Docker vs Kubernetes (clear separation)

| Layer      | Responsibility              |
| ---------- | --------------------------- |
| OCI        | Standards                   |
| runc       | Executes OCI runtime        |
| containerd | Manages container lifecycle |
| Docker     | UX + tooling                |
| Kubernetes | Orchestration               |

OCI sits **below everything**.

---

## 8. OCI vs CRI (very common confusion)

| OCI                      | CRI                             |
| ------------------------ | ------------------------------- |
| Open standard            | Kubernetes interface            |
| Defines container format | Defines kubelet ↔ runtime       |
| Used by all runtimes     | Kubernetes-only                 |
| Implemented by runc      | Implemented by containerd/CRI-O |

> Kubernetes uses **CRI**, which internally uses **OCI runtimes**.

---

## 9. OCI does NOT replace anything

OCI:

* ❌ Not a runtime
* ❌ Not a daemon
* ❌ Not Kubernetes
* ❌ Not Docker

OCI ensures **everything works together**.

---

## 10. One-page mental model (gold standard)

```
OCI Image Spec  → what an image looks like
OCI Runtime Spec → how a container must run
OCI Distribution → how images move

Docker/containerd → prepare OCI bundle
runc              → execute OCI runtime
Kernel            → enforce isolation & limits
```

---

## 11. Interview-ready one-liner

> **OCI is the standard; runc implements it; containerd manages it; Docker and Kubernetes orchestrate it.**

---

## 12. Why OCI matters in production

* Freedom to change runtimes
* No vendor lock-in
* Kubernetes compatibility
* Secure, auditable runtime behavior

---

## 13. Key Takeaways

* OCI = standards, not software
* `runc` = OCI runtime implementation
* Images and runtimes are portable
* Kubernetes & Docker rely on OCI

---

