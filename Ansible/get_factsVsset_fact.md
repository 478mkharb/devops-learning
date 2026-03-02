# Difference Between set_fact and gather_facts (get_facts) in Ansible

This document explains the **difference between `set_fact` and `gather_facts` (often called get_facts)** in Ansible.

Understanding this topic is **critical for writing dynamic playbooks, conditionals, templates, and roles**, and it is a **very common interview question**.

---

## 1. What Are Facts in Ansible?

**Facts** are variables that store information about:

* Managed hosts
* System properties
* Runtime values

Facts can be:

* **Automatically collected** by Ansible
* **Manually defined** during play execution

---

## 2. gather_facts (get_facts)

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

## 3. set_fact

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

## 4. Execution Timing (Very Important)

| Type         | When It Runs          |
| ------------ | --------------------- |
| gather_facts | Before any task       |
| set_fact     | During task execution |

---

## 5. Scope and Lifetime

### gather_facts

* Scope: Host-level
* Lifetime: Entire play
* Source: System data

### set_fact

* Scope: Host-level
* Lifetime: Remaining play (or longer with cache)
* Source: User-defined

---

## 6. Comparison Table

| Aspect         | gather_facts (get_facts) | set_fact                |
| -------------- | ------------------------ | ----------------------- |
| Purpose        | Collect system info      | Define custom variables |
| Level          | Play-level               | Task-level              |
| Module used    | setup                    | set_fact                |
| Default        | Enabled                  | Manual                  |
| Data source    | OS / system              | User logic              |
| Dynamic values | No                       | Yes                     |

---

## 7. Using Both Together (Real Example)

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

## 8. Common Mistakes

❌ Expecting `set_fact` to work before `gather_facts`

❌ Confusing get_facts with set_fact

❌ Forgetting to disable `gather_facts` when not needed

---

## 9. Best Practices

* Disable `gather_facts` if not required (performance)
* Use `set_fact` for derived or computed values
* Keep fact names meaningful
* Avoid excessive use of `set_fact`

---

## 10. Interview One-Line Answers

* **gather_facts**: Automatically collects system information
* **set_fact**: Manually defines runtime variables

---

## Final Summary

* `gather_facts` answers **“What does this system look like?”**
* `set_fact` answers **“What value do I want to calculate or store?”**

Both are essential for writing **dynamic, intelligent Ansible automation**.

---

Happy Automating with Ansible 🚀
