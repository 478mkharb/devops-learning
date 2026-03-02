# What Different Types of Strategies Are Used in Ansible?

Ansible **strategies** control **how tasks are executed across hosts** — specifically *ordering, parallelism, and failure behavior*.

> 🧠 **Simple idea**: Strategies decide *who runs what, and when*.

---

## 🔧 Default Strategy: `linear`

### How it Works

* All hosts execute **task 1 together**
* Then all hosts execute **task 2 together**
* Strict lock-step execution

```text
Task 1 → all hosts
Task 2 → all hosts
Task 3 → all hosts
```

### Example

```yaml
- name: Linear strategy example
  hosts: web
  strategy: linear
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present

    - name: Start nginx
      service:
        name: nginx
        state: started
```

### When to Use

* Predictable execution
* Safer deployments
* Default for most playbooks

---

## ⚡ Strategy: `free`

### How it Works

* Each host runs tasks **as fast as it can**
* No lock-step waiting
* Faster overall execution

```text
host1: task1 → task2 → task3
host2: task1 → task2
host3: task1 → task2 → task3
```

### Example

```yaml
- name: Free strategy example
  hosts: web
  strategy: free
  tasks:
    - name: Install packages
      apt:
        name: nginx
        state: present
```

### When to Use

* Large fleets
* Independent hosts
* Speed matters more than order

---

## 🌀 Strategy: `host_pinned`

### How it Works

* Each host sticks to **one worker**
* Tasks for that host are serialized
* Reduces race conditions

### Example

```yaml
- name: Host pinned example
  hosts: web
  strategy: host_pinned
  tasks:
    - name: Update app
      command: /opt/update.sh
```

### When to Use

* Stateful operations
* Avoiding concurrent access issues

---

## 🧱 Strategy: `debug`

### How it Works

* Executes **one task at a time**
* Waits for user input
* Used for troubleshooting

### Example

```yaml
- name: Debug strategy example
  hosts: web
  strategy: debug
  tasks:
    - name: Restart service
      service:
        name: nginx
        state: restarted
```

### When to Use

* Debugging playbooks
* Understanding execution flow

---

## 🔌 Strategy Plugins (Advanced)

Ansible strategies are implemented as **plugins**.

Common plugins:

* `linear` (default)
* `free`
* `host_pinned`
* `debug`

You can also **write custom strategies** in Python.

---

## ⚖️ Strategy vs `serial`

| Feature                | Strategy | serial |
| ---------------------- | -------- | ------ |
| Controls task ordering | ✅ Yes    | ❌ No   |
| Controls batch size    | ❌ No     | ✅ Yes  |
| Used together          | ✅ Yes    | ✅ Yes  |

Example:

```yaml
strategy: linear
serial: 2
```

---

## 📊 Summary Table

| Strategy    | Execution Style | Use Case        |
| ----------- | --------------- | --------------- |
| linear      | Lock-step       | Safe, default   |
| free        | Fast, async     | Large fleets    |
| host_pinned | Per-host worker | Stateful ops    |
| debug       | Step-by-step    | Troubleshooting |

---

## 🧠 One-Line Summary (Interview Gold)

> **Ansible strategies control how tasks are scheduled across hosts, balancing speed, safety, and order.**

---

## 🎯 Interview Tip

If asked:

> *"How do you speed up Ansible execution?"*

Answer:

* Increase forks
* Use `strategy: free`
* Use `serial` carefully

If you want, I can also add:

* Strategy + failure behavior
* Strategy comparison diagram
* Real production patterns
