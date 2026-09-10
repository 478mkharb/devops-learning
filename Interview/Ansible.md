# Ansible — Senior L2 Interview Questions & Answers

## Q1. What is Ansible?

Ansible is an automation platform used for configuration management, application deployment, orchestration, and operational automation.

A typical controller-driven architecture is:

```text
                         CONTROL NODE
                              |
                 +------------+------------+
                 |                         |
              Inventory                Playbook
                 |                         |
                 +------------+------------+
                              |
                     Variables / Facts
                              |
                         Task / Module
                              |
                    Connection mechanism
                    /                  \
                  SSH                 AWS SSM
                   |                    |
                   v                    v
             Managed host             EC2
```

Ansible automation is normally controller-driven. The exact execution mechanism depends on the connection plugin and module.

RH294 describes the control node as the system where Ansible is installed and run, and managed hosts as the systems listed in inventory.

---

## Q2. Explain Ansible architecture and execution flow.

The simplified execution flow is:

```text
Playbook
   ↓
YAML parsing
   ↓
Inventory / host pattern resolution
   ↓
Variable resolution
   ↓
Strategy / serial / forks
   ↓
Task selection
   ↓
Module resolution
   ↓
Connection plugin
   ↓
Module/API execution
   ↓
Structured result
   ↓
ok / changed / skipped / failed
   ↓
Handlers / next task
```

For normal Linux module execution:

```text
Controller
   ↓
Module implementation
   ↓
Connection
   ↓
Managed host
   ↓
Module execution
   ↓
JSON result
   ↓
Controller
```

RH294 explicitly describes the control node, inventory, plays, tasks, modules, and managed hosts as the core architecture.

---

## Q3. What is the difference between a Playbook, Play, Task, and Module?

| Object | Meaning | Example |
|---|---|---|
| Playbook | YAML file containing one or more plays | `site.yml` |
| Play | Maps hosts to tasks/roles/settings | `hosts: web` |
| Task | One unit of requested work | Install nginx |
| Module | Code that performs the operation | `ansible.builtin.apt` |

Example:

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

The hierarchy is:

```text
Playbook
  └── Play
       └── Task
            └── Module
```

---

## Q4. What is an Ansible module?

A module is a reusable unit of automation that performs a specific operation on a managed node or an external system.

Examples:

```yaml
ansible.builtin.copy:
ansible.builtin.template:
ansible.builtin.file:
ansible.builtin.apt:
ansible.builtin.systemd:
ansible.builtin.user:
ansible.builtin.command:
ansible.builtin.shell:
```

Modules generally accept structured arguments and return structured results.

For example:

```yaml
- name: Ensure nginx is installed
  ansible.builtin.apt:
    name: nginx
    state: present
```

Conceptually:

```text
Desired state
     ↓
Inspect current state
     ↓
Already correct? ---- yes ----> ok
     |
     no
     ↓
Make change
     ↓
changed
```

RH294 explains that a module is a small piece of code that takes specific arguments and can act on files, install software, or make API calls.

---

## Q5. What are the main types of Ansible modules?

There are two useful classification dimensions.

### By source/distribution

| Type | Meaning | Example |
|---|---|---|
| Built-in collection | Modules in the built-in collection | `ansible.builtin.copy` |
| Collection/vendor/community | Modules delivered through collections | `amazon.aws.ec2_instance` |
| Custom | Organization-written modules | `mycompany.platform.my_module` |

### By function

| Functional category | Examples |
|---|---|
| Package | `apt`, `dnf`, `yum`, `package` |
| File | `file`, `copy`, `fetch`, `template`, `stat` |
| Text/config | `lineinfile`, `blockinfile`, `replace` |
| Service | `service`, `systemd` |
| Identity | `user`, `group`, `authorized_key` |
| Command | `command`, `shell`, `raw`, `script` |
| Facts | `setup` |
| Validation | `assert`, `fail` |
| Network | OS/network-device-specific modules |
| Cloud | AWS and other cloud collections |

These classifications are not the same thing: "built-in" describes provenance, while "file" or "package" describes function.

---

## Q6. Where are Ansible modules located, and how does Ansible find them?

Modern Ansible organizes modules through collections.

Examples:

```yaml
ansible.builtin.copy:
ansible.builtin.command:
amazon.aws.ec2_instance:
community.general.some_module:
```

The controller resolves the requested module/collection.

A useful command is:

```bash
ansible --version
```

RH294 2.8-era output demonstrated module search locations such as:

```text
~/.ansible/plugins/modules
/usr/share/ansible/plugins/modules
```

The exact paths vary with Ansible version, installation method, collections, and configuration.

To inspect installed collections:

```bash
ansible-galaxy collection list
```

To inspect configuration:

```bash
ansible-config dump
```

### Important distinction

Module resolution is primarily a controller-side concern.

The managed node should not be treated as a machine where Ansible permanently installs every module like a normal application.

---

## Q7. What happens during playbook parsing versus execution?

A simplified model is:

```text
Playbook source
      ↓
Parse YAML
      ↓
Build play/task structure
      ↓
Static imports are processed
      ↓
Runtime execution begins
      ↓
Variables / facts / conditions / includes
      ↓
Module execution
```

The distinction becomes important for imports and includes.

```text
import_tasks
    → static

include_tasks
    → dynamic
```

RH294 states that imported content is preprocessed when the playbook is parsed, while included content is processed during execution as the play reaches it.

---

## Q8. How does Ansible execute a module internally?

For a typical Python-based Linux module:

```text
Task
 ↓
Resolve module
 ↓
Prepare module arguments
 ↓
Connection plugin
 ↓
Remote execution
 ↓
Structured result
 ↓
Controller
```

Historically, Ansible has used packaging mechanisms such as AnsiballZ to prepare Python module payloads.

A useful interview-level explanation is:

> The controller selects the module implementation, prepares the invocation, uses the connection mechanism to execute it, and receives a structured result.

Avoid treating a specific internal packaging implementation as immutable because implementation details can change between versions.

---

## Q9. What is the `.ansible` directory and why is `~/.ansible/tmp` important?

Ansible may create user-specific runtime directories such as:

```text
/home/ubuntu/.ansible/
```

A commonly encountered directory is:

```text
/home/ubuntu/.ansible/tmp/
```

This can be used as a remote temporary location for module execution artifacts.

```text
Controller
    |
    | prepare/execute module
    v
Managed host
    |
    v
~/.ansible/tmp/
    |
    v
temporary module files
    |
    v
module execution
```

| Path | Typical purpose |
|---|---|
| `~/.ansible/` | User-specific runtime-related Ansible data |
| `~/.ansible/tmp/` | Temporary module execution area |
| `/tmp` | Generic OS temporary directory |

