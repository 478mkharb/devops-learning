# What Are Ansible Facts and How Are They Collected?

**Ansible facts** are **system information automatically gathered from managed hosts** and stored as variables. They describe the **state and characteristics of a host**.

> 🧠 Think of facts as: *"Ansible asking the server: Who are you and what do you look like?"*

---

## 📦 What Are Ansible Facts?

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

## 🔍 How Are Facts Collected?

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

## ⚙️ Under the Hood (High Level)

1. Ansible connects to the host
2. Runs the `setup` module
3. Collects system data
4. Stores data as variables (facts)
5. Makes them available to all tasks

---

## 🧪 Example: Using Facts in a Playbook

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

## 🚫 Disabling Fact Gathering

If you don’t need facts, you can disable them to improve speed:

```yaml
- name: Fast play
  hosts: all
  gather_facts: false
```

---

## 🎯 Manually Gathering Facts

You can explicitly run the `setup` module:

```yaml
- name: Gather facts manually
  setup:
```

---

## 🧩 Subsets and Filters (Performance Optimization)

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

## 🏷️ Custom Facts

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

## 📊 Facts vs Variables

| Feature                 | Facts  | Variables         |
| ----------------------- | ------ | ----------------- |
| Collected automatically | ✅ Yes  | ❌ No              |
| Host-specific           | ✅ Yes  | ✅ Yes             |
| Dynamic                 | ✅ Yes  | ⚠️ Usually static |
| Source                  | System | User / inventory  |

---

## 🧠 One-Line Summary (Interview Gold)

> **Ansible facts are automatically gathered system details collected by the `setup` module and exposed as variables.**

---

## 🎯 Interview Tips

Common follow-ups:

* Difference between facts and inventory variables
* How to speed up fact gathering
* Custom facts vs dynamic facts

If you want, I can also add:

* `ansible_facts` namespace explanation
* Facts with `delegate_to`
* Fact caching (Redis / JSON)
* A facts collection flow diagram
