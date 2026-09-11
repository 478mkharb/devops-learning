# Ansible Senior L2 Interview — Part 6

**Focus:** Roles, AWS, dynamic inventory and mixed operating systems

## Q31. What is an Ansible role and what is its directory structure?

### Short explanation of the question

This tests whether I can organize automation into reusable production components.

### Answer

A role packages related tasks, handlers, templates, files, defaults, vars and metadata into a standard reusable structure. The structure gives each concern a predictable location and makes automation easier to reuse and maintain.

### Detailed explanation

### What is an Ansible Role?

An Ansible Role is a structured way to organize playbooks, variables, files, templates, and handlers so that automation becomes reusable and maintainable.

Roles help break complex playbooks into smaller reusable components.

---

### Standard Ansible Role Directory Structure 🗂

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

### Directory Explanation

### tasks/

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

### templates/

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

### vars/

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

### Example Playbook Using Role ▶

```
- hosts: web
  roles:
   - webserver
```

---

# Ansible Galaxy 🌌

### What is Ansible Galaxy?

Ansible Galaxy is the official repository for sharing and downloading Ansible roles.

It works similar to GitHub but specifically for reusable Ansible automation content.

---

### Why Use Ansible Galaxy? 🎯

* Reuse community roles
* Save development time
* Follow best practices
* Standardized role structure

---

### Common Ansible Galaxy Commands 💻

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

### Example Role Installation 📥

```
ansible-galaxy install geerlingguy.mysql
```

Directory created:

```
roles/
 └── geerlingguy.mysql
```

---

### Role Dependency via requirements.yml

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

### DevOps Interview Tips 💡

**Q: Why use roles instead of playbooks?**

Answer:

Roles provide modular, reusable, and scalable automation by separating tasks, variables, templates, and handlers into a standardized structure.

---

### Example

```text
roles/
└── webserver/
    ├── tasks/main.yml
    ├── handlers/main.yml
    ├── templates/nginx.conf.j2
    ├── files/
    ├── defaults/main.yml
    ├── vars/main.yml
    └── meta/main.yml
```

### Interviewer may cross-question

**Interviewer:** "Why use `defaults` instead of putting every variable in `vars`?"

**Candidate:** "Role defaults are intentionally easier for callers to override. I use role vars more selectively when stronger precedence is actually required." 

---

## Q32. How does Ansible work with AWS Systems Manager (SSM)?

### Short explanation of the question

This is a Senior L2 cloud question about the complete SSM execution path.

### Answer

With the AWS SSM connection plugin, Ansible can use Systems Manager instead of inbound SSH. The setup depends on the instance being managed by SSM, appropriate IAM permissions, controller-side AWS configuration, and the current collection's SSM/S3 requirements.

### Detailed explanation

This document explains **in depth** how Ansible executes tasks on EC2 instances **without SSH**, using **AWS Systems Manager (SSM)** and **AWS APIs**.

This is a **senior‑level concept** frequently asked in interviews and heavily used in **locked‑down AWS environments**.

---

### The Core Idea (Big Picture)

> **Ansible does NOT connect to EC2 instances when using SSM.**
> **It connects to AWS APIs, and AWS delivers the commands to the instance via the SSM Agent.**

This is why it works even when:

* SSH (22) is blocked ❌
* Instances are in private subnets 🔒
* No inbound ports are allowed

---

### Key Components Involved

### 1

* Runs Ansible playbooks
* Uses **AWS credentials** (IAM user/role)
* Calls **AWS SSM APIs**

---

### 2

* Managed AWS service
* Receives commands via API
* Queues and delivers commands to instances

---

### 3

Runs on each EC2 instance:

```text
amazon-ssm-agent
```

Responsibilities:

* Maintains **outbound HTTPS (443)** connection to AWS
* Polls for commands
* Executes commands locally (usually as root)
* Returns output back to AWS

📌 This is an **agent-based model**.

---

### 4

Controls:

* Who can send commands
* Which instances can receive commands
* What actions are allowed

No SSH keys involved ❌

---

### Full Execution Flow (Step by Step)

