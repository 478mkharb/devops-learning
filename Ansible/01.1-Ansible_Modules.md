# Ansible Modules Based on Functionality / Purpose

This is the **most common interview explanation**.
Here, modules are grouped by **what task they perform** — not by where they come from.

---

## 1️⃣ System Modules

Manage OS-level resources like users, groups, cron jobs.

**Examples:** `user`, `group`, `cron`

```yaml
- name: Create user
  ansible.builtin.user:
    name: devops
```

---

## 2️⃣ Package Modules

Install, update, or remove software packages.

**Examples:** `apt`, `yum`, `dnf`, `package`

```yaml
- name: Install nginx
  ansible.builtin.package:
    name: nginx
    state: present
```

---

## 3️⃣ File Modules

Manage files, directories, permissions, and configuration files.

**Examples:** `copy`, `template`, `file`, `lineinfile`

```yaml
- name: Create directory
  ansible.builtin.file:
    path: /opt/app
    state: directory
```

---

## 4️⃣ Service Modules

Control services like start, stop, restart, enable.

**Examples:** `service`, `systemd`

```yaml
- name: Start nginx
  ansible.builtin.service:
    name: nginx
    state: started
```

---

## 5️⃣ Network Modules

Configure network devices such as routers and switches.

**Examples:** `ios_config`, `junos_config`

```yaml
- name: Configure interface
  cisco.ios.ios_config:
    lines:
      - interface GigabitEthernet0/1
```

---

## 6️⃣ Cloud Modules

Provision and manage cloud infrastructure.

**Examples:** `ec2_instance`, `azure_rm_vm`

```yaml
- name: Launch EC2
  amazon.aws.ec2_instance:
    name: web-server
```

---

## 7️⃣ Database Modules

Create and manage databases and database users.

**Examples:** `mysql_db`, `postgresql_user`

```yaml
- name: Create database
  community.mysql.mysql_db:
    name: app_db
```

---

## 8️⃣ Container Modules

Manage Docker containers and images.

**Examples:** `docker_container`, `docker_image`

```yaml
- name: Run container
  community.docker.docker_container:
    name: nginx
    image: nginx
```

---

## 9️⃣ Kubernetes Modules

Manage Kubernetes resources.

**Examples:** `k8s`, `helm`

```yaml
- name: Deploy to Kubernetes
  kubernetes.core.k8s:
    state: present
    src: deployment.yaml
```

---

## 🔟 Monitoring & Notification Modules

Send alerts and integrate with monitoring tools.

**Examples:** `slack`, `mail`

```yaml
- name: Send alert
  community.general.slack:
    msg: "Deployment completed"
```

---

## 1️⃣1️⃣ Utility Modules

Used for debugging, variables, and flow control.

**Examples:** `debug`, `set_fact`, `assert`

```yaml
- name: Show variable
  ansible.builtin.debug:
    msg: "App version is {{ app_version }}"
```

---

## 🎯 Interview One‑Liner

**“Based on functionality, Ansible modules are grouped into system, package, file, service, network, cloud, database, container, Kubernetes, monitoring, and utility modules.”**
