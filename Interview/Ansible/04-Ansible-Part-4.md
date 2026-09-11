# Ansible Senior L2 Interview — Part 4

**Focus:** Failure handling, privilege and execution strategy

## Q19. What happens if a task fails in the middle of a play?

### Short explanation of the question

This tests whether I know Ansible's default failure behavior and the mechanisms for controlled recovery.

### Answer

By default, a failed task stops subsequent tasks for that host in the current play. Other hosts can continue unless broader failure controls are used. `block`/`rescue`/`always`, `ignore_errors`, `failed_when`, and failure policies provide controlled behavior.

### Detailed explanation

When a task fails in Ansible, **execution behavior depends on scope, settings, and error-handling directives**.

> 🧠 **Default rule**: If a task fails on a host, **Ansible stops executing further tasks for that host**, but continues for other hosts.

---

### Default Behavior (Most Important)

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

### Visual Execution Flow

```
web1: task1 ✅ → task2 ❌ → STOP
web2: task1 ✅ → task2 ✅ → task3 ✅
```

---

### Play Does NOT Stop by Default

❌ Common misconception:

> "If one task fails, the whole play stops"

✔ Reality:

* Ansible is **host-oriented**
* Failure is **isolated per host**

---

### Using `ignore_errors`

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

### Using `failed_when`

Control *when* a task should be considered failed:

```yaml
- name: Health check
  shell: curl -s http://localhost/health
  register: health
  failed_when: "'DOWN' in health.stdout"
```

Useful when exit codes are unreliable.

---

### Using `block / rescue / always`

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

### Stop the Entire Play: `any_errors_fatal`

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

### Stop After N Failures: `max_fail_percentage`

```yaml
- name: Rolling update
  hosts: web
  max_fail_percentage: 30
```

If more than 30% hosts fail → play stops.

---

### Summary Table

| Situation                | What Happens                |
| ------------------------ | --------------------------- |
| Task fails (default)     | Host stops, others continue |
| `ignore_errors: true`    | Task fails, host continues  |
| `block/rescue`           | Failure handled gracefully  |
| `any_errors_fatal: true` | Play stops completely       |
| `max_fail_percentage`    | Stops after threshold       |

---

### One-Line Summary (Interview Gold)

> **By default, a failed task stops execution only for that host, not the entire play.**

---

### Interview Tip

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

### Execution behavior

```text
Host A: Task 1 ✓ → Task 2 ✗ → later tasks skipped
Host B: Task 1 ✓ → Task 2 ✓ → continues
```

### Interviewer may cross-question

**Interviewer:** "How would you recover instead of simply ignoring the error?"

**Candidate:** "I would first classify the failure. For expected recoverable failures I can use `block`/`rescue`; for an acceptable non-zero result I can use `failed_when`; I use `ignore_errors` only when continuing is genuinely safe." 

---

## Q20. What is the difference between `assert` and `fail`?

### Short explanation of the question

This tests whether I use validation and deliberate stopping for their intended meanings.

### Answer

`assert` validates conditions and fails when they are false. `fail` deliberately stops execution with a failure message when the automation has determined that continuing is unsafe or inappropriate.

### Detailed explanation

Both `assert` and `fail` are **control / validation modules** in Ansible, used to **stop playbook execution intentionally** when conditions are not met.

They look similar, but they are used for **different purposes**.

---

### High-Level Difference (One Look)

| Aspect           | `assert`                          | `fail`                    |
| ---------------- | --------------------------------- | ------------------------- |
| Purpose          | Validate assumptions / conditions | Forcefully stop execution |
| Condition based  | ✅ Yes                             | ❌ No (manual logic)       |
| Best used for    | Pre-checks, validations           | Hard stop on errors       |
| Semantic meaning | "This must be true"               | "Stop here now"           |
| Readability      | Declarative                       | Imperative                |

---

### What is the `assert` Module?

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

### When to Use `assert`

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

### What is the `fail` Module?

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

### When to Use `fail`

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

### Side-by-Side Example (Very Important)

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

### Key Conceptual Difference (Interview Gold 🥇)

* `assert` expresses **expectations**
* `fail` expresses **decisions**

