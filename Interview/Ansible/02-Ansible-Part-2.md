# Ansible Senior L2 Interview — Part 2

**Focus:** Modules, commands, variables and facts

## Q7. What is the `raw` module and how is it different from normal modules?

### Short explanation of the question

This checks whether I know how to bootstrap a host when normal module prerequisites are not available.

### Answer

`raw` executes a command directly through the connection and does not require the normal remote Python module execution mechanism. I use it mainly for bootstrapping or very low-level operations, then move to purpose-built modules.

### Detailed explanation

This document explains **what the `raw` module is**, **how it differs from normal Ansible modules**, **why it exists**, and **which modules are idempotent or not**. Written for deep understanding and interviews.

---

### What is the `raw` Module?

The **`raw` module** runs a **plain shell command directly on the target host** over SSH.

Think of it as:

> ⚠️ *"Just run this command — don’t ask questions."*

```yaml
- name: Start nginx using raw
  ansible.builtin.raw: systemctl start nginx
```

No module logic. No state check. No JSON intelligence.

---

### What Are Normal Ansible Modules?

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

### How They Work Internally (Key Difference)

### Normal Module Execution

* Wrapped into **AnsiballZ** 🧩
* Copied to target via SSH
* Executed with Python 🐍
* Performs state inspection
* Returns JSON (`changed`, `failed`)
* Temporary files cleaned

---

### raw Module Execution

* No AnsiballZ
* No Python required
* No module copy
* Just SSH + command execution

Internally:

```bash
ssh host "systemctl start nginx"
```

---

### Why Does the `raw` Module Exist?

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

### Idempotency Explained (Very Important)

### Idempotent Modules (Safe to re-run)

These modules **check state before acting**:

* `package`
* `service`
* `copy`
* `file`
* `user`
* `template`

Running them multiple times = same result.

---

### Non-Idempotent by Default

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

### Can `command` / `shell` Be Made Idempotent?

🟡 Yes, but **manually**:

```yaml
- shell: touch /tmp/app.lock
  args:
    creates: /tmp/app.lock
```

But this is still weaker than real modules.

---

### Side-by-Side Comparison

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

### Mental Model (Remember This)

```text
Normal Module = check → compare → act → report
raw Module    = run → hope
```

---

### Interview-Perfect Summary

> **“Normal Ansible modules are state-aware and idempotent, executed via the AnsiballZ wrapper and returning structured JSON. The raw module bypasses this mechanism to execute commands directly over SSH, making it suitable only for bootstrapping or recovery scenarios where Python or modules cannot be used.”**

---

🟢 **Rule of Thumb:**
Use `raw` only to make Ansible usable — then stop using it.

### Example

```yaml
- name: Bootstrap Python on a minimal host
  ansible.builtin.raw: apt-get update && apt-get install -y python3
```

### Interviewer may cross-question

**Interviewer:** "Why not use shell for everything?"

**Candidate:** "Because raw, command and shell are lower-level tools. When a stateful module exists, it normally expresses the desired state more clearly and gives better idempotency." 

---

## Q8. What is the difference between `command` and `shell`?

### Short explanation of the question

This tests whether I choose the least powerful command-execution mechanism that still solves the problem.

### Answer

`command` executes without shell interpretation. `shell` invokes a shell and supports shell syntax such as pipes and redirection. I prefer `command` when shell features are unnecessary, and I prefer a purpose-built module over both when one exists.

### Detailed explanation

Both `command` and `shell` are used to run commands on remote hosts, but **they are NOT the same**.

> 🧠 **Golden rule**:

* Use **`command` by default**
* Use **`shell` only when you need shell features**

---

### Key Difference (One Line)

| Module    | How it runs                                  |
| --------- | -------------------------------------------- |
| `command` | Runs command **directly** (no shell)         |
| `shell`   | Runs command **through a shell** (`/bin/sh`) |

---

### `command` Module (Safer & Preferred)

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

### `shell` Module (Powerful but Risky)

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

### Side-by-Side Example

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

### Security Difference (Very Important)

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

### Idempotency Helpers

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

### Execution Summary

| Feature            | command   | shell             |
| ------------------ | --------- | ----------------- |
| Uses shell         | ❌ No      | ✅ Yes             |
| Pipes / redirects  | ❌ No      | ✅ Yes             |
| Wildcards          | ❌ No      | ✅ Yes             |
| Variable expansion | ❌ No      | ✅ Yes             |
| Security           | ✅ Safer   | ⚠️ Risky          |
| Recommended        | ✅ Default | ⚠️ Only if needed |

