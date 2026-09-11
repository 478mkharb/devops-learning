# Ansible 

| Priority | Questions | Main Focus |
|---|---|---|
| 🔴 P1 - High | 1–45 | Architecture, YAML, inventory, modules, idempotency, variables, strategies, handlers, roles, security, cloud and production scenarios |
| 🟠 P2 - Medium | 46–70 | Supporting features, debugging, safe deployments and operational practices |
| 🟢 P3 - Low | 71–75 | Other automation/configuration-management tools and their models |

---


# 2. P1 — Core Ansible, Architecture and Execution

## 1. What is Ansible and what problem does it solve?

**Priority: 🔴 P1**

Ansible is an automation and configuration-management tool used to automate:

- Server provisioning
- Package installation
- Configuration management
- Application deployment
- Service management
- User management
- Cloud operations

A typical architecture is:

```text
Developer / Jenkins
        |
        v
 Ansible Control Node
        |
        | SSH / WinRM / supported connection
        v
+-------+-------+-------+
|       |       |       |
Web01  Web02   DB01   DB02
```

Ansible uses YAML playbooks to describe desired operations and modules to perform them.

---

## 2. Is Ansible push-based or pull-based? Why is it called push-based?

**Priority: 🔴 P1 — Very Important**

Ansible is primarily **push-based**.

In a push model, the **control node initiates execution** and pushes the required automation to managed nodes.

```text
             PUSH
Control Node ---------> Managed Node
                         |
                         v
                    Task executes
```

For example:

```bash
ansible-playbook deploy.yml
```

The control node:

1. Reads the inventory.
2. Determines the target hosts.
3. Establishes the connection.
4. Executes the required modules/tasks on those hosts.
5. Collects results back.

### Why is it called push-based?

Because the managed node does not normally poll the Ansible controller asking:

> "Do you have any new configuration for me?"

Instead, the controller decides:

> "These servers need this configuration now."

### Push vs pull

| Model | Who initiates? | Example |
|---|---|---|
| Push | Controller | Ansible |
| Pull | Managed node/agent | Puppet, Chef client model |

Ansible can use different connection mechanisms, but its normal automation model remains controller-driven.

---

## 3. What are the basic requirements to run Ansible?

**Priority: 🔴 P1**

At minimum, you need:

### Control node

- Ansible installed
- Python environment suitable for the installed Ansible version
- Inventory
- Network connectivity to managed hosts
- Authentication/credentials

### Linux managed node

Normally:

- SSH connectivity
- A supported Python runtime for normal module execution
- Appropriate remote user
- Required privileges for the tasks

Typical flow:

```text
Control Node
   |
   | SSH
   v
Linux Managed Node
   |
   +-- Python
   +-- Required privileges
```

### Important exception

The **raw module** can be useful for bootstrapping a host where normal Ansible module execution cannot yet work, such as when Python is missing.

---

## 4. Explain the basic Ansible architecture.

**Priority: 🔴 P1**

| Component | Purpose |
|---|---|
| Control Node | Runs Ansible |
| Managed Node | Target system |
| Inventory | Defines targets |
| Playbook | Automation definition |
| Play | Maps tasks to hosts |
| Task | Individual operation |
| Module | Performs operation |
| Role | Reusable automation structure |
| Handler | Executes notified actions |
| Facts | Host information |

---

## 5. What is the difference between an ad-hoc command and a playbook?

**Priority: 🔴 P1**

Ad-hoc commands are useful for quick operations:

```bash
ansible web -m ansible.builtin.command -a "uptime"
```

Playbooks are structured, repeatable automation:

```yaml
- name: Configure web servers
  hosts: web
  become: true

  tasks:
    - name: Install nginx
      ansible.builtin.apt:
        name: nginx
        state: present
```

| Ad-hoc | Playbook |
|---|---|
| One-off operation | Repeatable automation |
| Quick troubleshooting | Production automation |
| Less structured | Version-controlled |
| Usually simple | Can contain roles, handlers, variables, etc. |

---

## 6. What are the basic types of Ansible modules?

**Priority: 🔴 P1**

There are several useful ways to classify modules. For interviews, keep **source-based classification** separate from **functionality-based classification**.

### Modules classified by source/origin

| Source | Meaning | Examples |
|---|---|---|
| `ansible.builtin` | Shipped with Ansible | `copy`, `file`, `template`, `command` |
| Community Collections | Maintained outside the Ansible core package | `community.general.*` |
| Vendor Collections | Maintained for a vendor/platform | `amazon.aws.*`, `kubernetes.core.*` |
| Custom modules | Written by an organization/user | Internal application modules |

Example:

```yaml
ansible.builtin.copy:
```

versus:

```yaml
amazon.aws.ec2_instance:
```

The fully qualified collection name (FQCN) makes the module's namespace explicit.

### Modules classified by functionality

| Functionality | Typical modules |
|---|---|
| Package management | `apt`, `dnf`, `package` |
| Service management | `service`, `systemd` |
| File management | `file`, `copy`, `template` |
| Text/configuration | `lineinfile`, `blockinfile`, `replace` |
| User management | `user`, `group` |
| Command execution | `command`, `shell`, `raw` |
| Cloud | `amazon.aws.*`, `azure.azcollection.*` |
| Network | `ansible.netcommon.*`, vendor network collections |
| Database | Database-specific collection modules |
| HTTP/API | `uri` |
| Source control | `git` |

---

## 7. YAML Scalar Styles: What are `|`, `>`, `|-`, `|+`, `>-`, and `>+`?

**Priority: 🔴 P1 — Important YAML/Ansible Fundamental**

These are YAML **block scalar indicators** used to control how multi-line strings are represented.