This difference matters a lot in **clean playbook design**.

---

### Best Practices

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

### Quick Decision Rule

> **If you are validating assumptions → use `assert`**
> **If you are intentionally stopping execution → use `fail`**

---

### Interview One-Liner 🎯

> **`assert` validates that conditions are true and fails if expectations are not met, while `fail` is used to explicitly stop execution based on custom logic or policy decisions.**

---

### Summary

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

### Comparison

| `assert` | `fail` |
|---|---|
| Validates conditions | Deliberately fails |
| Good for preconditions | Good for explicit stop conditions |

### Example

```yaml
- name: Validate supported OS
  ansible.builtin.assert:
    that:
      - ansible_facts.os_family in ['Debian', 'RedHat']
```

---

## Q21. What is the difference between `meta: flush_handlers` and `force_handlers`?

### Short explanation of the question

This tests a subtle handler distinction: when pending handlers run versus what happens to notified handlers after a failure.

### Answer

`meta: flush_handlers` runs pending notified handlers at that point in the play. `force_handlers` changes failure behavior so notified handlers can still run even if the host later encounters a failure.

### Detailed explanation

Handlers are a **special Ansible mechanism** that run tasks **only when notified**. By default, handlers run **at the end of a play**.

This document explains:

* Why handlers exist
* What `meta: flush_handlers` does
* What `force_handlers` does
* The **real difference** between them
* Practical examples & use cases

---

### Quick Refresher: What Are Handlers?

Handlers are tasks that:

* Run only when **notified**
* Run **once**, even if notified multiple times
* Run **at the end of a play** by default

Example:

```yaml
- name: Restart nginx
  service:
    name: nginx
    state: restarted
  listen: restart nginx
```

---

### Default Handler Behavior (Important)

```text
Tasks run
→ Handlers are queued
→ Play ends
→ Handlers execute
```

📌 This default behavior is intentional:

* Avoid repeated restarts
* Improve efficiency
* Keep playbooks predictable

---

### What is `meta: flush_handlers`?

### Why is `meta:` used here? (Very Important)

`flush_handlers` is **not a normal task** like `service`, `copy`, or `command`.

It is a **playbook control instruction** that tells Ansible to change its **internal execution behavior**.

That is why it is executed using the **`meta` keyword**.

👉 Think of `meta` as:

> 🧠 "Tell Ansible *how* to run the playbook, not *what* to run on the host"

---

### What Does `meta` Mean in Ansible?

The `meta` module is used for **special Ansible engine operations**, such as:

* 🔁 Flushing handlers
* 🔄 Ending a play early
* ⏭️ Skipping remaining tasks
* 🎯 Resetting host failures

These actions:

* Do **not** run on the remote host
* Do **not** use SSH
* Directly affect Ansible’s **execution flow**

---

### Why `flush_handlers` Cannot Be a Normal Module

Handlers are:

* Queued internally by Ansible
* Executed only at specific times

To run them immediately, Ansible must:

1. Pause normal task execution
2. Jump to the handler queue
3. Execute all pending handlers
4. Resume tasks

This requires **control over Ansible’s execution engine**, not a host-level operation.

👉 Hence:

```yaml
- meta: flush_handlers
```

and NOT something like:

```yaml
- flush_handlers: true   # ❌ invalid
```

---

### Simple Mental Model

| Type        | Example           | Runs Where            |
| ----------- | ----------------- | --------------------- |
| Normal task | `service`, `copy` | On remote host        |
| Handler     | `restart nginx`   | On remote host        |
| `meta` task | `flush_handlers`  | Inside Ansible engine |

---

### Definition (Clean & Interview-Ready)

> **`meta: flush_handlers` is used because flushing handlers is an internal Ansible control operation, not a host-level task, and must be executed through the Ansible execution engine.**

---

### Example: Why `flush_handlers` Is Needed

```yaml
- name: Update nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: restart nginx

- meta: flush_handlers

- name: Validate nginx config
  command: nginx -t
```

### What Happens Here?

1. Config changes → handler is notified
2. `flush_handlers` runs
3. Nginx is restarted immediately
4. Validation runs against the **new config**

