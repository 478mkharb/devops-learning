# Ansible Roles & Ansible Galaxy 🚀

## What is an Ansible Role? 📦

An Ansible Role is a structured way to organize playbooks, variables, files, templates, and handlers so that automation becomes reusable and maintainable.

Roles help break complex playbooks into smaller reusable components.

---

## Standard Ansible Role Directory Structure 🗂️

```
roles/
 └── webserver/
     ├── tasks/
     │   └── main.yml
     ├── handlers/
     │   └── main.yml
     ├── templates/
     │   └── nginx.conf.j2
     ├── files/
     │   └── index.html
     ├── vars/
     │   └── main.yml
     ├── defaults/
     │   └── main.yml
     ├── meta/
     │   └── main.yml
     └── README.md
```

---

## Directory Explanation 📚

### tasks/ ⚙️

Contains the main list of tasks executed by the role.

Example:

```
---
- name: Install nginx
  apt:
    name: nginx
    state: present

- name: Start nginx
  service:
    name: nginx
    state: started
```

---

### handlers/ 🔁

Handlers run when notified by tasks.

Example:

```
---
- name: restart nginx
  service:
    name: nginx
    state: restarted
```

---

### templates/ 🧩

Jinja2 templates used to dynamically generate configuration files.

Example:

```
server {
 listen 80;
 server_name {{ domain_name }};
}
```

---

### files/ 📁

Static files copied directly to the remote system.

Example:

```
index.html
```

---

### vars/ 🔐

Variables with higher priority.

Example:

```
nginx_port: 80
```

---

### defaults/ 🪶

Default variables with lowest priority.

Example:

```
nginx_port: 80
```

---

### meta/ 📑

Role metadata including dependencies.

Example:

```
dependencies:
 - role: common
```

---

## Example Playbook Using Role ▶️

```
- hosts: web
  roles:
   - webserver
```

---

# Ansible Galaxy 🌌

## What is Ansible Galaxy?

Ansible Galaxy is the official repository for sharing and downloading Ansible roles.

It works similar to GitHub but specifically for reusable Ansible automation content.

---

## Why Use Ansible Galaxy? 🎯

* Reuse community roles
* Save development time
* Follow best practices
* Standardized role structure

---

## Common Ansible Galaxy Commands 💻

Install role

```
ansible-galaxy install geerlingguy.nginx
```

Create role

```
ansible-galaxy init myrole
```

List installed roles

```
ansible-galaxy list
```

Remove role

```
ansible-galaxy remove role_name
```

---

## Example Role Installation 📥

```
ansible-galaxy install geerlingguy.mysql
```

Directory created:

```
roles/
 └── geerlingguy.mysql
```

---

## Role Dependency via requirements.yml 📦

```
roles:
 - name: geerlingguy.nginx
 - name: geerlingguy.mysql
```

Install all roles

```
ansible-galaxy install -r requirements.yml
```

---

## DevOps Interview Tips 💡

**Q: Why use roles instead of playbooks?**

Answer:

Roles provide modular, reusable, and scalable automation by separating tasks, variables, templates, and handlers into a standardized structure.

---

## Key Takeaways 🎯

* Roles make Ansible automation modular
* Galaxy is a role repository
* Roles follow a fixed directory structure
* Galaxy enables reusable infrastructure automation
