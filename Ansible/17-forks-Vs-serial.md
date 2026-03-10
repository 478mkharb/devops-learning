# Forks vs Serial in Ansible (With Examples, Use Cases & Execution Flow)

Both **`forks`** and **`serial`** control **how many hosts Ansible works on at a time**, but they operate at **different levels** and solve **different problems**.

> 🧠 **Core idea**:
>
> * `forks` controls **parallelism capacity** (how many hosts *can* run at once)
> * `serial` controls **deployment batches** (how many hosts *should* run at once)

---

# Example Setup Used in All Diagrams

Inventory (4 servers)

```
host1
host2
host3
host4
```

Example playbook with **two tasks**:

```yaml
- name: Demo play
  hosts: web
  tasks:
    - name: Task 1
      shell: echo "task1"

    - name: Task 2
      shell: echo "task2"
```

---

# What Is `forks`

## Definition

`forks` defines the **maximum number of parallel worker processes** Ansible can use on the controller.

* Global setting
* Limits controller concurrency
* Default value: **5**

Workers are controller-side processes that:

1. Pick a host
2. Open SSH connection
3. Send module
4. Execute task
5. Return result

---

# Where `forks` Is Configured

CLI

```
ansible-playbook site.yml -f 20
```

or in `ansible.cfg`

```
[defaults]
forks = 20
```

---

# Execution Flow — forks = 2

Example: **4 hosts, forks = 2**

All hosts belong to the play **from the beginning**.

Ansible only limits how many hosts run **simultaneously**.

## Task Execution Timeline

```
TASK 1

worker1 -> host1
worker2 -> host2

(wait)

worker1 -> host3
worker2 -> host4

TASK 2

worker1 -> host1
worker2 -> host2

(wait)

worker1 -> host3
worker2 -> host4
```

## Execution Flow Diagram

```
                Ansible Controller
                        |
                Worker Pool (forks=2)
                   /          \
             Worker1        Worker2
               |               |
             host1           host2

                (wait for worker free)

             Worker1        Worker2
               |               |
             host3           host4
```

### Important Behavior

```
All hosts are active in the play
Some hosts wait for a free worker
```

---

# What Is `serial`

## Definition

`serial` defines **how many hosts participate in the play at one time**.

It splits hosts into **batches** and runs the entire play on each batch sequentially.

* Play-level setting
* Used for rolling deployments

---

# Example Playbook Using Serial

```yaml
- name: Rolling deployment
  hosts: web
  serial: 2
  tasks:
    - name: Task 1
      shell: echo "task1"

    - name: Task 2
      shell: echo "task2"
```

---

# Execution Flow — serial = 2

Hosts are divided into batches.

```
Batch1 -> host1 host2
Batch2 -> host3 host4
```

## Task Execution Timeline

```
Batch 1

Task1
host1
host2

Task2
host1
host2

Batch1 finished

Batch 2

Task1
host3
host4

Task2
host3
host4
```

## Execution Flow Diagram

```
                Ansible Controller
                        |
                    Batch 1
                  /         \
               host1       host2

               Task1
               Task2

            Batch1 Complete

                    Batch 2
                  /         \
               host3       host4

               Task1
               Task2
```

### Important Behavior

```
Next batch does NOT start
until previous batch finishes
```

---

# Visual Comparison (Most Important)

## forks = 2

```
All hosts active

host1  running
host2  running
host3  waiting
host4  waiting
```

Controller only limits workers.

---

## serial = 2

```
Batch1 active

host1 running
host2 running

Batch2 not started

host3
host4
```

Hosts only enter play **batch by batch**.

---

# How forks and serial Work Together

Actual parallel hosts are determined by:

```
parallel hosts = min(forks, serial)
```

Example

```
forks = 10
serial = 2
```

Result

```
Only 2 hosts run at once
```

because serial restricts batch size.

---

# Real DevOps Use Cases

## Use `forks` When

Speed matters and hosts are independent.

Examples:

* collecting logs
* patching many servers
* gathering system facts
* installing packages

---

## Use `serial` When

Safety matters more than speed.

Examples:

* rolling deployments
* restarting web servers behind load balancer
* updating Kubernetes worker nodes
* database migrations

---

# Production Rolling Deployment Example

```yaml
- name: Zero downtime deploy
  hosts: web
  serial: 1
  tasks:
    - name: Remove from load balancer
      command: /opt/lb_remove.sh

    - name: Deploy application
      command: /opt/deploy.sh

    - name: Add back to load balancer
      command: /opt/lb_add.sh
```

Execution order

```
web1 deploy
web2 deploy
web3 deploy
web4 deploy
```

Service stays available.

---

# Interview One-Liner

```
forks = controller parallel capacity
serial = deployment batch size
```

Or

```
Forks define how much Ansible CAN do in parallel
Serial defines how much it SHOULD do at a time
```
