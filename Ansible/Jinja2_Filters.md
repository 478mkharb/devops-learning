# Jinja2 Commonly Used Filters (3.1) – Detailed Examples

This document explains the **most commonly used Jinja2 filters in Ansible** with **clear variables, templates, rendered output, real-world use cases, and interview tips**.

---

## 1. `upper` Filter

### Purpose

Converts a string to **uppercase**.

### Variable

```yaml
env: production
```

### Template

```jinja2
ENV={{ env | upper }}
```

### Output

```
ENV=PRODUCTION
```

### Use Case

Environment variables, constants

---

## 2. `lower` Filter

### Purpose

Converts a string to **lowercase**.

### Variable

```yaml
APP_NAME: MyApplication
```

### Template

```jinja2
app_name={{ APP_NAME | lower }}
```

### Output

```
app_name=myapplication
```

### Use Case

Linux usernames, case-sensitive configs

---

## 3. `default` Filter (VERY IMPORTANT)

### Purpose

Provides a **fallback value** if a variable is undefined or empty.

### Template

```jinja2
log_level={{ log_level | default('INFO') }}
```

### Output (if variable missing)

```
log_level=INFO
```

### Use Case

Production-safe templates

---

## 4. `length` Filter

### Purpose

Returns the **number of items** in a list or characters in a string.

### Variable

```yaml
servers:
  - web1
  - web2
  - web3
```

### Template

```jinja2
Total servers: {{ servers | length }}
```

### Output

```
Total servers: 3
```

### Use Case

Validation and conditional logic

---

## 5. `join` Filter

### Purpose

Joins list elements into a **single string**.

### Variable

```yaml
ports:
  - 80
  - 443
  - 8080
```

### Template

```jinja2
ports={{ ports | join(',') }}
```

### Output

```
ports=80,443,8080
```

### Use Case

Application configs, env variables

---

## 6. `replace` Filter

### Purpose

Replaces part of a string.

### Variable

```yaml
path: "/var/www/html"
```

### Template

```jinja2
{{ path | replace('/var', '/opt') }}
```

### Output

```
/opt/www/html
```

### Use Case

Path and string manipulation

---

## 7. `trim` Filter

### Purpose

Removes leading and trailing whitespace.

### Variable

```yaml
username: "  admin  "
```

### Template

```jinja2
user={{ username | trim }}
```

### Output

```
user=admin
```

### Use Case

Cleaning user input

---

## 8. `bool` Filter

### Purpose

Converts values to **boolean**.

### Variable

```yaml
enable_ssl: "true"
```

### Template

```jinja2
{% if enable_ssl | bool %}
SSL Enabled
{% endif %}
```

### Output

```
SSL Enabled
```

### Use Case

Conditional rendering

---

## 9. Filter Chaining (Interview Favorite)

### Variable

```yaml
app_name: "  MyApp  "
```

### Template

```jinja2
{{ app_name | trim | lower }}
```

### Output

```
myapp
```

### Use Case

Clean, standardized output

---

## 10. Real-World Example – Nginx Template

```jinja2
server_name {{ server_name | lower }};
listen {{ ports | join(' ') }};
```

### Variables

```yaml
server_name: MySite.COM
ports:
  - 80
  - 443
```

### Output

```
server_name mysite.com;
listen 80 443;
```

---

## Interview One-Liners

* `upper` / `lower`: Change case
* `default`: Prevent failures
* `length`: Count items
* `join`: Merge lists
* `replace`: Modify strings
* `trim`: Remove whitespace
* Filters can be **chained**

---

## Final Summary

Jinja2 filters are **essential tools** for transforming data in Ansible templates. Using them correctly makes templates **clean, safe, and production-ready**.

---

Happy Templating 🚀
