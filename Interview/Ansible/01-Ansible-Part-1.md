# Ansible Senior L2 Interview — Part 1

**Focus:** Fundamentals, architecture, modules and execution

## Q1. What is the difference between a Playbook, a Play and a Task?

### Short explanation of the question

The interviewer wants me to separate the four levels of Ansible thinking: the playbook is the automation document, a play targets hosts, a task is a unit of work, and a module performs the operation.

### Answer

A playbook is a YAML automation document that can contain one or more plays. A play maps tasks to a host pattern. A task is a single unit of work, normally invoking one module. The module performs the requested operation.

### Detailed explanation

This document explains the **core building blocks of Ansible** in a **clear, structured, and interview-ready manner**.

Understanding the difference between **Task**, **Play**, and **Playbook** is fundamental for writing correct Ansible automation and answering certification or interview questions.

---

### 1. Task

### What is a Task?

A **task** is the **smallest unit of work** in Ansible.

It represents a **single action** performed on a managed node using **one Ansible module**.

### Common Examples

* Install a package
* Copy a file
* Start or stop a service
* Execute a command

### Example Task

```yaml
- name: Install Apache
  yum:
    name: httpd
    state: present
```

### Key Characteristics

* Uses exactly **one module**
* Executes sequentially
* Can notify handlers
* Cannot run independently

---

### 2. Play

### What is a Play?

A **play** defines:

* **Which hosts** to target
* **What tasks** to run
* **How** those tasks should be executed

A play maps a **set of tasks** to a **group of hosts**.

### Example Play

```yaml
- name: Configure Web Servers
  hosts: web
  become: yes

  tasks:
    - name: Install Apache
      yum:
        name: httpd
        state: present

    - name: Start Apache
      service:
        name: httpd
        state: started
```

### Key Characteristics

* Contains multiple tasks
* Defines execution context (`hosts`, `become`, `vars`, etc.)
* Can include handlers, roles, and variables
* Cannot run independently

---

### 3. Playbook

### What is a Playbook?

A **playbook** is a **YAML file** that contains **one or more plays**.

It represents the **complete automation workflow**.

### Example Playbook

```yaml
---
- name: Configure Web Servers
  hosts: web
  become: yes
  tasks:
    - name: Install Apache
      yum:
        name: httpd
        state: present

- name: Configure Database Servers
  hosts: db
  become: yes
  tasks:
    - name: Install MySQL
      yum:
        name: mysql-server
        state: present
```

### Key Characteristics

* Entry point for Ansible execution
* Can contain multiple plays
* Executed using `ansible-playbook`
* Defines end-to-end automation

---

### 4. Hierarchy (Very Important)

```
Playbook
 ├── Play 1
 │    ├── Task 1
 │    ├── Task 2
 │    └── Handler
 └── Play 2
      ├── Task 1
      └── Task 2
```

This hierarchy is **critical for understanding execution flow**.

---

### 5. Comparison Table

| Aspect           | Task          | Play                  | Playbook                 |
| ---------------- | ------------- | --------------------- | ------------------------ |
| Purpose          | Single action | Apply tasks to hosts  | Full automation workflow |
| Contains         | Module call   | Tasks, handlers, vars | One or more plays        |
| Runs on          | Host          | Group of hosts        | Entire inventory         |
| Executable alone | No            | No                    | Yes                      |
| Smallest unit    | Yes           | No                    | No                       |

---

### 6. Real-World Analogy

Think of **cooking a meal**:

| Ansible  | Analogy         |
| -------- | --------------- |
| Task     | Chop vegetables |
| Play     | Cook a dish     |
| Playbook | Full meal plan  |

---

### 7. Common Confusions (Cleared)

❌ Task and Play are the same
❌ Playbook is a single task

✅ Task exists **inside a Play**
✅ Play exists **inside a Playbook**

---

### 8. Interview One-Line Answers

* **Task**: Smallest unit of work using one module
* **Play**: Maps tasks to a group of hosts
* **Playbook**: YAML file containing one or more plays