```text
┌────────────────────┐
│ Ansible Control    │
│ Node               │
│ (aws_ssm plugin)   │
└─────────┬──────────┘
          │ 1️⃣ AWS API Call (SendCommand)
          ▼
┌────────────────────┐
│ AWS SSM Service    │
│ (Regional API)     │
└─────────┬──────────┘
          │ 2️⃣ Command queued
          ▼
┌────────────────────┐
│ SSM Agent on EC2   │◀───────┐
│ (Outbound HTTPS)   │        │
└─────────┬──────────┘        │
          │ 3️⃣ Execute command│
          ▼                   │
┌────────────────────┐        │
│ EC2 Instance       │        │
│ (No SSH access)    │        │
└─────────┬──────────┘        │
          │ 4️⃣ Output sent back
          ▼                   │
┌────────────────────┐        │
│ AWS SSM Service    │────────┘
└─────────┬──────────┘
          │ 5️⃣ Result returned
          ▼
┌────────────────────┐
│ Ansible Output     │
└────────────────────┘
```

---

### Important Detail: Where Modules Execute

With SSH:

* Ansible copies modules to the host

With SSM:

* **Modules run inside the SSM execution environment**
* Commands are wrapped as SSM documents

👉 Ansible never opens a shell on the instance.

---

### How Ansible Is Configured to Use SSM

### Inventory Example

```ini
[ec2]
i-0abc123def456
```

---

### Variables Required

```yaml
ansible_connection: aws_ssm
ansible_aws_ssm_region: us-east-1
ansible_aws_ssm_s3_bucket: my-ssm-logs
```

---

### Example Playbook (No SSH)

```yaml
- hosts: ec2
  gather_facts: false
  tasks:
    - name: Check uptime via SSM
      command: uptime

    - name: Ensure nginx is installed
      shell: yum install -y nginx
```

What actually happens:

* Ansible → AWS API (`SendCommand`)
* SSM Agent executes commands
* Output returned to Ansible

---

### Security Model Explained

### IAM on Control Node

```json
{
  "Effect": "Allow",
  "Action": [
    "ssm:SendCommand",
    "ssm:GetCommandInvocation"
  ],
  "Resource": "*"
}
```

### IAM Role on EC2

```json
{
  "Effect": "Allow",
  "Action": [
    "ssm:UpdateInstanceInformation",
    "ssm:ListCommands"
  ],
  "Resource": "*"
}
```

---

### Why Enterprises Prefer SSM

✔ No inbound ports
✔ No SSH key management
✔ Centralized audit logs
✔ IAM-based access control
✔ Works in private subnets

---

### Key Limitations (Important)

⚠️ Slower than SSH for large file transfers
⚠️ Requires AWS-only environment
⚠️ Some Ansible modules behave differently
⚠️ Debugging is less interactive

---

### Comparison: SSH vs SSM

| Feature                 | SSH | AWS SSM |
| ----------------------- | --- | ------- |
| Agent required          | ❌   | ✅       |
| Inbound ports           | ✅   | ❌       |
| Uses AWS APIs           | ❌   | ✅       |
| Works in private subnet | ⚠️  | ✅       |
| Audit logging           | ⚠️  | ✅       |

---

### Mental Model

> **With SSH, Ansible talks to the server.**
> **With SSM, Ansible talks to AWS, and AWS talks to the server.**

---

### Interview One‑Liner 🎯

> **Ansible works with AWS SSM by sending commands through AWS APIs, which are executed by the SSM agent on EC2 instances over outbound HTTPS, eliminating the need for SSH.**

---

### Summary

* This is an agent‑based execution model
* Ansible never connects to the instance
* AWS SSM acts as the secure command broker
* Ideal for locked‑down cloud environments

---
# AWS SSM Networking – 5 Interview Questions with Answers 🎯

These questions focus on **AWS SSM networking (ports, SG, NACL)** and **Ansible + SSM**, exactly the depth expected in **mid–senior DevOps interviews**.

---

### 1

### Answer

AWS SSM requires **only outbound HTTPS (TCP 443)** from the EC2 instance.

* ❌ No inbound ports are required
* ❌ SSH (22) is NOT needed
* ❌ RDP (3389) is NOT needed

The SSM Agent communicates with AWS services using **outbound HTTPS connections only**.

📌 This is why SSM works in locked-down environments.

---

### 2

### Answer

Security Groups are **stateful**, so configuration is simple:

**Inbound rules:**

* ❌ None required

**Outbound rules:**

* ✅ Allow TCP 443 to AWS (or 0.0.0.0/0)

Because SGs are stateful, return traffic is automatically allowed.