❌ Without `flush_handlers`, validation might fail or test old state.

---

### When to Use `meta: flush_handlers`

Use it when:

* A handler’s effect is **required immediately**
* Later tasks depend on the handler’s result

Common use cases:

* Reloading services before validation
* Applying firewall rules before connectivity tests
* Restarting apps before health checks

---

### What is `force_handlers`?

### Definition

`force_handlers` ensures that **handlers run even if the play fails or a host errors out**.

By default:

* If a task fails → handlers **do not run**

With `force_handlers: true`:

> 🔥 Handlers will run **no matter what**

---

### Example: `force_handlers`

```yaml
- hosts: web
  force_handlers: true
  tasks:
    - name: Update nginx config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: restart nginx

    - name: Simulate failure
      command: /bin/false
```

### What Happens?

* Task fails ❌
* Play stops
* **Handler STILL runs** ✅

Without `force_handlers`, handler would be skipped.

---

### Why `force_handlers` Exists

Some handlers perform **cleanup or safety actions**, such as:

* Restarting a service after partial config changes
* Reloading firewall rules
* Cleaning temp files
* Restoring a safe state

You want these to run **even on failure**.

---

### Key Difference (Most Important Section)

| Aspect                  | `flush_handlers`     | `force_handlers`             |
| ----------------------- | -------------------- | ---------------------------- |
| Purpose                 | Run handlers early   | Run handlers even on failure |
| Timing                  | Immediate (mid-play) | End of play (guaranteed)     |
| Triggered by            | `meta` task          | Play-level option            |
| Solves                  | Dependency timing    | Failure safety               |
| Affects execution order | ✅ Yes                | ❌ No                         |

👉 They solve **different problems**.

---

### Can They Be Used Together?

Yes — and sometimes they should be.

Example:

```yaml
- hosts: web
  force_handlers: true
  tasks:
    - name: Update config
      template:
        src: app.conf.j2
        dest: /etc/app.conf
      notify: restart app

    - meta: flush_handlers

    - name: Run smoke tests
      command: curl http://localhost/health
```

✔ Handler runs immediately
✔ Handler will still run if later tasks fail

---

### Common Mistakes

* Using `flush_handlers` everywhere → defeats handler optimization
* Expecting `force_handlers` to run handlers early (it doesn’t)
* Forgetting handlers don’t run on failure by default

---

### Best Practices

✔ Use `flush_handlers` **sparingly**
✔ Use `force_handlers` for **cleanup / safety handlers**
✔ Document why handlers must run early or always
✔ Keep handlers **idempotent**

---

### Interview One‑Liners 🎯

* **`flush_handlers`**: "Runs notified handlers immediately instead of waiting for the end of the play."
* **`force_handlers`**: "Ensures handlers run even if the play fails."

---

### Final Summary

* Handlers normally run at play end
* `meta: flush_handlers` changes *when* they run
* `force_handlers` changes *whether* they run on failure
* They are complementary, not alternatives

---

If you want next:

* 🔥 Handler execution flow diagram
* ⚖️ `handlers` vs `post_tasks`
* 🧠 Production-safe handler patterns
* 🎤 Interview slide version

### Comparison

| `meta: flush_handlers` | `force_handlers` |
|---|---|
| Runs pending handlers at that point | Allows notified handlers to run despite later host failure |
| Changes timing | Changes failure behavior |

### Example

```yaml
- name: Update configuration
  ansible.builtin.template:
    src: app.conf.j2
    dest: /etc/app.conf
  notify: restart app

- name: Apply pending handler now
  ansible.builtin.meta: flush_handlers
```

---

## Q22. How does `become` work internally in Ansible?

### Short explanation of the question

This tests whether I understand privilege escalation as a separate layer from the initial connection.

### Answer

`become` tells Ansible to use privilege escalation for a task or play. A common Linux pattern is connecting as a normal user and using sudo to execute the operation with elevated privileges.

### Detailed explanation

`become` is Ansible’s **privilege escalation mechanism**. It allows Ansible to **run tasks as another user (usually root)** after connecting to a host.