---

### Final Summary

* A **task** performs a single action
* A **play** applies tasks to specific hosts
* A **playbook** orchestrates one or more plays

Mastering this hierarchy is essential for **roles, handlers, CI/CD pipelines, and large-scale automation**.

---

Happy Automating with Ansible 🚀

### Interviewer may cross-question

**Interviewer:** "Is a task the same thing as a module?"

**Candidate:** "No. A task is the unit of work declared in the playbook. The task normally invokes a module, and the module performs the operation."

### Example

```yaml
- name: Configure web server
  hosts: web
  tasks:
    - name: Install nginx
      ansible.builtin.package:
        name: nginx
        state: present
```

---

## Q2. How is Ansible different from Chef, Puppet and Salt?

### Short explanation of the question

This tests whether I can compare automation platforms by execution and operating model rather than by saying one is universally better.

### Answer

Ansible is commonly controller-driven and agentless. Chef and Puppet traditionally use agent/client-based configuration-management models. Salt is known for fast remote execution and can be deployed in multiple patterns. The right choice depends on network design, drift requirements, scale, compliance and team practices.

### Detailed explanation

Ansible, Chef, Puppet, and SaltStack are **configuration management (SCM) / automation tools**, but they differ **fundamentally in architecture, complexity, and operating model**.

> 🧠 **High-level idea**: Ansible focuses on *simplicity and agentless automation*, while Chef/Puppet focus on *continuous, agent-based configuration enforcement*.

---

### Core Architectural Difference

### Why Push-Based (Ansible) Is Often Preferred

**Push model** gives operators immediate, deterministic control over *when* and *how* changes happen.

### Advantages of Push

* Immediate execution (no polling delay)
* Strong orchestration (ordering, rolling updates)
* CI/CD friendly
* No long-running agents
* Easier debugging and auditing

### Where Pull Still Makes Sense

* Continuous drift correction
* Extremely large, long-lived fleets

🧠 **Summary**: Push excels at automation and orchestration; pull excels at continuous enforcement.

---

### Core Architectural Difference

### Ansible (Agentless, Push-based)

```
Control Node  ──SSH──▶  Managed Hosts
```

* No agent on managed nodes
* Uses SSH (Linux) / WinRM (Windows)
* Tasks are **pushed** from control node

---

### Chef / Puppet (Agent-based, Pull-based)

```
Chef/Puppet Server
        ▲
        │ (polls)
Agent ──┘
```

* Requires an **agent** on every node
* Agents **periodically pull** configuration
* Continuous enforcement model

---

### Comparison Table (Big Picture)

| Feature          | Ansible   | Chef        | Puppet       | SaltStack         |
| ---------------- | --------- | ----------- | ------------ | ----------------- |
| Architecture     | Agentless | Agent-based | Agent-based  | Agent / Agentless |
| Execution        | Push      | Pull        | Pull         | Push & Pull       |
| Primary Agent    | ❌ None    | chef-client | puppet-agent | salt-minion       |
| Language Used    | YAML      | Ruby DSL    | Puppet DSL   | YAML / Python     |
| Learning curve   | Low       | High        | Medium–High  | Medium            |
| Setup complexity | Very low  | High        | High         | Medium            |
| Speed to start   | Fast      | Slow        | Slow         | Medium            |

---

### Configuration Philosophy

### Ansible

* Task-based
* Procedural (what to do, step by step)
* Best for **orchestration + configuration**

```yaml
- name: Install nginx
  apt:
    name: nginx
    state: present
```

---

### Chef / Puppet

* Resource-based
* Declarative (desired end state)
* Best for **continuous drift correction**

```puppet
package { 'nginx':
  ensure => installed,
}
```

---

### Execution Model (Very Important)

### Ansible

* Runs **on demand**
* Does nothing unless you run it
* No background process

### Chef / Puppet

* Agent runs every X minutes
* Automatically fixes drift
* Always enforcing state

---

### Installation & Operations