RH294's command examples show temporary module files under `/home/devops/.ansible/tmp/...`.

This directory is **not** a Terraform-style state directory.

---

## Q10. Why does Ansible commonly need Python on Linux managed hosts?

Many Linux modules are implemented in Python.

Typical flow:

```text
Controller
   ↓
Python-based module
   ↓
Remote Python interpreter
   ↓
Module result
```

The Python interpreter can be discovered through facts.

Example:

```yaml
- ansible.builtin.debug:
    var: ansible_facts.python.executable
```

If the remote Python interpreter is missing or incompatible, normal Python-based module execution can fail.

---

## Q11. What is the difference between `raw`, `command`, and `shell`?

| Feature | `raw` | `command` | `shell` |
|---|---|---|---|
| Normal module subsystem | No | Yes | Yes |
| Shell interpretation | No | No | Yes |
| Pipes/redirection | No | No | Yes |
| Bootstrap use | Strong | Limited | Limited |
| Preferred routine choice | Rare | Usually | Only when shell features are needed |

Examples:

```yaml
- name: Command without shell processing
  ansible.builtin.command: uptime
```

```yaml
- name: Use shell pipeline
  ansible.builtin.shell: "grep ERROR /var/log/app.log | tail -20"
```

```yaml
- name: Bootstrap a host
  ansible.builtin.raw: apt-get update
```

Interview answer:

> Use a normal module whenever possible; use `command` for command execution without shell interpretation, `shell` when shell syntax is required, and `raw` for low-level/bootstrap situations where the normal module environment is unavailable.

---

## Q12. Which modules are generally idempotent and which are not?

Idempotency is usually a property of the operation plus task design.

### Generally idempotent configuration-oriented modules

| Module | Generally idempotent? |
|---|---|
| `file` | ✅ |
| `copy` | ✅ |
| `template` | ✅ |
| `lineinfile` | ✅ |
| `blockinfile` | ✅ |
| `replace` | ✅ |
| `user` | ✅ |
| `group` | ✅ |
| `apt` | ✅ |
| `dnf` | ✅ |
| `yum` | ✅ |
| `package` | ✅ |
| `service` | ✅ generally |
| `systemd` | ✅ generally |
| `authorized_key` | ✅ |
| `cron` | ✅ |
| `git` | ✅ generally |

### Not inherently idempotent

| Module | Inherently idempotent? | Reason |
|---|---|---|
| `command` | ❌ | Executes an instruction |
| `shell` | ❌ | Executes shell commands |
| `raw` | ❌ | Bypasses the normal module subsystem |
| `script` | ❌ | Runs a script; the script determines state |

Example:

```yaml
- name: Create marker
  ansible.builtin.command: touch /tmp/marker
```

Ansible does not automatically infer that the real requirement is "the file must exist."

### Making command execution conditionally idempotent

```yaml
- name: Initialize application
  ansible.builtin.command: /opt/app/init.sh
  args:
    creates: /opt/app/.initialized
```

You can also use:

```yaml
creates:
removes:
changed_when:
```

### Important nuance

Even an idempotent module can be used in a non-idempotent way.

For example, a template containing a changing timestamp produces different desired content every run.

```text
Module capability
      +
Task design
      +
Input data
      ↓
Actual idempotency
```

RH294 explicitly emphasizes that playbooks/tasks should be safely repeatable and warns that arbitrary command modules need care to remain idempotent.

---

## Q13. What is `rc` in an Ansible command result?

`rc` means **return code** or **exit code**.

Example:

```yaml
- name: Run health check
  ansible.builtin.command: /usr/local/bin/healthcheck
  register: result

- ansible.builtin.debug:
    var: result.rc
```

Typical meanings:

```text
rc = 0      → command conventionally succeeded
rc != 0     → command reported an error condition
```

The exact meaning of a non-zero code depends on the command.

Example:

```yaml
result:
  rc: 1
  stdout: ""
  stderr: "Connection refused"
```

Compare:

| Field | Meaning |
|---|---|
| `rc` | Command exit code |
| `stdout` | Standard output |
| `stderr` | Standard error |
| `changed` | Whether task reports a change |
| `failed` | Whether task reports failure |

RH294 examples show command output containing `rc=0`.

---

## Q14. What are Ansible ad-hoc commands?

Ad-hoc commands run a module directly from the command line without creating a playbook.

Syntax:

```bash
ansible <pattern> -m <module> -a "<arguments>"
```

Example:

```bash
ansible all -m ping
```

They are useful for:

- connectivity testing
- quick diagnostics
- inspecting facts
- one-off operational actions
- validating module behavior

RH294 explicitly includes running ad-hoc tasks as a core Ansible objective.

---

## Q15. What are important Ansible ad-hoc commands?

### Connectivity

```bash
ansible all -m ping
```

### Facts

```bash
ansible all -m setup
```

### Command

```bash
ansible all -m command -a "uptime"
```

### Shell

```bash
ansible all -m shell -a "df -h | grep /"
```

### Package with privilege escalation

```bash
ansible web -m dnf -a "name=httpd state=present" -b
```

### Service

```bash
ansible web -m service -a "name=httpd state=started" -b
```

### Copy

```bash
ansible all -m copy \
  -a 'content="Managed by Ansible\n" dest=/etc/motd' \
  -b
```

RH294 uses a similar `copy` ad-hoc example with `--become`.

### Inventory

```bash
ansible-inventory --graph
ansible-inventory --list
ansible-inventory --host web01
```

### Module documentation

```bash
ansible-doc ansible.builtin.copy
ansible-doc ansible.builtin.template
ansible-doc ansible.builtin.command
```

### Verbose

```bash
ansible all -m ping -vvv
```

---

## Q16. What is an Ansible inventory?

Inventory defines the managed hosts and organizes them into groups.

Example:

```ini
[web]
web01
web02

[db]
db01
db02
```

Inventory can be:

```text
Static
  ↓
Files / YAML / INI

Dynamic
  ↓
External source/plugin
```

RH294 explicitly covers inventory as the source for managed hosts and groups.

---

## Q17. What are Ansible host patterns?

A host pattern determines which inventory hosts a play or ad-hoc command targets.

Examples:

```yaml
hosts: web
```

```bash
ansible web --list-hosts
```

Single host:

```yaml
hosts: web01
```

Multiple groups:

```yaml
hosts: web:db
```

Intersection:

```yaml
hosts: lab,&datacenter1
```

Exclusion:

```yaml
hosts: datacenter,!test02
```

RH294 specifically covers group intersection and `!`-based exclusion patterns.

---

## Q18. Why are host patterns important?

Host patterns allow targeting decisions to be expressed in inventory instead of scattering host-selection logic across tasks.

For example:

```yaml
- name: Update production web servers
  hosts: prod_web
```

is usually clearer than targeting every host and adding multiple conditions.

RH294 recommends carefully designed host patterns and inventory groups rather than complex task conditions for host selection.

---

## Q19. What are `all` and `ungrouped` in inventory?

`all` represents all hosts in the inventory.

`ungrouped` represents hosts that are not members of another user-defined group.

Example:

```ini
web01

[web]
web02
```

Conceptually:

```text
all
├── web01
└── web02

ungrouped
└── web01
```

---

## Q20. What is dynamic inventory?

Dynamic inventory generates the inventory from an external system.

Examples:

```text
AWS EC2
Cloud APIs
CMDB
Directory service
```

Flow:

```text
External source
      ↓
Inventory plugin
      ↓
Hosts / groups / variables
      ↓
Ansible
```

RH294 explains that dynamic inventory can be generated from external sources such as directory services or cloud-management systems.

---

## Q21. How does AWS dynamic inventory work?

Conceptually:

```text
Ansible
   ↓
AWS inventory plugin
   ↓
AWS API
   ↓
EC2 instances
   ↓
Filters / tags
   ↓
Ansible hosts and groups
```

For example, EC2 instances can be grouped by tags:

```text
Environment=dev
Application=otms
Role=web
```

A dynamic inventory system discovers current infrastructure. It does **not** itself create EC2 instances.

---

## Q22. How do you troubleshoot incorrect dynamic inventory results?

Check:

```text
AWS account
   ↓
Credentials
   ↓
Region
   ↓
Plugin configuration
   ↓
Filters
   ↓
Tags
   ↓
Instance state
   ↓
Permissions
   ↓
Cache
```

Useful commands:

```bash
ansible-inventory --list
ansible-inventory --graph
```

Compare the output against the actual AWS resource state.

---

## Q23. What is `ansible.cfg`?

`ansible.cfg` is Ansible's configuration file.

Typical settings include:

```ini
[defaults]
inventory = ./inventory
remote_user = devops
forks = 10

[privilege_escalation]
become = false
become_method = sudo
become_user = root
```

RH294 explicitly teaches configuring inventory, remote user, and privilege escalation in `ansible.cfg`.

---

## Q24. Where does Ansible look for `ansible.cfg`?

Ansible searches for its configuration file according to a precedence/order of locations and uses the first relevant configuration it finds.

RH294 emphasizes that Ansible looks for its configuration file in several locations and that the first configuration file found is used.

To determine what configuration is active:

```bash
ansible --version
```

and:

```bash
ansible-config dump -v --only-changed
```

RH294 demonstrates that `ansible-config dump -v --only-changed` identifies the configuration file being used.

---

## Q25. What is the difference between `group_vars` and `host_vars`?

Recommended project structure:

```text
project/
├── inventory
├── group_vars/
│   └── web.yml
├── host_vars/
│   └── web01.yml
└── site.yml
```

`group_vars` applies to a group:

```yaml
# group_vars/web.yml
app_port: 8080
```

`host_vars` applies to a specific host:

```yaml
# host_vars/web01.yml
app_port: 8081
```

RH294 explicitly recommends `group_vars` and `host_vars` directories instead of putting inventory variables directly in the inventory file.

---

## Q26. Explain Ansible variable precedence.

Ansible has many variable sources.

A simplified interview graph is:

```text
             LOWER PRECEDENCE
                    |
                    v
             Role defaults
                    |
                    v
          Inventory group vars
                    |
                    v
           Inventory host vars
                    |
                    v
               Play vars
                    |
                    v
               Task vars
                    |
                    v
               set_fact
                    |
                    v
              Extra vars
                    |
                    v
             HIGHER PRECEDENCE
```

This is a simplified interview model, not the complete official hierarchy.

### Example

```text
role default   → 8080
group_vars     → 8081
host_vars      → 8082
play vars      → 8083
extra vars     → 8084
```

Run:

```bash
ansible-playbook site.yml -e "app_port=8084"
```

The effective value can become `8084`.

RH294 shows that extra variables can override variables defined in the playbook/inventory.

---

## Q27. What are extra variables (`-e`)?

Extra variables are variables supplied at execution time.

Example:

```bash
ansible-playbook site.yml -e "environment=prod"
```

They are useful for one-off overrides.

Example:

```yaml
- name: Deploy application
  hosts: app
  tasks:
    - ansible.builtin.debug:
        msg: "Deploying to {{ environment }}"
```

RH294 explicitly describes command-line variables as extra variables that can override existing values.

---

## Q28. What are Ansible facts?

Facts are information automatically gathered from managed hosts.

Examples:

```text
OS/distribution
kernel
CPU
memory
network interfaces
IP addresses
mounts
Python interpreter
```

Example:

```yaml
- name: Show OS
  ansible.builtin.debug:
    var: ansible_facts.distribution
```

RH294 describes facts as variables automatically discovered from managed hosts.

---

## Q29. What is the `setup` module?

`setup` gathers facts from managed hosts.

Ad-hoc:

```bash
ansible web -m setup
```

Playbook:

```yaml
- hosts: web
  gather_facts: true
```

Facts are then available under:

```yaml
ansible_facts
```

RH294 demonstrates `ansible webserver -m setup` to inspect facts.

---

## Q30. What is the difference between `gather_facts` and `set_fact`?

| `gather_facts` | `set_fact` |
|---|---|
| Discovers target information | Creates variables during execution |
| Usually uses `setup` | Uses `set_fact` |
| Hardware/OS/network/etc. | Derived/application-specific values |

Example:

```yaml
- hosts: all
  gather_facts: true
```

Versus:

```yaml
- name: Build deployment identifier
  ansible.builtin.set_fact:
    deployment_id: "{{ app_name }}-{{ environment }}"
```

---

## Q31. What are magic variables?

Magic variables are supplied by Ansible to expose execution/inventory context.

Important examples:

| Variable | Meaning |
|---|---|
| `hostvars` | Variables for hosts |
| `groups` | Inventory groups |
| `group_names` | Groups for current host |
| `inventory_hostname` | Current inventory name |
| `ansible_play_hosts` | Hosts in current play |
| `ansible_facts` | Gathered facts |
| `playbook_dir` | Playbook directory |

RH294 specifically explains `hostvars`, `group_names`, `groups`, and `inventory_hostname`.

---

## Q32. What does `register` do?

`register` captures a task result into a variable.

Example:

```yaml
- name: Check application
  ansible.builtin.command: /usr/local/bin/healthcheck
  register: result
```

Then:

```yaml
- ansible.builtin.debug:
    var: result
```

Typical result information can include:

```text
rc
stdout
stderr
changed
failed
```