---

### Real-World Usage Guidelines

### Use `command` when

* Running a single binary
* Predictability matters
* Security is important

### Use `shell` when

* You need pipes or redirects
* You need `&&`, `||`
* You rely on shell features

---

### One-Line Summary (Interview Gold)

> **`command` runs without a shell and is safer; `shell` runs through a shell and supports pipes and redirects.**

---

### Interview Trap Question

> *"Why is `shell` discouraged?"*

✔ Because it invokes a shell, making it less secure and less predictable.

If you want, I can also add:

* `command` vs `raw`
* `shell` with `executable`
* Best practices to replace shell with modules

### Comparison

| `command` | `shell` |
|---|---|
| No shell parsing | Shell parsing |
| Preferred for simple commands | Use when shell syntax is required |
| No normal shell pipes/redirection | Supports shell operators, pipes and redirection |

### Example

```yaml
- name: Simple command
  ansible.builtin.command: systemctl is-active nginx

- name: Pipeline requiring a shell
  ansible.builtin.shell: journalctl -u nginx | tail -50
```

---

## Q9. What are Ansible ad-hoc commands and when should you use them?

### Short explanation of the question

This tests whether I know when a one-line operation is appropriate and when it should become version-controlled automation.

### Answer

An ad-hoc command runs a single Ansible task from the CLI. It is useful for quick checks, troubleshooting and controlled one-off actions. Repeated or multi-step work should normally be moved into a playbook or role.

### Detailed explanation

This document provides a **detailed, practical, and interview-ready guide** to **Ansible ad-hoc commands** with **multiple real-world examples**.

Ad-hoc commands are used for **quick, one-time operations** without writing a playbook.

---

### 1. What Are Ansible Ad-Hoc Commands?

An **ad-hoc command** is a **single-line Ansible command** executed from the CLI to perform a **simple task** on managed nodes.

They are best suited for:

* Quick checks
* Troubleshooting
* One-time changes
* Emergency operations

---
### 2. General Syntax

```bash
ansible <host-pattern> -m <module> -a "module arguments" [options]
```

### Components Explained

* `ansible` → CLI tool for ad-hoc commands
* `<host-pattern>` → Inventory group or host
* `-m` → Module name
* `-a` → Module arguments
* `-i` → Inventory file (optional)
* `-b` → Become (sudo)

---

### 3. Verify Connectivity (Ping Module)

```bash
ansible all -m ping
```

Checks:

* SSH connectivity
* Python availability

---

### 4. Execute Shell / Command Modules

### Run a Simple Command

```bash
ansible web -m command -a "uptime"
```

### Using Shell (supports pipes, redirects)

```bash
ansible web -m shell -a "df -h | grep /dev"
```

---

### 5. File Operations

### Create a File

```bash
ansible web -m file -a "path=/tmp/demo.txt state=touch"
```

### Remove a File

```bash
ansible web -m file -a "path=/tmp/demo.txt state=absent"
```

---

### 6. Copy Files to Remote Hosts

```bash
ansible web -m copy -a "src=./index.html dest=/var/www/html/index.html"
```

---

### 7. Manage Packages

### Install Package (YUM)

```bash
ansible web -m yum -a "name=httpd state=present" -b
```

### Install Package (APT)

```bash
ansible ubuntu -m apt -a "name=nginx state=present update_cache=yes" -b
```

---

### 8. Manage Services

### Start a Service

```bash
ansible web -m service -a "name=httpd state=started" -b
```

### Restart a Service

```bash
ansible web -m service -a "name=httpd state=restarted" -b
```

---

### 9. User Management

### Create a User

```bash
ansible all -m user -a "name=deploy state=present" -b
```

### Delete a User

```bash
ansible all -m user -a "name=deploy state=absent" -b
```

---

### 10. Disk and System Information

### Check Disk Usage

```bash
ansible all -m shell -a "df -h"
```

### Check Memory

```bash
ansible all -m shell -a "free -m"
```

---

### 11. Fetch Files from Remote Hosts

```bash
ansible web -m fetch -a "src=/var/log/messages dest=./logs flat=yes"
```

---

### 12. Manage Permissions

```bash
ansible web -m file -a "path=/var/www/html owner=apache group=apache mode=0644" -b
```

---

### 13. Using Variables in Ad-Hoc Commands

```bash
ansible web -m shell -a "echo {{ inventory_hostname }}"
```

---

### 14. Using Custom Inventory

