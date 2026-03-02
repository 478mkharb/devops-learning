# Ansible `assert` vs `fail` Modules – Complete Guide ✅❌

Both `assert` and `fail` are **control / validation modules** in Ansible, used to **stop playbook execution intentionally** when conditions are not met.

They look similar, but they are used for **different purposes**.

---

## High-Level Difference (One Look)

| Aspect           | `assert`                          | `fail`                    |
| ---------------- | --------------------------------- | ------------------------- |
| Purpose          | Validate assumptions / conditions | Forcefully stop execution |
| Condition based  | ✅ Yes                             | ❌ No (manual logic)       |
| Best used for    | Pre-checks, validations           | Hard stop on errors       |
| Semantic meaning | "This must be true"               | "Stop here now"           |
| Readability      | Declarative                       | Imperative                |

---

## What is the `assert` Module?

The **`assert` module** is used to **verify that a condition is true**.

If the condition evaluates to **false**, Ansible:

* Marks the task as **FAILED**
* Stops execution (unless handled)

Think of `assert` as:

> 🧠 *A guardrail that ensures expectations are met before continuing.*

---

### Example 1: Basic Assert

```yaml
- name: Ensure memory is at least 4GB
  assert:
    that:
      - ansible_memtotal_mb >= 4096
    fail_msg: "❌ Not enough memory"
    success_msg: "✅ Memory requirement met"
```

📌 If memory < 4GB → playbook fails immediately.

---

### Example 2: Assert Multiple Conditions

```yaml
- name: Validate OS and version
  assert:
    that:
      - ansible_os_family == "RedHat"
      - ansible_distribution_major_version | int >= 8
```

✔ Clean, readable **pre-flight validation**

---

## When to Use `assert`

Use `assert` when:

* You want to **validate environment assumptions**
* You are writing **reusable roles**
* You want failures to clearly say *what expectation failed*

Common use cases:

* OS checks
* Version checks
* Variable sanity checks
* Feature flags

---

## What is the `fail` Module?

The **`fail` module** is used to **explicitly stop execution** with a custom message.

It does **not evaluate conditions itself** — you control *when* it runs using `when`.

Think of `fail` as:

> 🚨 *An emergency brake you pull intentionally.*

---

### Example 1: Basic Fail

```yaml
- name: Stop playbook execution
  fail:
    msg: "❌ This environment is not supported"
```

✔ Always fails when executed.

---

### Example 2: Fail with Condition (Most Common)

```yaml
- name: Stop if running on production
  fail:
    msg: "❌ This playbook must not run on production"
  when: env == "prod"
```

📌 Logic lives in `when`, not in `fail`.

---

## When to Use `fail`

Use `fail` when:

* A condition is **business / policy driven**
* You want a **hard stop**
* Logic is complex or already computed

Common use cases:

* Preventing prod changes
* License / compliance checks
* Manual safety checks
* Guarding destructive actions

---

## Side-by-Side Example (Very Important)

### Using `assert` (Validation Style)

```yaml
- name: Validate input variable
  assert:
    that:
      - app_port is defined
      - app_port | int > 1024
```

### Using `fail` (Control Style)

```yaml
- name: Stop if app_port is invalid
  fail:
    msg: "❌ app_port must be greater than 1024"
  when: app_port is not defined or app_port | int <= 1024
```

👉 Same outcome, **different intent**.

---

## Key Conceptual Difference (Interview Gold 🥇)

* `assert` expresses **expectations**
* `fail` expresses **decisions**

This difference matters a lot in **clean playbook design**.

---

## Best Practices 🧠

✔ Use `assert` for:

* Role inputs
* Preconditions
* Sanity checks

✔ Use `fail` for:

* Safety stops
* Policy enforcement
* Destructive guardrails

❌ Don’t use `fail` where `assert` is clearer
❌ Don’t hide validations deep inside logic

---

## Quick Decision Rule

> **If you are validating assumptions → use `assert`**
> **If you are intentionally stopping execution → use `fail`**

---

## Interview One-Liner 🎯

> **`assert` validates that conditions are true and fails if expectations are not met, while `fail` is used to explicitly stop execution based on custom logic or policy decisions.**

---

## Summary

* Both stop execution
* `assert` = validation mindset
* `fail` = control & safety mindset
* Choosing correctly improves readability and maintainability

---

If you want next:

* ⚖️ `assert` vs `when` vs `failed_when`
* 🧠 Role design best practices
* 🚀 Real-world production guard examples
* 🎤 Interview slide version
