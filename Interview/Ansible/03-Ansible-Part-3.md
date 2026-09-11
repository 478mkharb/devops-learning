# Ansible Senior L2 Interview — Part 3

**Focus:** Templating, inventory context and task control

## Q13. What is Jinja2 and why is it important in Ansible?

### Short explanation of the question

This tests whether I understand how Ansible turns variables and facts into dynamic configuration.

### Answer

Jinja2 is the templating language Ansible uses to render dynamic content. The `template` module commonly uses Jinja2 to create configuration files from variables, facts, conditions, loops and filters.

### Detailed explanation

This document explains **Jinja2 templating in Ansible from zero to advanced**, covering **syntax, filters, conditionals, loops, variables, facts, best practices, and interview questions**.

This is **README-ready**, **interview-ready**, and **production-ready**.

---

### 1. What Is Jinja2?

**Jinja2** is a **powerful templating engine** used by Ansible to dynamically generate files such as:

* Configuration files
* Scripts
* Application property files

Ansible uses Jinja2 mainly through the **`template` module**.

---

### 2. Why Jinja2 Is Important in Ansible

Without Jinja2:

* Static configuration files

With Jinja2:

* Dynamic values
* Environment-specific configs
* Conditional logic
* Loop-based rendering

---

### 3. Jinja2 File Extension

Jinja2 template files use the extension:

```
.j2
```

Example:

```
nginx.conf.j2
```

---

### 4. Basic Jinja2 Syntax

### 4.1 Variable Substitution

```jinja2
{{ variable_name }}
```

Example:

```jinja2
server_name {{ nginx_server_name }};
```

---

### 4.2 Using Variables in Playbook

```yaml
- name: Deploy config
  template:
    src: app.conf.j2
    dest: /etc/app/app.conf
```

---

### 5. Comments in Jinja2

```jinja2
{# This is a comment #}
```

---

### 6. Conditionals (if / elif / else)

```jinja2
{% if env == 'prod' %}
debug=false
{% elif env == 'dev' %}
debug=true
{% else %}
debug=false
{% endif %}
```

---

### 7. Loops (for)

### 7.1 Loop Over a List

```jinja2
{% for port in ports %}
listen {{ port }};
{% endfor %}
```

Variables:

```yaml
ports:
  - 80
  - 443
```

---

### 7.2 Loop Over Dictionary

```jinja2
{% for key, value in users.items() %}
{{ key }}={{ value }}
{% endfor %}
```

---

### 8. Filters (Very Important)

### Common Filters

```jinja2
{{ my_var | upper }}
{{ my_var | lower }}
{{ my_list | length }}
{{ my_list | join(',') }}
{{ my_var | default('value') }}
```

---

### 9. Using Ansible Facts in Jinja2

```jinja2
Hostname: {{ ansible_hostname }}
IP Address: {{ ansible_default_ipv4.address }}
OS: {{ ansible_distribution }}
```

Facts come from `gather_facts: true`.

---

### 10. Whitespace Control

```jinja2
{{ variable -}}
{%- if condition %}
```

Prevents unwanted blank lines.

---

### 11. Include Another Template

```jinja2
{% include 'common.conf.j2' %}
```

---

### 12. Macros (Reusable Logic)

```jinja2
{% macro user(name, shell) %}
{{ name }}:{{ shell }}
{% endmacro %}

{{ user('john', '/bin/bash') }}
```

---

### 13. Template vs Copy Module

| Feature         | template | copy |
| --------------- | -------- | ---- |
| Dynamic content | Yes      | No   |
| Uses Jinja2     | Yes      | No   |
| Variables       | Yes      | No   |

---

### 14. Common Mistakes

❌ Forgetting `.j2` extension
❌ Missing `gather_facts`
❌ Incorrect indentation
❌ Overusing logic in templates

---

### 15. Best Practices

* Keep logic minimal in templates
* Use variables, not hardcoding
* Validate rendered configs
* Use handlers with templates

---

### 16. Real-World Example – Nginx Config

```jinja2
server {
    listen {{ nginx_port }};
    server_name {{ nginx_server_name }};

    {% if enable_ssl %}
    ssl on;
    {% endif %}
}
```

---

### 17. Interview Questions and Answers

### Q1. What is Jinja2 in Ansible?

**Answer:** It is a templating engine used to create dynamic configuration files.

---

### Q2. Difference between template and copy module?

**Answer:** `template` supports variables and logic; `copy` is static.

---

### Q3. Can you use loops and conditions in templates?

**Answer:** Yes, using Jinja2 syntax.

