# Jinja2 Templates in Ansible – Complete, In-Depth Guide

This document explains **Jinja2 templating in Ansible from zero to advanced**, covering **syntax, filters, conditionals, loops, variables, facts, best practices, and interview questions**.

This is **README-ready**, **interview-ready**, and **production-ready**.

---

## 1. What Is Jinja2?

**Jinja2** is a **powerful templating engine** used by Ansible to dynamically generate files such as:

* Configuration files
* Scripts
* Application property files

Ansible uses Jinja2 mainly through the **`template` module**.

---

## 2. Why Jinja2 Is Important in Ansible

Without Jinja2:

* Static configuration files

With Jinja2:

* Dynamic values
* Environment-specific configs
* Conditional logic
* Loop-based rendering

---

## 3. Jinja2 File Extension

Jinja2 template files use the extension:

```
.j2
```

Example:

```
nginx.conf.j2
```

---

## 4. Basic Jinja2 Syntax

### 4.1 Variable Substitution

```jinja2
{{ variable_name }}
```

Example:

```jinja2
server_name {{ nginx_server_name }};
```

---

### 4.2 Using Variables in Playbook

```yaml
- name: Deploy config
  template:
    src: app.conf.j2
    dest: /etc/app/app.conf
```

---

## 5. Comments in Jinja2

```jinja2
{# This is a comment #}
```

---

## 6. Conditionals (if / elif / else)

```jinja2
{% if env == 'prod' %}
debug=false
{% elif env == 'dev' %}
debug=true
{% else %}
debug=false
{% endif %}
```

---

## 7. Loops (for)

### 7.1 Loop Over a List

```jinja2
{% for port in ports %}
listen {{ port }};
{% endfor %}
```

Variables:

```yaml
ports:
  - 80
  - 443
```

---

### 7.2 Loop Over Dictionary

```jinja2
{% for key, value in users.items() %}
{{ key }}={{ value }}
{% endfor %}
```

---

## 8. Filters (Very Important)

### Common Filters

```jinja2
{{ my_var | upper }}
{{ my_var | lower }}
{{ my_list | length }}
{{ my_list | join(',') }}
{{ my_var | default('value') }}
```

---

## 9. Using Ansible Facts in Jinja2

```jinja2
Hostname: {{ ansible_hostname }}
IP Address: {{ ansible_default_ipv4.address }}
OS: {{ ansible_distribution }}
```

Facts come from `gather_facts: true`.

---

## 10. Whitespace Control

```jinja2
{{ variable -}}
{%- if condition %}
```

Prevents unwanted blank lines.

---

## 11. Include Another Template

```jinja2
{% include 'common.conf.j2' %}
```

---

## 12. Macros (Reusable Logic)

```jinja2
{% macro user(name, shell) %}
{{ name }}:{{ shell }}
{% endmacro %}

{{ user('john', '/bin/bash') }}
```

---

## 13. Template vs Copy Module

| Feature         | template | copy |
| --------------- | -------- | ---- |
| Dynamic content | Yes      | No   |
| Uses Jinja2     | Yes      | No   |
| Variables       | Yes      | No   |

---

## 14. Common Mistakes

❌ Forgetting `.j2` extension
❌ Missing `gather_facts`
❌ Incorrect indentation
❌ Overusing logic in templates

---

## 15. Best Practices

* Keep logic minimal in templates
* Use variables, not hardcoding
* Validate rendered configs
* Use handlers with templates

---

## 16. Real-World Example – Nginx Config

```jinja2
server {
    listen {{ nginx_port }};
    server_name {{ nginx_server_name }};

    {% if enable_ssl %}
    ssl on;
    {% endif %}
}
```

---

## 17. Interview Questions and Answers

### Q1. What is Jinja2 in Ansible?

**Answer:** It is a templating engine used to create dynamic configuration files.

---

### Q2. Difference between template and copy module?

**Answer:** `template` supports variables and logic; `copy` is static.

---

### Q3. Can you use loops and conditions in templates?

**Answer:** Yes, using Jinja2 syntax.

---

### Q4. Where are templates stored?

**Answer:** In the `templates/` directory of a role or project.

---

## 18. One-Line Interview Summary

* **Jinja2** enables dynamic file generation in Ansible using variables, loops, and conditions.

---

## Final Summary

Jinja2 is **mandatory knowledge** for:

* Configuration management
* Environment-based deployments
* Ansible interviews

Mastering Jinja2 = mastering Ansible automation.

---

Happy Templating 🚀