```bash
ansible web -i inventory.ini -m ping
```

---

### 15. Become (sudo) Usage

```bash
ansible web -m yum -a "name=git state=present" -b
```

---

### 16. Limit Hosts

```bash
ansible all --limit web1 -m ping
```

---

### 17. Run Commands in Parallel

```bash
ansible all -m shell -a "uptime" -f 10
```

---

### 18. Dry Run (Check Mode)

```bash
ansible web -m copy -a "src=a.txt dest=/tmp/a.txt" --check
```

---

### 19. Common Mistakes

❌ Using `shell` instead of `command` unnecessarily

❌ Forgetting `-b` for privileged tasks

❌ Using ad-hoc commands for complex workflows

---

### 20. Best Practices

* Use ad-hoc commands for **simple tasks only**
* Prefer **idempotent modules** over shell
* Move repeated tasks to playbooks
* Use inventory groups wisely

---

### 21. Interview One-Line Answers

* **Ad-hoc command**: One-time Ansible command without a playbook
* Best module for checks: `ping`
* Supports sudo: Yes, using `-b`

---

### Final Summary

* Ad-hoc commands are fast and powerful
* Ideal for troubleshooting and quick fixes
* Not a replacement for playbooks

Mastering ad-hoc commands makes you **much faster and more effective with Ansible**.

---

Happy Automating with Ansible 🚀

### Example

```bash
ansible web -m ansible.builtin.ping
ansible web -m ansible.builtin.command -a 'uptime'
ansible web -m ansible.builtin.service -a 'name=nginx state=started'
```

### Interviewer may cross-question

**Interviewer:** "Would you put a long production procedure into an ad-hoc command?"

**Candidate:** "No. I would turn repeatable work into a playbook or role so it is reviewable, version-controlled and reproducible." 

---

## Q10. What is variable precedence in Ansible?

### Short explanation of the question

This tests whether I can explain why the same variable name can produce different runtime values.

### Answer

Ansible has many variable sources with different precedence. Extra vars supplied with `-e` are among the highest-precedence inputs, while role defaults are deliberately easy to override. I debug precedence by finding where the value is defined rather than relying on an incomplete memorized ladder.

### Detailed explanation

Variable precedence defines **which variable wins** when the same variable name is defined in multiple places.

Rule to remember:

> 🔼 **Higher precedence overrides lower precedence**

---

### Variable Precedence Pyramid (Lowest → Highest)

```text
                🔺 Extra vars (-e)
              🔺 Task vars / set_fact
            🔺 Block vars
          🔺 Role vars (roles/vars)
        🔺 Play vars
      🔺 Host vars (host_vars)
    🔺 Group vars (group_vars)
  🔺 Inventory vars
🔺 Role defaults (roles/defaults)
```

📌 Bottom = weakest priority
📌 Top = strongest priority

---

### Precedence Explained with Examples

We will use **one variable name** everywhere:

```yaml
app_port
```

---

### Role Defaults (Lowest Priority)

📁 `roles/web/defaults/main.yml`

```yaml
app_port: 80
```

Used for **safe defaults**.

---

### Inventory Variables

📁 `inventory.ini`

```ini
[web]
web1 app_port=81
```

Overrides role defaults.

---

### Group Variables

📁 `group_vars/web.yml`

```yaml
app_port: 82
```

Applies to all hosts in the group.

---

### Host Variables

📁 `host_vars/web1.yml`

```yaml
app_port: 83
```

Overrides group-level values.

---

### Play Variables

```yaml
- hosts: web
  vars:
    app_port: 84
```

Overrides inventory, group, and host vars.

---

### Role Vars (High Priority)

📁 `roles/web/vars/main.yml`

```yaml
app_port: 85
```

⚠️ Very strong — avoid unless necessary.

---

### Block Variables

```yaml
block:
  - debug:
      msg: "{{ app_port }}"
  vars:
    app_port: 86
```

Overrides play and role vars.

---

### Task Variables / set_fact

```yaml
- set_fact:
    app_port: 87
```

Overrides almost everything below.

---

### Extra Vars (Highest Priority)

```bash
ansible-playbook site.yml -e app_port=88
```

Nothing can override this.

---

### Final Winner Example

If `app_port` is defined **everywhere**, the value used will be:

```text
app_port = 88
```

(from extra vars)

---

### Interview-Perfect Answer

> **“Ansible variable precedence follows a layered model where role defaults have the lowest priority and extra vars have the highest. When the same variable is defined multiple times, the one with the highest precedence wins.”**

