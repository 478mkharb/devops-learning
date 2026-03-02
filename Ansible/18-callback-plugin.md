# Ansible Callback Plugins

## What is a Callback Plugin in Ansible?

A **callback plugin** in Ansible is a plugin that hooks into Ansible’s execution engine and **reacts to events** during a playbook run.

In simple words:

> Callback plugins let you **control how Ansible reports, logs, or reacts** to task execution.

They do **not** change *what* Ansible does — they change **how results are displayed, recorded, or processed**.

---

## Where Are Callback Plugins Used?

Callback plugins are used when you want to:

* Customize **output format** (JSON, minimal, rich UI)
* Send execution data to **external systems**

  * Slack
  * Email
  * Log files
  * Databases
  * Monitoring tools
* Collect **metrics** about playbook execution
* Integrate Ansible with **CI/CD pipelines**
* Improve **debugging and observability**

---

## Common Built‑in Callback Plugins

| Plugin Name     | Purpose                       |
| --------------- | ----------------------------- |
| `default`       | Standard Ansible output       |
| `minimal`       | Very short output             |
| `yaml`          | Clean, readable YAML output   |
| `json`          | Machine‑readable output       |
| `profile_tasks` | Shows task execution time     |
| `timer`         | Total playbook execution time |
| `log_plays`     | Logs playbook runs            |

---

## How Callback Plugins Work (Execution Flow)

1. Playbook starts
2. Task executes on a host
3. Ansible generates an event

   * Task started
   * Task succeeded
   * Task failed
   * Host unreachable
4. Callback plugin **listens to the event**
5. Plugin performs an action

   * Print output
   * Log data
   * Send notification

---

## Example 1: Using a Built‑in Callback Plugin

### Enable `yaml` Output

Edit `ansible.cfg`:

```ini
[defaults]
stdout_callback = yaml
```

### Run Playbook

```bash
ansible-playbook site.yml
```

### Result

* Output becomes **structured and readable**
* Ideal for demos and debugging

---

## Example 2: Enable Task Execution Time (`profile_tasks`)

```ini
[defaults]
stdout_callback = profile_tasks
```

### Output Shows

```text
TASK [Install nginx] *********************
0.84s

TASK [Start nginx] ***********************
0.12s
```

👉 Very useful for **performance tuning**

---

## Example 3: Custom Callback Plugin (Simple)

### File Structure

```text
project/
├── ansible.cfg
└── callback_plugins/
    └── notify.py
```

### `ansible.cfg`

```ini
[defaults]
callback_plugins = ./callback_plugins
```

### Custom Callback Plugin (`notify.py`)

```python
from ansible.plugins.callback import CallbackBase

class CallbackModule(CallbackBase):
    CALLBACK_VERSION = 2.0
    CALLBACK_TYPE = 'notification'
    CALLBACK_NAME = 'notify'

    def v2_runner_on_ok(self, result):
        host = result._host.get_name()
        task = result.task_name
        print(f"✅ Task '{task}' succeeded on {host}")
```

### What This Does

* Listens for **successful tasks**
* Prints a custom success message

---

## Real‑World Use Cases 🚀

* 📣 Send Slack message when playbook fails
* 📧 Email notifications on critical failures
* 📝 Store execution logs for auditing & compliance
* 🔄 Improve CI/CD pipeline visibility
* 🤫 Reduce noisy output in production

---

## 🔥 Prometheus Monitoring Use Case (Very Important)

Callback plugins are extremely useful when you want **Ansible + Prometheus observability**.

### 🧠 Problem

You want to:

* Track **Ansible playbook success / failure**
* Measure **task execution time**
* Expose metrics to **Prometheus**

Prometheus cannot directly scrape Ansible.
👉 **Callback plugins act as the bridge**.

---

### 🏗️ Architecture Flow

```text
Ansible Playbook
      ↓
Callback Plugin 🎧
      ↓
Expose Metrics (file / HTTP / pushgateway)
      ↓
Prometheus Scrapes Metrics 📊
      ↓
Grafana Dashboard 📈
```

---

### 📊 Example Metrics You Can Expose

| Metric                            | Description               |
| --------------------------------- | ------------------------- |
| `ansible_playbook_runs_total`     | Total playbook executions |
| `ansible_playbook_failures_total` | Failed playbook runs      |
| `ansible_task_duration_seconds`   | Task execution time       |
| `ansible_host_unreachable_total`  | Unreachable hosts         |

---

### 🧩 Example: Prometheus‑Friendly Callback Plugin

```python
from ansible.plugins.callback import CallbackBase
from prometheus_client import Counter, Histogram, start_http_server

# Metrics
PLAYBOOK_RUNS = Counter('ansible_playbook_runs_total', 'Total playbook runs')
TASK_TIME = Histogram('ansible_task_duration_seconds', 'Task execution time')

class CallbackModule(CallbackBase):
    CALLBACK_VERSION = 2.0
    CALLBACK_TYPE = 'notification'
    CALLBACK_NAME = 'prometheus'

    def __init__(self):
        super().__init__()
        start_http_server(8000)  # Prometheus scrapes here
        PLAYBOOK_RUNS.inc()

    def v2_runner_on_ok(self, result):
        TASK_TIME.observe(result._result.get('delta', 0))
```

---

### ⚙️ Prometheus Scrape Config

```yaml
scrape_configs:
  - job_name: 'ansible'
    static_configs:
      - targets: ['ansible-host:8000']
```

---

### 🎯 What You Get

* 📈 Grafana dashboard showing Ansible reliability
* ⏱️ Slow tasks identified visually
* 🚨 Alerts when failure count increases
* 🧠 Better production observability

---

### 💡 Interview Gold Line ✨

> *Callback plugins enable exporting Ansible execution metrics to Prometheus, making infrastructure automation observable and measurable.*

---

## Interview‑Ready One‑Liner

> **Ansible callback plugins allow you to customize how playbook execution events are handled, displayed, or sent to external systems without changing task logic.**

---

If you want, I can also:

* Draw a **visual flow diagram**
* Create a **Slack notification callback**
* Show **callback vs action vs lookup plugins**
* Convert this into **presentation slides**
