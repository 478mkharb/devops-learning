# How Ansible Maintains State Without a State File

This canvas explains the **full concept** of how Ansible maintains state **without storing any state file**, and how this is fundamentally different from **Terraform**.

---

## 1️⃣ Key Concept First (Very Important)

### ❌ Ansible does NOT have a state file

* No `state.tf`
* No central state database
* No stored desired-vs-actual snapshot

### ✅ Ansible is **state-aware**, not **state-storing**

It **re-discovers the current state every time it runs** by inspecting the target system.

---

## 2️⃣ How Ansible Maintains State (Runtime Model)

Ansible maintains correctness using **three mechanisms**:

1. **Live system inspection**
2. **Idempotent modules**
3. **Desired state declaration**

There is **no memory between runs**.

---

## 3️⃣ Step-by-Step: How Ansible Knows the State (Example)

### Desired State (Playbook)

```yaml
- name: Ensure nginx is installed
  ansible.builtin.package:
    name: nginx
    state: present
```

---

### What Ansible Does at Runtime

#### Step 1: Connect to target host

* Uses SSH
* Copies a temporary Python module
* Executes it remotely

#### Step 2: Inspect current state

The module queries the system directly.

Examples (internally):

* Ubuntu → `dpkg -s nginx`
* RHEL → `rpm -q nginx`

#### Step 3: Compare with desired state

| Current State | Desired State | Action     |
| ------------- | ------------- | ---------- |
| Installed     | present       | Do nothing |
| Not installed | present       | Install    |

#### Step 4: Act only if mismatch

* If mismatch → change system
* If match → exit safely

#### Step 5: Return result

```json
{
  "changed": false
}
```

👉 This is how Ansible enforces state **without storing it**.

---

## 4️⃣ Another Example: File Permissions

### Desired State

```yaml
- name: Ensure directory permissions
  ansible.builtin.file:
    path: /opt/app
    state: directory
    mode: '0755'
```

### Runtime Check

* Does `/opt/app` exist?
* Is it a directory?
* Are permissions already `0755`?

Only if **any answer is NO**, Ansible applies a change.

---

## 5️⃣ What Happens If Someone Changes the Server Manually?

* Engineer runs: `chmod 777 /opt/app`
* Ansible is run again

✅ Ansible re-checks live state
✅ Detects drift
✅ Corrects it automatically

No state file needed.

---

## 6️⃣ Optional: Facts (Still Not a State File)

Ansible gathers facts at runtime:

```yaml
gather_facts: true
```

Facts include:

* OS type
* IP addresses
* CPU, memory

Stored temporarily in memory as:

```yaml
ansible_facts
```

⚠️ Facts are **runtime data**, not desired-state storage.

---

## 7️⃣ Terraform: Completely Different Model

Terraform uses a **persistent state file** to track infrastructure.

### Terraform State File

* `terraform.tfstate`
* Stored locally or remotely (S3, GCS, etc.)
* Maps **resources ↔ real-world objects**

---

## 8️⃣ Terraform Example

### Desired State

```hcl
resource "aws_instance" "web" {
  instance_type = "t3.micro"
}
```

Terraform compares:

* **State file** vs **real cloud**

If state file is lost or corrupted → Terraform is blind.

---

## 9️⃣ Ansible vs Terraform (Side-by-Side)

| Aspect          | Ansible                  | Terraform                   |
| --------------- | ------------------------ | --------------------------- |
| State storage   | ❌ None                   | ✅ State file                |
| State detection | Live system inspection   | State file comparison       |
| Drift detection | On every run             | Based on state refresh      |
| Failure risk    | No state corruption      | State file critical         |
| Best for        | Configuration management | Infrastructure provisioning |

---

## 🔟 How Ansible Confirms State (No State File, No Guessing)

Ansible confirms state **from module output**, not from a stored file.

### Key Rule

* ❌ Exit code ≠ state
* ✅ Structured JSON output = state confirmation

---

### Execution Flow (What Really Happens)

1. Ansible copies a Python module to the target host
2. Module inspects the live system
3. Module compares current vs desired state
4. Module returns **structured JSON** on STDOUT
5. Ansible core reads this JSON and decides the result

---

### Example: Package Module (nginx)

**Playbook**

```yaml
- name: Ensure nginx is installed
  ansible.builtin.package:
    name: nginx
    state: present
```

**Module logic (simplified)**

```text
IF nginx already installed
  changed = false
ELSE
  install nginx
  changed = true
```

**Returned JSON (no change)**

```json
{
  "changed": false,
  "msg": "nginx already installed"
}
```

---

### Role of Exit Codes

| Exit Code | Meaning                      |
| --------- | ---------------------------- |
| 0         | Module executed successfully |
| !=0       | Module crashed or failed     |

⚠️ Exit code only shows **execution success**, not configuration state.

---

### What Ansible Actually Uses to Decide State

| Field     | Purpose                                  |
| --------- | ---------------------------------------- |
| `changed` | Whether desired state differed           |
| `failed`  | Logical task failure                     |
| `rc`      | Command return code (shell/command only) |

---

### Why This Matters

* Enables idempotency
* Drives handlers (`notify`)
* Supports check mode (`--check`)
* Allows safe parallel execution

---

## 🔟 Why Ansible Was Designed This Way

* Stateless control node
* Safe for configuration management
* Works even with manual changes
* No shared lock or state corruption

Terraform **must** store state because cloud APIs do not expose full intent.

---

## 🎯 Interview-Perfect Summary

**“Ansible does not maintain a state file. It enforces state by inspecting the live system at runtime using idempotent modules and comparing it with the desired configuration. Terraform, in contrast, relies on a persistent state file to map and manage infrastructure resources.”**

---

✅ This conceptual difference explains **why Ansible and Terraform are complementary, not competitors**.