### Ansible

* Install only on control node
* Managed hosts need only:

  * Python
  * SSH access
* **No agent lifecycle to manage**

### Chef / Puppet

* Require server + agents
* Agent names:

  * Chef: `chef-client`
  * Puppet: `puppet-agent`
* Written in:

  * Chef → Ruby
  * Puppet → Puppet DSL (Ruby-based)
* Certificates, keys, onboarding
* Higher operational overhead

### SaltStack

* Agent name: `salt-minion`
* Language: Python
* Can run agentless via SSH
---

### Security Model

| Aspect      | Ansible     | Chef / Puppet |
| ----------- | ----------- | ------------- |
| Transport   | SSH / WinRM | HTTPS         |
| Credentials | SSH keys    | Agent certs   |
| Firewall    | Simple      | More ports    |

---

### Real-World Use Cases

### When Ansible Is Better

* One-time provisioning
* CI/CD pipelines
* Orchestration (rolling restarts, DB failover)
* Cloud & Kubernetes automation
* Small to large teams

### When Chef / Puppet Are Better

* Very large fleets (100k+ nodes)
* Strict configuration enforcement
* Long-lived servers
* Regulated environments

---

### Performance & Scale

* Ansible scales via **forks & parallelism**
* Chef/Puppet scale via **agent autonomy**
* SaltStack is fastest due to event-driven model

---

### Drift Management

| Tool    | Drift Handling |
| ------- | -------------- |
| Ansible | Manual re-run  |
| Chef    | Auto-correct   |
| Puppet  | Auto-correct   |
| Salt    | Event-driven   |

---

### Ecosystem & Community

* Ansible: Huge community, Red Hat backed
* Puppet: Mature enterprise tooling
* Chef: Strong infra-as-code philosophy
* Salt: Powerful but smaller ecosystem

---

### Interview-Ready Summary (One Paragraph)

> **Ansible is agentless, push-based, and simple to adopt, making it ideal for orchestration and CI/CD automation. Chef and Puppet are agent-based, pull-driven systems designed for continuous configuration enforcement at large scale.**

---

### Interview Trap Question

**Q:** *Why do companies move from Puppet/Chef to Ansible?*

**A:**

* Lower operational overhead
* Faster onboarding
* Better orchestration capabilities

---

### Final Verdict

| Team Size / Need      | Best Choice |
| --------------------- | ----------- |
| Fast automation       | Ansible     |
| Heavy compliance      | Puppet      |
| Infra-as-code purists | Chef        |
| Event-driven ops      | SaltStack   |

---

If you want, I can also add:

* Ansible vs Terraform comparison
* Production architecture diagrams
* Real migration story (Puppet → Ansible)
* MCQs & interview questions

---

## Q3. What are the types of Ansible modules?

### Short explanation of the question

This tests whether I understand that modules can be classified by how they are delivered as well as by what they do.

### Answer

Ansible modules can come from built-in collections, community or vendor collections, and custom implementations. In practice, I choose a module based on the resource and operation I need to manage.

### Detailed explanation

### Primary Types of Ansible Modules (How They Are Delivered)
---
### 1

* Shipped **with Ansible by default**
* No installation required
* Always available once Ansible is installed
* Lives in the `ansible.builtin` collection

**Example**

```yaml
- name: Copy file to server
  ansible.builtin.copy:
    src: app.conf
    dest: /etc/app.conf
```

---

### 2

* Developed and maintained by the **community**
* Installed separately using collections
* Used for cloud, containers, databases, monitoring, etc.

**Install example**

```bash
ansible-galaxy collection install community.docker
```

**Usage example**

```yaml
- name: Run nginx container
  community.docker.docker_container:
    name: nginx
    image: nginx
    state: started
```

---

### 3

* Written by **you or your team**
* Used when no built-in or community module fits
* Usually written in Python

**Location**

```text
library/my_custom_module.py
```

**Usage example**

```yaml
- name: Call custom module
  my_custom_module:
    option: value
```