---

### 3

### Answer

NACLs are **stateless**, so both request and response traffic must be explicitly allowed.

**Outbound NACL rules:**

* ✅ Allow TCP 443 (EC2 → AWS)

**Inbound NACL rules:**

* ✅ Allow TCP 1024–65535 (ephemeral ports)

The ephemeral ports are required for **return traffic from AWS services**.

📌 This difference is a very common production issue.

---

### 4

### Answer

SSM uses an **agent-based, outbound-only model**:

* The SSM Agent runs on the EC2 instance
* It maintains an outbound HTTPS connection to AWS
* AWS never initiates a connection to the instance

This design:

* Improves security
* Eliminates bastion hosts
* Avoids inbound firewall rules

---

### 5

### Answer

Ansible does **not connect to the EC2 instance** when using SSM.

Flow:

```text
Ansible → AWS SSM API → SSM Service → SSM Agent → EC2
```

* Ansible uses the `aws_ssm` connection plugin
* Communicates with AWS via APIs (HTTPS 443)
* The SSM Agent executes commands locally

📌 SSH is completely bypassed.

---

### Interview Summary (Quick Revision

* SSM → outbound HTTPS only
* SG → outbound 443 is enough
* NACL → outbound 443 + inbound ephemeral ports
* No inbound access ever
* Ansible talks to AWS, not EC2

---

### Execution flow

```text
Ansible Controller
      |
      v
AWS APIs / SSM connection plugin
      |
      v
AWS Systems Manager
      |
      v
SSM Agent on EC2
      |
      v
Command / module execution
```

### Interviewer may cross-question

**Interviewer:** "The EC2 instance is private and SSH is blocked. Is SSM automatically enough?"

**Candidate:** "No. I would verify SSM Agent, instance IAM permissions, controller AWS permissions, region/configuration, required S3 access, and the network prerequisites of Systems Manager and the current Ansible collection." 

---

## Q33. What are boto3 and botocore and how do they relate to Ansible?

### Short explanation of the question

This tests whether I understand the AWS SDK layer used by Ansible AWS automation.

### Answer

`boto3` is the high-level AWS SDK for Python and `botocore` provides lower-level AWS service/client machinery. AWS Ansible collections use this SDK stack to call AWS APIs.

### Detailed explanation

This document explains **exactly how Ansible talks to AWS**, step by step, using **AWS APIs**, **IAM**, and **boto3** — without SSH, agents, or logging into servers.

This is a **fundamental cloud‑automation concept** and a **common senior‑level interview topic**.

---

# The Core Idea

**Ansible connects to AWS by calling AWS service APIs over HTTPS using the AWS SDK (boto3), authenticated by IAM credentials.**

Ansible does **not connect to EC2 instances** unless you explicitly configure **SSH or SSM connections**.

---

# Big Mental Model

```
Ansible Playbook
      |
      v
AWS Ansible Module
      |
      v
boto3 / botocore (AWS SDK)
      |
      v
Signed HTTPS Request
      |
      v
AWS Service API (EC2 / S3 / IAM / SSM)
```

Ansible behaves exactly like:

* AWS CLI
* Terraform
* Custom Python SDK apps

---

# Core Components Involved

### 1. Ansible Control Node

The controller machine where playbooks run.

Responsibilities:

* Executes playbooks
* Loads AWS modules from collections
* Sends API requests to AWS

No SSH connection to AWS services happens here.

---

### 2. AWS Ansible Collections

AWS functionality is provided by collections such as:

```
amazon.aws
community.aws
```

Example modules:

```
amazon.aws.ec2_instance
amazon.aws.ec2_vpc
amazon.aws.s3_object
amazon.aws.iam_role
amazon.aws.ssm_parameter
```

Each module maps **directly to AWS API operations**.

Example mapping:

```
ec2_instance  -> RunInstances API
s3_object     -> PutObject API
```

---

### 3. boto3 and botocore

Under the hood execution chain:

```
Ansible Module
      |
      v
boto3 (AWS SDK)
      |
      v
botocore (API engine)
      |
      v
AWS REST API
```

**Important distinction:**

```
boto3 / botocore are required ONLY when Ansible is interacting with AWS APIs
(for example creating, modifying, or deleting AWS resources).
```

Examples where **boto3 IS required**:

* creating EC2 instances
* creating VPCs
* managing security groups
* creating S3 buckets
* modifying IAM roles

Because these actions call AWS service APIs.

Examples where **boto3 is NOT required**:

* installing packages on EC2
* copying files to EC2
* configuring nginx, docker, or applications
* restarting services on EC2

Those tasks use **SSH-based configuration** and run directly on the EC2 instance, not through AWS APIs.

---

# IAM Authentication Flow

AWS authentication is handled via IAM.

Ansible **does not manage credentials itself**.

Credential lookup order used by boto3:

```
1 Environment variables
2 ~/.aws/credentials
3 ~/.aws/config
4 IAM role (EC2 / EKS / ECS)
```

Best practice in production:

```
Run Ansible on EC2 with IAM role
```

This avoids storing access keys.

---

# End-to-End Execution Flow Diagram

```
┌──────────────────────────┐
│ Ansible Controller       │
│ (Playbook Execution)     │
└──────────┬───────────────┘
           │
           │ 1 Load AWS module
           ▼
┌──────────────────────────┐
│ AWS Ansible Module       │
│ (amazon.aws collection)  │
└──────────┬───────────────┘
           │
           │ 2 Call AWS SDK
           ▼
┌──────────────────────────┐
│ boto3 / botocore         │
│ Sign request using IAM   │
└──────────┬───────────────┘
           │
           │ 3 HTTPS request
           ▼
┌──────────────────────────┐
│ AWS Service API          │
│ EC2 / S3 / IAM etc.      │
└──────────┬───────────────┘
           │
           │ 4 IAM policy validation
           │ 5 Service performs action
           ▼
┌──────────────────────────┐
│ AWS JSON Response        │
└──────────┬───────────────┘
           │
           │ 6 Response returned
           ▼
┌──────────────────────────┐
│ Ansible Output           │
│ ok / changed / failed    │
└──────────────────────────┘
```

---

# Example: Creating an EC2 Instance

Playbook:

```yaml
- name: Create EC2 instance
  amazon.aws.ec2_instance:
    name: web01
    instance_type: t3.micro
    image_id: ami-0abc123
    region: us-east-1
```

Internal flow:

```
Ansible
  -> boto3.run_instances()
  -> HTTPS request to EC2 API
  -> IAM authorization
  -> Instance launched
  -> JSON response returned
```

Ansible marks task as:

```
changed
```

---

# EC2 Configuration After Creation (SSH Model)

Once instances exist, Ansible may switch to **SSH configuration mode**.

Flow:

```
Ansible Controller
        |
        | SSH
        v
EC2 Instance
        |
        v
Run tasks (apt, yum, copy, service)
```

This is **different from AWS API mode**.

---

# Example: AWS SSM Connection Model

When using:

```
ansible_connection: aws_ssm
```

Execution flow becomes:

```
Ansible
   |
   v
SSM API (SendCommand)
   |
   v
AWS SSM Service
   |
   v
SSM Agent on EC2
   |
   v
Command execution
```

No inbound SSH required.

---

# API Model vs SSH Model

| Aspect         | AWS API Model | SSH Model |
| -------------- | ------------- | --------- |
| Target         | AWS service   | Server    |
| Transport      | HTTPS         | SSH       |
| Auth           | IAM           | SSH key   |
| Execution      | Control plane | On host   |
| Agent required | No            | No        |

---

# Why This Design Is Powerful

Advantages:

* No inbound ports required
* IAM based access control
* Fully auditable using CloudTrail
* Works inside private VPC
* Same model used by Terraform and AWS CLI

---

# Common Misconceptions

Incorrect beliefs:

```
Ansible logs into AWS
Ansible uses SSH to control AWS
AWS requires agents for Ansible
```

Correct understanding:

```
Ansible interacts with AWS using APIs
```

---

# Interview One‑Liner

**Ansible communicates with AWS by using boto3 to send IAM‑authenticated HTTPS requests to AWS service APIs.**

---

# Final Summary

* Ansible acts as an API client
* boto3 enables AWS communication
* IAM provides authentication and authorization
* No SSH or agents are needed for infrastructure creation
* SSH is only used when configuring EC2 instances

### Execution flow

```text
Ansible AWS module
       |
       v
boto3 / botocore
       |
       v
AWS service API
       |
       v
AWS resource
```

### Interviewer may cross-question

**Interviewer:** "Is boto3 responsible for connecting to an EC2 operating system?"