| Syntax | Name | Behavior | Common Ansible use |
|---|---|---|---|
| `|` | Literal | Preserves line breaks | Multi-line configuration/content |
| `>` | Folded | Folds line breaks into spaces | Long logical text |
| `|-` | Literal + strip | Preserves line breaks, strips final newline | Exact multi-line content |
| `|+` | Literal + keep | Preserves line breaks and trailing newlines | Rare trailing-newline cases |
| `>-` | Folded + strip | Folds lines, strips final newline | Long text without final newline |
| `>+` | Folded + keep | Folds lines, keeps trailing newlines | Rare |

### Examples

```yaml
content: |
  line 1
  line 2
  line 3
```

The line breaks are preserved.

```yaml
message: >
  This is a long
  message written
  across several
  lines.
```

Conceptually becomes:

```text
This is a long message written across several lines.
```

### Interview shortcut

```text
|  → Literal → line breaks stay
>  → Folded  → line breaks become spaces

-  → strip final newline
+  → keep trailing newlines
```

> **Important:** `|` and `>` are **YAML syntax**, not Ansible modules.

---

## 8. What is idempotency in Ansible?

**Priority: 🔴 P1**

Idempotency means repeated execution converges to the same desired state without causing unnecessary changes.

Good:

```yaml
- name: Ensure nginx is installed
  ansible.builtin.apt:
    name: nginx
    state: present
```

Bad pattern:

```yaml
- name: Install nginx
  ansible.builtin.shell: apt-get install -y nginx
```

The second approach gives Ansible less information about desired state.

### Interview brain teaser

```yaml
- name: Create user
  ansible.builtin.command: useradd mukesh
```

Run it twice.

The second execution may fail because the user already exists.

Better:

```yaml
- name: Ensure user exists
  ansible.builtin.user:
    name: mukesh
    state: present
```

---

## 9. What is the difference between `command`, `shell`, and `raw`?

**Priority: 🔴 P1**

| Feature | `command` | `shell` | `raw` |
|---|---|---|---|
| Uses shell features | No | Yes | Remote shell directly |
| Pipes/redirection | No | Yes | Yes, shell-dependent |
| Normal module execution | Yes | Yes | No |
| Common use | Simple command | Shell-specific command | Bootstrap/special cases |

Prefer:

```yaml
- ansible.builtin.command: uptime
```

Use `shell` when shell interpretation is actually required:

```yaml
- ansible.builtin.shell: "cat app.log | grep ERROR"
```

### Why use `raw`?

`raw` executes a command directly through the connection without requiring the normal Ansible module execution environment on the remote host.

Example:

```yaml
- name: Bootstrap Python
  ansible.builtin.raw: apt-get update && apt-get install -y python3
```

This can be useful when a newly provisioned Linux host does not yet have Python available.

### Important limitation

`raw` should not become the default replacement for normal modules. You lose many of the advantages of module-based automation, including easier idempotency and structured results.

---

## 10. What is an Ansible inventory?

**Priority: 🔴 P1**

Inventory defines the systems Ansible can manage and groups them logically.

```ini
[web]
web01
web02

[db]
db01
db02
```

It can also contain connection information:

```ini
web01 ansible_host=10.0.1.25
```

Inventory may be static or dynamically generated through inventory plugins.

---


# 3. P1 — Inventory and Dynamic Inventory

## 11. What is dynamic inventory and why is it important in cloud environments?

**Priority: 🔴 P1 — Very Important**

Static inventory becomes difficult to maintain in environments where machines are frequently created and destroyed.

Example:

```text
Before:
web01 = 10.0.1.10
web02 = 10.0.1.11

ASG replaces web02

After:
web01 = 10.0.1.10
web-new = 10.0.1.35
```

A static inventory can become stale.

Dynamic inventory queries the source of truth, such as AWS, and builds the Ansible inventory from current resources.

```text
             AWS
              |
        EC2 / ASG API
              |
              v
     Dynamic Inventory Plugin
              |
              v
        Ansible Groups
              |
              v
          Playbook
```

---

## 12. How does dynamic inventory work with cloud platforms?

**Priority: 🔴 P1 — Very Important**

The process is:

```text
1. Ansible invokes inventory plugin
                ↓
2. Plugin queries cloud API
                ↓
3. Cloud returns current resources
                ↓
4. Plugin applies filters/grouping
                ↓
5. Ansible builds inventory in memory
                ↓
6. Playbook targets resulting groups
```

For AWS, instances can be grouped using tags.

Example conceptual tags:

```text
Environment = dev
Application = notification
Role        = app
```

The dynamic inventory can create a group representing matching instances.

### Key interview point

Dynamic inventory is not simply "automatically changing an IP file."

It is a mechanism for **discovering current infrastructure from an external source of truth**.

---

## 13. How would you use dynamic inventory with an AWS Auto Scaling Group?

**Priority: 🔴 P1 — Scenario**

Suppose an ASG manages 20 application servers.

Instances can be replaced at any time.

Do not maintain:

```ini
[app]
10.0.1.10
10.0.1.11
...
```

Instead:

```text
AWS ASG
  |
  +-- Current EC2 instances
  |
  +-- Tags
       |
       v
amazon.aws.aws_ec2
       |
       v
Ansible group
       |
       v
Deployment playbook
```

This ensures that a newly created instance can be discovered without manually editing inventory.

---


# 4. P1 — Variables and Variable Precedence

## 14. What are `group_vars` and `host_vars`?

**Priority: 🔴 P1**

`group_vars` defines values common to a group:

```text
group_vars/
└── web.yml
```

```yaml
app_port: 8080
```

`host_vars` defines host-specific values:

```text
host_vars/
└── web01.yml
```

```yaml
app_port: 9090
```

This separates configuration from automation logic.

---

## 15. Explain Ansible variable precedence.

**Priority: 🔴 P1 — Very Important**

## 🔺 Variable Precedence Pyramid (Lowest → Highest)

```text
                🔺 Extra vars (-e)
              🔺 Task vars
            🔺 Block vars
          🔺 Include vars / set_fact
        🔺 Role vars (role/vars)
      🔺 Play vars
    🔺 Host vars
  🔺 Group vars
🔺 Role defaults (role/defaults)
```

📌 Bottom = weakest priority
📌 Top = strongest priority

### Practical example

```yaml
# role defaults
app_port: 80
```

```yaml
# group_vars/web.yml
app_port: 8080
```

```yaml
# host_vars/web01.yml
app_port: 9090
```

Command:

```bash
ansible-playbook site.yml -e "app_port=7070"
```

For this example, `7070` wins because extra vars have very high precedence.

### Interview advice

If the interviewer asks for the exact complete precedence order, state that Ansible has a detailed precedence hierarchy and then explain the important high-level levels rather than inventing an oversimplified order.

---

## 16. What is `set_fact`?

**Priority: 🔴 P1**

`set_fact` creates variables during execution.

```yaml
- name: Set deployment version
  ansible.builtin.set_fact:
    app_version: "2.4.1"
```

It is useful when a value is calculated or discovered at runtime.

---

## 17. What is `register`?

**Priority: 🔴 P1**

`register` captures a task's result.

```yaml
- name: Check nginx
  ansible.builtin.command: systemctl is-active nginx
  register: nginx_status
```

You can then use:

```yaml
when: nginx_status.rc == 0
```

or inspect:

```yaml
- ansible.builtin.debug:
    var: nginx_status
```

---

## 18. Explain `when`, `changed_when`, and `failed_when`.

**Priority: 🔴 P1**

| Keyword | Purpose |
|---|---|
| `when` | Whether a task should execute |
| `changed_when` | Whether result should be marked changed |
| `failed_when` | Whether result should be treated as failure |

Example:

```yaml
- name: Check application
  ansible.builtin.command: /opt/app/check.sh
  register: result
  changed_when: false
  failed_when: result.rc == 1
```

---


# 5. P1 — Strategies, Forks and Serial

## 19. What is Ansible's strategy?

**Priority: 🔴 P1 — Very Important**

Strategy controls **how Ansible schedules task execution across hosts**.

Common strategies include:

| Strategy | Behavior |
|---|---|
| `linear` | Default style; hosts progress through tasks in coordinated batches |
| `free` | A host can continue to later tasks without waiting for slower hosts |
| `host_pinned` | Similar to free execution, but each host remains associated with a worker process |

Example:

```yaml
- hosts: web
  strategy: free
```

### Linear

Suppose there are:

```text
web01
web02
web03
```

Conceptually:

```text
Task 1 → all hosts
Task 2 → all hosts
Task 3 → all hosts
```

A slower host can affect progression because hosts remain coordinated by the strategy.

### Free

With:

```yaml
strategy: free
```

one host can progress while another is still working on an earlier task.

```text
web01: Task1 → Task2 → Task3
web02: Task1 → [slow]
web03: Task1 → Task2 → Task3
```

This can improve throughput when hosts are independent.

---

## 20. What is `host_pinned` strategy and when would you use it?

**Priority: 🔴 P1**

`host_pinned` provides free-like host progression while maintaining a stronger association between hosts and worker processes.

The practical idea is:

```text
Host A → Worker A
Host B → Worker B
Host C → Worker C
```

A host can progress independently, but its execution remains pinned to its worker.

This can be useful where the behavior of `free` is desirable but worker/host affinity matters.

### Interview distinction

Do not say:

> "`host_pinned` is the same as `free`."

A better answer:

> "`free` allows hosts to progress independently; `host_pinned` also permits independent host progression while keeping a host associated with a worker process."

---

## 21. How do strategies interact with `forks`?

**Priority: 🔴 P1 — Very Important**

These solve different problems.

### `forks`

Controls how many worker processes Ansible can use concurrently.

Example:

```ini
[defaults]
forks = 10
```

### Strategy

Controls **how those workers schedule task execution across hosts**.

Think:

```text
forks = execution capacity
strategy = execution behavior
```

Example:

```text
100 hosts
forks = 10
```

Ansible cannot normally process all 100 hosts simultaneously through 100 workers if only 10 forks are configured.

The strategy determines how those available workers progress through tasks.

---

## 22. What is the difference between `forks` and `serial`?

**Priority: 🔴 P1 — Very Important**

This is one of the most useful L2 questions.

| `forks` | `serial` |
|---|---|
| Controls worker concurrency | Controls deployment batch size |
| Mainly affects execution capacity | Mainly controls rollout scope |
| Global/configuration level | Play-level rollout control |
| Example: 20 workers | Example: 2 hosts at a time |

Example:

```yaml
- hosts: web
  serial: 2
```

means only two hosts participate in the current rollout batch.

If:

```ini
forks = 20
```

Ansible may have capacity for many concurrent workers, but `serial: 2` intentionally limits the active deployment batch.

### Critical interview point

`forks` does **not** mean:

> "Always run tasks on exactly N hosts."

`serial` is what you use to deliberately control rollout batches.

---

## 23. Give an example using `forks` and `serial` together.

**Priority: 🔴 P1 — Scenario**

Suppose:

```text
100 application servers
forks = 20
serial = 5
```

The practical result is that the play processes only **5 target hosts in the current serial batch**, even though the controller has capacity for up to 20 workers.