---

### Final Answer (No Ambiguity)

**How many types of Ansible modules exist?**
👉 **Three (3): Built-in, Community, and Custom**

### Comparison

| Delivery source | Meaning |
|---|---|
| Built-in collections | Modules maintained as part of Ansible's supported collection set |
| Community/vendor collections | Additional platform-specific modules |
| Custom modules | Organization-specific operations |

### Example

`ansible.builtin.copy` is a built-in module. AWS-specific modules are commonly supplied by the `amazon.aws` collection.

---

## Q4. What are the major Ansible module categories by functionality?

### Short explanation of the question

This tests whether I can choose a module family from a requirement instead of memorizing module names.

### Answer

Common functionality groups include system, package, file, service, user, command, cloud, database, networking and platform-specific modules. I prefer the module that represents the desired state of the resource.

### Detailed explanation

This is the **most common interview explanation**.
Here, modules are grouped by **what task they perform** — not by where they come from.

---

### 1

Manage OS-level resources like users, groups, cron jobs.

**Examples:** `user`, `group`, `cron`

```yaml
- name: Create user
  ansible.builtin.user:
    name: devops
```

---

### 2

Install, update, or remove software packages.

**Examples:** `apt`, `yum`, `dnf`, `package`

```yaml
- name: Install nginx
  ansible.builtin.package:
    name: nginx
    state: present
```

---

### 3

Manage files, directories, permissions, and configuration files.

**Examples:** `copy`, `template`, `file`, `lineinfile`

```yaml
- name: Create directory
  ansible.builtin.file:
    path: /opt/app
    state: directory
```

---

### 4

Control services like start, stop, restart, enable.

**Examples:** `service`, `systemd`

```yaml
- name: Start nginx
  ansible.builtin.service:
    name: nginx
    state: started
```

---

### 5

Configure network devices such as routers and switches.

**Examples:** `ios_config`, `junos_config`

```yaml
- name: Configure interface
  cisco.ios.ios_config:
    lines:
      - interface GigabitEthernet0/1
```

---

### 6

Provision and manage cloud infrastructure.

**Examples:** `ec2_instance`, `azure_rm_vm`

```yaml
- name: Launch EC2
  amazon.aws.ec2_instance:
    name: web-server
```

---

### 7

Create and manage databases and database users.

**Examples:** `mysql_db`, `postgresql_user`

```yaml
- name: Create database
  community.mysql.mysql_db:
    name: app_db
```

---

### 8

Manage Docker containers and images.

**Examples:** `docker_container`, `docker_image`

```yaml
- name: Run container
  community.docker.docker_container:
    name: nginx
    image: nginx
```

---

### 9

Manage Kubernetes resources.

**Examples:** `k8s`, `helm`

```yaml
- name: Deploy to Kubernetes
  kubernetes.core.k8s:
    state: present
    src: deployment.yaml
```

---

### Monitoring & Notification Modules

Send alerts and integrate with monitoring tools.

**Examples:** `slack`, `mail`

```yaml
- name: Send alert
  community.general.slack:
    msg: "Deployment completed"
```

---

### 1

Used for debugging, variables, and flow control.

**Examples:** `debug`, `set_fact`, `assert`

```yaml
- name: Show variable
  ansible.builtin.debug:
    msg: "App version is {{ app_version }}"
```

---

### Interview One‑Liner

**“Based on functionality, Ansible modules are grouped into system, package, file, service, network, cloud, database, container, Kubernetes, monitoring, and utility modules.”**

### Example

For a Linux application I might combine:

```yaml
- name: Install package
  ansible.builtin.package:
    name: nginx
    state: present

- name: Render configuration
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf

- name: Ensure service is running
  ansible.builtin.service:
    name: nginx
    state: started
```

### Interviewer may cross-question

**Interviewer:** "Why not do all of this with shell?"

**Candidate:** "Because purpose-built modules understand resource state and normally give me better idempotency and clearer intent." 

---