**Candidate:** "No. Boto3/botocore is the AWS API layer. Operating-system management is a separate path such as SSH or SSM." 

---

## Q34. How do you create EC2 instances with Ansible and discover them dynamically?

### Short explanation of the question

This checks whether I understand the difference between provisioning an EC2 instance and discovering it through dynamic inventory.

### Answer

Ansible can provision EC2 resources through AWS APIs. The `amazon.aws.aws_ec2` inventory plugin is a separate discovery mechanism that finds existing instances using regions, filters, tags and state. Dynamic inventory does not create instances.

### Detailed explanation

This README provides a **complete end-to-end, step-by-step guide** to:

* Create EC2 instances using Ansible
* Configure AWS authentication
* Use **Ansible Dynamic Inventory (`aws_ec2`)**
* Manage and configure EC2 instances after creation

> ⚠️ **Important Concept**: Dynamic inventory **does NOT create EC2 instances**. It is only used to **discover and manage already created instances**.

---

### Architecture Overview

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

### Prerequisites

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

### Step 1: Install Required Ansible Collections

```bash
ansible-galaxy collection install amazon.aws community.aws
```

Verify:

```bash
ansible-galaxy collection list | grep amazon.aws
```

---

### Step 2: Configure AWS Credentials

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

### Step 3: Create EC2 Instance Using Ansible

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

### Step 4: Configure Ansible Dynamic Inventory

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

### Step 5: Configure EC2 Using Dynamic Inventory

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

### Step 6: SSH Access Verification

Ensure:

* Port 22 is open in security group
* Correct key pair used

Test:

```bash
ssh -i mykeypair.pem ec2-user@<public-ip>
```

---

### Step 7: Cleanup – Terminate EC2 Instance

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

### Common Mistakes

❌ Trying to create EC2 using dynamic inventory

❌ Missing AWS credentials

❌ Wrong AMI for region

❌ SSH blocked in security group

---

### Best Practices

* Use **tags** aggressively
* Separate **infra** and **config** playbooks
* Never hardcode credentials
* Use IAM roles in production

---

### Tooling Summary

| Purpose           | Tool                     |
| ----------------- | ------------------------ |
| Create EC2        | amazon.aws.ec2_instance  |
| Dynamic Inventory | aws_ec2 plugin           |
| Configuration     | Ansible Playbooks        |
| Automation        | Jenkins / GitHub Actions |

---

### Final Notes

This setup is:

* Jenkins friendly
* Scalable
* Production aligned
* Cloud-native

---

Happy Automating 🚀

### Execution flow

```text
Ansible AWS module
      |
      v
EC2 API
      |
      v
EC2 instance created
      |
      v
AWS dynamic inventory
      |
      v
Host discovered
      |
      v
Configuration play
```

### Example

```yaml
plugin: amazon.aws.aws_ec2
regions:
  - us-east-1
filters:
  instance-state-name: running
keyed_groups:
  - key: tags.Name
    prefix: tag
```

### Interviewer may cross-question

**Interviewer:** "Does dynamic inventory create EC2 instances?"

**Candidate:** "No. It discovers existing AWS resources. Provisioning and discovery are separate operations." 

---

## Q35. How does Ansible manage both Windows and Linux servers?

### Short explanation of the question

This tests whether I can manage mixed operating systems without assuming the same connection or module set for both.

### Answer

Ansible can manage Linux and Windows from one controller. Linux commonly uses SSH and Windows commonly uses WinRM, with platform-appropriate modules and tasks.

### Detailed explanation

This guide explains **how a DevOps engineer configures multiple Linux and Windows servers using Ansible in a real production environment**.

Example scenario used in this document:

* 1 Ansible Control Node
* 5 Linux servers
* 5 Windows servers

The key idea is that **Ansible uses different connection protocols for different operating systems**.

| OS      | Protocol Used | Default Port |
| ------- | ------------- | ------------ |
| Linux   | SSH           | 22           |
| Windows | WinRM         | 5985 / 5986  |

---

# 1. Overall Architecture

In production, Ansible runs from a **control node** which connects to target machines.

```
                +-----------------------+
                |   Ansible Controller  |
                +-----------------------+
                   /               \
                  /                 \
               SSH                   WinRM
                |                     |
        +----------------+    +----------------+
        |  Linux Servers |    | Windows Servers|
        | (5 machines)   |    | (5 machines)   |
        +----------------+    +----------------+
```