---

### Best Practices

* ✅ Put defaults in `roles/defaults`
* ⚠️ Use `roles/vars` sparingly
* 🚫 Avoid relying heavily on extra vars
* 🧠 Keep variable definitions predictable

---

🧩 **Mnemonic to remember:**

> *Defaults sink, extra vars rule.*

### Example

```yaml
# role defaults
app_port: 8080
```

Runtime:

```bash
ansible-playbook site.yml -e app_port=9090
```

The extra variable has higher precedence, so the effective value is `9090`.

### Interviewer may cross-question

**Interviewer:** "How do you debug a surprising variable value?"

**Candidate:** "I search for every definition, inspect inventory and host/group variables, check role defaults/vars and play/task variables, and then check runtime overrides such as `-e`. I use `debug` or `ansible-inventory` where appropriate." 

---

## Q11. What are Ansible facts and how are they collected?

### Short explanation of the question

This tests whether I understand automatically discovered host information and its performance impact.

### Answer

Facts are information Ansible gathers about a managed host, such as OS, network, memory and processor details. They are normally collected at play start through the setup mechanism unless fact gathering is disabled.

### Detailed explanation

**Ansible facts** are **system information automatically gathered from managed hosts** and stored as variables. They describe the **state and characteristics of a host**.

> 🧠 Think of facts as: *"Ansible asking the server: Who are you and what do you look like?"*

---

### What Are Ansible Facts?

Facts include information such as:

* OS name and version
* IP addresses
* CPU architecture
* Memory details
* Disk and mount points
* Network interfaces

Example fact variable:

```yaml
ansible_os_family: Debian
ansible_memory_mb:
  real:
    total: 7855
```

---

### How Are Facts Collected?

Facts are collected using the **`setup` module**.

### Default Behavior

* Facts are gathered **at the start of every play**
* Happens before any task runs

```yaml
- name: Example play
  hosts: all
  tasks:
    - debug:
        var: ansible_hostname
```

➡️ Even though `setup` is not written, it is executed automatically.

---

### Under the Hood (High Level)

1. Ansible connects to the host
2. Runs the `setup` module
3. Collects system data
4. Stores data as variables (facts)
5. Makes them available to all tasks

---

### Example: Using Facts in a Playbook

```yaml
- name: Install web server based on OS
  hosts: all
  tasks:
    - name: Install nginx on Debian
      apt:
        name: nginx
        state: present
      when: ansible_os_family == "Debian"

    - name: Install nginx on RedHat
      yum:
        name: nginx
        state: present
      when: ansible_os_family == "RedHat"
```

---

### Disabling Fact Gathering

If you don’t need facts, you can disable them to improve speed:

```yaml
- name: Fast play
  hosts: all
  gather_facts: false
```

---

### Manually Gathering Facts

You can explicitly run the `setup` module:

```yaml
- name: Gather facts manually
  setup:
```

---

### Subsets and Filters (Performance Optimization)

Collect only specific facts:

```yaml
- setup:
    gather_subset:
      - network
```

Or exclude facts:

```yaml
- setup:
    gather_subset:
      - '!hardware'
```

---

### Custom Facts

You can define **custom facts** on the host.

### Example (Linux)

```bash
/etc/ansible/facts.d/app.fact
```

```ini
[app]
version=1.2.3
env=production
```

Access in playbook:

```yaml
ansible_local.app.version
```

---

### Facts vs Variables

| Feature                 | Facts  | Variables         |
| ----------------------- | ------ | ----------------- |
| Collected automatically | ✅ Yes  | ❌ No              |
| Host-specific           | ✅ Yes  | ✅ Yes             |
| Dynamic                 | ✅ Yes  | ⚠️ Usually static |
| Source                  | System | User / inventory  |

---

### One-Line Summary (Interview Gold)

> **Ansible facts are automatically gathered system details collected by the `setup` module and exposed as variables.**

---

### Interview Tips

Common follow-ups:

* Difference between facts and inventory variables
* How to speed up fact gathering
* Custom facts vs dynamic facts

If you want, I can also add:

* `ansible_facts` namespace explanation
* Facts with `delegate_to`
* Fact caching (Redis / JSON)
* A facts collection flow diagram

### Example

```yaml
- hosts: web
  gather_facts: false
  tasks:
    - name: Gather only network facts when needed
      ansible.builtin.setup:
        gather_subset:
          - network
```

### Interviewer may cross-question

**Interviewer:** "Why disable fact gathering?"