---

### Q4. Where are templates stored?

**Answer:** In the `templates/` directory of a role or project.

---

### 18. One-Line Interview Summary

* **Jinja2** enables dynamic file generation in Ansible using variables, loops, and conditions.

---

### Final Summary

Jinja2 is **mandatory knowledge** for:

* Configuration management
* Environment-based deployments
* Ansible interviews

Mastering Jinja2 = mastering Ansible automation.

---

Happy Templating 🚀

### Example

Template:

```jinja2
server {
    listen {{ app_port }};
    server_name {{ inventory_hostname }};
}
```

If `app_port` is `8080` and the inventory hostname is `web01`, the rendered configuration contains those concrete values.

---

## Q14. What are commonly used Jinja2 filters in Ansible?

### Short explanation of the question

This checks whether I can transform values cleanly while rendering templates.

### Answer

Jinja2 filters transform values before rendering. Common examples include `upper`, `lower` and `default`. I use filters for small, readable transformations and avoid turning templates into complex application logic.

### Detailed explanation

This document explains the **most commonly used Jinja2 filters in Ansible** with **clear variables, templates, rendered output, real-world use cases, and interview tips**.

---

### 1. `upper` Filter

### Purpose

Converts a string to **uppercase**.

### Variable

```yaml
env: production
```

### Template

```jinja2
ENV={{ env | upper }}
```

### Output

```
ENV=PRODUCTION
```

### Use Case

Environment variables, constants

---

### 2. `lower` Filter

### Purpose

Converts a string to **lowercase**.

### Variable

```yaml
APP_NAME: MyApplication
```

### Template

```jinja2
app_name={{ APP_NAME | lower }}
```

### Output

```
app_name=myapplication
```

### Use Case

Linux usernames, case-sensitive configs

---

### 3. `default` Filter (VERY IMPORTANT)

### Purpose

Provides a **fallback value** if a variable is undefined or empty.

### Template

```jinja2
log_level={{ log_level | default('INFO') }}
```

### Output (if variable missing)

```
log_level=INFO
```

### Use Case

Production-safe templates

---

### 4. `length` Filter

### Purpose

Returns the **number of items** in a list or characters in a string.

### Variable

```yaml
servers:
  - web1
  - web2
  - web3
```

### Template

```jinja2
Total servers: {{ servers | length }}
```

### Output

```
Total servers: 3
```

### Use Case

Validation and conditional logic

---

### 5. `join` Filter

### Purpose

Joins list elements into a **single string**.

### Variable

```yaml
ports:
  - 80
  - 443
  - 8080
```

### Template

```jinja2
ports={{ ports | join(',') }}
```

### Output

```
ports=80,443,8080
```

### Use Case

Application configs, env variables

---

### 6. `replace` Filter

### Purpose

Replaces part of a string.

### Variable

```yaml
path: "/var/www/html"
```

### Template

```jinja2
{{ path | replace('/var', '/opt') }}
```

### Output

```
/opt/www/html
```

### Use Case

Path and string manipulation

---

### 7. `trim` Filter

### Purpose

Removes leading and trailing whitespace.

### Variable

```yaml
username: "  admin  "
```

### Template

```jinja2
user={{ username | trim }}
```

### Output

```
user=admin
```

### Use Case

Cleaning user input

---

### 8. `bool` Filter

### Purpose

Converts values to **boolean**.

### Variable

```yaml
enable_ssl: "true"
```

### Template

```jinja2
{% if enable_ssl | bool %}
SSL Enabled
{% endif %}
```

### Output

```
SSL Enabled
```

### Use Case

Conditional rendering

---

### 9. Filter Chaining (Interview Favorite)

### Variable

```yaml
app_name: "  MyApp  "
```

### Template

```jinja2
{{ app_name | trim | lower }}
```

### Output

```
myapp
```

### Use Case

Clean, standardized output

---

### 10. Real-World Example – Nginx Template

```jinja2
server_name {{ server_name | lower }};
listen {{ ports | join(' ') }};
```

### Variables

```yaml
server_name: MySite.COM
ports:
  - 80
  - 443
```

### Output

```
server_name mysite.com;
listen 80 443;
```

---

### Interview One-Liners

* `upper` / `lower`: Change case
* `default`: Prevent failures
* `length`: Count items
* `join`: Merge lists
* `replace`: Modify strings
* `trim`: Remove whitespace
* Filters can be **chained**

---

### Final Summary

Jinja2 filters are **essential tools** for transforming data in Ansible templates. Using them correctly makes templates **clean, safe, and production-ready**.