Linux machines communicate using **SSH**, while Windows machines communicate using **WinRM (Windows Remote Management)**.

---

# 2. Real Production Repository Structure

In real DevOps environments, Ansible code is organized carefully.

```
ansible-infrastructure/

├── inventory/
│   └── production.ini
│
├── playbooks/
│   └── site.yml
│
├── roles/
│   ├── linux_common/
│   │   └── tasks/
│   │       └── main.yml
│   │
│   └── windows_common/
│       └── tasks/
│           └── main.yml
│
├── group_vars/
│   ├── linux.yml
│   └── windows.yml
│
└── ansible.cfg
```

Why this structure is used:

* Easy to maintain
* Supports large infrastructure
* Enables reuse through roles

---

# 3. Inventory Configuration

Inventory defines **which servers Ansible will manage**.

File:

```
inventory/production.ini
```

Example:

```
[linux]
linux1 ansible_host=10.0.1.10
linux2 ansible_host=10.0.1.11
linux3 ansible_host=10.0.1.12
linux4 ansible_host=10.0.1.13
linux5 ansible_host=10.0.1.14

[windows]
win1 ansible_host=10.0.2.10
win2 ansible_host=10.0.2.11
win3 ansible_host=10.0.2.12
win4 ansible_host=10.0.2.13
win5 ansible_host=10.0.2.14
```

Two host groups are created:

* **linux**
* **windows**

---

# 4. Linux Connection Configuration

Linux hosts use **SSH authentication**.

File:

```
group_vars/linux.yml
```

```
ansible_user: ubuntu
ansible_become: true
ansible_ssh_private_key_file: ~/.ssh/id_rsa
```

Explanation:

| Parameter      | Purpose               |
| -------------- | --------------------- |
| ansible_user   | SSH user              |
| ansible_become | Enable sudo           |
| ssh key        | Secure authentication |

---

# 5. Windows Connection Configuration

Windows requires **WinRM configuration**.

File:

```
group_vars/windows.yml
```

Example configuration:

```
ansible_user: Administrator
ansible_password: StrongPassword
ansible_connection: winrm
ansible_port: 5985
ansible_winrm_transport: ntlm
ansible_winrm_server_cert_validation: ignore
```

Explanation:

| Parameter          | Purpose               |
| ------------------ | --------------------- |
| ansible_connection | Use WinRM             |
| ansible_port       | WinRM port            |
| ntlm               | authentication method |

Default ports:

| Port | Protocol    |
| ---- | ----------- |
| 5985 | HTTP WinRM  |
| 5986 | HTTPS WinRM |

---

# 6. Preparing Windows Servers

WinRM must be enabled on each Windows server.

Run PowerShell:

```
winrm quickconfig
```

This command:

* Enables WinRM service
* Opens firewall port
* Creates listener

To enable basic authentication:

```
Set-Item WSMan:\localhost\Service\Auth\Basic $true
```

---

# 7. Linux Role Example

File:

```
roles/linux_common/tasks/main.yml
```

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

This installs and starts **nginx on all Linux servers**.

---

# 8. Windows Role Example

File:

```
roles/windows_common/tasks/main.yml
```

```
---

- name: Install IIS
  win_feature:
    name: Web-Server
    state: present

- name: Start IIS
  win_service:
    name: W3SVC
    state: started
```

This installs the **IIS web server on Windows machines**.

---

# 9. Main Playbook

File:

```
playbooks/site.yml
```

```
---

- name: Configure Linux servers
  hosts: linux
  become: true

  roles:
    - linux_common

- name: Configure Windows servers
  hosts: windows

  roles:
    - windows_common
```

The playbook contains **two plays**.

One for Linux and one for Windows.

---

# 10. Execution Process

Command used:

```
ansible-playbook -i inventory/production.ini playbooks/site.yml
```

Execution flow:

```
Step 1
Ansible reads inventory

Step 2
Hosts grouped (linux/windows)

Step 3
Linux → SSH connection

Step 4
Windows → WinRM connection

Step 5
Roles executed
```

---

# 11. Internal Communication Flow

### Linux Communication

```
Ansible Controller
      |
      | SSH
      v
Linux Host
      |
      v
Execute module
```

### Windows Communication