RH294 explicitly teaches `register` as a way to capture command output for use by later tasks.

---

## Q33. What is Jinja2 in Ansible?

Jinja2 is Ansible's templating/expression language.

It supports:

- variable interpolation
- expressions
- conditionals
- loops
- filters
- template rendering

Example:

```jinja2
server={{ inventory_hostname }}
port={{ app_port }}
environment={{ environment }}
```

---

## Q34. What Jinja2 filters are commonly used?

Examples:

```jinja2
{{ name | lower }}
{{ name | upper }}
{{ value | default(8080) }}
{{ items | sort }}
{{ items | unique }}
{{ value | int }}
{{ data | to_json }}
{{ data | to_nice_json }}
```

Filters transform or normalize values before they are used.

---

## Q35. How do you use arrays/dictionaries as Ansible variables?

Instead of multiple related variables:

```yaml
user1_name: Bob
user1_home: /users/bob

user2_name: Anne
user2_home: /users/anne
```

use a structured object:

```yaml
users:
  bob:
    name: Bob
    home: /users/bob
  anne:
    name: Anne
    home: /users/anne
```

Then:

```jinja2
{{ users.bob.name }}
{{ users.anne.home }}
```

RH294 explicitly introduces structured arrays/dictionaries as a way to organize related configuration data.

---

## Q36. What is Ansible Vault?

Ansible Vault protects sensitive Ansible data by encrypting files containing secrets.

Typical use cases:

```text
Passwords
API keys
Password hashes
Private keys
Sensitive variables
```

RH294 explicitly teaches Vault for protecting sensitive variables and structured data files used by Ansible.

Example:

```bash
ansible-vault create secrets.yml
```

---

## Q37. What can Ansible Vault encrypt?

Vault can be used for structured data files used by Ansible, such as:

```text
vars files
inventory variables
role variables
other variable files
```

A common structure is:

```text
project/
├── group_vars/
│   └── web.yml
├── vars/
│   └── secrets.yml
└── site.yml
```

RH294 explicitly notes that inventory variables, included variable files, variables passed to playbooks, and role variables can be protected with Vault.

---

## Q38. What are the important `ansible-vault` commands?

### Create

```bash
ansible-vault create secrets.yml
```

### Edit

```bash
ansible-vault edit secrets.yml
```

### Encrypt existing file

```bash
ansible-vault encrypt secrets.yml
```

### Decrypt

```bash
ansible-vault decrypt secrets.yml
```

### View

```bash
ansible-vault view secrets.yml
```

### Change password

```bash
ansible-vault rekey secrets.yml
```

RH294 explicitly demonstrates create, edit, decrypt and rekey operations.

---

## Q39. How do you run a playbook that uses a Vault-encrypted file?

Example:

```bash
ansible-playbook site.yml --vault-id @prompt
```

Or with a password file:

```bash
ansible-playbook site.yml \
  --vault-password-file vault-pass
```

Modern Ansible commonly uses `--vault-id`.

RH294 demonstrates interactive Vault password entry and password-file usage.

---

## Q40. What is a Vault ID?

A Vault ID associates an encrypted file with a named vault-password source.

Conceptually:

```text
dev vault     → dev password
prod vault    → prod password
```

This is useful when a playbook uses files encrypted with different Vault passwords.

RH294 explicitly discusses assigning Vault IDs when multiple Vault passwords are used.

---

## Q41. Does Ansible Vault encrypt the entire project?

No.

Vault is primarily used to encrypt sensitive structured data, such as variable files.

A normal architecture is:

```text
Playbooks ─────────────── plain text
Roles ─────────────────── plain text
Secrets file ─────────── encrypted with Vault
```

This allows automation logic to remain reviewable while secrets remain protected.

---

## Q42. What is the difference between `when` and `failed_when`?

`when` controls whether a task executes.

```yaml
- name: Run only on Red Hat
  ansible.builtin.command: cat /etc/redhat-release
  when: ansible_facts.os_family == "RedHat"
```

`failed_when` controls whether the result is treated as failure.

```yaml
- name: Check health
  ansible.builtin.command: /usr/local/bin/check.sh
  register: result
  failed_when: "'CRITICAL' in result.stdout"
```

Think:

```text
when
 ↓
Should the task run?

failed_when
 ↓
Should this result count as failure?
```

---

## Q43. What is the difference between `block`, `rescue`, `always`, and `ignore_errors`?

| Mechanism | Purpose |
|---|---|
| `block` | Group related tasks |
| `rescue` | Recover from block failure |
| `always` | Cleanup/finalization |
| `ignore_errors` | Continue after a task failure |

Example:

```yaml
- block:
    - name: Deploy
      ...

    - name: Validate
      ...
  rescue:
    - name: Roll back
      ...
  always:
    - name: Cleanup
      ...
```

Use `ignore_errors` only when the failure is genuinely safe to ignore.

---

## Q44. What is the difference between `assert` and `fail`?

### `assert`

Validates an assumption:

```yaml
- name: Validate environment
  ansible.builtin.assert:
    that:
      - environment in ['dev', 'staging', 'prod']
```

### `fail`

Explicitly fails:

```yaml
- name: Abort deployment
  ansible.builtin.fail:
    msg: "Deployment is locked"
```

```text
assert → validate
fail   → explicitly stop
```

---

## Q45. What are loops in Ansible?

Loops execute the same task for multiple values.

Example:

```yaml
- name: Create users
  ansible.builtin.user:
    name: "{{ item }}"
    state: present
  loop:
    - alice
    - bob
    - charlie
```

The loop variable is:

```text
item
```

RH294 has a dedicated section for writing loops and conditional tasks.

---

## Q46. How do you loop over a list of dictionaries?

Variables:

```yaml
users:
  - username: mukesh
    group: devops
  - username: rahul
    group: developers
```

Task:

```yaml
- name: Create users
  ansible.builtin.user:
    name: "{{ item.username }}"
    groups: "{{ item.group }}"
    state: present
  loop: "{{ users }}"
```

This pattern is common in reusable playbooks and roles.

---

## Q47. How do loops interact with `when`?

Example:

```yaml
- name: Install packages
  ansible.builtin.dnf:
    name: "{{ item }}"
    state: present
  loop:
    - httpd
    - vim
    - git
  when: ansible_facts.os_family == "RedHat"
```

The condition controls task execution while the loop controls the repeated values.

---

## Q48. What is the difference between `include_tasks` and `import_tasks`?

| `include_tasks` | `import_tasks` |
|---|---|
| Dynamic | Static |
| Runtime processing | Parse-time/static processing |
| Useful with runtime conditions | Useful for known task structure |
| Include is conditional as a unit | Conditions can propagate to imported tasks |

Example:

```yaml
- name: Load OS-specific tasks
  ansible.builtin.include_tasks: "{{ ansible_facts.os_family }}.yml"
```

Static import:

```yaml
- ansible.builtin.import_tasks: firewall.yml
```

RH294 explains that `import_tasks` inserts tasks when the playbook is parsed, whereas `include_tasks` processes the content when execution reaches it.

---

## Q49. What is `import_playbook`?

`import_playbook` imports a complete playbook containing plays.

Example:

```yaml
- name: Configure web
  import_playbook: web.yml

- name: Configure database
  import_playbook: db.yml
```

It is used at the top level of a playbook, not inside a play.

RH294 explicitly describes `import_playbook` and notes that imported playbooks execute in order.

---

## Q50. What does `delegate_to` do?

`delegate_to` changes where a task is executed.

Example:

```yaml
- name: Register backend
  ansible.builtin.command: /usr/local/bin/register-backend
  delegate_to: localhost
```

Possible use cases:

```text
Load balancer operations
Central API calls
Database coordination
Controller-side validation
```

Conceptually:

```text
Current target host
       |
       | delegate_to
       v
Different execution host
```

Delegation changes execution location but does not magically change every variable/fact context.

---

## Q51. What are handlers and how do they work?

Handlers are tasks triggered with `notify`.

Example:

```yaml
- name: Deploy configuration
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: Reload nginx

handlers:
  - name: Reload nginx
    ansible.builtin.service:
      name: nginx
      state: reloaded
```

Flow:

```text
Task changes configuration
          ↓
       notify
          ↓
 Handler is scheduled
          ↓
Handler phase executes
```

RH294 explicitly teaches handlers as tasks that run when another task changes the managed host.

---

## Q52. What is the difference between `flush_handlers` and `force_handlers`?

### Flush

```yaml
- ansible.builtin.meta: flush_handlers
```

Runs notified handlers immediately at that point.

### Force

```yaml
force_handlers: true
```

Changes behavior so notified handlers can still run despite later task failures where possible.

Think:

```text
flush_handlers
    → timing

force_handlers
    → failure behavior
```

---

## Q53. What happens if a task fails after a handler has been notified?

Example:

```text
Task 1
  ↓
Config changed
  ↓
notify handler

Task 2
  ↓
FAIL

Handler phase
```

By default, the failure can prevent the notified handler from running on that host.

Use:

```yaml
force_handlers: true
```

where the handler must run despite later failure.

Or:

```yaml
meta: flush_handlers
```

when you need the handler to execute before a subsequent task.

---

## Q54. What is a role?

A role is a standardized structure for reusable automation.

Example:

```text
roles/
└── nginx/
    ├── defaults/
    ├── files/
    ├── handlers/
    ├── meta/
    ├── tasks/
    ├── templates/
    ├── tests/
    └── vars/
```

Roles package tasks, variables, handlers, templates, files and metadata into reusable units.

RH294 explicitly describes roles as a standardized directory structure for reusable tasks, variables, files, templates and other resources.

---

## Q55. Explain the important directories in a role.

| Directory | Purpose |
|---|---|
| `tasks/` | Main task logic |
| `handlers/` | Handlers |
| `templates/` | Jinja2 templates |
| `files/` | Static files |
| `defaults/` | Default variables |
| `vars/` | Role variables |
| `meta/` | Dependencies/metadata |
| `tests/` | Tests |

Typical entry points:

```text
tasks/main.yml
handlers/main.yml
defaults/main.yml
vars/main.yml
meta/main.yml
```

---

## Q56. What is the difference between role `defaults` and `vars`?

`defaults/main.yml` contains values designed to be easily overridden.

Example:

```yaml
app_port: 8080
```

`vars/main.yml` is normally used for values that should have stronger role-variable precedence.

Interview answer:

> Use defaults for user-tunable defaults and vars for stronger role-specific values.

RH294 specifically identifies `defaults/main.yml` as the place for initial parameter values to a role.

---

## Q57. How do role dependencies work?

Role dependencies can be declared in:

```text
meta/main.yml
```

Example concept:

```yaml
dependencies:
  - role: infra.apache
```

This allows one role to depend on another.

RH294 explicitly identifies `meta/main.yml` as the location for role dependencies.

---

## Q58. What is `roles/requirements.yml`?

It describes external role/collection dependencies that should be installed for a project.

Example:

```yaml
- name: infra.apache
  source: git@github.com:company/infra.apache.git
  scm: git
  version: v1.4
```

Install:

```bash
ansible-galaxy install -r roles/requirements.yml
```

RH294 demonstrates using `roles/requirements.yml` to install a role from Git at a specific version.

---

## Q59. What is Ansible Galaxy?

Ansible Galaxy is an ecosystem for finding, sharing and installing roles and collections.

Commands:

```bash
ansible-galaxy role list
ansible-galaxy collection list
ansible-galaxy role install <role>
ansible-galaxy collection install <collection>
```

RH294 explicitly covers Galaxy for retrieving and installing reusable roles.

---

## Q60. What are RHEL System Roles?

RHEL System Roles are supported/reusable Ansible roles for configuring common RHEL subsystems.

RH294 includes examples such as:

```text
rhel-system-roles.kdump
rhel-system-roles.network
rhel-system-roles.selinux
rhel-system-roles.timesync
```

They can reduce the need to write repetitive OS-specific configuration logic yourself.

---

## Q61. How do you manage files using Ansible?

Common file-related modules include:

```text
copy
template
file
fetch
stat
lineinfile
blockinfile
replace
```

Example:

```yaml
- name: Create application directory
  ansible.builtin.file:
    path: /opt/myapp
    state: directory
    owner: appuser
    group: appuser
    mode: '0755'
```

RH294 states that Ansible's file modules cover creating, copying, editing and modifying file attributes.

---

## Q62. What is the difference between `copy`, `template`, `fetch`, and `stat`?

| Module | Purpose |
|---|---|
| `copy` | Copy static content from controller to target |
| `template` | Render Jinja2 template and deploy |
| `fetch` | Copy file from target to controller |
| `stat` | Inspect file metadata/state |

Example:

```yaml
- name: Deploy config
  ansible.builtin.template:
    src: app.conf.j2
    dest: /etc/myapp/app.conf
```

Stat example:

```yaml
- name: Inspect file
  ansible.builtin.stat:
    path: /etc/myapp/app.conf
  register: config
```

Then:

```yaml
when: config.stat.exists
```

---

## Q63. How do package modules work?

Example:

```yaml
- name: Install Apache
  ansible.builtin.dnf:
    name: httpd
    state: present
```

Common desired states:

```text
present
absent
latest
```

The package module compares the requested state with the current state.

RH294 explicitly demonstrates `yum` desired states such as `present`, `absent`, and `latest`.

