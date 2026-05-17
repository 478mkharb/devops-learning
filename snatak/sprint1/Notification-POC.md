# <h1 align="center">POC - Notification Integration using MailHog & Slack | OT-Microservices </h1>

<div align="center">
<img width="100" alt="Python" src="https://www.python.org/static/community_logos/python-logo.png" />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="120" alt="MailHog" src="https://github.com/mailhog/MailHog/raw/master/docs/images/hog.png" />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="100" alt="Slack" src="https://upload.wikimedia.org/wikipedia/commons/d/d5/Slack_icon_2019.svg" />
</div>

<br/>

---

<div align="center">

<table>
  <tr>
    <th align="center">Author</th>
    <th align="center">Created On</th>
    <th align="center">Version</th>
    <th align="center">Last Updated By</th>
    <th align="center">Last Edited On</th>
    <th align="center">Pre Reviewer</th>
    <th align="center">L0 Reviewer</th>
    <th align="center">L1 Reviewer</th>
    <th align="center">L2 Reviewer</th>
  </tr>

  <tr>
    <td align="center">Mukesh Kharb</td>
    <td align="center">16/05/2026</td>
    <td align="center">1.0</td>
    <td align="center">Mukesh Kharb</td>
    <td align="center">16/05/2026</td>
    <td align="center">Team</td>
    <td align="center">Mohit Kumar</td>
    <td align="center">Faisal Khan</td>
    <td align="center">Mahesh Kumar</td>
  </tr>
</table>

