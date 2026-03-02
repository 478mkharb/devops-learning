# Ansible Modules 

---

## 1. script Module

### Purpose

The `script` module copies a **local script** from the Ansible control node to the managed host and executes it there.

### Key Points

* Script runs on the **remote node**
* Script must exist on the **control machine**
* No need to pre-copy the script manually

### Common Parameters

* `cmd`: Path of the script to run (mandatory)
* `creates`: Prevents re-running if a file exists
* `removes`: Runs script only if a file exists

### Example

```yaml
- name: Run a shell script on remote server
  script: install_app.sh
```

### Use Cases

* Legacy scripts execution
* One-time setup scripts
* Quick automation without converting scripts to tasks

### Best Practices

* Prefer Ansible modules over scripts when possible
* Use `creates` for idempotency

---

## 2. firewalld Module

### Purpose

Manages firewall rules on systems using **firewalld** (RHEL, CentOS, Rocky, AlmaLinux).

### Key Points

* Works only when firewalld service is running
* Supports zones, services, ports, rich rules

### Common Parameters

* `service`: Firewall service name (http, https)
* `port`: Port/protocol (80/tcp)
* `zone`: Firewall zone (public, internal)
* `state`: enabled / disabled
* `permanent`: yes/no
* `immediate`: yes/no

### Example

```yaml
- name: Allow HTTP service
  firewalld:
    service: http
    permanent: yes
    state: enabled
```

### Use Cases

* Opening application ports
* Securing servers dynamically

### Best Practices

* Use `permanent: yes` with `immediate: yes`
* Reload firewalld only when needed

---

## 3. setup Module

### Purpose

Collects **system facts** from managed nodes.

### Key Points

* Automatically runs at play start (unless disabled)
* Facts stored in `ansible_facts`

### Common Parameters

* `filter`: Collect specific facts
* `gather_subset`: Limit fact collection

### Example

```yaml
- name: Collect OS facts
  setup:
    filter: ansible_distribution*
```

### Common Facts

* `ansible_os_family`
* `ansible_hostname`
* `ansible_memtotal_mb`
* `ansible_default_ipv4.address`

### Use Cases

* Conditional task execution
* Environment-specific logic

---

## 4. git Module

### Purpose

Manages Git repositories on remote nodes.

### Key Points

* Supports clone, pull, checkout
* Works with public and private repos

### Common Parameters

* `repo`: Repository URL
* `dest`: Destination path
* `version`: Branch/tag/commit
* `update`: yes/no
* `force`: yes/no

### Example

```yaml
- name: Clone application repo
  git:
    repo: https://github.com/org/app.git
    dest: /opt/app
    version: main
```

### Use Cases

* Deploying applications
* Configuration sync

---

## 5. blockinfile Module

### Purpose

Manages **blocks of text** in files.

### Key Points

* Inserts content between markers
* Ensures idempotency

### Common Parameters

* `path`: File path
* `block`: Text block
* `marker`: Custom marker text
* `create`: yes/no

### Example

```yaml
- name: Add config block
  blockinfile:
    path: /etc/myapp.conf
    block: |
      server=prod
      port=8080
```

### Use Cases

* Appending config sections
* Managing application configs

---

## 6. stat Module

### Purpose

Checks **file or directory properties**.

### Key Points

* Does not modify anything
* Used with conditionals

### Common Parameters

* `path`: File or directory path

### Registered Variables

* `exists`
* `isdir`
* `size`
* `mtime`

### Example

```yaml
- name: Check if file exists
  stat:
    path: /etc/nginx/nginx.conf
  register: nginx_conf
```

### Use Cases

* Conditional execution
* Validation checks

---

## 7. cron Module

### Purpose

Manages **cron jobs** for users.

### Key Points

* Ensures cron job presence or absence
* Prevents duplicate entries

### Common Parameters

* `name`: Job description
* `minute`, `hour`, `day`, `month`, `weekday`
* `job`: Command
* `user`: Target user

### Example

```yaml
- name: Schedule backup job
  cron:
    name: "Daily backup"
    hour: "2"
    job: "/opt/backup.sh"
```

### Use Cases

* Scheduled maintenance
* Log cleanup

---

## 8. pip Module

### Purpose

Manages Python packages using **pip**.

### Key Points

* Works with virtualenvs
* Requires Python & pip installed

### Common Parameters

* `name`: Package name
* `version`: Specific version
* `virtualenv`: Virtual environment path
* `state`: present / absent

### Example

```yaml
- name: Install Flask
  pip:
    name: flask
```

### Use Cases

* Python application dependencies
* Automation scripts

---

## 9. yum_key Module

### Purpose

Manages **GPG keys** for YUM repositories.

### Key Points

* Ensures package authenticity
* Used before adding repositories

### Common Parameters

* `key`: URL or path of GPG key
* `state`: present / absent

### Example

```yaml
- name: Add Docker GPG key
  yum_key:
    key: https://download.docker.com/linux/centos/gpg
    state: present
```

