# How `delegate_to` Works in Ansible

`delegate_to` is an **Ansible task-level directive** that tells Ansible to **execute a task on a different host than the one currently being targeted**, while still keeping the **context of the original host**.

In simple words:

> 🧠 *"This task belongs to host A, but run it on host B."*

---

## 🔧 How `delegate_to` Works (Behind the Scenes)

1. Ansible loops over hosts in the play as usual.
2. When it reaches a task with `delegate_to`, it **does not run the task on the inventory host**.
3. Instead, the task is executed on the **delegated host**.
4. **Variables, facts, and host context** still belong to the original host (unless overridden).

✅ Facts are stored against the *original* host, not the delegated one.

---

## 📌 Basic Syntax

```yaml
- name: Task executed elsewhere
  some_module:
    ...
  delegate_to: other_host
```

---

## 🧪 Example 1: Restart a Load Balancer from a Web Server Play

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

## 🛑 Prevent Multiple Runs (Common Pattern)

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

## 🌍 Example 2: Running a Task on the Control Node (localhost)

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

## 📦 Example 3: Fetch Files via a Bastion Host

```yaml
- name: Copy logs from private server via bastion
  fetch:
    src: /var/log/app.log
    dest: ./logs/
  delegate_to: bastion
```

---

## ⭐ Common Real-World Use Cases

| Use Case             | Why `delegate_to` is Used            |
| -------------------- | ------------------------------------ |
| Load balancer reload | LB not part of target group          |
| Bastion / jump host  | Private nodes not directly reachable |
| API / webhook calls  | Needs to run from control node       |
| DNS / cloud updates  | Centralized execution                |
| Database leader ops  | Execute only from primary            |

---

## 🔄 `delegate_to` vs `local_action`

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

## ⚠️ Important Gotchas

* Delegated tasks **still loop over hosts** unless `run_once` is used
* Facts belong to the **original host**
* SSH connection is made to the **delegated host**
* Inventory variables of delegated host are accessible via `hostvars`

---

## 🧠 One-Line Summary

> `delegate_to` lets you **run a task on another host while keeping the logic tied to the original host**.

If you want, I can also add:

* delegate_to + facts example
* delegate_to vs include_tasks
* delegate_to flow diagram
* interview-style explanation