## Q5. What happens internally when Ansible executes a module?

### Short explanation of the question

This tests whether I understand the execution lifecycle and can distinguish supported Ansible concepts from internal implementation details.

### Answer

At a high level, Ansible resolves the task and variables, selects the module and connection, prepares the execution, runs it on the target, and processes the returned result. Python module packaging has historically involved an AnsiballZ-style mechanism, but that is an implementation detail.

### Detailed explanation

---

### What is an Ansible Module?

An **Ansible module** is a **self-contained unit of work** that Ansible runs on a target machine to do a specific job.

Think of a module as:

> 🛠️ *"One tool, one responsibility"*

Modules:

* ⚙️ Perform the **actual work** (install, copy, start, configure)
* 🔁 Are **idempotent** (safe to run multiple times)
* 👀 Are **state-aware** (they inspect the system first)
* 📦 Return **structured JSON output** (`changed`, `failed`)
* 🖥️ Run on the **target host**, not the control node

---

### Simple Module Example

```yaml
- name: Ensure nginx is installed
  ansible.builtin.package:
    name: nginx
    state: present
```

🧠 What this really says:

> "I want nginx installed. If it’s already there, don’t touch anything."

The **module itself** decides whether work is needed.

---

### How Ansible Executes a Module (Big Picture)

Ansible is **agentless** 🚫🤖 — target nodes do NOT run Ansible.

So Ansible follows this smart approach:

➡️ Wrap the module into a temporary runner (**AnsiballZ**)
➡️ Transfer it to the target host over SSH
➡️ Execute it locally
➡️ Read structured output
➡️ Clean everything up

Nothing stays behind.

---

### What is AnsiballZ?

**AnsiballZ** is a **temporary Python wrapper** created at runtime.

It contains:

* 🧩 Python bootstrap code
* 📜 Embedded module logic (base64 encoded)
* 🎯 Task arguments from the playbook
* 📤 JSON result handling

📍 Location on target host (temporary):

```text
/tmp/ansible_<random>/AnsiballZ_<module>.py
```

---

### Internal Execution Flow (Step-by-Step)

🔹 Ansible reads the playbook on the **control node**
🔹 Selects the required module + arguments
🔹 Dynamically builds the **AnsiballZ wrapper**
🔹 Transfers it over SSH (scp / sftp)
🔹 Executes it using Python on the target host
🔹 Module inspects current system state
🔹 Compares it with desired state
🔹 Makes changes **only if required**
🔹 Returns structured JSON output
🔹 Ansible cleans up temporary files

---

### Example: `service` Module in Action

```yaml
- name: Ensure nginx is running
  ansible.builtin.service:
    name: nginx
    state: started
```

What happens internally:

* 🧪 Checks: `systemctl is-active nginx`
* 🔍 If already running → no change
* ▶️ If stopped → start service

Returned result:

```json
{ "changed": false }
```

👆 This JSON — **not exit codes** — tells Ansible what happened.

---

### Full Execution Flow Diagram

```text
Playbook (YAML)
     │
     ▼
Ansible Control Node
     │  (wrap module + args)
     │
     ▼
AnsiballZ Wrapper
     │  (SSH: scp / sftp)
     ▼
Target Host
     │  /tmp/ansible_xxx/AnsiballZ_module.py
     │
     ▼
Python Execution
     │
     ▼
Module Logic
     │  check state → compare → act
     ▼
JSON Result (changed / failed)
     │
     ▼
Ansible Core
     │
     ▼
Next Task / Handler
```

---

### Normal Module vs `raw` Command

| Feature            | 🧠 Normal Module | ⚠️ raw Command |
| ------------------ | ---------------- | -------------- |
| Uses AnsiballZ     | ✅ Yes            | ❌ No           |
| State-aware        | ✅ Yes            | ❌ No           |
| Idempotent         | ✅ Yes            | ❌ No           |
| Returns JSON       | ✅ Yes            | ❌ No           |
| Handlers supported | ✅ Yes            | ❌ No           |