```
Ansible Controller
      |
      | WinRM
      v
Windows Host
      |
      v
PowerShell module execution
```

---

# 12. Important Ansible Modules

### Linux Modules

| Module  | Purpose          |
| ------- | ---------------- |
| apt     | install packages |
| yum     | install packages |
| service | manage services  |
| copy    | copy files       |
| file    | manage files     |

### Windows Modules

| Module      | Purpose                  |
| ----------- | ------------------------ |
| win_feature | install Windows features |
| win_service | manage services          |
| win_copy    | copy files               |
| win_package | install software         |

---

# 13. Scaling in Real Production

Large companies often manage:

* 100+ Linux servers
* 100+ Windows servers

Maintaining static inventory files becomes difficult in cloud environments because servers are created and destroyed frequently.

To solve this, production environments use **Dynamic Inventory**.

Dynamic inventory automatically pulls server information from cloud platforms like:

* AWS
* Azure
* GCP
* VMware

Instead of manually writing IP addresses in inventory files, Ansible **queries the cloud API** and generates the host list automatically.

---

# 14. Dynamic Inventory Architecture

```
                +----------------------+
                |   Ansible Controller |
                +----------------------+
                          |
                          |
                    Cloud API Query
                          |
                          v
               +-----------------------+
               |  Dynamic Inventory    |
               |  Plugin / Script      |
               +-----------------------+
                    /             \
                   /               \
            Linux Instances     Windows Instances
```

Flow:

1. Ansible runs inventory plugin
2. Plugin queries cloud provider API
3. API returns instance list
4. Ansible groups hosts automatically

---

# 15. Example Dynamic Inventory (AWS)

Production environments commonly use the **AWS EC2 inventory plugin**.

Directory structure:

```
inventory/
  aws_ec2.yml
```

Example configuration:

```
plugin: amazon.aws.aws_ec2
regions:
  - ap-south-1

filters:
  instance-state-name: running

keyed_groups:
  - key: tags.OS
    prefix: os
```

Example EC2 tags:

| Instance | Tag        |
| -------- | ---------- |
| EC2-1    | OS=linux   |
| EC2-2    | OS=windows |

Inventory groups generated automatically:

```
os_linux
os_windows
```

---

# 16. Example Playbook with Dynamic Inventory

```
---

- name: Configure Linux servers
  hosts: os_linux
  become: true

  roles:
    - linux_common

- name: Configure Windows servers
  hosts: os_windows

  roles:
    - windows_common
```

Now whenever a new EC2 instance is launched with tag:

```
OS=linux
```

Ansible **automatically discovers it**.

No manual inventory update required.

---

# 17. Running Dynamic Inventory

Command:

```
ansible-inventory -i inventory/aws_ec2.yml --graph
```

Example output:

```
@all
 |--@os_linux
 |   |--10.0.1.10
 |   |--10.0.1.11
 |
 |--@os_windows
     |--10.0.2.10
     |--10.0.2.11
```

Run playbook:

```
ansible-playbook -i inventory/aws_ec2.yml playbooks/site.yml
```

---

# 18. Benefits of Dynamic Inventory

| Benefit                 | Explanation                        |
| ----------------------- | ---------------------------------- |
| Auto discovery          | New servers detected automatically |
| Cloud integration       | Works with AWS, Azure, GCP         |
| No manual IP management | Uses instance metadata             |
| Scales easily           | Works with thousands of servers    |

---

# 19. Testing Connectivity

Test Linux connectivity:

```
ansible os_linux -i inventory/aws_ec2.yml -m ping
```

Test Windows connectivity:

```
ansible os_windows -i inventory/aws_ec2.yml -m win_ping
```

---

Test Linux connectivity:

```
ansible linux -i inventory/production.ini -m ping
```

Test Windows connectivity:

```
ansible windows -i inventory/production.ini -m win_ping
```

---

# 15. DevOps Interview Answer

If asked:

**How do you manage both Windows and Linux servers using Ansible?**

A good answer is:

Ansible manages Linux servers using SSH and Windows servers using WinRM. We define separate host groups in the inventory and configure connection parameters using group variables. Linux systems use modules like apt or yum while Windows systems use modules such as win_feature and win_service. A single playbook can contain multiple plays targeting different host groups, enabling centralized automation of heterogeneous infrastructure.

### Comparison

