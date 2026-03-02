# Create and Manage AWS EC2 Instances Using Ansible Dynamic Inventory

This README provides a **complete end-to-end, step-by-step guide** to:

* Create EC2 instances using Ansible
* Configure AWS authentication
* Use **Ansible Dynamic Inventory (`aws_ec2`)**
* Manage and configure EC2 instances after creation

> ⚠️ **Important Concept**: Dynamic inventory **does NOT create EC2 instances**. It is only used to **discover and manage already created instances**.

---

## Architecture Overview

```
Developer/Jenkins
      |
      v
Ansible Control Node
      |
      | (Create EC2)
      v
AWS EC2 API
      |
      | (Discover EC2)
      v
Dynamic Inventory (aws_ec2)
      |
      v
Configure EC2 Instances
```

---

## Prerequisites

### 1. System Requirements

* Linux / macOS
* Python >= 3.8
* Ansible >= 2.14
* AWS Account

Verify:

```bash
ansible --version
python3 --version
```

---

## Step 1: Install Required Ansible Collections

```bash
ansible-galaxy collection install amazon.aws community.aws
```

Verify:

```bash
ansible-galaxy collection list | grep amazon.aws
```

---

## Step 2: Configure AWS Credentials

### Option 1: Using AWS CLI (Recommended)

```bash
aws configure
```

Provide:

* AWS Access Key ID
* AWS Secret Access Key
* Default region (example: ap-south-1)
* Output format (json)

Credentials are stored in:

```
~/.aws/credentials
~/.aws/config
```

---

### Option 2: Using Environment Variables

```bash
export AWS_ACCESS_KEY_ID=XXXX
export AWS_SECRET_ACCESS_KEY=YYYY
export AWS_DEFAULT_REGION=ap-south-1
```

Verify:

```bash
aws sts get-caller-identity
```

---

## Step 3: Create EC2 Instance Using Ansible

Dynamic inventory **cannot create EC2**. We use a playbook.

### 3.1 Create Playbook: `create_ec2.yml`

```yaml
---
- name: Create EC2 instance
  hosts: localhost
  connection: local
  gather_facts: false

  tasks:
    - name: Launch EC2 instance
      amazon.aws.ec2_instance:
        name: ansible-demo-ec2
        key_name: mykeypair
        instance_type: t2.micro
        image_id: ami-0f5ee92e2d63afc18
        wait: true
        region: ap-south-1
        security_group: web-sg
        vpc_subnet_id: subnet-xxxxxxxx
        count: 1
        tags:
          Environment: dev
          ManagedBy: Ansible
      register: ec2_output
```

> Replace:

* `key_name`
* `image_id`
* `security_group`
* `subnet_id`

---

### 3.2 Run the Playbook

```bash
ansible-playbook create_ec2.yml
```

Verify EC2 creation from AWS Console.

---

## Step 4: Configure Ansible Dynamic Inventory

### 4.1 Create Inventory File: `aws_ec2.yml`

```yaml
plugin: amazon.aws.aws_ec2

regions:
  - ap-south-1

filters:
  tag:ManagedBy: Ansible

keyed_groups:
  - key: tags.Environment
    prefix: env
  - key: instance_type
    prefix: type

hostnames:
  - private-ip-address
```

---

### 4.2 Enable Dynamic Inventory Plugin

Create or update `ansible.cfg`:

```ini
[inventory]
enable_plugins = amazon.aws.aws_ec2
```

---

### 4.3 Test Dynamic Inventory

```bash
ansible-inventory -i aws_ec2.yml --graph
```

Example Output:

```
@env_dev:
  |--10.0.1.25
```

---

## Step 5: Configure EC2 Using Dynamic Inventory

### 5.1 Create Playbook: `configure_ec2.yml`

```yaml
---
- name: Configure EC2 instances
  hosts: env_dev
  become: yes

  tasks:
    - name: Install Apache
      yum:
        name: httpd
        state: present

    - name: Start Apache
      service:
        name: httpd
        state: started
```

---

### 5.2 Run Configuration Playbook

```bash
ansible-playbook -i aws_ec2.yml configure_ec2.yml
```

---

## Step 6: SSH Access Verification

Ensure:

* Port 22 is open in security group
* Correct key pair used

Test:

```bash
ssh -i mykeypair.pem ec2-user@<public-ip>
```

---

## Step 7: Cleanup – Terminate EC2 Instance

### 7.1 Create Termination Playbook

```yaml
---
- name: Terminate EC2 instance
  hosts: localhost
  connection: local
  gather_facts: false

  tasks:
    - name: Terminate instance
      amazon.aws.ec2_instance:
        state: absent
        filters:
          tag:ManagedBy: Ansible
```

Run:

```bash
ansible-playbook terminate_ec2.yml
```

---

## Common Mistakes

❌ Trying to create EC2 using dynamic inventory

❌ Missing AWS credentials

❌ Wrong AMI for region

❌ SSH blocked in security group

---

## Best Practices

* Use **tags** aggressively
* Separate **infra** and **config** playbooks
* Never hardcode credentials
* Use IAM roles in production

---

## Tooling Summary

| Purpose           | Tool                     |
| ----------------- | ------------------------ |
| Create EC2        | amazon.aws.ec2_instance  |
| Dynamic Inventory | aws_ec2 plugin           |
| Configuration     | Ansible Playbooks        |
| Automation        | Jenkins / GitHub Actions |

---

## Final Notes

This setup is:

* Jenkins friendly
* Scalable
* Production aligned
* Cloud-native

---

Happy Automating 🚀
