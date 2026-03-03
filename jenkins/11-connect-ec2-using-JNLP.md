## 🔗 How Can You Connect an AWS EC2 Instance to Jenkins Using JNLP?

This explains **how JNLP (inbound agent) connection works** and **how to connect an EC2 instance to Jenkins step by step**, including **what happens internally**.

---

## 🧠 What is JNLP in Jenkins?

**JNLP (Java Network Launch Protocol)** in Jenkins means:

> 👉 The **agent initiates the connection to the Jenkins controller**, not the other way around.

This is called an **inbound agent**.

---

## ❓ Why Use JNLP for AWS EC2?

JNLP is preferred when:

* Jenkins controller **cannot SSH into EC2**
* EC2 is in **private subnet**
* Firewall / NAT restrictions exist
* Dynamic or auto-scaled agents are used

---

## 🏗️ High-Level Architecture

```
AWS EC2 (Agent)
   |
   |  JNLP (Outbound)
   v
Jenkins Controller
```

✔️ EC2 connects **outbound** → easier networking

---

## 🔑 Prerequisites

* Jenkins controller reachable via HTTPS
* EC2 has:

  * Java installed (Java 11+)
  * Network access to Jenkins URL
* Jenkins credentials for agent (secret)

---

## 1️⃣ Step 1: Create a JNLP Agent in Jenkins

### In Jenkins UI

```
Manage Jenkins → Nodes → New Node
```

Configure:

* Node name: `ec2-jnlp-agent`
* Type: Permanent Agent
* Remote root directory: `/home/jenkins`
* Launch method: **Launch agent by connecting it to the controller (JNLP)**

Save the node.

---

## 2️⃣ Step 2: Get JNLP Command & Secret

After saving the node, Jenkins shows:

* Agent secret
* JNLP connection command

Example:

```
java -jar agent.jar \
  -url https://jenkins.example.com \
  -secret abc123 \
  -name ec2-jnlp-agent
```

👉 This command is **unique per agent**.

---

## 3️⃣ Step 3: Prepare the EC2 Instance

### Install Java

```
sudo yum install java-11 -y   # Amazon Linux
```

### Download agent.jar

```
curl -O https://jenkins.example.com/jnlpJars/agent.jar
```

---

## 4️⃣ Step 4: Start JNLP Agent on EC2

Run:

```
java -jar agent.jar \
  -url https://jenkins.example.com \
  -secret abc123 \
  -name ec2-jnlp-agent
```

Once connected:

* Node shows **ONLINE** in Jenkins
* Executors become available

---

## 5️⃣ Step 5: Make Agent Persistent (Systemd)

Create a service:

```
/etc/systemd/system/jenkins-agent.service
```

```
[Unit]
Description=Jenkins JNLP Agent
After=network.target

[Service]
User=ec2-user
ExecStart=/usr/bin/java -jar /home/ec2-user/agent.jar \
  -url https://jenkins.example.com \
  -secret abc123 \
  -name ec2-jnlp-agent
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable it:

```
sudo systemctl daemon-reexec
sudo systemctl enable jenkins-agent
sudo systemctl start jenkins-agent
```

---

## 🔄 Internal Flow (Interview Important)

```
Pipeline needs executor
   ↓
JNLP Agent already connected
   ↓
Controller assigns job
   ↓
Agent runs steps
   ↓
Logs streamed back
```

---

## 🔐 Security Considerations

* Use HTTPS only
* Rotate agent secrets
* Restrict outbound access
* Prefer IAM roles (no creds on EC2)

---

## 🆚 JNLP vs SSH (Quick Compare)

| Aspect            | JNLP               | SSH                |
| ----------------- | ------------------ | ------------------ |
| Who initiates     | Agent → Controller | Controller → Agent |
| Firewall friendly | ✅ Yes              | ❌ No               |
| Cloud friendly    | ✅ Yes              | ⚠️ Sometimes       |
| Setup complexity  | Medium             | Easy               |

---

## 🎤 Interview Golden Lines

* 🔗 *JNLP agents connect inbound to Jenkins controller*
* ☁️ *Ideal for EC2 in private subnets*
* 🔐 *More firewall-friendly than SSH*

---

## 💡 Pro Interview Tip

> “Most cloud-based Jenkins setups use JNLP or Kubernetes agents, not SSH.”

---
Just say 👍
