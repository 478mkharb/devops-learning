# Magic Variables in Ansible

## Definition

Magic variables are special predefined variables automatically created by Ansible. They provide information about the inventory, hosts, playbook execution, and environment, and cannot be set or modified by users.

**Simple definition:**
Magic variables are automatically generated variables that give information about the playbook run, inventory, and hosts.

---

# Common Magic Variables

| Magic Variable           | Description                           |
| ------------------------ | ------------------------------------- |
| hostvars                 | Access variables of other hosts       |
| groups                   | Dictionary of all inventory groups    |
| group_names              | Groups the current host belongs to    |
| inventory_hostname       | Current host name from inventory      |
| inventory_hostname_short | Short hostname                        |
| ansible_play_hosts       | Hosts involved in the play            |
| ansible_play_batch       | Current batch of hosts being executed |
| playbook_dir             | Directory of the running playbook     |
| inventory_dir            | Inventory file location               |

---

# Example 1 – inventory_hostname

```yaml
- name: Show hostname
  debug:
    msg: "This host is {{ inventory_hostname }}"
```

Example output:

```
This host is web1
```

---

# Example 2 – groups

```yaml
- name: Print all webservers
  debug:
    msg: "{{ groups['webservers'] }}"
```

Example output:

```
['web1', 'web2', 'web3']
```

---

# Example 3 – hostvars

Access variables of another host.

```yaml
- name: Show IP of another host
  debug:
    msg: "{{ hostvars['web1']['ansible_host'] }}"
```

This lets one host read variables of another host.

---

# Example 4 – group_names

```yaml
- name: Print groups for this host
  debug:
    var: group_names
```

Example output:

```
["webservers", "production"]
```

---

# Example Inventory

```ini
[webservers]
web1
web2

[db]
db1
```

Magic variables allow playbooks to dynamically understand the inventory structure.

---

# Key Characteristics

| Feature       | Description                 |
| ------------- | --------------------------- |
| Created by    | Ansible automatically       |
| User editable | No                          |
| Used for      | Inventory and runtime info  |
| Accessed in   | Playbooks, templates, tasks |

---

## One-line Interview Answer

Magic variables in Ansible are automatically generated variables that provide information about hosts, inventory, and playbook execution during runtime.