> 🧠 Think of `become` as: *"Log in as a normal user, then temporarily switch identity to do privileged work."*

---

### What Problem Does `become` Solve?

* SSH best practice: **don’t log in as root**
* Many tasks require **root privileges**
* `become` bridges this gap safely

---

### High-Level Internal Flow

```
Ansible Control Node
        │
        │ SSH (normal user)
        ▼
Remote Host (e.g. ubuntu)
        │
        │ become (sudo / su / doas)
        ▼
Target User (root)
```

---

### Step-by-Step: How `become` Works Internally

1. Ansible connects to the host via SSH as `ansible_user`
2. Task is marked with `become: true`
3. Ansible wraps the command with a **privilege escalation command**
4. Command is executed as the **become user** (default: root)
5. Output is returned to Ansible

---

### Basic Example

```yaml
- name: Install nginx
  apt:
    name: nginx
    state: present
  become: true
```

### What Ansible Executes Internally

```bash
sudo -H -S -n apt-get install nginx
```
(Not shown to user, but conceptually accurate)
### `-H` → Set HOME for Target User

### What it does

* Sets the `HOME` environment variable to the **target user’s home directory**
* Usually `/root` when becoming root

### Why Ansible uses it

* Prevents tools from reading config files from the **original user’s home**
* Avoids permission and config conflicts

### Example

Without `-H`:

```bash
HOME=/home/ansible
```

With `-H`:

```bash
HOME=/root
```

✅ Important for package managers, git, pip, etc.

---

### `-S` → Read Password from STDIN

### What it does

* Tells `sudo` to **read the password from standard input (STDIN)**
* Instead of prompting interactively

### Why Ansible uses it

* Ansible is **non-interactive**
* Password (if required) is piped securely via STDIN

⚠️ Ansible does this internally and **never stores the password on disk**.

---

### `-n` → Non-Interactive Mode

### What it does

* Disables password prompts completely
* If sudo needs a password → command **fails immediately**

### Why Ansible uses it

* Prevents playbooks from **hanging forever**
* Ensures predictable automation behavior
---

### `become_user`

Run task as a specific user:

```yaml
- name: Run as postgres user
  command: whoami
  become: true
  become_user: postgres
```

---

### `become_method`

Defines **how** privilege escalation happens.

Common methods:

| Method | Used On         |
| ------ | --------------- |
| sudo   | Linux (default) |
| su     | Legacy systems  |
| doas   | OpenBSD         |
| pbrun  | PowerBroker     |
| dzdo   | Centrify        |

Example:

```yaml
become: true
become_method: su
```

---

### Password Handling (`become_pass`)

If sudo requires a password:

```bash
ansible-playbook site.yml --ask-become-pass
```

Internally:

* Password is sent via **STDIN**
* Never written to disk

---

### `become` vs `remote_user`

| Feature               | remote_user | become |
| --------------------- | ----------- | ------ |
| Used for SSH login    | ✅ Yes       | ❌ No   |
| Changes user mid-task | ❌ No        | ✅ Yes  |
| Typical user          | ansible     | root   |

---

### Common Failure Scenarios

### sudo: a password is required"

Cause:

* User not in sudoers
* Missing `--ask-become-pass`

### become_user does not exist"

Cause:

* Target user missing on system

---

### Security Best Practices

* Use **passwordless sudo** for Ansible user
* Limit sudo permissions
* Avoid `become: true` at play level unless needed

---

### One-Line Summary (Interview Gold)

> **Ansible connects as a normal user and uses `become` to wrap commands with sudo (or similar) to run them as another user.**

---

### Interview Follow-Ups

Common questions:

* `become` vs `sudo`
* `become` with `delegate_to`
* Why root login is discouraged

If you want, I can also add:

* Execution flow diagram
* `become` with roles
* Real-world sudoers example

### Execution flow

```text
Remote connection user
        |
        v
Connection established
        |
        v
Privilege escalation mechanism
        |
        v
Effective execution user
        |
        v
Task / module executes
```

### Example

```yaml
- hosts: web
  become: true
  tasks:
    - name: Install nginx
      ansible.builtin.package:
        name: nginx
        state: present
```

