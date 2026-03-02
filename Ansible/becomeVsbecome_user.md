# Difference Between become and become_user in Ansible

This document explains **privilege escalation in Ansible** with a clear focus on the difference between **`become`** and **`become_user`**.

This topic is **very important for interviews, real-world automation, and security-sensitive environments**.

---

## 1. What Is Privilege Escalation in Ansible?

By default, Ansible connects to managed nodes using a **normal user** (for example: `ec2-user`, `ubuntu`, or `ansible`).

Some tasks require **higher privileges**, such as:

* Installing packages
* Modifying system configuration
* Managing services
* Accessing protected files

Ansible uses **privilege escalation** to handle this securely.

---

## 2. become

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

## 3. become_user

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

## 4. Using become and become_user Together

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

## 5. Task-Level vs Play-Level Usage

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

## 6. Comparison Table

| Aspect        | become                      | become_user         |
| ------------- | --------------------------- | ------------------- |
| Purpose       | Enable privilege escalation | Specify target user |
| Required      | Yes                         | Only with become    |
| Default value | false                       | root                |
| Scope         | Play / Task                 | Play / Task         |
| Used alone    | Yes                         | No                  |

---

## 7. Common Mistakes

❌ Using `become_user` without `become`

❌ Assuming `become` always means root explicitly

❌ Running app-level commands as root unnecessarily

---

## 8. Best Practices

* Use `become` only when required
* Prefer least-privilege (`become_user` != root)
* Avoid running application logic as root
* Use task-level escalation where possible

---

## 9. Interview One-Line Answers

* **become**: Enables privilege escalation
* **become_user**: Specifies the user to run the task as

---

## Final Summary

* `become` controls **whether** privilege escalation happens
* `become_user` controls **who** the task runs as
* Both together provide **secure and controlled execution**

---

Happy Automating with Ansible 🚀