**Candidate:** "If a play does not need facts, especially across a large inventory, avoiding unnecessary discovery can reduce execution time." 

---

## Q12. What is the difference between `gather_facts` and `set_fact`?

### Short explanation of the question

This tests whether I distinguish information discovered from the host from variables deliberately created during execution.

### Answer

`gather_facts` controls automatic host fact collection. `set_fact` creates variables during a play. Facts describe discovered host state; `set_fact` is useful for values I derive or intentionally establish.

### Detailed explanation

This document explains the **difference between `set_fact` and `gather_facts` (often called get_facts)** in Ansible.

Understanding this topic is **critical for writing dynamic playbooks, conditionals, templates, and roles**, and it is a **very common interview question**.

---

### 1. What Are Facts in Ansible?

**Facts** are variables that store information about:

* Managed hosts
* System properties
* Runtime values

Facts can be:

* **Automatically collected** by Ansible
* **Manually defined** during play execution

---

### 2. gather_facts (get_facts)

### What is `gather_facts`?

`gather_facts` is a **play-level setting** that tells Ansible to **automatically collect system information** from managed nodes using the `setup` module.

> Many people call this **get_facts**, but the actual keyword is `gather_facts`.

---

### Example: Using gather_facts

```yaml
- name: Collect system facts
  hosts: web
  gather_facts: yes

  tasks:
    - name: Print OS name
      debug:
        msg: "OS is {{ ansible_distribution }}"
```

---

### What Information Is Collected?

Examples of gathered facts:

* OS name and version
* IP addresses
* CPU architecture
* Memory details
* Disk information
* Network interfaces

These facts are stored as **host variables**.

---

### Key Characteristics of gather_facts

* Enabled by default (`gather_facts: yes`)
* Uses the `setup` module internally
* Runs **before any task executes**
* Collects **static system information**

---

### 3. set_fact

### What is `set_fact`?

`set_fact` is a **task-level module** used to **define or modify custom variables (facts)** during play execution.

These facts are created **at runtime**.

---

### Example: Using set_fact

```yaml
- name: Define custom facts
  hosts: web
  gather_facts: no

  tasks:
    - name: Set environment variable
      set_fact:
        app_env: production

    - name: Print custom fact
      debug:
        msg: "Environment is {{ app_env }}"
```

---

### Key Characteristics of set_fact

* Task-level operation
* Creates **custom facts**
* Values can be dynamic
* Available for the rest of the play

---

### 4. Execution Timing (Very Important)

| Type         | When It Runs          |
| ------------ | --------------------- |
| gather_facts | Before any task       |
| set_fact     | During task execution |

---

### 5. Scope and Lifetime

### gather_facts

* Scope: Host-level
* Lifetime: Entire play
* Source: System data

### set_fact

* Scope: Host-level
* Lifetime: Remaining play (or longer with cache)
* Source: User-defined

---

### 6. Comparison Table

| Aspect         | gather_facts (get_facts) | set_fact                |
| -------------- | ------------------------ | ----------------------- |
| Purpose        | Collect system info      | Define custom variables |
| Level          | Play-level               | Task-level              |
| Module used    | setup                    | set_fact                |
| Default        | Enabled                  | Manual                  |
| Data source    | OS / system              | User logic              |
| Dynamic values | No                       | Yes                     |

---

### 7. Using Both Together (Real Example)

```yaml
- name: Combine facts
  hosts: web

  tasks:
    - name: Set app path based on OS
      set_fact:
        app_path: "/opt/app"
      when: ansible_os_family == "RedHat"

    - name: Show app path
      debug:
        msg: "App path is {{ app_path }}"
```

This example:

* Uses **gathered facts** (`ansible_os_family`)
* Creates a **custom fact** (`app_path`)

---

### 8. Common Mistakes

❌ Expecting `set_fact` to work before `gather_facts`

❌ Confusing get_facts with set_fact

❌ Forgetting to disable `gather_facts` when not needed

---

### 9. Best Practices

* Disable `gather_facts` if not required (performance)
* Use `set_fact` for derived or computed values
* Keep fact names meaningful
* Avoid excessive use of `set_fact`

---

### 10. Interview One-Line Answers

* **gather_facts**: Automatically collects system information
* **set_fact**: Manually defines runtime variables

---

### Final Summary

* `gather_facts` answers **“What does this system look like?”**
* `set_fact` answers **“What value do I want to calculate or store?”**

Both are essential for writing **dynamic, intelligent Ansible automation**.

---

Happy Automating with Ansible 🚀

---