Conceptually:

```text
100 hosts
    |
serial: 5
    |
+---+---+---+---+---+
| 5 hosts            |
+--------------------+
        |
  available worker
    capacity = 20
```

The smaller applicable constraint controls the active rollout.

### Production use

For a risky production deployment:

```yaml
serial: 5
```

can limit blast radius while a reasonable `forks` value keeps the controller capable of handling normal concurrency.

---

## 24. How does `serial` work in rolling deployments?

**Priority: 🔴 P1**

With:

```yaml
- hosts: web
  serial: 2
```

and six servers:

```text
Batch 1: web01 web02
Batch 2: web03 web04
Batch 3: web05 web06
```

This is useful when deploying applications behind a load balancer.

A production flow could be:

```text
Take batch out of traffic
        ↓
Deploy
        ↓
Health check
        ↓
Return batch to traffic
        ↓
Next batch
```

---

## 25. What happens if a server fails during a serial deployment?

**Priority: 🔴 P1 — Scenario**

`serial` controls batching; it does not automatically define the complete failure policy.

You should combine rollout control with failure thresholds, health checks, and appropriate error handling.

Useful concepts include:

```yaml
serial: 2
max_fail_percentage: 20
```

The exact behavior depends on the play and failure conditions.

### Interview answer

> "I would use `serial` to reduce blast radius and combine it with health checks and failure thresholds so that a bad release does not continue blindly across the fleet."

---


# 6. P1 — File and Configuration Management

## 26. What is the difference between `lineinfile`, `blockinfile`, and `replace`?

**Priority: 🔴 P1 — Very Important**

| Module | Best use |
|---|---|
| `lineinfile` | Ensure one specific line exists/has a specific value |
| `blockinfile` | Manage a logical multi-line block |
| `replace` | Replace text matching a regular expression |

### `lineinfile`

Use for a single line.

```yaml
- name: Set SSH port
  ansible.builtin.lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^#?Port '
    line: 'Port 2222'
```

### `blockinfile`

Use when you own/manage a block of multiple lines.

```yaml
- name: Add application environment block
  ansible.builtin.blockinfile:
    path: /etc/profile.d/app.sh
    marker: "# {mark} ANSIBLE APP CONFIG"
    block: |
      export APP_ENV=prod
      export APP_PORT=8080
      export APP_NAME=notification
```

### `replace`

Use regex-based replacement.

```yaml
- name: Replace all HTTP ports
  ansible.builtin.replace:
    path: /etc/app/config.conf
    regexp: 'port=80'
    replace: 'port=8080'
```

### Interview decision rule

```text
One logical line?
        ↓
   lineinfile

Multiple managed lines?
        ↓
   blockinfile

Regex-based text replacement?
        ↓
     replace
```

For complex application configuration, a `template` is often cleaner than repeatedly editing individual lines.

---


# 7. P1 — Handlers and Execution Control

## 27. What is a handler?

**Priority: 🔴 P1**

A handler is a task that runs when notified by a changed task.

```yaml
tasks:
  - name: Update nginx config
    ansible.builtin.template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify: Restart nginx

handlers:
  - name: Restart nginx
    ansible.builtin.service:
      name: nginx
      state: restarted
```

If the template does not change, the handler is not triggered.

---

## 28. When does a handler execute?

**Priority: 🔴 P1 — Brain Teaser**

By default, handlers run after the normal tasks for the relevant play have completed.

If several tasks notify the same handler:

```text
Task 1 → notify Restart
Task 2 → notify Restart
Task 3 → notify Restart
```

the handler normally executes once for that host.

It is a queued notification, not an immediate task execution.

---

## 29. What happens to a notified handler if a later task fails?

**Priority: 🔴 P1 — Brain Teaser**

Example:

```text
Task 1 → config changed → notify Restart
Task 2 → success
Task 3 → failure
```

By default, a failed host may not execute its pending handlers.

Use:

```yaml
force_handlers: true
```

when the play's design requires notified handlers to execute despite later task failure.

---

## 30. What is `delegate_to`?

**Priority: 🔴 P1**

`delegate_to` changes where a task executes.

```yaml
- name: Update deployment system
  ansible.builtin.command: /opt/update.sh
  delegate_to: localhost
```

The play can target 20 application servers, while this particular task runs on the control node.

Common uses:

- Load balancer operations
- API calls
- DNS operations
- Orchestration steps

---

## 31. What is `run_once`?

**Priority: 🔴 P1**

It executes a task once for the current play rather than once per target host.

```yaml
- name: Create deployment record
  ansible.builtin.command: /opt/create-record.sh
  run_once: true
```

With 20 target hosts, the task runs once.

Together:

```yaml
delegate_to: localhost
run_once: true
```

means the task runs once on `localhost`.

---


# 8. P1 — Roles, Templates and Reuse

## 32. What is an Ansible Role?

**Priority: 🔴 P1**

A role packages reusable automation.

```text
roles/
└── nginx/
    ├── tasks/
    ├── handlers/
    ├── templates/
    ├── files/
    ├── defaults/
    ├── vars/
    └── meta/
```

Roles improve:

- Reusability
- Maintainability
- Separation of concerns
- Team collaboration

---

## 33. What is the difference between role `defaults` and `vars`?

**Priority: 🔴 P1**

`defaults/main.yml` is intended for values that consumers can override.

```yaml
nginx_port: 80
```

`vars/main.yml` has higher precedence and is generally used for role-controlled values.

### Practical rule

> Put configurable role inputs in `defaults`; don't unnecessarily put user-configurable values in `vars`.

---

## 34. What is Jinja2 templating?

**Priority: 🔴 P1**