---

## Q23. What is the difference between `become` and `become_user`?

### Short explanation of the question

This checks whether I can distinguish enabling escalation from choosing the account used after escalation.

### Answer

`become` enables privilege escalation. `become_user` specifies the user Ansible should become after escalation. The connection user and effective execution user can therefore be different.

### Detailed explanation

This document explains **privilege escalation in Ansible** with a clear focus on the difference between **`become`** and **`become_user`**.

This topic is **very important for interviews, real-world automation, and security-sensitive environments**.

---

### 1. What Is Privilege Escalation in Ansible?

By default, Ansible connects to managed nodes using a **normal user** (for example: `ec2-user`, `ubuntu`, or `ansible`).

Some tasks require **higher privileges**, such as:

* Installing packages
* Modifying system configuration
* Managing services
* Accessing protected files

Ansible uses **privilege escalation** to handle this securely.

---

### 2. become

### What is `become`?

`become` is a **boolean flag** that tells Ansible:

> “Execute this task (or play) with elevated privileges.”

By default, `become` escalates privileges to the **root user**.

---

### Example: Using `become`

```yaml
- name: Install Apache
  hosts: web
  become: yes

  tasks:
    - name: Install httpd
      yum:
        name: httpd
        state: present
```

### What Happens Internally?

* Ansible connects as a normal user
* Uses `sudo` (default method)
* Executes tasks as **root**

---

### Key Points About `become`

* Enables privilege escalation
* Default target user: `root`
* Can be set at **play level** or **task level**
* Uses `sudo` by default (configurable)

---

### 3. become_user

### What is `become_user`?

`become_user` specifies **which user** Ansible should switch to **after privilege escalation**.

It is used **together with `become`**.

---

### Example: Using `become_user`

```yaml
- name: Run task as another user
  hosts: web
  become: yes
  become_user: appuser

  tasks:
    - name: Run application script
      command: /opt/app/start.sh
```

### What Happens Internally?

* Ansible connects as a normal user
* Escalates privileges using `sudo`
* Switches execution to **`appuser`**

---

### Key Points About `become_user`

* Defines **target user**
* Requires `become: yes`
* Default value is `root`
* Commonly used for application or service users

---

### 4. Using become and become_user Together

```yaml
- hosts: web
  become: yes
  become_user: postgres

  tasks:
    - name: Run DB maintenance
      command: psql -c "VACUUM;"
```

This runs the command **as the `postgres` user**, not root.

---

### 5. Task-Level vs Play-Level Usage

### Play-Level

```yaml
- hosts: web
  become: yes
  become_user: root
```

Applies to **all tasks** in the play.

---

### Task-Level Override

```yaml
- name: Install package
  yum:
    name: nginx
    state: present
  become: yes

- name: Run app command
  command: ./run.sh
  become: yes
  become_user: appuser
```

---

### 6. Comparison Table

| Aspect        | become                      | become_user         |
| ------------- | --------------------------- | ------------------- |
| Purpose       | Enable privilege escalation | Specify target user |
| Required      | Yes                         | Only with become    |
| Default value | false                       | root                |
| Scope         | Play / Task                 | Play / Task         |
| Used alone    | Yes                         | No                  |

---

### 7. Common Mistakes

❌ Using `become_user` without `become`

❌ Assuming `become` always means root explicitly

❌ Running app-level commands as root unnecessarily

---

### 8. Best Practices

* Use `become` only when required
* Prefer least-privilege (`become_user` != root)
* Avoid running application logic as root
* Use task-level escalation where possible

---

### 9. Interview One-Line Answers

* **become**: Enables privilege escalation
* **become_user**: Specifies the user to run the task as

---

### Final Summary

* `become` controls **whether** privilege escalation happens
* `become_user` controls **who** the task runs as
* Both together provide **secure and controlled execution**

---

Happy Automating with Ansible 🚀

### Example

```yaml
- name: Run application command as appuser
  ansible.builtin.command: /opt/app/bin/status
  become: true
  become_user: appuser
```

### Interviewer may cross-question

**Interviewer:** "If SSH works, can I assume become will work?"