---

## Q64. How do you manage services with Ansible?

Example:

```yaml
- name: Enable and start Apache
  ansible.builtin.systemd:
    name: httpd
    state: started
    enabled: true
```

Desired state:

```text
service running
+
service enabled at boot
```

A handler is commonly used when a configuration change requires reload/restart.

---

## Q65. How do you manage users and SSH access with Ansible?

Common modules:

```text
user
group
authorized_key
lineinfile
template
```

Example:

```yaml
- name: Create deployment user
  ansible.builtin.user:
    name: deploy
    groups: wheel
    append: true
    state: present
```

SSH key:

```yaml
- name: Add SSH key
  ansible.posix.authorized_key:
    user: deploy
    key: "{{ lookup('file', 'deploy.pub') }}"
```

RH294 explicitly covers user creation, groups, SSH authorized keys, sudoers and SSH configuration.

---

## Q66. What is check mode?

Check mode attempts to show what would change without applying changes for modules that support check mode.

```bash
ansible-playbook site.yml --check
```

Use it before production changes.

Important limitation:

> Not every module can completely predict changes in check mode, especially arbitrary command/script operations.

RH294 explicitly describes `--check` as showing changes to be performed for modules that support check mode.

---

## Q67. What is diff mode?

Diff mode shows differences for modules that support it.

```bash
ansible-playbook site.yml --diff
```

Very useful with:

```text
template
copy
lineinfile
blockinfile
```

Combined:

```bash
ansible-playbook site.yml --check --diff
```

---

## Q68. How do you troubleshoot an Ansible playbook?

A practical sequence is:

```text
Syntax
  ↓
Inventory
  ↓
Connectivity
  ↓
Facts
  ↓
Variables
  ↓
Privilege
  ↓
Module
  ↓
Target system
```

Commands:

```bash
ansible-playbook site.yml --syntax-check
ansible-playbook site.yml -vvv
ansible-inventory --graph
ansible host01 -m ping -vvv
ansible host01 -m setup
```

The `debug` module is particularly useful for inspecting variables and facts.

RH294 recommends `--syntax-check`, verbosity, `debug`, check mode and ad-hoc validation as troubleshooting tools.

---

## Q69. What do `-v`, `-vv`, `-vvv`, and `-vvvv` provide?

A simplified view:

| Option | Typical use |
|---|---|
| `-v` | More result information |
| `-vv` | Input/output details |
| `-vvv` | Connection troubleshooting |
| `-vvvv` | Very detailed execution/debug information |

Example:

```bash
ansible-playbook site.yml -vvv
```

RH294 specifically describes increasing verbosity through these levels and notes that `-vvv` includes connection information.

---

## Q70. How can Ansible log output to a file?

Ansible can write execution logs when configured with:

```ini
[defaults]
log_path = /var/log/ansible/ansible.log
```

or via the environment variable:

```bash
export ANSIBLE_LOG_PATH=/var/log/ansible/ansible.log
```

RH294 states that logging is not enabled by default and describes `log_path` and `ANSIBLE_LOG_PATH`.

---

## Q71. What is the `linear` strategy?

`linear` is the synchronized/default-style strategy where hosts progress through tasks in a coordinated fashion.

Conceptually:

```text
Task 1
 ↓
eligible hosts
 ↓
Task 2
 ↓
eligible hosts
 ↓
Task 3
```

It is useful when ordering and coordination across hosts matter.

---

## Q72. What is the `free` strategy?

`free` allows hosts to progress through tasks independently.

```text
Host A: T1 → T2 → T3 → T4
Host B: T1 ------→ T2 → T3
Host C: T1 → T2 → T3 → T4
```

A fast host does not need to wait for a slower host to finish an earlier task.

Use it only when hosts can safely progress independently.

---

## Q73. What is `host_pinned`?

Where supported by the installed Ansible version/execution environment, `host_pinned` provides host-oriented scheduling behavior.

For interviews, understand the distinction:

```text
linear
  → coordinated progression

free
  → independent host progression

host_pinned
  → host-oriented scheduling
```

Always verify exact semantics against the Ansible version used in production.

---

## Q74. What are `forks`?

`forks` controls controller-side worker concurrency.

Example:

```ini
[defaults]
forks = 20
```

Think:

```text
Target hosts
    ↓
Available controller workers
    ↓
Concurrent execution
```

Increasing forks can improve throughput only when the real bottleneck is available worker capacity.

RH294 describes `forks` as controlling the maximum number of parallel connections to managed hosts.

---

## Q75. What is `serial`?

`serial` controls the number of hosts processed in each batch of a play.

Example:

```yaml
- hosts: production
  serial: 10
```

For 100 hosts:

```text
Batch 1 → 10
Batch 2 → 10
...
Batch 10 → 10
```

RH294 demonstrates `serial: 3` and shows the play running fully on three hosts before moving to the remaining host.

---

## Q76. What is the difference between `forks`, `serial`, and `strategy`?

| Setting | Controls |
|---|---|
| `forks` | Worker concurrency |
| `serial` | Host batch size |
| `strategy` | How hosts progress through tasks |

Example:

```yaml
- hosts: production
  serial: 10
  strategy: linear
```

with:

```ini
forks = 20
```

does **not** mean 20 production hosts are rolled out at once.

`serial: 10` restricts the active rollout batch to ten hosts; `forks` provides available worker capacity inside that execution.

---

## Q77. How would you safely deploy to 100 production servers?

Use controlled batching:

```yaml
- hosts: production_app
  serial: 10
  tasks:
    - name: Deploy
      ...

    - name: Validate health
      ...
```

Flow:

```text
100 hosts
   ↓
10-host batch
   ↓
Precheck
   ↓
Deploy
   ↓
Health check
   ↓
Success?
 /     \
No      Yes
|        |
Stop     Next batch
```

This limits blast radius.

RH294 specifically uses `serial` for rolling updates so the entire web-server fleet is not taken out of service at once.

---

## Q78. How does pipelining improve Ansible performance?

Pipelining reduces the number of remote transfer/command operations associated with some module execution paths.

Conceptually:

```text
Without pipelining
Controller
 ↓
multiple SSH operations
 ↓
remote temp/module handling

With pipelining
Controller
 ↓
fewer SSH operations
 ↓
module execution
```

It can reduce SSH overhead, but it must be evaluated with privilege-escalation configuration and environment constraints.

---

## Q79. How would you optimize a slow Ansible deployment?

Check:

```text
Controller CPU/memory
        ↓
Connection latency
        ↓
Fact gathering
        ↓
Loops/redundant tasks
        ↓
Forks
        ↓
Serial
        ↓
Strategy
        ↓
Pipelining
        ↓
AWS API latency/throttling
```