Jinja2 allows dynamic configuration generation.

Template:

```jinja2
server {
    listen {{ nginx_port }};
    server_name {{ server_name }};
}
```

Task:

```yaml
- name: Generate nginx config
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
```

Variables are rendered before the resulting file is placed on the target.

---

## 35. What is the difference between `copy` and `template`?

**Priority: 🔴 P1**

| `copy` | `template` |
|---|---|
| Static content | Dynamic content |
| Copies file | Renders Jinja2 first |
| Good for fixed files | Good for environment-specific configuration |

Example:

```yaml
- ansible.builtin.copy:
    src: app.conf
    dest: /etc/app/app.conf
```

versus:

```yaml
- ansible.builtin.template:
    src: app.conf.j2
    dest: /etc/app/app.conf
```

---

## 36. What is the difference between `include_tasks` and `import_tasks`?

**Priority: 🔴 P1**

The interview-friendly distinction:

| `import_tasks` | `include_tasks` |
|---|---|
| Static | Dynamic |
| Processed during parsing | Included during execution |
| Good for predictable task structure | Good for runtime conditions/selection |

Example:

```yaml
- ansible.builtin.include_tasks: "{{ ansible_os_family }}.yml"
```

is useful when the task file depends on runtime facts.

---


# 9. P1 — Security, Validation and Troubleshooting

## 37. What is Ansible Vault?

**Priority: 🔴 P1**

Vault encrypts sensitive Ansible data such as:

- Passwords
- API tokens
- Secrets
- Private configuration

Example:

```bash
ansible-vault encrypt secrets.yml
```

Then:

```bash
ansible-playbook site.yml --ask-vault-pass
```

### Brain teaser

If this is committed to Git:

```yaml
db_password: MySecret123
```

Vault cannot protect it.

Only encrypted data is protected by Vault.

---

## 38. What is check mode?

**Priority: 🔴 P1**

Check mode previews changes where the relevant modules support it.

```bash
ansible-playbook site.yml --check
```

It is useful before production changes.

---

## 39. What is diff mode?

**Priority: 🔴 P1**

Diff mode shows supported configuration differences.

```bash
ansible-playbook site.yml --diff
```

Often:

```bash
ansible-playbook site.yml --check --diff
```

Be careful because diffs can expose sensitive configuration.

---

## 40. `ansible all -m ping` works but the playbook fails with sudo/password errors. Why?

**Priority: 🔴 P1 — Scenario**

`ping` proves basic connectivity/module execution. It does not prove that privilege escalation works.

If the playbook uses:

```yaml
become: true
```

Ansible may need sudo/become credentials.

Useful distinction:

```text
SSH/connectivity
       ≠
Privilege escalation
```

Investigate:

```bash
ansible all -m ping
ansible all -m command -a "id" -b
```

and the become configuration.

---

## 41. `ansible all -m ping` fails with SSH timeout. What do you check?

**Priority: 🔴 P1 — Troubleshooting Scenario**

Check systematically:

```text
Instance running?
      ↓
Correct address?
      ↓
Network route?
      ↓
Security Group?
      ↓
NACL?
      ↓
Port 22 reachable?
      ↓
Bastion/proxy required?
      ↓
Correct username/key?
      ↓
sshd running?
      ↓
Ansible connection variables correct?
```

Use:

```bash
ansible-inventory --graph
ansible all -m ping -vvv
```

---

## 42. Playbook reports success, but application still uses old configuration. What do you investigate?

**Priority: 🔴 P1 — Troubleshooting Brain Teaser**

Follow the chain:

```text
Did template/copy change?
        ↓
Was notify triggered?
        ↓
Did handler execute?
        ↓
Did another task overwrite the file?
        ↓
Is application reading that file?
        ↓
Did reload/restart succeed?
        ↓
Is running process using new configuration?
```

Useful:

```bash
ansible-playbook site.yml --check --diff
ansible-playbook site.yml -vv
```

The important L2 skill is structured diagnosis rather than immediately adding a restart.

---


# 10. P1 — Cloud and Production Scenarios

## 43. Can Ansible manage EC2 instances in private subnets?

**Priority: 🔴 P1**

Yes, provided the control node has a supported connection path.

Examples:

```text
Control Node
     |
    VPN
     |
Private EC2
```

or:

```text
Control Node
     |
  Bastion
     |
Private EC2
```

Other architectures can use AWS Systems Manager-based connection mechanisms where supported by the Ansible setup.

The important point is:

> Agentless does not mean connectionless. The control node still needs a valid management path.

---

## 44. How would you perform a zero/low-downtime deployment with Ansible?

**Priority: 🔴 P1 — Scenario**

For servers behind a load balancer:

```yaml
serial: 2
```

Then conceptually:

```text
Take 2 servers out of traffic
        ↓
Deploy application
        ↓
Run health checks
        ↓
Put servers back
        ↓
Continue
```

Combine:

- `serial`
- Load-balancer integration
- Health checks
- Failure thresholds
- Handlers
- Idempotent tasks

The goal is to control **blast radius** and prevent a bad release from reaching the whole fleet.

---

## 45. How would you troubleshoot a playbook that is slow on hundreds of servers?

**Priority: 🔴 P1**

Check:

- `forks`
- Fact gathering
- Network latency
- Expensive shell commands
- Unnecessary repeated operations
- Strategy
- Serial constraints
- External API rate limits
- Slow individual tasks

If facts are unnecessary:

```yaml
gather_facts: false
```

If appropriate:

```ini
[defaults]
forks = 20
```

Do not simply increase forks without checking controller and target capacity.

---


# 11. P2 — Supporting L2 Questions

## 46. What is `become`?

