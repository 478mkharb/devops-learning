# Difference Between `when` and `failed_when` in Ansible

Both `when` and `failed_when` are **conditional controls** in Ansible, but they serve **very different purposes**.

> 🧠 **Key idea**:

* `when` decides **whether a task should run**
* `failed_when` decides **whether a task should be marked as failed after it runs**

---

## 🔍 `when` — Conditional Task Execution

### What it does

`when` **skips the task entirely** if the condition is false.

* Task is **not executed**
* No command is run
* Status shows as **SKIPPED**

### Syntax

```yaml
- name: Run only on Ubuntu
  apt:
    name: nginx
    state: present
  when: ansible_os_family == "Debian"
```

### Result

| Condition | Task Status |
| --------- | ----------- |
| True      | Executed    |
| False     | Skipped     |

---

## 🔥 `failed_when` — Conditional Failure

### What it does

`failed_when` **runs the task first**, then evaluates a condition to decide if the task **should be considered failed**.

* Task **always executes**
* You control **failure logic**
* Useful when command exit codes are unreliable

### Syntax

```yaml
- name: Check application health
  shell: curl -s http://localhost/health
  register: health_check
  failed_when: "'DOWN' in health_check.stdout"
```

### Result

| Condition | Task Status |
| --------- | ----------- |
| True      | FAILED      |
| False     | SUCCESS     |

---

## 🧪 Side-by-Side Example

### Scenario

* Check disk usage
* Skip check on test servers
* Fail only if usage > 80%

```yaml
- name: Check disk usage
  shell: df -h / | awk 'NR==2 {print $5}' | tr -d '%'
  register: disk_usage
  when: env != "test"
  failed_when: disk_usage.stdout | int > 80
```

### Execution Flow

1. `when` evaluated first
2. If false → task skipped
3. If true → command runs
4. `failed_when` evaluated after execution

---

## 🧠 Execution Order (Important for Interviews)

```
when  →  task execution  →  failed_when
```

---

## 🚫 Common Mistake

❌ Expecting `failed_when` to skip a task

```yaml
failed_when: some_condition
```

➡️ The task will still run.

---

## ✅ Using Both Together (Very Common Pattern)

```yaml
- name: Validate config file
  command: nginx -t
  register: nginx_test
  when: nginx_installed
  failed_when: nginx_test.rc != 0
```

---

## 📊 Comparison Table

| Feature            | `when`            | `failed_when`         |
| ------------------ | ----------------- | --------------------- |
| Controls execution | ✅ Yes             | ❌ No                  |
| Controls failure   | ❌ No              | ✅ Yes                 |
| Task runs?         | Only if true      | Always                |
| Evaluated          | Before execution  | After execution       |
| Common use         | OS/env conditions | Custom error handling |

---

## ⭐ Real-World Use Cases

### `when`

* OS-specific tasks
* Environment-based execution (prod/dev)
* Feature flags

### `failed_when`

* Health checks
* API calls with 200 but error in body
* Commands returning non-standard exit codes

---

## 🧠 One-Line Summary (Interview Gold)

> **`when` decides if a task should run, `failed_when` decides if a task should fail.**

If you want, I can also add:

* `changed_when` vs `failed_when`
* `ignore_errors` vs `failed_when`
* Complex expressions with `register`
* Interview trick questions with answers