---

Happy Templating 🚀

### Example

```jinja2
ENV={{ env | upper }}
APP_USER={{ app_user | lower }}
PORT={{ app_port | default(8080) }}
```

If `env` is `production`, the first expression renders `PRODUCTION`.

---

## Q15. What are Ansible magic variables?

### Short explanation of the question

This tests whether I understand the execution-context variables Ansible provides automatically.

### Answer

Magic variables expose inventory, host, group and play context. Common examples include `inventory_hostname`, `groups`, `group_names`, `hostvars`, `ansible_play_batch` and `playbook_dir`.

### Detailed explanation

### Definition

Magic variables are special predefined variables automatically created by Ansible. They provide information about the inventory, hosts, playbook execution, and environment, and cannot be set or modified by users.

**Simple definition:**
Magic variables are automatically generated variables that give information about the playbook run, inventory, and hosts.

---

# Common Magic Variables

| Magic Variable           | Description                           |
| ------------------------ | ------------------------------------- |
| hostvars                 | Access variables of other hosts       |
| groups                   | Dictionary of all inventory groups    |
| group_names              | Groups the current host belongs to    |
| inventory_hostname       | Current host name from inventory      |
| inventory_hostname_short | Short hostname                        |
| ansible_play_hosts       | Hosts involved in the play            |
| ansible_play_batch       | Current batch of hosts being executed |
| playbook_dir             | Directory of the running playbook     |
| inventory_dir            | Inventory file location               |

---

# Example 1 – inventory_hostname

```yaml
- name: Show hostname
  debug:
    msg: "This host is {{ inventory_hostname }}"
```

Example output:

```
This host is web1
```

---

# Example 2 – groups

```yaml
- name: Print all webservers
  debug:
    msg: "{{ groups['webservers'] }}"
```

Example output:

```
['web1', 'web2', 'web3']
```

---

# Example 3 – hostvars

Access variables of another host.

```yaml
- name: Show IP of another host
  debug:
    msg: "{{ hostvars['web1']['ansible_host'] }}"
```

This lets one host read variables of another host.

---

# Example 4 – group_names

```yaml
- name: Print groups for this host
  debug:
    var: group_names
```

Example output:

```
["webservers", "production"]
```

---

# Example Inventory

```ini
[webservers]
web1
web2

[db]
db1
```

Magic variables allow playbooks to dynamically understand the inventory structure.

---

# Key Characteristics

| Feature       | Description                 |
| ------------- | --------------------------- |
| Created by    | Ansible automatically       |
| User editable | No                          |
| Used for      | Inventory and runtime info  |
| Accessed in   | Playbooks, templates, tasks |

---

### One-line Interview Answer

Magic variables in Ansible are automatically generated variables that provide information about hosts, inventory, and playbook execution during runtime.

### Example

```yaml
- name: Show current inventory name
  ansible.builtin.debug:
    msg: "Running on {{ inventory_hostname }}"
```

### Interviewer may cross-question

**Interviewer:** "Why would you use `hostvars`?"

**Candidate:** "When a task genuinely needs information belonging to another inventory host, for example a database endpoint or another host's discovered value." 

---

## Q16. What is the difference between `include_tasks` and `import_tasks`?

### Short explanation of the question

This tests whether I understand static versus dynamic task inclusion.

### Answer

`import_tasks` is a static import processed as part of parsing, while `include_tasks` is dynamic and evaluated during execution. The distinction matters when conditions, variables, tags and task structure depend on runtime information.

### Detailed explanation

Both `include_tasks` and `import_tasks` are used to **reuse task files**, but they behave **very differently**. This difference is a **classic interview question**.

---

### Core Difference (One Line)

> **`import_tasks` is static (decided at playbook parse time), while `include_tasks` is dynamic (decided at runtime).**

---

### How `import_tasks` Works (Static)

`import_tasks` is processed **when Ansible parses the playbook**, before execution starts.

### Example

```yaml
- hosts: web
  tasks:
    - import_tasks: install.yml
```

### What happens internally

* Tasks from `install.yml` are **merged into the playbook**
* Ansible knows **all tasks in advance**
* Conditions are applied **per task**, not to the import itself

### Key traits

* 📦 Tasks are loaded **once**
* 🧠 Execution plan is fixed
* 🚫 Cannot use variables to decide whether to import

---

### How `include_tasks` Works (Dynamic)

`include_tasks` is processed **during execution**, at runtime.

### Example

```yaml
- hosts: web
  tasks:
    - include_tasks: install.yml
      when: ansible_os_family == "Debian"
```