**Priority: 🟠 P2**

`become` enables privilege escalation.

```yaml
become: true
```

Commonly this means using sudo to execute tasks as root.

---

## 47. What are tags?

**Priority: 🟠 P2**

Tags allow selective execution.

```yaml
- name: Install nginx
  ansible.builtin.apt:
    name: nginx
  tags:
    - install
```

Run:

```bash
ansible-playbook site.yml --tags install
```

---

## 48. What is `block`, `rescue`, and `always`?

**Priority: 🟠 P2**

They provide structured error handling.

```yaml
- block:
    - name: Deploy
      ansible.builtin.command: /opt/deploy.sh

  rescue:
    - name: Roll back
      ansible.builtin.command: /opt/rollback.sh

  always:
    - name: Cleanup
      ansible.builtin.file:
        path: /tmp/deploy
        state: absent
```

Think:

```text
block  = normal work
rescue = recovery
always = cleanup/finalization
```

---

## 49. What is Ansible Galaxy?

**Priority: 🟠 P2**

Galaxy is an ecosystem for reusable Ansible content, including roles and collections.

It reduces duplicated automation and helps teams consume reusable community/vendor content.

---

## 50. What are Ansible Collections?

**Priority: 🟠 P2**

Collections package Ansible content such as:

- Modules
- Plugins
- Roles
- Supporting utilities

Examples:

```yaml
amazon.aws.ec2_instance:
```

and:

```yaml
community.general.some_module:
```

---

## 51. What is a module versus a plugin?

**Priority: 🟠 P2**

A **module** performs an operation on a managed system.

A **plugin** extends Ansible behavior.

Examples of plugins:

- Inventory plugins
- Connection plugins
- Lookup plugins
- Filter plugins
- Callback plugins

---

## 52. What is a lookup?

**Priority: 🟠 P2**

A lookup retrieves data from a supported source.

Example:

```jinja2
{{ lookup('env', 'HOME') }}
```

Lookups are especially useful when data needs to be retrieved from the control-node environment or other supported lookup sources.

---

## 53. What is `async` and `poll`?

**Priority: 🟠 P2**

They support long-running asynchronous tasks.

```yaml
- name: Run long deployment
  ansible.builtin.command: /opt/long-deploy.sh
  async: 1800
  poll: 10
```

This allows the task to run asynchronously while Ansible polls for completion.

---

## 54. What is the difference between `inventory_hostname` and `ansible_hostname`?

**Priority: 🟠 P2**

```text
inventory_hostname
    = name used by Ansible inventory

ansible_hostname
    = hostname reported by the managed system's facts
```

They can be different.

---

## 55. What happens if a task is not idempotent?

**Priority: 🟠 P2**

Repeated playbook execution may:

- Report unnecessary changes
- Recreate resources
- Restart services unnecessarily
- Fail on subsequent runs
- Produce configuration drift

L2 engineers should identify non-idempotent operations and replace them with state-aware modules where possible.

---

## 56. What is `changed_when: false` useful for?

**Priority: 🟠 P2**

For read-only commands that Ansible cannot automatically recognize as non-changing:

```yaml
- name: Check application
  ansible.builtin.command: /opt/check.sh
  changed_when: false
```

This keeps reporting accurate.

---

## 57. How do you run only one part of a large playbook?

**Priority: 🟠 P2**

Use tags:

```bash
ansible-playbook site.yml --tags deploy
```

or limit hosts:

```bash
ansible-playbook site.yml --limit web01
```

They solve different problems:

```text
--tags  → which tasks
--limit → which hosts
```

---

## 58. Why should you prefer FQCNs?

**Priority: 🟠 P2**

Fully Qualified Collection Names make module origin explicit.

Instead of:

```yaml
copy:
```

use:

```yaml
ansible.builtin.copy:
```

Benefits:

- Clear namespace
- Less ambiguity
- Better readability
- Better compatibility with modern Ansible content

---

## 59. What is strategy `free` useful for?

**Priority: 🟠 P2**

Use it when target hosts are relatively independent and waiting for the slowest host is undesirable.

```yaml
strategy: free
```

It can improve throughput but should not be used blindly when task ordering across hosts matters.

---

## 60. What is the difference between a module, role, play, and playbook?

**Priority: 🟠 P2**

```text
Module
  ↓
does one type of operation

Task
  ↓
calls a module

Play
  ↓
maps tasks to hosts

Role
  ↓
packages reusable tasks/configuration

Playbook
  ↓
contains one or more plays
```

---


# 12. P2 — Production and Operational Questions

## 61. How do you make an Ansible deployment safe for production?

**Priority: 🟠 P2**

Use:

- Idempotent modules
- `--check` where supported
- `--diff` where safe
- Small `serial` batches
- Health checks
- Failure thresholds
- Handlers
- Vault/secret management
- Dynamic inventory
- CI validation
- Code review
- Rollback strategy

---

## 62. How would you structure Dev, QA and Prod inventories?

**Priority: 🟠 P2**

A common approach:

```text
inventories/
├── dev/
├── qa/
└── prod/
```

Environment-specific values can be managed through group variables and inventory variables.

Keep the automation logic reusable and move environment differences into configuration.

---

## 63. Why should you avoid excessive `shell` usage?

**Priority: 🟠 P2**

Because native modules generally provide:

- Better idempotency
- Structured results
- Better portability
- Clearer intent
- Easier maintenance

Use:

```yaml
ansible.builtin.user
ansible.builtin.file
ansible.builtin.service
ansible.builtin.apt
```

instead of shelling out to equivalent OS commands whenever a suitable module exists.

---

## 64. How do you debug an Ansible task?