</div>

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Pre-requisites](#2-pre-requisites)
3. [Install MailHog](#3-install-mailhog)
4. [Create Slack Webhook](#4-create-slack-webhook)
5. [Update config.yaml](#5-update-configyaml)
6. [Update notification.py](#6-update-notificationpy)
7. [Run Notification Service](#7-run-notification-service)
8. [Verify Email Notifications](#8-verify-email-notifications)
9. [Verify Slack Notifications](#9-verify-slack-notifications)
10. [Expected Results](#10-expected-results)
11. [FAQs](#11-faqs)
12. [Contact Information](#12-contact-information)
13. [References](#13-references)

---

<a id="1-introduction"></a>

## 1. Introduction

This POC demonstrates implementation of Email and Slack notifications in OT-Microservices using Python Notification Service, MailHog SMTP, and Slack Incoming Webhooks.

The notification worker fetches records from ElasticSearch and sends notifications through:

* MailHog SMTP for Email Notifications
* Slack Incoming Webhooks for Slack Alerts

---

<a id="2-pre-requisites"></a>

## 2. Pre-requisites

| Requirement     | Version / Details        |
| --------------- | ------------------------ |
| Ubuntu          | 22.04                    |
| Python          | 3.11+                    |
| pip             | Latest                   |
| ElasticSearch   | 7.x                      |
| MailHog         | v1.0.1                   |
| Slack Workspace | Active Workspace         |
| Slack App       | Incoming Webhook Enabled |

---

<a id="3-install-mailhog"></a>

## 3. Install MailHog

```bash
wget https://github.com/mailhog/MailHog/releases/download/v1.0.1/MailHog_linux_amd64
chmod +x MailHog_linux_amd64
sudo mv MailHog_linux_amd64 /usr/local/bin/mailhog
```

Start MailHog:

```bash
nohup mailhog > ~/mailhog.log 2>&1 &
```

Verify MailHog:

```bash
ss -tulpn | grep 1025
ss -tulpn | grep 8025
```

Access MailHog UI:

```text
http://<VM-IP>:8025
```

<details>
<summary>📸 <strong>Click to view Screenshot - MailHog UI</strong></summary>

<!-- Add Screenshot Here -->

</details>

---

<a id="4-create-slack-webhook"></a>

## 4. Create Slack Webhook

### Create Slack App

* Open Slack API Portal
* Click Create New App
* Select From Scratch
* Enter App Name
* Select Workspace

### Enable Incoming Webhook

* Open Incoming Webhooks
* Enable Activate Incoming Webhooks
* Click Add New Webhook to Workspace
* Select Slack Channel
* Allow Permissions

Copy generated Webhook URL.

Example:

```text
https://hooks.slack.com/services/XXXX/YYYY/ZZZZ
```

<details>
<summary>📸 <strong>Click to view Screenshot - Slack Webhook</strong></summary>

<!-- Add Screenshot Here -->

</details>

---

<a id="5-update-configyaml"></a>

## 5. Update config.yaml

```yaml
smtp:
  from: notifications@otmicro.local
  smtp_server: localhost
  smtp_port: 1025

slack:
  webhook_url: "https://hooks.slack.com/services/XXXX/YYYY/ZZZZ"
```

> [!NOTE]
> MailHog does not require SMTP authentication or TLS configuration.

---

<a id="6-update-notificationpy"></a>

## 6. Update notification.py

### Install Required Python Packages

```bash
pip3 install requests emails elasticsearch schedule
```

### Add Slack Library

```python
import requests
```

### Add Slack Notification Function

```python
def send_slack_notification(message):
    config_content = read_configuration()

    payload = {
        "text": message
    }

    requests.post(
        config_content.getProperty("slack.webhook_url"),
        json=payload,
        timeout=5
    )
```

### Trigger Slack Notification

```python
send_slack_notification(
    f"Salary slip notification sent to {email_id}"
)
```

### Updated MailHog SMTP Configuration

```python
smtp={
    "host": config_content.getProperty("smtp.smtp_server"),
    "port": config_content.getProperty("smtp.smtp_port")
}
```

---

<a id="7-run-notification-service"></a>

## 7. Run Notification Service

```bash
python3 notification.py
```

Run in background:

```bash
nohup python3 notification.py > notification.log 2>&1 &
```

Verify Process:

```bash
ps aux | grep notification
```

---

<a id="8-verify-email-notifications"></a>

## 8. Verify Email Notifications

* Trigger notification workflow
* Open MailHog UI
* Verify emails are received

MailHog UI:

```text
http://<VM-IP>:8025
```

<details>
<summary>📸 <strong>Click to view Screenshot - Email Notification</strong></summary>

<!-- Add Screenshot Here -->

</details>

---

<a id="9-verify-slack-notifications"></a>

## 9. Verify Slack Notifications

* Trigger notification workflow
* Open configured Slack Channel
* Verify alert message is delivered

Expected Slack Message:

```text
Salary slip notification sent successfully
```

<details>
<summary>📸 <strong>Click to view Screenshot - Slack Notification</strong></summary>

<!-- Add Screenshot Here -->

</details>

---

<a id="10-expected-results"></a>

## 10. Expected Results

| Component                   | Expected Result                           |
| --------------------------- | ----------------------------------------- |
| MailHog SMTP                | Emails visible in MailHog UI              |
| Slack Webhook               | Notifications delivered to Slack channel  |
| Python Notification Service | Notifications processed successfully      |
| ElasticSearch               | Notification records fetched successfully |

---

<a id="11-faqs"></a>

## 11. FAQs

### Q1. Why is MailHog used in this POC?

**Answer:**
MailHog allows safe local email testing without sending emails to real users.

### Q2. Why are Slack Incoming Webhooks used?

**Answer:**
Incoming Webhooks provide a simple method to send automated alerts to Slack channels.

### Q3. Does MailHog require SMTP authentication?

**Answer:**
No. MailHog works locally without SMTP authentication or TLS.

### Q4. What happens if Slack Webhook URL is invalid?

**Answer:**
Slack notifications fail and errors appear in notification service logs.

### Q5. Can both Email and Slack notifications run together?

**Answer:**
Yes. The Python notification service supports multiple notification channels.

### Q6. Why is ElasticSearch used in notification flow?

**Answer:**
ElasticSearch stores searchable records used by the notification worker.

---

<a id="12-contact-information"></a>

## 12. Contact Information

| Name         | ✉️ Contact                                                                        |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

<a id="13-references"></a>

## 13. References

| S.No | Description                 | Click to View                                                                                                                                                                       |
| ---- | --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | MailHog Documentation       | [![MailHog](https://img.shields.io/badge/MAILHOG-DOCUMENTATION-2F2F2F?style=flat-square\&logo=maildotru\&logoColor=white)](https://github.com/mailhog/MailHog)                      |
| 2    | Slack Incoming Webhooks     | [![Slack](https://img.shields.io/badge/SLACK-WEBHOOKS-3A3A3A?style=flat-square\&logo=slack\&logoColor=white)](https://api.slack.com/messaging/webhooks)                             |
| 3    | Python Emails Library       | [![Python Emails](https://img.shields.io/badge/PYTHON-EMAILS-404040?style=flat-square\&logo=python\&logoColor=white)](https://github.com/lavr/python-emails)                        |
| 4    | Python Requests Library     | [![Requests](https://img.shields.io/badge/PYTHON-REQUESTS-2B2B2B?style=flat-square\&logo=python\&logoColor=white)](https://requests.readthedocs.io/)                                |
| 5    | ElasticSearch Documentation | [![ElasticSearch](https://img.shields.io/badge/ELASTICSEARCH-DOCUMENTATION-1F1F1F?style=flat-square\&logo=elasticsearch\&logoColor=white)](https://www.elastic.co/guide/index.html) |