### What happens internally

* Ansible reaches this task during execution
* Condition is evaluated
* Tasks are included **only if the condition is true**

### Key traits

* 🔁 Tasks can be included **multiple times**
* 🎯 Conditions apply to the **entire include**
* ✅ Variables and facts fully supported

---

### Side-by-Side Comparison

| Aspect              | import_tasks | include_tasks |
| ------------------- | ------------ | ------------- |
| Evaluation time     | Parse time   | Runtime       |
| Static / Dynamic    | Static       | Dynamic       |
| Conditional include | ❌ No         | ✅ Yes         |
| Loop support        | ❌ No         | ✅ Yes         |
| Uses facts          | ❌ No         | ✅ Yes         |
| Execution plan      | Fixed        | Flexible      |

---

### Real DevOps Example

### Scenario

Install packages differently based on OS.

### Using `include_tasks` (Correct)

```yaml
- include_tasks: debian.yml
  when: ansible_os_family == "Debian"

- include_tasks: redhat.yml
  when: ansible_os_family == "RedHat"
```

### Why not `import_tasks`?

Because OS facts are available **only at runtime**.

---

### Common Mistake

```yaml
- import_tasks: install.yml
  when: ansible_os_family == "Debian"
```

❌ The `when` is applied to **each task inside**, not to the import itself.

---

### When to Use What

### Use `import_tasks` when

* Task structure is fixed
* No runtime conditions
* You want faster parsing

### Use `include_tasks` when

* Tasks depend on facts or variables
* Conditional execution is required
* Tasks need to run in loops

---

### Interview-Perfect Answer

> **“`import_tasks` is static and processed at playbook parse time, while `include_tasks` is dynamic and evaluated at runtime, making `include_tasks` suitable for conditional and fact-based task execution.”**

### Comparison

| `import_tasks` | `include_tasks` |
|---|---|
| Static | Dynamic |
| Parse-time structure | Runtime inclusion |
| Good for fixed task structure | Useful when runtime data determines inclusion |

### Example

```yaml
- name: Select tasks at runtime
  ansible.builtin.include_tasks: "{{ ansible_facts.os_family }}.yml"
```

---

## Q17. How does `delegate_to` work in Ansible?

### Short explanation of the question

This tests whether I understand that a task can execute on a different host from the one currently being iterated.

### Answer

`delegate_to` changes the execution target of a task while the play still iterates over the original hosts. Delegated tasks can still be concurrent, so shared delegated resources may require `run_once` or controlled concurrency.

### Detailed explanation

`delegate_to` is an **Ansible task-level directive** that tells Ansible to **execute a task on a different host than the one currently being targeted**, while still keeping the **context of the original host**.

In simple words:

> 🧠 *"This task belongs to host A, but run it on host B."*

---

### How `delegate_to` Works (Behind the Scenes)

1. Ansible loops over hosts in the play as usual.
2. When it reaches a task with `delegate_to`, it **does not run the task on the inventory host**.
3. Instead, the task is executed on the **delegated host**.
4. **Variables, facts, and host context** still belong to the original host (unless overridden).

✅ Facts are stored against the *original* host, not the delegated one.

---

### Basic Syntax

```yaml
- name: Task executed elsewhere
  some_module:
    ...
  delegate_to: other_host
```

---

### Example 1: Restart a Load Balancer from a Web Server Play

### Scenario

* You are configuring **web servers**
* After updating a web server, you want to **reload NGINX on the load balancer**

### Inventory

```ini
[web]
web1
web2

[lb]
lb1
```

### Playbook

```yaml
- name: Deploy app on web servers
  hosts: web
  tasks:
    - name: Update application files
      copy:
        src: app/
        dest: /var/www/html/

    - name: Reload NGINX on load balancer
      service:
        name: nginx
        state: reloaded
      delegate_to: lb1
```

### What Happens?

| Current Host | Task Runs On |
| ------------ | ------------ |
| web1         | lb1          |
| web2         | lb1          |

⚠️ The reload runs **twice** (once per web host). This is important!

---

### Prevent Multiple Runs (Common Pattern)

To avoid repeated execution, combine with `run_once`:

```yaml
- name: Reload NGINX once
  service:
    name: nginx
    state: reloaded
  delegate_to: lb1
  run_once: true
```

---

### Example 2: Running a Task on the Control Node (localhost)

### Use Case

* Call an API
* Generate a file
* Update DNS
* Send a notification

```yaml
- name: Call external API
  uri:
    url: https://example.com/deploy
    method: POST
  delegate_to: localhost
```