**Priority: 🟠 P2**

Useful tools:

```bash
ansible-playbook site.yml -vvv
ansible-playbook site.yml --check
ansible-playbook site.yml --diff
ansible-inventory --graph
```

Also inspect registered variables:

```yaml
- ansible.builtin.debug:
    var: result
```

---

## 65. What is the difference between failure and unreachable?

**Priority: 🟠 P2**

A **failed** host was reachable but a task failed.

An **unreachable** host could not be contacted.

Conceptually:

```text
Could not connect
      → UNREACHABLE

Connected but task failed
      → FAILED
```

This distinction matters when designing recovery and troubleshooting logic.

---

## 66. How would you prevent a bad configuration from being deployed?

**Priority: 🟠 P2**

Use:

```text
Template generation
      ↓
Validation
      ↓
Deploy configuration
      ↓
Notify handler
      ↓
Reload/restart
```

For supported services/modules, validate the generated configuration before applying it.

For example, use an application's native configuration test before restarting it.

---

## 67. How would you roll back an application deployment?

**Priority: 🟠 P2**

A good design keeps a known-good version available.

```text
Current version
      ↓
Deploy new version
      ↓
Health check
   /       \
 PASS      FAIL
  |          |
Continue   Rollback
```

Ansible can implement this with roles, blocks/rescue, versioned artifacts, and deployment state.

---

## 68. How do you avoid exposing secrets in Ansible logs?

**Priority: 🟠 P2**

Use:

```yaml
no_log: true
```

for sensitive tasks where appropriate.

Also:

- Use Ansible Vault or an external secret manager
- Avoid passing secrets unnecessarily in command-line arguments
- Avoid debugging secret variables
- Restrict CI/CD log access

---

## 69. What is the purpose of `ansible.cfg`?

**Priority: 🟠 P2**

`ansible.cfg` controls Ansible behavior.

Examples:

```ini
[defaults]
inventory = ./inventory
forks = 20
host_key_checking = False
```

Be careful with security-sensitive configuration such as disabling host key checking.

---

## 70. What is check mode not guaranteed to do?

**Priority: 🟠 P2**

`--check` depends on module support.

Some modules can accurately predict changes; others cannot.

Therefore:

> Check mode is a useful safety mechanism, not a universal simulation of every side effect.

---


# 13. P3 — Other Automation and Configuration-Management Tools

## 71. What other configuration-management tools are commonly used?

**Priority: 🟢 P3**

Common tools include:

- Puppet
- Chef
- Salt
- CFEngine
- Terraform
- Pulumi
- AWS Systems Manager
- Kubernetes configuration mechanisms

They solve overlapping but not identical problems.

---

## 72. Which tools are generally push-based and which are pull-based?

**Priority: 🟢 P3**

The exact architecture can vary by mode/version, so interview answers should use the common/default model rather than treating every deployment mode as absolute.

| Tool | Common model | Basic idea |
|---|---|---|
| Ansible | Push | Controller initiates execution |
| Puppet | Pull | Agent periodically retrieves/applies catalog |
| Chef | Pull | Chef client periodically contacts server |
| Salt | Push-oriented | Master can send commands to minions |
| CFEngine | Pull | Agents periodically converge toward policy |
| Terraform | Push/control-plane driven | Terraform process calls provider APIs |
| AWS SSM | Control-plane driven | Commands/jobs are delivered through SSM infrastructure |

### Important distinction

Terraform is not simply a "configuration-management push tool." It is primarily **declarative infrastructure provisioning/state management**.

Likewise, Kubernetes is not normally described using only "push vs pull" because its control plane uses controllers and reconciliation loops.

---

## 73. Why did Ansible become popular compared with agent-based tools?

**Priority: 🟢 P3**

Major reasons include:

- No traditional persistent agent requirement on Linux
- SSH-based management
- YAML playbooks
- Large module ecosystem
- Easy integration with CI/CD
- Good support for cloud automation
- Human-readable automation

Agent-based tools can still be preferable when continuous policy enforcement and autonomous convergence are primary requirements.

---

## 74. Is pull-based automation always better than push-based automation?

**Priority: 🟢 P3**

No.

### Push advantages

- Immediate execution
- Simple operational model
- Good CI/CD integration
- Easy targeted deployments

### Pull advantages

- Continuous convergence
- Managed nodes can operate independently
- Useful when controllers cannot directly reach every node

The choice depends on network topology, compliance requirements, scale, drift-management needs, and operational model.

---

## 75. In an interview, how would you compare Ansible with Puppet?

**Priority: 🟢 P3**

A concise answer:

> "Ansible is commonly used in a push-oriented, controller-driven model and is popular for orchestration, configuration management, and CI/CD-driven automation. Puppet traditionally uses an agent-based pull/convergence model, where agents periodically retrieve policy and enforce the desired state."

Then mention that the tools overlap and modern deployments can use additional mechanisms, so the distinction is about their **common architecture and operational model**, not an absolute rule for every feature.

---

# 14. Frequently Used Ansible Modules in DevOps

**Priority: 🔴 P1/P2 — Practical Reference**

