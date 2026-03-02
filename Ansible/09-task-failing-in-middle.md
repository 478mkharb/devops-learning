# What Happens If a Task Fails in the Middle of a Play? (Ansible)

When a task fails in Ansible, **execution behavior depends on scope, settings, and error-handling directives**.

> 🧠 **Default rule**: If a task fails on a host, **Ansible stops executing further tasks for that host**, but continues for other hosts.

---

## 🔴 Default Behavior (Most Important)

### Scenario

Inventory:

```ini
[web]
web1
web2
```

Playbook:

```yaml
- name: Configure web servers
  hosts: web
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present

    - name: Start nginx (this fails on web1)
      service:
        name: nginx
        state: started

    - name: Deploy app
      copy:
        src: app/
        dest: /var/www/html/
```

### What Happens?

| Host | Result                 |
| ---- | ---------------------- |
| web1 | ❌ Stops at failed task |
| web2 | ✅ Continues all tasks  |

✔ Remaining tasks are **skipped only for the failed host**, not globally.

---

## 🔄 Visual Execution Flow

```
web1: task1 ✅ → task2 ❌ → STOP
web2: task1 ✅ → task2 ✅ → task3 ✅
```

---

## 🚫 Play Does NOT Stop by Default

❌ Common misconception:

> "If one task fails, the whole play stops"

✔ Reality:

* Ansible is **host-oriented**
* Failure is **isolated per host**

---

## 🟡 Using `ignore_errors`

### Allow Play to Continue Even After Failure

```yaml
- name: Run risky command
  command: /opt/may_fail.sh
  ignore_errors: true
```

### Result

* Task marked as **FAILED**
* Execution **continues for the same host**

---

## 🟠 Using `failed_when`

Control *when* a task should be considered failed:

```yaml
- name: Health check
  shell: curl -s http://localhost/health
  register: health
  failed_when: "'DOWN' in health.stdout"
```

Useful when exit codes are unreliable.

---

## 🟢 Using `block / rescue / always`

### Graceful Error Handling (Try/Catch Style)

```yaml
- block:
    - name: Restart app
      service:
        name: myapp
        state: restarted

  rescue:
    - name: Roll back config
      copy:
        src: backup.conf
        dest: /etc/myapp/app.conf

  always:
    - name: Notify
      debug:
        msg: "App restart attempted"
```

### Behavior

* `block` fails → `rescue` runs
* `always` runs **no matter what**

---

## ⛔ Stop the Entire Play: `any_errors_fatal`

```yaml
- name: Critical operation
  hosts: web
  any_errors_fatal: true
  tasks:
    - name: Update shared config
      copy:
        src: config.yml
        dest: /etc/app/config.yml
```

### Result

❌ Failure on **any host** → **entire play stops**

---

## 🧨 Stop After N Failures: `max_fail_percentage`

```yaml
- name: Rolling update
  hosts: web
  max_fail_percentage: 30
```

If more than 30% hosts fail → play stops.

---

## 📊 Summary Table

| Situation                | What Happens                |
| ------------------------ | --------------------------- |
| Task fails (default)     | Host stops, others continue |
| `ignore_errors: true`    | Task fails, host continues  |
| `block/rescue`           | Failure handled gracefully  |
| `any_errors_fatal: true` | Play stops completely       |
| `max_fail_percentage`    | Stops after threshold       |

---

## 🧠 One-Line Summary (Interview Gold)

> **By default, a failed task stops execution only for that host, not the entire play.**

---

## 🎯 Interview Tip

If interviewer asks:

> *"How do you ensure consistency when a task fails?"*

Answer:

* `block/rescue`
* `any_errors_fatal`
* `serial` + `max_fail_percentage`

If you want, I can also add:

* Failure behavior with `serial`
* Failure with `delegate_to`
* Real production failure patterns
* Failure flow diagram
