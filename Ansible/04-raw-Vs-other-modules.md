# ⚔️ Ansible `raw` Module vs Normal Modules (Complete Concept)

This document explains **what the `raw` module is**, **how it differs from normal Ansible modules**, **why it exists**, and **which modules are idempotent or not**. Written for deep understanding and interviews.

---

## 🧩 What is the `raw` Module?

The **`raw` module** runs a **plain shell command directly on the target host** over SSH.

Think of it as:

> ⚠️ *"Just run this command — don’t ask questions."*

```yaml
- name: Start nginx using raw
  ansible.builtin.raw: systemctl start nginx
```

No module logic. No state check. No JSON intelligence.

---

## 🧠 What Are Normal Ansible Modules?

Normal modules (like `package`, `copy`, `service`) are **state-aware programs**.

They:

* Inspect current system state 👀
* Compare with desired state 🎯
* Change only if needed 🔁
* Return structured JSON 📦

```yaml
- name: Ensure nginx is running
  ansible.builtin.service:
    name: nginx
    state: started
```

---

## ⚙️ How They Work Internally (Key Difference)

### 🧠 Normal Module Execution

* Wrapped into **AnsiballZ** 🧩
* Copied to target via SSH
* Executed with Python 🐍
* Performs state inspection
* Returns JSON (`changed`, `failed`)
* Temporary files cleaned

---

### ⚠️ raw Module Execution

* No AnsiballZ
* No Python required
* No module copy
* Just SSH + command execution

Internally:

```bash
ssh host "systemctl start nginx"
```

---

## 🔍 Why Does the `raw` Module Exist?

The `raw` module exists for **special situations**:

✔️ Bootstrapping new servers (Python not installed)
✔️ Minimal OS images
✔️ Recovery / broken systems
✔️ Installing Python so Ansible can work

Example (very common):

```yaml
- name: Install Python on fresh server
  ansible.builtin.raw: apt-get install -y python3
```

➡️ After this, switch back to normal modules.

---

## 🔁 Idempotency Explained (Very Important)

### ✅ Idempotent Modules (Safe to re-run)

These modules **check state before acting**:

* `package`
* `service`
* `copy`
* `file`
* `user`
* `template`

Running them multiple times = same result.

---

### ❌ Non-Idempotent by Default

These modules **do not guarantee idempotency**:

* `raw`
* `command`
* `shell`

Why?

* They execute commands blindly
* They rely on exit codes, not state

Example:

```yaml
- shell: echo hello >> file.txt
```

➡️ File grows every run ❌

---

## ⚠️ Can `command` / `shell` Be Made Idempotent?

🟡 Yes, but **manually**:

```yaml
- shell: touch /tmp/app.lock
  args:
    creates: /tmp/app.lock
```

But this is still weaker than real modules.

---

## 📊 Side-by-Side Comparison

| Feature            | 🧠 Normal Modules | ⚠️ raw Module |
| ------------------ | ----------------- | ------------- |
| Uses AnsiballZ     | ✅ Yes             | ❌ No          |
| Python required    | ✅ Yes             | ❌ No          |
| State inspection   | ✅ Yes             | ❌ No          |
| Idempotent         | ✅ Yes             | ❌ No          |
| Returns JSON       | ✅ Yes             | ❌ No          |
| Handlers supported | ✅ Yes             | ❌ No          |
| Check mode works   | ✅ Yes             | ❌ No          |

---

## 🧠 Mental Model (Remember This)

```text
Normal Module = check → compare → act → report
raw Module    = run → hope
```

---

## 🎯 Interview-Perfect Summary

> **“Normal Ansible modules are state-aware and idempotent, executed via the AnsiballZ wrapper and returning structured JSON. The raw module bypasses this mechanism to execute commands directly over SSH, making it suitable only for bootstrapping or recovery scenarios where Python or modules cannot be used.”**

---

🟢 **Rule of Thumb:**
Use `raw` only to make Ansible usable — then stop using it.