| Requirement | Preferred module | Typical purpose |
|---|---|---|
| Directory/file state | `file` | Directories, empty files, permissions, ownership |
| Static file/content | `copy` | Copy fixed files/content |
| Dynamic configuration | `template` | Render Jinja2 templates |
| One configuration line | `lineinfile` | Ensure/change one line |
| Multi-line managed block | `blockinfile` | Add/update a block |
| Regex replacement | `replace` | Replace matching text |
| Package management | `package` / `apt` / `dnf` | Install/remove packages |
| Service management | `service` / `systemd_service` | Start/stop/restart/enable services |
| Users/groups | `user` / `group` | Account management |
| Git deployment | `git` | Clone/update repositories |
| File download | `get_url` | HTTP/HTTPS downloads |
| Archive extraction | `unarchive` | Extract deployment artifacts |
| Simple command | `command` | Execute without shell interpretation |
| Shell syntax | `shell` | Pipes/redirection/shell features |
| Bootstrap | `raw` | Run directly when normal module execution is unavailable |
| HTTP/API | `uri` | REST/API calls and health checks |
| Debugging | `debug` | Inspect values/results |
| File inspection | `stat` | File metadata/state |
| File discovery | `find` | Locate files |
| Validation | `assert` | Enforce assumptions |
| Scheduling | `cron` | Recurring jobs |
| Kernel settings | `sysctl` | Manage kernel parameters |
| AWS | `amazon.aws.*` | AWS resource management |

## 14.1 High-value examples

### `file`

```yaml
- name: Create application directory
  ansible.builtin.file:
    path: /opt/myapp
    state: directory
    owner: ubuntu
    group: ubuntu
    mode: '0755'
```

### `copy`

```yaml
- name: Copy static configuration
  ansible.builtin.copy:
    src: app.conf
    dest: /etc/myapp/app.conf
    mode: '0644'
```

### `template`

```yaml
- name: Generate dynamic configuration
  ansible.builtin.template:
    src: app.conf.j2
    dest: /etc/myapp/app.conf
```

### `lineinfile`

```yaml
- name: Set application port
  ansible.builtin.lineinfile:
    path: /etc/myapp/app.conf
    regexp: '^APP_PORT='
    line: 'APP_PORT=8080'
```

### `blockinfile`

```yaml
- name: Add application environment block
  ansible.builtin.blockinfile:
    path: /etc/profile.d/myapp.sh
    marker: "# {mark} ANSIBLE MYAPP"
    block: |
      export APP_ENV=prod
      export APP_PORT=8080
```

### `replace`

```yaml
- name: Replace HTTP port
  ansible.builtin.replace:
    path: /etc/myapp/app.conf
    regexp: 'port=80'
    replace: 'port=8080'
```

### `git`

```yaml
- name: Clone application
  ansible.builtin.git:
    repo: https://github.com/example/myapp.git
    dest: /opt/myapp
    version: main
```

### `get_url`

```yaml
- name: Download application artifact
  ansible.builtin.get_url:
    url: https://example.com/myapp.tar.gz
    dest: /tmp/myapp.tar.gz
```

### `uri`

```yaml
- name: Check application health
  ansible.builtin.uri:
    url: http://localhost:8080/health
    method: GET
    status_code: 200
```

### `systemd_service`

```yaml
- name: Restart application
  ansible.builtin.systemd_service:
    name: myapp
    state: restarted
```

### `assert`

```yaml
- name: Validate application port
  ansible.builtin.assert:
    that:
      - app_port | int > 0
      - app_port | int < 65536
    fail_msg: "Invalid application port"
```

## 14.2 Module selection rule

```text
Empty file / directory / permissions?
        ↓
      file

Static content?
        ↓
      copy

Dynamic/Jinja2 content?
        ↓
    template

One logical line?
        ↓
   lineinfile

Multiple managed lines?
        ↓
   blockinfile

Regex replacement?
        ↓
     replace

Download a file?
        ↓
     get_url

Call an HTTP/API endpoint?
        ↓
       uri

Simple command?
        ↓
     command

Need shell features?
        ↓
      shell

Normal module execution unavailable?
        ↓
       raw
```

> **Interview rule:** Prefer a purpose-built module when one exists. Use `shell`/`command` only when required, and use `raw` mainly for bootstrap or special environments.

---

# 15. Quick Revision Table

| Topic | Remember This |
|---|---|
| Ansible model | Primarily push/controller-driven |
| Agentless | No traditional persistent Ansible agent required on Linux |
| Dynamic inventory | Discovers current infrastructure from an external source |
| Module | Performs an operation |
| Playbook | Defines automation |
| Role | Packages reusable automation |
| Idempotency | Repeated execution converges to desired state |
| `|` YAML scalar | Literal; preserve line breaks |
| `>` YAML scalar | Folded; line breaks become spaces |
| `|-` / `>-` | Strip final newline |
| `|+` / `>+` | Keep trailing newlines |
| `raw` | Bootstrap/special environments |
| `lineinfile` | One logical line |
| `blockinfile` | Managed multi-line block |
| `replace` | Regex-based replacement |
| Strategy | Controls host/task scheduling |
| `linear` | Coordinated task progression |
| `free` | Hosts can progress independently |
| `host_pinned` | Free-like progression with host/worker affinity |
| Forks | Worker concurrency capacity |
| Serial | Rollout batch size |
| Handler | Runs after notification |
| `delegate_to` | Changes execution host |
| `run_once` | Executes once per play |
| `group_vars` | Group-level configuration |
| `host_vars` | Host-specific configuration |
| `set_fact` | Creates runtime fact/variable |
| `register` | Stores task result |
| `when` | Task execution condition |
| `changed_when` | Controls changed status |
| `failed_when` | Controls failure status |
| `template` | Renders Jinja2 |
| `copy` | Copies static content |
| Vault | Encrypts Ansible secrets |
| `--check` | Preview supported changes |
| `--diff` | Show supported differences |
| `become` | Privilege escalation |
| Collections | Namespaced Ansible content |
| `uri` | HTTP/API interaction |
| `get_url` | File download |
| `git` | Git repository management |
| `systemd_service` | systemd-specific service management |
| Dynamic AWS inventory | Useful with EC2/ASG |