First identify the bottleneck.

Blindly increasing `forks` can simply move the bottleneck to:

```text
controller
network
managed hosts
AWS APIs
```

---

## Q80. Is Ansible push-based or pull-based?

Standard Ansible execution is controller-driven/push-oriented:

```text
Controller
    |
    | connect / execute
    v
Managed node
```

The target normally does not run an Ansible agent that periodically asks for work in the classic SSH model.

RH294 describes the control node connecting to managed hosts through SSH or WinRM.

---

## Q81. Which other configuration-management tools commonly use a pull model?

### Puppet

```text
Puppet Server
      ↓
Puppet Agent
      ↓
Node convergence
```

The agent can periodically retrieve/apply configuration.

### Chef

```text
Chef Server
     ↓
Chef Client
     ↓
Node convergence
```

The client retrieves configuration and converges the node.

### General pull model

```text
Node-side agent
      ↓
Periodic check
      ↓
Central service
      ↓
Desired configuration
      ↓
Convergence
```

The primary architectural difference is **who initiates the configuration run**.

---

## Q82. Why might an organization choose push vs pull?

| Push | Pull |
|---|---|
| Central operator/controller initiates run | Node initiates convergence |
| Good for orchestration/deployments | Good for periodic convergence |
| Simple target-side agent model | Requires agent lifecycle |
| Target must generally be reachable at run time | Node can act according to its schedule |
| Strong central control | More node-side autonomy |

Neither is universally superior.

---

## Q83. How does Ansible communicate with AWS services?

A useful stack is:

```text
Playbook
   ↓
Ansible AWS collection
   ↓
boto3
   ↓
botocore
   ↓
AWS API
```

For example:

```yaml
- name: Gather EC2 information
  amazon.aws.ec2_instance_info:
    region: us-east-1
```

The collection provides Ansible-facing modules/plugins; the Python AWS SDK stack handles communication with AWS.

---

## Q84. Why are boto3 and botocore important for Ansible AWS automation?

### boto3

Higher-level AWS SDK for Python.

Example:

```python
import boto3

ec2 = boto3.client("ec2")
response = ec2.describe_instances()
```

### botocore

Lower-level AWS client/service machinery used by boto3.

It handles functionality such as:

```text
AWS service models
Request construction
Signing/authentication
Endpoints
Serialization
Low-level client behavior
```

Conceptually:

```text
Ansible AWS module
       ↓
Ansible AWS collection
       ↓
boto3
       ↓
botocore
       ↓
AWS API
```

This allows Ansible's AWS integration to use the AWS SDK rather than reimplementing the AWS API protocol stack.

---

## Q85. What is the difference between the AWS collection, boto3, and botocore?

| Component | Responsibility |
|---|---|
| Ansible AWS collection | Ansible modules/plugins |
| boto3 | High-level Python AWS SDK |
| botocore | Lower-level AWS SDK/client machinery |
| AWS API | Actual cloud service |

Think:

```text
Automation interface → Ansible
AWS-specific Ansible implementation → collection
Python AWS SDK → boto3
Low-level AWS machinery → botocore
Cloud operation → AWS API
```

---

## Q86. How does Ansible manage EC2 through AWS Systems Manager instead of SSH?

Traditional:

```text
Ansible Controller
       |
       | SSH :22
       v
EC2
```

SSM-based:

```text
Ansible Controller
       |
       | AWS APIs / SSM
       v
AWS Systems Manager
       |
       v
SSM Agent
       |
       v
Private EC2
```

The management path is different from SSH.

This is particularly useful for private EC2 instances where inbound SSH is intentionally unavailable.

---

## Q87. What is required for Ansible + EC2 + SSM?

Think in layers.

### IAM

The EC2 instance needs an appropriate Systems Manager IAM role.

### Agent

SSM Agent must be installed/running and successfully registered.

### Network

A private instance needs access to required AWS Systems Manager endpoints, typically through:

```text
NAT
or
VPC interface endpoints
```

### Controller

The Ansible controller needs required AWS permissions.

### Ansible

The inventory/playbook must use the correct SSM connection mechanism.

---

## Q88. How would you troubleshoot private EC2 with no SSH?

Use this sequence:

```text
EC2 running?
   ↓
SSM Agent running?
   ↓
Correct IAM instance role?
   ↓
Network access to SSM?
   ↓
Managed node online?
   ↓
Controller AWS permissions?
   ↓
Ansible SSM connection correct?
```

Do not weaken the security model by opening SSH unless that is intentionally part of the architecture.

---

## Q89. How do you separate infrastructure provisioning from Ansible configuration?

A common production boundary is:

```text
Terraform / CloudFormation
       ↓
VPC / subnet / SG / EC2 / IAM / ALB
       ↓
Dynamic inventory
       ↓
Ansible
       ↓
Packages / files / services / application config
```

The point is separation of:

```text
Infrastructure lifecycle
        vs
Configuration management
```

This is a design choice, not a hard requirement.

---

## Q90. Why can SSH connectivity succeed while an Ansible task still fails?

Connectivity proves only the transport/authentication layer.

Example:

```text
SSH works
  ↓
Ansible task fails
```

Possible causes:

```text
Python interpreter
Privilege escalation
Module dependency
Permissions
SELinux/AppArmor
Variables
Application state
OS/package differences
```

A ping test does not prove task compatibility.

---

## Q91. A playbook works on one server but fails on another. How do you troubleshoot it?

Layer the investigation:

```text
Connectivity
   ↓
Inventory
   ↓
Facts
   ↓
Variables
   ↓
Privileges
   ↓
OS/package versions
   ↓
Filesystem
   ↓
Security policy
   ↓
Application state
```

Useful commands:

```bash
ansible host01 -m ping -vvv
ansible-inventory --host host01
ansible host01 -m setup
```

And use:

```yaml
- ansible.builtin.debug:
    var: variable_name
```

to inspect effective values.

---

## Q92. A task always reports `changed`. How do you troubleshoot it?

Check:

```text
Is the module capable of desired-state comparison?
        ↓
Is it command/shell/script based?
        ↓
Does input change each run?
        ↓
Are timestamps/random values involved?
        ↓
Would creates/removes help?
        ↓
Would changed_when better represent the result?
```

Do not simply suppress the result:

```yaml
changed_when: false
```

unless that is truly the correct semantic result.

---

## Q93. How would you troubleshoot a "module not found" error?

Check:

```bash
ansible --version
ansible-doc <module>
ansible-galaxy collection list
ansible-config dump
```

Then verify:

1. Module name.
2. FQCN.
3. Collection installation.
4. Active Python/Ansible environment.
5. Multiple Ansible installations.
6. `collections_paths`/configured search locations.

Example:

```yaml
ansible.builtin.copy:
```

rather than an ambiguous short name where appropriate.

---

## Q94. What happens when an Ansible task fails?

By default, Ansible stops executing later tasks on the failed host for that play.

The exact behavior across other hosts depends on strategy and play-level failure controls.

Execution states include:

```text
ok
changed
skipped
failed
unreachable
```

RH294's play recap and execution examples show these result categories and per-host counts.

---

## Q95. What is the difference between `unreachable` and `failed`?

### `unreachable`

Ansible could not establish/maintain the required connection to the target.

Typical causes:

```text
SSH failure
Network failure
Wrong host
DNS problem
SSM connectivity problem
```

### `failed`

The target was reachable, but task execution failed.

Typical causes:

```text
command returned non-zero
permissions
module error
invalid task arguments
application failure
```

This distinction is extremely useful during incident troubleshooting.

---

## Q96. How do you design an idempotent production deployment?

Use desired-state modules:

```yaml
- name: Ensure package exists
  ansible.builtin.dnf:
    name: nginx
    state: present

- name: Ensure service is enabled and running
  ansible.builtin.systemd:
    name: nginx
    state: started
    enabled: true

- name: Deploy configuration
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: Reload nginx
```

Expected behavior:

```text
Run 1 → expected changes
Run 2 → no unnecessary changes
```

---

## Q97. How would you handle configuration drift?

Ansible detects drift when automation checks actual host state against the desired state.

```text
Desired state
      +
Current state
      ↓
Difference?
   /      \
 no        yes
 |          |
 ok      remediate
```

Ansible is not a continuous drift-monitoring daemon by itself.

Periodic drift remediation can be scheduled through CI/CD, automation controllers, timers, or other operational workflows.

---

## Q98. What are the important differences between Ansible and Puppet/Chef?

| Area | Ansible | Puppet | Chef |
|---|---|---|---|
| Common model | Controller-driven/push | Agent-oriented/pull | Client/agent-oriented |
| Agent on classic target | Not required for SSH | Yes | Yes |
| Configuration convergence | Controller initiates | Agent initiates | Client initiates |
| Strong orchestration use | Yes | Yes | Yes |
| Target-side agent lifecycle | Lower in SSH model | Required | Required |

The strongest interview answer focuses on architecture rather than declaring one tool universally better.

---

## Q99. What is the difference between `become` and `become_user`?

```yaml
become: true
become_user: appuser
```

- `become` enables privilege escalation.
- `become_user` specifies the effective user to become.

Example:

```yaml
- name: Run as application user
  ansible.builtin.command: whoami
  become: true
  become_user: appuser
```

---

## Q100. How does privilege escalation work in Ansible?

Conceptually:

```text
Connect as deployment user
        ↓
become mechanism
        ↓
Execute as elevated/effective user
```

Example:

```yaml
- name: Install package
  ansible.builtin.dnf:
    name: nginx
    state: present
  become: true
```

Failures can arise from:

```text
sudo policy
become method
target user permissions
password requirements
SELinux/security policy
```

---

## Q101. What are callback plugins?

Callback plugins allow Ansible to react to execution events and customize output/reporting/integration.

Possible uses:

```text
Logging
Reporting
Custom terminal output
Metrics
External integrations
```

Conceptually:

```text
Ansible execution
      |
      +----> normal output
      |
      +----> callback
                |
                +----> reporting/integration
```

---

## Q102. How do you troubleshoot an Ansible controller/environment problem?

Check:

```bash
ansible --version
ansible-config dump -v --only-changed
ansible-galaxy collection list
ansible-galaxy role list
```

Then verify:

```text
Python environment
Ansible executable
Collections path
Inventory path
ansible.cfg
Credentials
```

This is especially important when multiple Python virtual environments or Ansible installations exist.

---

## Q103. What is the practical relationship between Ansible, inventory, modules, and roles?

Think of the responsibilities separately:

```text
Inventory
    → WHICH systems?

Variables
    → WHAT values?

Playbook
    → WHAT workflow?

Task
    → WHAT step?

Module
    → HOW is that operation performed?

Role
    → HOW is reusable automation organized?

Strategy
    → HOW do hosts progress?

Connection
    → HOW do we reach the target?
```

This is a useful mental model for Senior L2 interviews.

---

## Q104. How would you design Ansible for a large production environment?

A practical architecture can be:

```text
Git
 ↓
Ansible project
 ↓
CI validation
 ↓
ansible-lint / syntax / test
 ↓
Automation Controller / Jenkins
 ↓
Dynamic inventory
 ↓
serial + strategy + forks
 ↓
Managed hosts / AWS SSM
```

Security:

```text
Secrets → Ansible Vault / approved secret system
Access  → IAM / SSH / SSM
Changes → Git review
Rollout → serial batches
Validation → check/diff/health checks
```

The exact tooling depends on the organization, but the principles are repeatability, controlled rollout, observability, and secure secret handling.

---

## Q105. What are the most important Ansible commands to know for a Senior L2 interview?

```bash
# Version
ansible --version

# Configuration
ansible-config dump
ansible-config dump -v --only-changed

# Inventory
ansible-inventory --list
ansible-inventory --graph
ansible-inventory --host <host>

# Connectivity
ansible all -m ping
ansible all -m ping -vvv

# Facts
ansible all -m setup

# Ad-hoc
ansible all -m command -a "uptime"
ansible all -m shell -a "df -h"
ansible all -m copy -a "src=file dest=/tmp/file"

# Playbook
ansible-playbook site.yml

# Validate
ansible-playbook site.yml --syntax-check
ansible-playbook site.yml --check
ansible-playbook site.yml --diff

# Debug
ansible-playbook site.yml -vvv

# Module docs
ansible-doc ansible.builtin.copy
ansible-doc ansible.builtin.template
ansible-doc ansible.builtin.command

# Roles / collections
ansible-galaxy role list
ansible-galaxy collection list
```

---

## Q106. What are the most important Ansible concepts to explain in one interview answer?

A strong Senior L2 explanation connects the concepts:

```text
Inventory
   ↓
Select hosts
   ↓
Variables / facts
   ↓
Playbook
   ↓
Task
   ↓
Module
   ↓
Connection
   ↓
Current-state inspection
   ↓
Desired-state convergence
   ↓
ok / changed / failed
   ↓
Handler
```

For scale:

```text
Strategy
+
serial
+
forks
```

For cloud:

```text
Dynamic inventory
+
AWS collection
+
boto3
+
botocore
+
SSM
```

For secure automation:

```text
Ansible Vault
+
IAM / least privilege
+
Git review
+
controlled rollout
```

For reusable automation:

```text
Roles
+
requirements.yml
+
Galaxy
+
group_vars / host_vars
```