💡 Very common in CI/CD pipelines.

---

### Example 3: Fetch Files via a Bastion Host

```yaml
- name: Copy logs from private server via bastion
  fetch:
    src: /var/log/app.log
    dest: ./logs/
  delegate_to: bastion
```

---

### Common Real-World Use Cases

| Use Case             | Why `delegate_to` is Used            |
| -------------------- | ------------------------------------ |
| Load balancer reload | LB not part of target group          |
| Bastion / jump host  | Private nodes not directly reachable |
| API / webhook calls  | Needs to run from control node       |
| DNS / cloud updates  | Centralized execution                |
| Database leader ops  | Execute only from primary            |

---

### `delegate_to` vs `local_action`

| Feature       | delegate_to | local_action  |
| ------------- | ----------- | ------------- |
| Flexible host | ✅ Yes       | ❌ No          |
| Modern usage  | ✅ Preferred | ❌ Deprecated  |
| Clear syntax  | ✅ Yes       | ⚠️ Less clear |

Equivalent example:

```yaml
# Old
local_action: shell echo "done"

# New
delegate_to: localhost
```

---

### Important Gotchas

* Delegated tasks **still loop over hosts** unless `run_once` is used
* Facts belong to the **original host**
* SSH connection is made to the **delegated host**
* Inventory variables of delegated host are accessible via `hostvars`

---

### One-Line Summary

> `delegate_to` lets you **run a task on another host while keeping the logic tied to the original host**.

If you want, I can also add:

* delegate_to + facts example
* delegate_to vs include_tasks
* delegate_to flow diagram
* interview-style explanation

### Example

```yaml
- name: Remove this server from the load balancer
  ansible.builtin.command:
    cmd: "/usr/local/bin/remove-from-lb {{ inventory_hostname }}"
  delegate_to: lb-controller
```

### Interviewer may cross-question

**Interviewer:** "If 20 web servers delegate to one controller, will the delegated tasks run one by one?"

**Candidate:** "Not necessarily. Delegated tasks can still execute concurrently for the original hosts. If the shared resource cannot handle that, I need coordination such as `run_once`, batching, or another concurrency control." 

---

## Q18. What is the difference between `when` and `failed_when`?

### Short explanation of the question

This tests whether I can distinguish deciding whether a task runs from deciding whether an executed result is a failure.

### Answer

`when` controls whether a task executes. `failed_when` evaluates an executed task's result and can redefine whether Ansible considers that result failed.

### Detailed explanation

Both `when` and `failed_when` are **conditional controls** in Ansible, but they serve **very different purposes**.

> 🧠 **Key idea**:

* `when` decides **whether a task should run**
* `failed_when` decides **whether a task should be marked as failed after it runs**

---

### `when` — Conditional Task Execution

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

### `failed_when` — Conditional Failure

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

### Side-by-Side Example

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

### Execution Order (Important for Interviews)

```
when  →  task execution  →  failed_when
```

---

### Common Mistake

❌ Expecting `failed_when` to skip a task

```yaml
failed_when: some_condition
```

➡️ The task will still run.

---

### Using Both Together (Very Common Pattern)

```yaml
- name: Validate config file
  command: nginx -t
  register: nginx_test
  when: nginx_installed
  failed_when: nginx_test.rc != 0
```

---

### Comparison Table

| Feature            | `when`            | `failed_when`         |
| ------------------ | ----------------- | --------------------- |
| Controls execution | ✅ Yes             | ❌ No                  |
| Controls failure   | ❌ No              | ✅ Yes                 |
| Task runs?         | Only if true      | Always                |
| Evaluated          | Before execution  | After execution       |
| Common use         | OS/env conditions | Custom error handling |

---

### Real-World Use Cases

### `when`

* OS-specific tasks
* Environment-based execution (prod/dev)
* Feature flags

### `failed_when`

* Health checks
* API calls with 200 but error in body
* Commands returning non-standard exit codes

---

### One-Line Summary (Interview Gold)

> **`when` decides if a task should run, `failed_when` decides if a task should fail.**

If you want, I can also add:

* `changed_when` vs `failed_when`
* `ignore_errors` vs `failed_when`
* Complex expressions with `register`
* Interview trick questions with answers

### Example

```yaml
- name: Check application
  ansible.builtin.command: /usr/local/bin/check-app
  register: result
  failed_when: "'CRITICAL' in result.stdout"

- name: Restart only on Debian
  ansible.builtin.service:
    name: nginx
    state: restarted
  when: ansible_facts.os_family == "Debian"
```

---
