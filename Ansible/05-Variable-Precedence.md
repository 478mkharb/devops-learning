# 🧱 Ansible Variable Precedence Order

Variable precedence defines **which variable wins** when the same variable name is defined in multiple places.

Rule to remember:

> 🔼 **Higher precedence overrides lower precedence**

---

## 🔺 Variable Precedence Pyramid (Lowest → Highest)

```text
                🔺 Extra vars (-e)
              🔺 Task vars / set_fact
            🔺 Block vars
          🔺 Role vars (roles/vars)
        🔺 Play vars
      🔺 Host vars (host_vars)
    🔺 Group vars (group_vars)
  🔺 Inventory vars
🔺 Role defaults (roles/defaults)
```

📌 Bottom = weakest priority
📌 Top = strongest priority

---

## 🧠 Precedence Explained with Examples

We will use **one variable name** everywhere:

```yaml
app_port
```

---

### 🟢 Role Defaults (Lowest Priority)

📁 `roles/web/defaults/main.yml`

```yaml
app_port: 80
```

Used for **safe defaults**.

---

### 🟢 Inventory Variables

📁 `inventory.ini`

```ini
[web]
web1 app_port=81
```

Overrides role defaults.

---

### 🟢 Group Variables

📁 `group_vars/web.yml`

```yaml
app_port: 82
```

Applies to all hosts in the group.

---

### 🟢 Host Variables

📁 `host_vars/web1.yml`

```yaml
app_port: 83
```

Overrides group-level values.

---

### 🟢 Play Variables

```yaml
- hosts: web
  vars:
    app_port: 84
```

Overrides inventory, group, and host vars.

---

### 🟢 Role Vars (High Priority)

📁 `roles/web/vars/main.yml`

```yaml
app_port: 85
```

⚠️ Very strong — avoid unless necessary.

---

### 🟢 Block Variables

```yaml
block:
  - debug:
      msg: "{{ app_port }}"
  vars:
    app_port: 86
```

Overrides play and role vars.

---

### 🟢 Task Variables / set_fact

```yaml
- set_fact:
    app_port: 87
```

Overrides almost everything below.

---

### 🔴 Extra Vars (Highest Priority)

```bash
ansible-playbook site.yml -e app_port=88
```

Nothing can override this.

---

## 🏆 Final Winner Example

If `app_port` is defined **everywhere**, the value used will be:

```text
app_port = 88
```

(from extra vars)

---

## 🎯 Interview-Perfect Answer

> **“Ansible variable precedence follows a layered model where role defaults have the lowest priority and extra vars have the highest. When the same variable is defined multiple times, the one with the highest precedence wins.”**

---

## ⚠️ Best Practices

* ✅ Put defaults in `roles/defaults`
* ⚠️ Use `roles/vars` sparingly
* 🚫 Avoid relying heavily on extra vars
* 🧠 Keep variable definitions predictable

---

🧩 **Mnemonic to remember:**

> *Defaults sink, extra vars rule.*