### Use Cases

* Third-party repositories
* Secure package installation

---

## 10. find Module

### Purpose

Finds files and directories based on criteria.

### Key Points

* Does not delete files (use with `file` module)
* Powerful filtering options

### Common Parameters

* `paths`: Search paths
* `patterns`: File patterns
* `age`: File age
* `size`: File size
* `file_type`: file / directory

### Example

```yaml
- name: Find log files
  find:
    paths: /var/log
    patterns: "*.log"
  register: log_files
```

### Use Cases

* Cleanup automation
* Reporting

---

## Interview Tips

* Prefer **modules over scripts**
* Ensure **idempotency**
* Use `stat` + `when` together
* Use `setup` facts for dynamic logic

---

## 11. telnet Module

### Purpose

Used to **test connectivity** to remote hosts on a specific port using the Telnet protocol.

### Key Points

* Mainly for **testing**, not configuration
* Useful in troubleshooting network/service availability

### Common Parameters

* `host`: Target host
* `port`: Port number
* `timeout`: Connection timeout

### Example

```yaml
- name: Test connectivity to web server
  telnet:
    host: 192.168.1.10
    port: 80
```

---

## 12. user (user_modify)

### Purpose

Manages **user accounts** on managed nodes.

### Common Parameters

* `name`: Username
* `uid`: User ID
* `group`: Primary group
* `groups`: Supplementary groups
* `state`: present / absent

### Example

```yaml
- name: Create user
  user:
    name: devuser
    state: present
```

---

## 13. timezone Module

### Purpose

Manages system **timezone settings**.

### Example

```yaml
- name: Set timezone to Asia/Kolkata
  timezone:
    name: Asia/Kolkata
```

---

## 14. wget Module

### Purpose

Downloads files from HTTP/HTTPS/FTP URLs.

### Example

```yaml
- name: Download file
  get_url:
    url: https://example.com/app.tar.gz
    dest: /opt/app.tar.gz
```

(Note: `get_url` is preferred over wget in Ansible)

---

## 15. set_facts Module

### Purpose

Defines **custom variables (facts)** during play execution.

### Example

```yaml
- name: Set custom fact
  set_fact:
    env: production
```

---

## 16. apt_key Module

### Purpose

Manages **APT GPG keys** for Debian-based systems.

### Example

```yaml
- name: Add repo key
  apt_key:
    url: https://example.com/key.gpg
    state: present
```

---

## 17. apt_repository Module

### Purpose

Manages **APT repositories**.

### Example

```yaml
- name: Add repository
  apt_repository:
    repo: ppa:ansible/ansible
```

---

## 18. mail Module

### Purpose

Sends email notifications from playbooks.

### Example

```yaml
- name: Send mail
  mail:
    to: admin@example.com
    subject: Deployment Status
    body: Deployment completed successfully
```

---

## 19. fetch Module

### Purpose

Copies files **from managed nodes to control node**.

### Example

```yaml
- name: Fetch logs
  fetch:
    src: /var/log/syslog
    dest: ./logs/
```

---

## 20. expect Module

### Purpose

Automates **interactive commands** requiring user input.

### Example

```yaml
- name: Run interactive script
  expect:
    command: passwd user1
    responses:
      "New password": "Pass@123"
```

---

## 21. systemd Module

### Purpose

Manages **systemd services**.

### Example

```yaml
- name: Start nginx
  systemd:
    name: nginx
    state: started
    enabled: yes
```

---

## 22. debug Module

### Purpose

Prints variables and messages for **debugging**.

### Example

```yaml
- debug:
    var: ansible_hostname
```

---

## 23. yum_repository Module

### Purpose

Manages **YUM repositories**.

### Example

```yaml
- name: Add yum repo
  yum_repository:
    name: myrepo
    baseurl: http://repo.example.com
    enabled: yes
```

---

## 24. service Module

### Purpose

Manages services across init systems.

### Example

```yaml
- name: Restart service
  service:
    name: httpd
    state: restarted
```

---

## 25. group Module

### Purpose

Manages **Linux groups**.

### Example

```yaml
- name: Create group
  group:
    name: devops
```

---

## 26. command Module

### Purpose

Executes commands on remote nodes (no shell features).

### Example

```yaml
- command: uptime
```

---

## 27. apk Module

### Purpose

Manages packages on **Alpine Linux**.

### Example

```yaml
- name: Install package
  apk:
    name: nginx
    state: present
```

---

## 28. raw Module

### Purpose

Runs raw SSH commands (no Python required).

### Example

```yaml
- raw: yum install -y python3
```

---

## 29. meta Module

### Purpose

Controls **play execution behavior**.

### Example

```yaml
- meta: end_play
```

---

## Best Practices Summary

* Prefer **modules over command/raw**
* Use `debug` during development
* Ensure idempotency
* Use OS-specific modules (apt/yum/apk)

---

✅ This completes all modules listed in your table and image.
