# How Ansible Detects Configuration Drift 🔍

## What is Configuration Drift?

**Configuration drift** happens when the actual state of a system **diverges from the desired state** defined in configuration management.

Examples:

* A package manually upgraded on a server
* A config file edited by hand
* A service stopped outside automation

Over time, servers that *should be identical* become different.

---

## Short Answer (Interview‑Ready)

> **Ansible detects configuration drift by comparing the desired state defined in playbooks with the current state of the target system during execution, using idempotent modules and reporting changes or differences.**

---

## Core Concept: Desired State + Idempotency

Ansible does **not continuously monitor** systems.

Instead, it detects drift **when a playbook is run**.

Key idea:

* You declare the **desired state**
* Ansible checks the **current state**
* If they differ → drift is detected

This works because most Ansible modules are **idempotent**.

---

## How Idempotent Modules Detect Drift

Each idempotent module:

1. Reads the current system state
2. Compares it with the desired state
3. Decides:

   * ✅ No change needed → `ok`
   * 🔄 Change needed → `changed`

That comparison step is **drift detection**.

---

## Example 1: Package Drift Detection 📦

### Desired State

```yaml
- name: Ensure nginx is installed
  yum:
    name: nginx
    state: present
```

### Drift Scenario

* Someone manually removes nginx

### Playbook Result

```text
changed: [web01]
```

👉 Ansible detected drift and corrected it.

---

## Example 2: Service Drift Detection 🔁

### Desired State

```yaml
- name: Ensure nginx is running
  service:
    name: nginx
    state: started
    enabled: true
```

### Drift Scenario

* Service stopped manually

### Result

```text
changed: [web01]
```

---

## Example 3: Configuration File Drift 📝

### Desired State

```yaml
- name: Enforce nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
```

### Drift Scenario

* File edited manually on the server

---

## 🔐 How Checksum Rule Is Implemented (Key Internal Mechanism)

When Ansible manages files (via `copy`, `template`, `lineinfile`, etc.), it uses a **checksum comparison algorithm** to detect drift.

### Step‑by‑Step Internal Flow

1️⃣ **Read current file on target host**
Ansible calculates a checksum (SHA1 by default) of the existing file:

```text
Current file → SHA1 checksum
```

2️⃣ **Generate desired file content**

* For `template`: Jinja2 is rendered
* For `copy`: source file is read

```text
Rendered content → SHA1 checksum
```

3️⃣ **Compare checksums**

```text
If current_checksum == desired_checksum → OK (no drift)
If current_checksum != desired_checksum → CHANGED (drift detected)
```

4️⃣ **Apply correction if needed**
If drift exists:

* File is replaced atomically
* Permissions/ownership reapplied

---

### Why Ansible Uses Checksums (Not Timestamps)

| Method     | Reason                                        |
| ---------- | --------------------------------------------- |
| Checksums  | Content‑accurate, reliable                    |
| Timestamps | Unreliable, can change without content change |
| File size  | Too weak                                      |

👉 This ensures **true drift detection**, not false positives.

---

### Checksum Example (Conceptual)

```text
Desired file checksum:  a94a8fe5ccb19ba61c4c0873d391e987
Current file checksum:  8b1a9953c4611296a827abf8c47804d7

→ Drift detected
```

---

### What Happens in `--check` Mode

* Checksums are still calculated
* Comparison still happens
* ❌ File is NOT replaced
* ✅ Drift is reported as `changed`

```bash
ansible-playbook site.yml --check --diff
```

Perfect for **audits and compliance scans**.

---

### Important Notes ⚠️

* Checksums are calculated **on the remote host**
* Binary and text files are handled the same way
* Large files may impact performance
* Non‑idempotent modules may skip checksum logic

---

### Mental Model 🧠

> **Ansible does not ask “Did this file change?”**
> **It asks “Is the file exactly what I declared?”**

---

## Check Mode: Drift Detection Without Changes 🔎

Ansible can detect drift **without fixing it** using check mode.

```bash
ansible-playbook site.yml --check
```

Output:

* `changed` → drift exists
* `ok` → no drift

📌 Useful for audits and compliance.

---

## Diff Mode: See Exactly What Drifted 🧾

```bash
ansible-playbook site.yml --diff
```

Shows:

* Before vs after
* Line‑by‑line config differences

Very useful for:

* Debugging
* Reviews
* Compliance evidence

---

## Drift Detection vs Drift Prevention

| Aspect                    | Ansible |
| ------------------------- | ------- |
| Continuous monitoring     | ❌ No    |
| Detects drift on run      | ✅ Yes   |
| Automatically fixes drift | ✅ Yes   |
| Declarative desired state | ✅ Yes   |

👉 Ansible is **reactive**, not continuous.

---

## Real‑World Drift Detection Workflow 🏗️

```text
Desired State (Playbooks)
        ↓
Run Ansible
        ↓
Compare current vs desired
        ↓
Report drift (changed)
        ↓
Optionally fix drift
```

---

## Common Drift Sources (Reality Check)

* SSH manual changes
* Emergency hotfixes
* OS auto‑updates
* Partial playbook runs
* Human error

---

## Best Practices to Control Drift 🧠

✔ Run Ansible regularly (cron / CI)
✔ Use check mode for audits
✔ Avoid manual changes
✔ Enforce configs via templates
✔ Combine with monitoring alerts

---

## Important Limitation ⚠️

Ansible **cannot detect drift** if:

* You never run the playbook
* The resource is unmanaged
* A module is not idempotent

---

## Interview One‑Liner 🎯

> **Ansible detects configuration drift by re‑evaluating system state during playbook execution and reporting differences through idempotent modules, rather than continuously monitoring systems.**

---

## Summary

* Drift = deviation from desired state
* Ansible detects drift at runtime
* Idempotency is the key mechanism
* `--check` and `--diff` enhance visibility
* Regular runs are essential

---

If you want next:

* ⚖️ Ansible vs Puppet drift handling
* 🔄 Continuous drift detection patterns
* 🚀 CI‑based compliance checks
* 🎤 Interview slide version
