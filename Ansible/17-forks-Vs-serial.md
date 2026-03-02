# Forks vs Serial in Ansible (With Examples & Use Cases)

Both **`forks`** and **`serial`** control **how many hosts Ansible works on at a time**, but they operate at **different levels** and solve **different problems**.

> 🧠 **Core idea**:

* `forks` controls **parallelism capacity** (how many hosts *can* run at once)
* `serial` controls **deployment batches** (how many hosts *should* run at once)

---

## 🔧 What Is `forks`?

### Definition

`forks` defines the **maximum number of parallel worker processes** Ansible can use.

* Global setting
* Upper limit on concurrency
* Default value: **5**

### Where It Is Set

```bash
ansible-playbook site.yml -f 20
```

or in `ansible.cfg`:

```ini
[defaults]
forks = 20
```

---

## 🧪 Example: Forks in Action

### Inventory

```ini
[web]
web1
web2
web3
web4
web5
web6
```

### Playbook

```yaml
- name: Install nginx
  hosts: web
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
```

### Execution with `forks = 3`

```
Batch 1: web1, web2, web3
Batch 2: web4, web5, web6
```

➡️ Forks decide **capacity**, not intention.

---

## 🚦 What Is `serial`?

### Definition

`serial` defines **how many hosts Ansible processes at a time per play**.

* Play-level setting
* Enforces **rolling / staged execution**
* Overrides aggressive parallelism

---

## 🧪 Example: Serial in Action

```yaml
- name: Rolling deployment
  hosts: web
  serial: 2
  tasks:
    - name: Restart application
      service:
        name: myapp
        state: restarted
```

### Execution Flow

```
Batch 1: web1, web2
Batch 2: web3, web4
Batch 3: web5, web6
```

➡️ Serial decides **policy**, not capacity.

---

## ⚖️ Forks vs Serial (Side-by-Side)

| Aspect          | forks                | serial             |
| --------------- | -------------------- | ------------------ |
| Scope           | Global               | Per play           |
| Purpose         | Max parallel workers | Batch size control |
| Controls speed  | ✅ Yes                | ⚠️ Indirect        |
| Controls safety | ❌ No                 | ✅ Yes              |
| Default         | 5                    | All hosts          |

---

## 🧠 How They Work Together (Important)

> **Actual parallel hosts = min(forks, serial)**

### Example

```yaml
serial: 2
```

```ini
forks = 10
```

➡️ Only **2 hosts run at a time**, even though forks allow 10.

---

## 🎯 Real-World Use Cases

### Use `forks` When

* Running tasks on many independent hosts
* Speed matters (patching, info gathering)
* Hosts are stateless

Examples:

* Log collection
* Package installation
* Metrics gathering

---

### Use `serial` When

* High-risk operations
* Services must stay available
* Order and safety matter

Examples:

* Rolling deployments
* Database migrations
* Load-balanced services

---

## 🧨 Production Pattern Example

```yaml
- name: Zero-downtime deployment
  hosts: web
  serial: 1
  tasks:
    - name: Remove from LB
      command: /opt/lb_remove.sh

    - name: Deploy app
      command: /opt/deploy.sh

    - name: Add back to LB
      command: /opt/lb_add.sh
```

---

## 🧠 One-Line Summary (Interview Gold)

> **Forks define how much Ansible can do in parallel; serial defines how much it should do at a time.**

---

## 🎯 Interview Trap Question

**Q:** *If forks=20 and serial=2, how many hosts run in parallel?*

**A:** 2 hosts.

---

If you want, I can also add:

* forks vs strategy
* serial with failure handling
* real outage case study
