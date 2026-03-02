# Difference Between `command` and `shell` Modules in Ansible

Both `command` and `shell` are used to run commands on remote hosts, but **they are NOT the same**.

> 🧠 **Golden rule**:

* Use **`command` by default**
* Use **`shell` only when you need shell features**

---

## 🧩 Key Difference (One Line)

| Module    | How it runs                                  |
| --------- | -------------------------------------------- |
| `command` | Runs command **directly** (no shell)         |
| `shell`   | Runs command **through a shell** (`/bin/sh`) |

---

## 🔒 `command` Module (Safer & Preferred)

### What it does

* Executes binaries **without invoking a shell**
* No variable expansion, pipes, redirects, or wildcards
* More secure and predictable

### Example

```yaml
- name: Check uptime
  command: uptime
```

❌ This will FAIL:

```yaml
- command: ls *.log
```

---

## 🔥 `shell` Module (Powerful but Risky)

### What it does

* Executes command **inside a shell**
* Supports:

  * Pipes (`|`)
  * Redirects (`>`, `>>`)
  * Logical operators (`&&`, `||`)
  * Environment variables

### Example

```yaml
- name: Find error logs
  shell: cat /var/log/app.log | grep ERROR > /tmp/errors.txt
```

---

## 🧪 Side-by-Side Example

### Goal

Count number of running nginx processes

```yaml
# ❌ Won't work
- name: Count nginx processes
  command: ps aux | grep nginx | wc -l

# ✅ Works
- name: Count nginx processes
  shell: ps aux | grep nginx | wc -l
```

---

## ⚠️ Security Difference (Very Important)

### `shell` is vulnerable to injection

```yaml
shell: rm -rf {{ user_input }}
```

If `user_input` is unsafe → 💥

### `command` is safer

```yaml
command:
  argv:
    - rm
    - -rf
    - /tmp/mydir
```

---

## 🚀 Idempotency Helpers

Both support:

```yaml
creates: /path/file
removes: /path/file
```

Example:

```yaml
- name: Run once
  shell: echo "done" > /tmp/flag
  creates: /tmp/flag
```

---

## 🧠 Execution Summary

| Feature            | command   | shell             |
| ------------------ | --------- | ----------------- |
| Uses shell         | ❌ No      | ✅ Yes             |
| Pipes / redirects  | ❌ No      | ✅ Yes             |
| Wildcards          | ❌ No      | ✅ Yes             |
| Variable expansion | ❌ No      | ✅ Yes             |
| Security           | ✅ Safer   | ⚠️ Risky          |
| Recommended        | ✅ Default | ⚠️ Only if needed |

---

## ⭐ Real-World Usage Guidelines

### Use `command` when:

* Running a single binary
* Predictability matters
* Security is important

### Use `shell` when:

* You need pipes or redirects
* You need `&&`, `||`
* You rely on shell features

---

## 🧠 One-Line Summary (Interview Gold)

> **`command` runs without a shell and is safer; `shell` runs through a shell and supports pipes and redirects.**

---

## 🎯 Interview Trap Question

> *"Why is `shell` discouraged?"*

✔ Because it invokes a shell, making it less secure and less predictable.

If you want, I can also add:

* `command` vs `raw`
* `shell` with `executable`
* Best practices to replace shell with modules