🧩 `raw` is mainly used for **bootstrapping or recovery**.

---

### Final Takeaway

> 🟢 **Ansible modules are intelligent, state-aware units of work.**
> 🟢 **AnsiballZ is the temporary wrapper that makes agentless execution possible.**
> 🟢 **Modules inspect, compare, act, report — then disappear.**

---

## Q6. How does Ansible maintain state without a state file?

### Short explanation of the question

This tests a common misconception: Ansible can reconcile current state without maintaining a Terraform-style persistent state file.

### Answer

Ansible modules inspect the current state of the target during execution and compare it with the state requested by the task. Idempotent modules change the resource only when necessary, so Ansible can correct drift without a Terraform-style state file.

### Detailed explanation

This canvas explains the **full concept** of how Ansible maintains state **without storing any state file**, and how this is fundamentally different from **Terraform**.

---

### 1

### Ansible does NOT have a state file

* No `state.tf`
* No central state database
* No stored desired-vs-actual snapshot

### Ansible is **state-aware**, not **state-storing**

It **re-discovers the current state every time it runs** by inspecting the target system.

---

### 2

Ansible maintains correctness using **three mechanisms**:

1. **Live system inspection**
2. **Idempotent modules**
3. **Desired state declaration**

There is **no memory between runs**.

---

### 3

### Desired State (Playbook)

```yaml
- name: Ensure nginx is installed
  ansible.builtin.package:
    name: nginx
    state: present
```

---

### What Ansible Does at Runtime

### Step 1: Connect to target host

* Uses SSH
* Copies a temporary Python module
* Executes it remotely

### Step 2: Inspect current state

The module queries the system directly.

Examples (internally):

* Ubuntu → `dpkg -s nginx`
* RHEL → `rpm -q nginx`

### Step 3: Compare with desired state

| Current State | Desired State | Action     |
| ------------- | ------------- | ---------- |
| Installed     | present       | Do nothing |
| Not installed | present       | Install    |

### Step 4: Act only if mismatch

* If mismatch → change system
* If match → exit safely

### Step 5: Return result

```json
{
  "changed": false
}
```

👉 This is how Ansible enforces state **without storing it**.

---

### 4

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

### 5

* Engineer runs: `chmod 777 /opt/app`
* Ansible is run again

✅ Ansible re-checks live state
✅ Detects drift
✅ Corrects it automatically

No state file needed.

---

### 6

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

### 7

Terraform uses a **persistent state file** to track infrastructure.

### Terraform State File

* `terraform.tfstate`
* Stored locally or remotely (S3, GCS, etc.)
* Maps **resources ↔ real-world objects**

---

### 8

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

### 9

| Aspect          | Ansible                  | Terraform                   |
| --------------- | ------------------------ | --------------------------- |
| State storage   | ❌ None                   | ✅ State file                |
| State detection | Live system inspection   | State file comparison       |
| Drift detection | On every run             | Based on state refresh      |
| Failure risk    | No state corruption      | State file critical         |
| Best for        | Configuration management | Infrastructure provisioning |

---

### How Ansible Confirms State (No State File, No Guessing)

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

### Why Ansible Was Designed This Way

* Stateless control node
* Safe for configuration management
* Works even with manual changes
* No shared lock or state corruption

Terraform **must** store state because cloud APIs do not expose full intent.

---

### Interview-Perfect Summary

**“Ansible does not maintain a state file. It enforces state by inspecting the live system at runtime using idempotent modules and comparing it with the desired configuration. Terraform, in contrast, relies on a persistent state file to map and manage infrastructure resources.”**

---

✅ This conceptual difference explains **why Ansible and Terraform are complementary, not competitors**.

### Comparison

```text
Ansible:
desired state + runtime inspection
          ↓
     reconcile target

Terraform:
configuration + persisted state
          ↓
     plan infrastructure
```

### Example

If an administrator stops a service manually, a later Ansible run can inspect the service state and start it again because the service module knows the desired state.

---