**Candidate:** "No. SSH authentication and privilege escalation are separate layers. Sudo policy, passwords, permissions and the configured become method can still fail." 

---

## Q24. What strategy plugins are used in Ansible?

### Short explanation of the question

This tests whether I understand how Ansible schedules task progression across hosts.

### Answer

Strategy plugins control how tasks progress across hosts. `linear` is the default coordinated strategy, while `free` lets faster hosts continue without waiting for slower hosts.

### Detailed explanation

Ansible **strategies** control **how tasks are executed across hosts** — specifically *ordering, parallelism, and failure behavior*.

> 🧠 **Simple idea**: Strategies decide *who runs what, and when*.

---

### Default Strategy: `linear`

### How it Works

* All hosts execute **task 1 together**
* Then all hosts execute **task 2 together**
* Strict lock-step execution

```text
Task 1 → all hosts
Task 2 → all hosts
Task 3 → all hosts
```

### Example

```yaml
- name: Linear strategy example
  hosts: web
  strategy: linear
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present

    - name: Start nginx
      service:
        name: nginx
        state: started
```

### When to Use

* Predictable execution
* Safer deployments
* Default for most playbooks

---

### Strategy: `free`

### How it Works

* Each host runs tasks **as fast as it can**
* No lock-step waiting
* Faster overall execution

```text
host1: task1 → task2 → task3
host2: task1 → task2
host3: task1 → task2 → task3
```

### Example

```yaml
- name: Free strategy example
  hosts: web
  strategy: free
  tasks:
    - name: Install packages
      apt:
        name: nginx
        state: present
```

### When to Use

* Large fleets
* Independent hosts
* Speed matters more than order

---

### Strategy: `host_pinned`

### How it Works

* Each host sticks to **one worker**
* Tasks for that host are serialized
* Reduces race conditions

### Example

```yaml
- name: Host pinned example
  hosts: web
  strategy: host_pinned
  tasks:
    - name: Update app
      command: /opt/update.sh
```

### When to Use

* Stateful operations
* Avoiding concurrent access issues

---

### Strategy: `debug`

### How it Works

* Executes **one task at a time**
* Waits for user input
* Used for troubleshooting

### Example

```yaml
- name: Debug strategy example
  hosts: web
  strategy: debug
  tasks:
    - name: Restart service
      service:
        name: nginx
        state: restarted
```

### When to Use

* Debugging playbooks
* Understanding execution flow

---

### Strategy Plugins (Advanced)

Ansible strategies are implemented as **plugins**.

Common plugins:

* `linear` (default)
* `free`
* `host_pinned`
* `debug`

You can also **write custom strategies** in Python.

---

### Strategy vs `serial`

| Feature                | Strategy | serial |
| ---------------------- | -------- | ------ |
| Controls task ordering | ✅ Yes    | ❌ No   |
| Controls batch size    | ❌ No     | ✅ Yes  |
| Used together          | ✅ Yes    | ✅ Yes  |

Example:

```yaml
strategy: linear
serial: 2
```

---

### Summary Table

| Strategy    | Execution Style | Use Case        |
| ----------- | --------------- | --------------- |
| linear      | Lock-step       | Safe, default   |
| free        | Fast, async     | Large fleets    |
| host_pinned | Per-host worker | Stateful ops    |
| debug       | Step-by-step    | Troubleshooting |

---

### One-Line Summary (Interview Gold)

> **Ansible strategies control how tasks are scheduled across hosts, balancing speed, safety, and order.**

---

### Interview Tip

If asked:

> *"How do you speed up Ansible execution?"*

Answer:

* Increase forks
* Use `strategy: free`
* Use `serial` carefully

If you want, I can also add:

* Strategy + failure behavior
* Strategy comparison diagram
* Real production patterns

### Comparison

| Strategy | Practical behavior |
|---|---|
| `linear` | Hosts progress through tasks in a coordinated manner |
| `free` | A host can continue without waiting for slower hosts |

### Interviewer may cross-question

**Interviewer:** "Is `free` the same as increasing forks?"

**Candidate:** "No. Forks controls available worker capacity; strategy controls how hosts progress through tasks." 

---
