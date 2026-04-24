# Notification API — OT-Microservices
<p align="center">
  <img src="https://img.shields.io/badge/service-notification--api-blue" />
  <img src="https://img.shields.io/badge/language-python-yellow" />
  <img src="https://img.shields.io/badge/framework-flask-pink" />
  <img src="https://img.shields.io/badge/database-scylladb-green" />
  <img src="https://img.shields.io/badge/search-elasticsearch-orange" />
  <img src="https://img.shields.io/badge/email-smtp%20(mailhog)-navy" />
</p>

| Author       | Created on | Version | Last updated by | Last edited on | Pre Reviewer | L0 Reviewer | L1 Reviewer | L2 Reviewer |
| ------------ | ---------- | ------- | --------------- | -------------- | ------------ | ----------- | ----------- | ----------- |
| Mukesh Kharb | 22/04/2026 | 1.0     | Mukesh Kharb    | 22/04/2026     | Team         | Mohit Kumar | Faisal Khan | Mahesh Kumar| 

---
## 📚 Table of Contents

- [Purpose](#purpose)
- [Pre-requisites](#pre-requisites)
- [System Requirements](#system-requirements)
- [Dependencies](#dependencies)
  - [Build Time Dependency](#build-time-dependency)
  - [Run Time Dependency](#run-time-dependency)
  - [Other Dependency](#other-dependency)
- [Important Ports](#important-ports)
- [Architecture](#architecture)
- [Dataflow Diagram](#dataflow-diagram)
- [Step-by-step Installation of Notification API](#step-by-step-installation-of-notification-api)
  - [Step1: Installation of Software Dependencies](#step1-installation-of-software-dependencies)
  - [Step2: Build / Artifact Generation](#step2-build--artifact-generation)
  - [Step3: Application Deployment](#step3-application-deployment)
- [Monitoring (Health Checks)](#monitoring-health-checks)
- [Logging](#logging)
  - [Application Logs](#application-logs)
  - [System Logs](#system-logs)
  - [Event Logs](#event-logs)
  - [Debugging Commands](#debugging-commands)
- [Disaster Recovery](#disaster-recovery)
  - [Strategy](#strategy)
  - [Database Recovery](#database-recovery)
  - [ElasticSearch Recovery](#elasticsearch-recovery)
- [High Availability](#high-availability)
  - [Fault Tolerance](#fault-tolerance)
- [Troubleshooting](#troubleshooting)
- [FAQs](#faqs)
- [Contact Information](#contact-information)
- [References](#references)

## Purpose

The Notification API is a Python-based microservice designed to automate the generation and delivery of salary notifications to employees.

It performs the following key functions:

* Fetches salary data from upstream services
* Generates salary slips in PDF format
* Sends notifications via email using SMTP

By decoupling notification processing from core business services, this microservice improves scalability, reduces system load, and ensures asynchronous communication.

---

## Pre-requisites

| Requirement                | Description                                      |
|---------------------------|--------------------------------------------------|
| Operating System          | Ubuntu 22.04 or compatible Linux OS              |
| Python                    | Python 3.x installed                             |
| Package Manager           | pip3 available                                   |
| Database                  | ScyllaDB service running and accessible          |
| Search Engine             | ElasticSearch 7.x running on port 9200           |
| SMTP Tool                 | MailHog installed for email testing              |
| Network                   | Connectivity between all microservices           |

---

## System Requirements

| Hardware Specifications | Minimum Recommendation |
| ----------------------- | ---------------------- |
| Processor               | Dual-core              |
| RAM                     | 4GB                    |
| Disk                    | 20GB                   |
| OS                      | Ubuntu 22.04           |

---

## Dependencies

### Build Time Dependency

| Name   | Version | Description         |
| ------ | ------- | ------------------- |
| Python | 3.x     | Runtime environment |
| pip3   | latest  | Package installer   |

### Run Time Dependency

| Name              | Version | Description                        |
| ----------------- | ------- | ---------------------------------- |
| cassandra-driver  | latest  | ScyllaDB connection               |
| elasticsearch     | 7.17.0  | Indexing & search                 |
| emails            | latest  | SMTP email handling               |
| schedule          | latest  | Job scheduling                    |
| pyyaml            | latest  | Config file handling              |

### Other Dependency

| Name          | Version | Description               |
| ------------- | ------- | ------------------------- |
| ScyllaDB      | latest  | Primary database          |
| ElasticSearch | 7.17    | Indexing layer            |
| MailHog       | 1.0.1   | SMTP testing tool         |

---

## Important Ports

| Inbound Traffic | Description      |
| --------------- | ---------------- |
| 5000            | Notification API |
| 9042            | ScyllaDB         |
| 9200            | ElasticSearch    |
| 8025            | MailHog UI       |

| Outbound Traffic | Description |
| ---------------- | ----------- |
| 587              | SMTP Email  |
| 1025             | MailHog SMTP |

---

## Others

| Component   | Description                               |
| ----------- | ----------------------------------------- |
| Config file | config.yaml for environment configuration |
| Logs        | Stored under ~/logs directory             |

---

## Architecture

<img width="1000" height="auto" src="https://github.com/user-attachments/assets/f5689fe3-115f-4d2d-aab5-a57db493ae4d" />

---

## Dataflow Diagram

<img width="1774" height="887" src="https://github.com/user-attachments/assets/af0a0905-f395-417d-8189-8cbc0bf76a99" />

### Explanation

1. Salary data is generated by Salary API  
2. Data is stored in ScyllaDB  
3. Bridge Script syncs data from ScyllaDB to ElasticSearch (only new records using ES exists check, default `notified: False`)  
4. Notification Worker fetches only records where `notified = False`  
5. PDF salary slip is generated  
6. Email is sent via SMTP  
7. After sending email, Worker updates `notified = True` to prevent duplicate emails  

---

## Step-by-step Installation of Notification API

### Step1: Installation of Software Dependencies

#### Build Dependency

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install python3-pip python3-venv -y
python3 -m venv venv
source venv/bin/activate
```

#### Run Time Dependency

```bash
pip3 install cassandra-driver elasticsearch==7.17.0 emails schedule pyyaml
```

#### Other Dependency

```bash
sudo apt-get install elasticsearch -y
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
curl http://localhost:9200
```

Install MailHog:
```bash
wget https://github.com/mailhog/MailHog/releases/download/v1.0.1/MailHog_linux_amd64
chmod +x MailHog_linux_amd64
sudo mv MailHog_linux_amd64 /usr/local/bin/mailhog
nohup mailhog > ~/mailhog.log 2>&1 &
```

---

### Step2: Build / Artifact Generation

```bash
git clone https://github.com/OT-MICROSERVICES/notification-worker.git
cd notification-worker
```

---

### Step3: Application Deployment

Create config file:

```bash
nano config.yaml
```

Example:

```yaml
smtp:
  from: "admin@test.com"
  smtp_server: "127.0.0.1"
  smtp_port: "1025"

elasticsearch:
  host: "127.0.0.1"
  port: 9200
```

Start Bridge Script:

```bash
nohup python3 ~/scylla_to_es_sync.py > ~/sync.log 2>&1 &
```

Start Notification Worker:

```bash
export CONFIG_FILE=$(pwd)/config.yaml
nohup python3 notification_api.py --mode scheduled > ~/notification.log 2>&1 &
```

---

## Monitoring (Health Checks)

| Check         | Command                          | Expected Result |
| ------------- | -------------------------------- | --------------- |
| Sync Logs     | tail -f ~/sync.log               | Sync running    |
| Worker Logs   | tail -f ~/notification.log       | Worker running  |
| ElasticSearch | curl http://localhost:9200       | Cluster info    |
| MailHog UI    | http://<IP>:8025                 | Dashboard       |

---

## Logging

### Application Logs

```bash
tail -f ~/notification.log
grep ERROR ~/notification.log
```

### System Logs

```bash
sudo journalctl -xe
sudo journalctl -u notification.service
cat /var/log/syslog
```

### Event Logs

```bash
grep "Notification" ~/notification.log
grep "PDF" ~/notification.log
```

### Debugging Commands

```bash
lsof -i :5000
netstat -tulnp | grep 5000
ps aux | grep notification
```

---

## Disaster Recovery

In case of failures, recovery strategies should be in place.

### Strategy

* Backup ScyllaDB data using snapshots  
* Backup Elasticsearch indices  

### Database Recovery

```bash
nodetool snapshot
nodetool refresh
```

### ElasticSearch Recovery

```bash
curl -X PUT "localhost:9200/_snapshot/backup/snap1"
curl -X POST "localhost:9200/_snapshot/backup/snap1/_restore"
```

---

## High Availability

To ensure high availability:

* Deploy multiple worker instances  
* Keep application stateless  
* Use database and search clustering  

### Fault Tolerance

* Worker retries prevent data loss  
* Elasticsearch ensures data buffering  

---

## Troubleshooting

| Issue                       | Cause              | Solution              |
| --------------------------- | ------------------ | --------------------- |
| ModuleNotFoundError         | Missing dependency | pip3 install module   |
| Duplicate Emails            | Logic issue        | Check notified flag   |
| ElasticSearch not reachable | Service down       | Restart elasticsearch |
| MailHog not accessible      | Port blocked       | Open port 8025        |

---

## FAQs

* What is the purpose of the Notification API?  
  > It generates salary PDFs and sends them via email.

* Is the Notification API stateful?  
  > No, it is stateless.

* How can I verify the service is running?  
  > Check logs or MailHog UI.

* Which database is used?  
  > ScyllaDB.

* How are duplicate emails prevented?  
  > Using ES exists check and notified flag.

---

## Contact Information

| Name   | Email                                                                             |
| ------ | --------------------------------------------------------------------------------- |
| Mukesh | mukesh.Kharb.snaatak@mygurukulam.co |

---

## References

* OT-Microservices Documentation Template
