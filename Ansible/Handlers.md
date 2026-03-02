# Ansible Handlers: flush_handlers and force_handlers (Complete Guide)

This document explains **everything in detail** about:

* What handlers are
* How handlers work internally
* How and when handlers execute
* `flush_handlers`
* `force_handlers`
* Real-world examples
* Common mistakes and best practices

This is **README.md ready** and suitable for **assignments, interviews, and production usage**.

---

## 1. What Are Handlers in Ansible?

Handlers are **special tasks** that run **only when notified** by another task.

They are commonly used for:

* Restarting services
* Reloading daemons
* Applying configuration changes safely

### Key Characteristics

* Handlers run **at the end of a play by default**
* A handler runs **only once**, even if notified multiple times
* Handlers run **only if at least one task changed**

---

## 2. Basic Handler Example

```yaml
---
- name: Basic handler example
  hosts: web
  become: yes

  tasks:
    - name: Update Apache config
      copy:
        src: httpd.conf
        dest: /etc/httpd/conf/httpd.conf
      notify: Restart Apache

  handlers:
    - name: Restart Apache
      service:
        name: httpd
        state: restarted
```

### Execution Flow

1. Task runs
2. If task reports **changed**, handler is notified
3. Handler runs **after all tasks finish**

---

## 3. Why flush_handlers Is Needed

Sometimes you need the handler to run **immediately**, not at the end.

Example scenarios:

* Config file must be reloaded before next task
* Service restart required mid-play
* Sequential dependency between tasks

This is where `flush_handlers` is used.

---

## 4. Using flush_handlers

### Syntax

```yaml
- meta: flush_handlers
```

This forces **all notified handlers** to run **immediately at that point** in the play.

---

## 5. flush_handlers Example (Step-by-Step)

```yaml
---
- name: flush_handlers demo
  hosts: web
  become: yes

  tasks:
    - name: Update Apache config
      copy:
        src: httpd.conf
        dest: /etc/httpd/conf/httpd.conf
      notify: Restart Apache

    - name: Flush handlers now
      meta: flush_handlers

    - name: Verify Apache status
      command: systemctl status httpd

  handlers:
    - name: Restart Apache
      service:
        name: httpd
        state: restarted
```

### Execution Order

1. Config updated
2. Handler notified
3. `flush_handlers` executes handler immediately
4. Apache is restarted
5. Status check runs

---

## 6. What Is force_handlers?

By default:

* If a task fails → handlers **do not run**

`force_handlers` changes this behavior.

### force_handlers Ensures:

* Handlers **always run**, even if:

  * A task fails
  * The play stops early
  * `any_errors_fatal` is set

---

## 7. Using force_handlers

### Syntax (Play Level)

```yaml
- hosts: web
  become: yes
  force_handlers: true
```

---

## 8. force_handlers Example

```yaml
---
- name: force_handlers demo
  hosts: web
  become: yes
  force_handlers: true

  tasks:
    - name: Update Apache config
      copy:
        src: httpd.conf
        dest: /etc/httpd/conf/httpd.conf
      notify: Restart Apache

    - name: Simulate failure
      command: /bin/false

  handlers:
    - name: Restart Apache
      service:
        name: httpd
        state: restarted
```

### Result

* Task fails
* Play stops
* **Handler still executes**

---

## 9. flush_handlers vs force_handlers (Comparison)

| Feature          | flush_handlers      | force_handlers   |
| ---------------- | ------------------- | ---------------- |
| Execution timing | Immediate           | End of play      |
| Runs on failure  | ❌ No                | ✅ Yes            |
| Scope            | Task-level          | Play-level       |
| Use case         | Mid-play dependency | Cleanup / safety |

---

## 10. Combined Example (Real-World)

```yaml
---
- name: Real-world handler control
  hosts: web
  become: yes
  force_handlers: true

  tasks:
    - name: Deploy config
      template:
        src: app.conf.j2
        dest: /etc/app/app.conf
      notify: Restart App

    - name: Flush handlers before migration
      meta: flush_handlers

    - name: Run DB migration
      command: /usr/local/bin/migrate-db

  handlers:
    - name: Restart App
      service:
        name: app
        state: restarted
```

### Why This Matters

* App restarted **before migration**
* Restart happens **even if migration fails**

---

## 11. Common Mistakes

❌ Expecting handler to run without `notify`

❌ Using `flush_handlers` without any notified handlers

❌ Assuming handlers run immediately by default

❌ Forgetting `force_handlers` during failure-sensitive operations

---

## 12. Best Practices

* Use handlers only for **state changes**
* Keep handlers **idempotent**
* Use `flush_handlers` sparingly
* Enable `force_handlers` for cleanup tasks
* Name handlers clearly

---

## 13. Interview Quick Answers

**Q: When do handlers run?**
A: End of the play unless flushed

**Q: Can handlers run on failure?**
A: Yes, with `force_handlers: true`

**Q: Can a handler run multiple times?**
A: No, only once per play

---

## Final Summary

* `notify` triggers handlers
* `flush_handlers` forces immediate execution
* `force_handlers` guarantees execution on failure
* Together they provide **fine-grained control** over service management

---

Happy Automating with Ansible 🚀
