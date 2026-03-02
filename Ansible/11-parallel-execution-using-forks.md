# How Does Ansible Handle Parallel Execution?

Ansible is **parallel by default**. It executes tasks on **multiple hosts at the same time** using a **fork-based worker model**.

> 🧠 **Key idea**: One task, many hosts — executed concurrently.

---

## 🔧 Default Parallel Execution Model

* Ansible runs a task on **multiple hosts simultaneously**
* Parallelism is controlled by **forks**
* Default forks value: **5**

```bash
ansible-config dump | grep DEFAULT_FORKS
```

---

## 🧠 What Are Forks?

* A **fork** is a worker process
* Each fork handles **one host at a time**
* More forks = more parallel hosts

### Example

| Forks | Hosts | Parallel Execution |
| ----- | ----- | ------------------ |
| 5     | 20    | 5 at a time        |
| 10    | 20    | 10 at a time       |

---

## 🧪 Simple Parallel Execution Example

### Inventory

```ini
[web]
web1
web2
web3
web4
```

### Playbook

```yaml
- name: Install nginx in parallel
  hosts: web
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
```

### What Happens?

```
Task runs concurrently on:
web1, web2, web3, web4
```

---

## ⏱️ Controlling Parallelism with `forks`

### Command Line

```bash
ansible-playbook site.yml -f 10
```

### ansible.cfg

```ini
[defaults]
forks = 10
```

---

## 🚦 Sequential Execution with `serial`

Use `serial` to **limit how many hosts are updated at once**.

```yaml
- name: Rolling update
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
```

---

## 🧨 Parallelism vs Safety (Very Important)

| Scenario             | Recommendation    |
| -------------------- | ----------------- |
| Package installs     | High parallelism  |
| Database changes     | Low / serial      |
| Load balancer reload | serial + run_once |

---

## 🟡 Interaction with Failures

* Failure affects **only that host** by default
* Other hosts continue in parallel

```yaml
any_errors_fatal: true
```

➡️ Stops play on all hosts if one fails

---

## 🧩 Parallelism with `delegate_to`

```yaml
- name: Reload LB
  service:
    name: nginx
    state: reloaded
  delegate_to: lb1
  run_once: true
```

Even though play is parallel, this task runs **once**.

---

## ⚙️ Parallel Execution Internals (High Level)

```
Task
 ├─ Fork 1 → host1
 ├─ Fork 2 → host2
 ├─ Fork 3 → host3
```

---

## 📊 Summary Table

| Feature         | Behavior         |
| --------------- | ---------------- |
| Default         | Parallel         |
| Controlled by   | forks            |
| Rolling updates | serial           |
| Failure scope   | Per-host         |
| Global stop     | any_errors_fatal |

---

## 🧠 One-Line Summary (Interview Gold)

> **Ansible executes tasks in parallel across hosts using forks, and you control it with `forks` and `serial`.**

---

## 🎯 Interview Tips

Common follow-ups:

* Difference between `forks` and `serial`
* How to do zero-downtime deploys
* How failures affect parallel runs

If you want, I can also add:

* Parallelism with `strategy: free`
* Performance tuning best practices
* Execution flow diagram
