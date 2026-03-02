# Types of Modules in Ansible

## 🔵 Primary Types of Ansible Modules (How They Are Delivered)
---
### 1️⃣ Ansible Built-in Modules

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

### 2️⃣ Community Modules

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

### 3️⃣ Custom Modules

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

### ✅ Final Answer (No Ambiguity)

**How many types of Ansible modules exist?**
👉 **Three (3): Built-in, Community, and Custom**