| Area | Linux | Windows |
|---|---|---|
| Common connection | SSH | WinRM |
| Typical service module | `ansible.builtin.service` | `ansible.windows.win_service` |

### Example

```yaml
- name: Start nginx on Linux
  ansible.builtin.service:
    name: nginx
    state: started

- name: Start Windows service
  ansible.windows.win_service:
    name: Spooler
    state: started
```

---


---

## Managed Node Readiness Checklist

This is a **corrected, production-ready checklist** to prepare **managed/worker nodes** so they can be safely and reliably managed by an **Ansible control node**.

You can copy this into:

* README.md
* Onboarding docs
* Platform standards
* Audit / compliance notes

---

### 1

Ansible executes modules using Python on managed nodes.

### Minimum Requirement

* Python **2.7** (legacy)
* Python **3.x** (recommended)

```bash
# Check Python
python3 --version || python --version
```

### Install if Missing

**RHEL / CentOS / Rocky / Alma**

```bash
sudo yum install -y python3
```

**Ubuntu / Debian**

```bash
sudo apt update && sudo apt install -y python3
```

📌 If Python is not available, Ansible will fail immediately.

---

### 2

Ansible uses **SSH** for Linux/Unix systems (unless using SSM, WinRM, etc.).

### Generate SSH Key (Control Node)

```bash
ssh-keygen -t rsa -b 4096
```

### Copy Public Key to Managed Node

```bash
ssh-copy-id user@managed-node
```

### Verify Access

```bash
ssh user@managed-node uptime
```

📌 Passwordless SSH is strongly recommended.

---

### 3

Most playbooks require **root-level access**.

### Verify Sudo Access

```bash
sudo -l
```

### Add User to Sudoers

**Ubuntu / Debian**

```bash
sudo usermod -aG sudo user
```

**RHEL / CentOS**

```bash
sudo usermod -aG wheel user
```

### Playbook Usage

```yaml
become: true
become_user: root
```

📌 If `sudo` is blocked or restricted, many modules will fail.

---

### 4

### Disable `requiretty` (If Present)

Some hardened systems enable `requiretty`, which breaks Ansible pipelining.

```bash
sudo visudo
```

Ensure:

```text
Defaults !requiretty
```

📌 Required only if using **pipelining = true**.

---

### 5

These are commonly used by Ansible modules and fact gathering.

**RHEL / CentOS**

```bash
sudo yum install -y sudo tar unzip curl rsync git
```

**Ubuntu / Debian**

```bash
sudo apt install -y sudo tar unzip curl rsync git
```

---

### 6

### Requirements

* SSH port (default **22**) must be reachable **from the control node**
* No inbound internet access required

### Test from Control Node

```bash
nc -zv managed-node-ip 22
```

📌 If SSH is blocked, Ansible cannot manage the node.

---

### 7

Required if playbooks install Python packages.

**Ubuntu / Debian**

```bash
sudo apt install -y python3-pip
```

**RHEL / CentOS**

```bash
sudo yum install -y python3-pip
```

---

### 8

For accurate `gather_facts` results:

```bash
# Ubuntu/Debian
sudo apt install -y lsb-release

# RHEL/CentOS
sudo yum install -y redhat-lsb-core
```

---

### 9

SELinux may block Ansible actions.

### Check Status

```bash
getenforce
```

### Common Fix (If Needed)

```bash
sudo setsebool -P ssh_sysadm_login on
```

📌 Avoid disabling SELinux; fix policies instead.

---

### 1

Incorrect system time can break:

* SSL
* Package installs
* API calls

```bash
timedatectl status
```

---

### Final Readiness Summary

| Requirement       | Status Check           |
| ----------------- | ---------------------- |
| Python installed  | `python3 --version`    |
| SSH access        | `ssh user@host uptime` |
| Sudo access       | `sudo -l`              |
| Utilities present | tar, curl, git, rsync  |
| Firewall          | Port 22 reachable      |
| SELinux (if any)  | `getenforce`           |
| Time sync         | `timedatectl`          |

---

### Interview One‑Liner

> **An Ansible-managed node must have Python installed, allow SSH access from the control node, provide sudo privileges, and permit required system utilities and network connectivity.**

---

### Summary

* Python + SSH are non‑negotiable
* Sudo is required for most automation
* Firewall and SELinux are common failure points
* This checklist prevents 90% of Ansible runtime issues

---
