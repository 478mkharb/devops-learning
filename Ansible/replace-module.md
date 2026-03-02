# Ansible `replace` Module – Complete Guide

This document explains **what the `replace` module is**, **why it is used**, **how it works**, **how it differs from `lineinfile` and `blockinfile`**, and includes **real-world examples and interview questions**.

The `replace` module is frequently tested in **interviews** and heavily used in **legacy configuration management**.

---

## 1. What Is the replace Module?

The **`replace` module** is used to **search and replace text patterns in a file using regular expressions (regex)**.

Unlike `lineinfile`, it:

* Can replace **parts of a line**
* Can replace **multiple matching lines**
* Works best for **pattern-based modifications**

---

## 2. When Should You Use replace?

Use `replace` when:

* You need to change **only a portion of a line**
* You are dealing with **legacy or unstructured config files**
* Multiple lines need the **same regex-based update**

Avoid using it for:

* Single key-value settings → use `lineinfile`
* Structured blocks → use `blockinfile`

---

## 3. Basic Syntax

```yaml
- replace:
    path: <file_path>
    regexp: <regex_pattern>
    replace: <replacement_text>
```

---

## 4. Basic Example – Replace Text in a File

```yaml
- name: Replace port number
  replace:
    path: /etc/myapp/app.conf
    regexp: 'port=8080'
    replace: 'port=9090'
```

This replaces **only the matched pattern**, not the entire line.

---

## 5. Regex-Based Replacement (Very Important)

### Example: Replace IP Address Pattern

```yaml
- name: Update IP address
  replace:
    path: /etc/hosts
    regexp: '(192\.168\.1\.)[0-9]+'
    replace: '\g<1>100'
```

✔ Demonstrates **regex groups**

---

## 6. Replace Multiple Lines at Once

```yaml
- name: Disable debug flags
  replace:
    path: /etc/myapp/app.conf
    regexp: 'debug=true'
    replace: 'debug=false'
```

All matching occurrences are replaced.

---

## 7. Using Backup (Highly Recommended)

```yaml
- name: Replace with backup
  replace:
    path: /etc/nginx/nginx.conf
    regexp: 'worker_processes\s+1'
    replace: 'worker_processes auto'
    backup: yes
```

Creates a backup before modifying the file.

---

## 8. replace vs lineinfile vs blockinfile

| Feature               | replace          | lineinfile         | blockinfile   |
| --------------------- | ---------------- | ------------------ | ------------- |
| Purpose               | Replace patterns | Ensure single line | Manage blocks |
| Uses regex            | Yes (mandatory)  | Optional           | No            |
| Partial line replace  | Yes              | No                 | No            |
| Multiple replacements | Yes              | No                 | Yes (block)   |
| Markers               | No               | No                 | Yes           |

---

## 9. Common Mistakes

❌ Using `replace` when `lineinfile` is safer

❌ Writing incorrect regex (escaping issues)

❌ Forgetting backups on critical files

❌ Assuming replace manages line uniqueness

---

## 10. Best Practices

* Always test regex separately
* Use `backup: yes` in production
* Prefer `lineinfile` for simple configs
* Keep replace operations minimal and targeted

---

## 11. Interview Questions and Answers

### Q1. What does the replace module do?

**Answer:** It replaces text patterns in a file using regular expressions.

---

### Q2. Difference between replace and lineinfile?

**Answer:** `replace` changes parts of lines using regex, while `lineinfile` ensures a full line exists or is removed.

---

### Q3. Can replace modify multiple lines?

**Answer:** Yes, all matching patterns are replaced.

---

### Q4. Is replace idempotent?

**Answer:** Yes, if the regex and replacement are written correctly.

---

### Q5. When should you avoid replace?

**Answer:** When managing structured or key-value configs where `lineinfile` or templates are safer.

---

## 12. One-Line Interview Summary

* **replace**: Regex-based text replacement inside files

---

## Final Summary

* `replace` is powerful but dangerous if misused
* Best suited for **pattern-based edits**
* Complements `lineinfile` and `blockinfile`

Used correctly, it is invaluable for **legacy system automation**.

---

Happy Automating with Ansible 🚀
