# <h1 align="center">POC - Notification Integration using MailHog & Slack | OT-Microservices </h1>

<div align="center">
<img width="50" alt="Python" src="https://upload.wikimedia.org/wikipedia/commons/c/c3/Python-logo-notext.svg" />
<img width="50" alt="Slack" src="https://upload.wikimedia.org/wikipedia/commons/d/d5/Slack_icon_2019.svg" />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="80" alt="mailhog" alt="mailhog" src="https://github.com/user-attachments/assets/6bb008e3-6d52-45c9-af65-1f62cfd04bf1" />
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
    <td align="center">17/05/2026</td>
    <td align="center">1.0</td>
    <td align="center">Mukesh Kharb</td>
    <td align="center">17/05/2026</td>
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
7. [Trigger and Run Notification Worker](#7-trigger-and-run-notification-worker)
8. [Verify Email and Slack Notifications](#8-verify-email-and-slack-notifications)
9. [Expected Results](#9-expected-results)
10. [FAQs](#10-faqs)
11. [Contact Information](#11-contact-information)
12. [References](#12-references)

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
| ElasticSearch   | 7.8.0                    |
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
ss -tulpn | grep 8025
ss -tulpn | grep 1025
```

Access MailHog UI:

```text
http://<VM-IP>:8025
```

<details>
<summary>📸 <strong>Click to view Screenshot - MailHog UI</strong></summary>
<img width="1542" height="788" alt="image" src="https://github.com/user-attachments/assets/291218be-4ad6-4905-af7c-e635f0283415" />

<img width="1542" height="709" alt="image" src="https://github.com/user-attachments/assets/373faca6-2162-4f22-988e-41e7f1a8e599" />


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

<details>
<summary>📸 <strong>Click to view Screenshot - Slack Webhook</strong></summary>

<img width="1836" height="990" alt="image" src="https://github.com/user-attachments/assets/62355585-da2b-4446-9754-eec3e2dff29e" />


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

<details>
<summary>📸 <strong>Click to view Screenshot - Update config.yaml</strong></summary>
<img width="1519" height="555" alt="image" src="https://github.com/user-attachments/assets/035b273e-a485-450b-a538-fab0b35ae2c6" />


</details>

---

<a id="6-update-notificationpy"></a>

## 6. Update notification_api.py

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
<details>
<summary>📸 <strong>Click to view - Updated notification_api.py</strong></summary>

```python
#!/usr/bin/python3
# pylint: disable=invalid-name,broad-except

"""
Notification Service
Author:- Opstree Solutions

This service:
- Fetches employee records from ElasticSearch
- Sends Email notifications using MailHog SMTP
- Sends Slack notifications using Incoming Webhooks
"""

import argparse
import os
import sys
import logging
import time
import requests
import emails
import schedule
import config_with_yaml as config

from elasticsearch import Elasticsearch

CONFIG_FILE = os.environ.get("CONFIG_FILE")

FORMATTER = logging.Formatter(
    "%(asctime)s — %(name)s — %(levelname)s — %(message)s"
)


def init_logger():
    """Initialize logger"""
    console_handler = logging.StreamHandler(sys.stdout)
    console_handler.setFormatter(FORMATTER)
    return console_handler


def get_logger():
    """Return logger instance"""
    logger = logging.getLogger("notification-service")

    if not logger.handlers:
        logger.setLevel(logging.DEBUG)
        logger.addHandler(init_logger())

    return logger


def read_configuration():
    """Read configuration file"""
    logger = get_logger()

    try:
        cfg = config.load(CONFIG_FILE)
        return cfg

    except Exception as error:
        logger.error(
            "Unable to parse configuration file: %s",
            error
        )
        return None


def send_slack_notification(message):
    """Send notification to Slack channel"""

    logger = get_logger()
    config_content = read_configuration()

    try:
        webhook_url = config_content.getProperty(
            "slack.webhook_url"
        )

        payload = {
            "text": message
        }

        response = requests.post(
            webhook_url,
            json=payload,
            timeout=5
        )

        if response.status_code == 200:
            logger.info(
                "Slack notification sent successfully"
            )
        else:
            logger.error(
                "Slack notification failed: %s",
                response.text
            )

    except Exception as error:
        logger.error(
            "Unable to send Slack notification: %s",
            error
        )


def send_mail(email_id):
    """Send email notification"""

    logger = get_logger()
    config_content = read_configuration()

    try:
        message = emails.html(
            html="""
            <strong>
            Your salary slip has been generated successfully.
            </strong>
            """,
            subject="Salary Slip Notification",
            mail_from=config_content.getProperty(
                "smtp.from"
            ),
        )

        message.send(
            to=email_id,
            smtp={
                "host": config_content.getProperty(
                    "smtp.smtp_server"
                ),
                "port": config_content.getProperty(
                    "smtp.smtp_port"
                ),
            },
        )

        logger.info(
            "Email notification sent to %s",
            email_id
        )

        send_slack_notification(
            f"Salary slip notification sent to {email_id}"
        )

    except Exception as error:
        logger.error(
            "Unable to send email notification: %s",
            error
        )


def send_mail_to_all_users():
    """Fetch users from ElasticSearch"""

    logger = get_logger()
    config_content = read_configuration()

    try:
        es_client = Elasticsearch(
            [config_content.getProperty(
                "elasticsearch.host"
            )],
            http_auth=(
                config_content.getProperty(
                    "elasticsearch.username"
                ),
                config_content.getProperty(
                    "elasticsearch.password"
                ),
            ),
            scheme="http",
            port=config_content.getProperty(
                "elasticsearch.port"
            ),
        )

        result = es_client.search(
            index="employee-management",
            body={
                "query": {
                    "match_all": {}
                }
            }
        )

        for data in result["hits"]["hits"]:
            send_mail(
                data["_source"]["email_id"]
            )

    except Exception as error:
        logger.error(
            "ElasticSearch query execution failed: %s",
            error
        )


def schedule_operation():
    """Run notification scheduler"""

    logger = get_logger()

    schedule.every().hour.do(
        send_mail_to_all_users
    )

    while True:
        logger.info(
            "Waiting for scheduled notification execution"
        )

        schedule.run_pending()
        time.sleep(1)


if __name__ == "__main__":

    parser = argparse.ArgumentParser()

    parser.add_argument(
        "-m",
        "--mode",
        help="""
        Application mode:
        scheduled or external
        """,
        default="scheduled"
    )

    args = parser.parse_args()

    if args.mode == "scheduled":
        schedule_operation()

    else:
        send_mail_to_all_users()
```
</details> 

---

<a id="7-trigger-and-run-notification-worker"></a>

## 7. Trigger and Run Notification Worker

* Trigger notification workflow
```bash
curl -X POST "http://localhost:9200/employee-management/_doc/1" \
-H 'Content-Type: application/json' \
-d '
{
  "employee_name": "Mukesh",
  "email_id": "mukesh@test.com"
}'
```
* Verify
```bash
curl -X GET "http://localhost:9200/employee-management/_search?pretty"
```
* Run Notification worker again

```bash
export CONFIG_FILE=config.yaml
python3 notification_api.py -m external
```
<details>
<summary>📸 <strong>Click to view Screenshot - Trigger and Run Notification Worker</strong></summary>
<img width="1500" height="244" alt="image" src="https://github.com/user-attachments/assets/8fece967-59fd-43fc-9b67-32aecc4b3182" />
<img width="1500" height="729" alt="image" src="https://github.com/user-attachments/assets/d879c3af-ffd6-4907-acd5-84287870853d" />
<img width="1500" height="290" alt="image" src="https://github.com/user-attachments/assets/c4026cd4-40ff-475d-828e-1fbc9da38515" />

</details>

---

<a id="8-verify-email-and-slack-notifications"></a>

## 8. Verify Email and Slack Notifications

### MailHog UI:

```text
http://<VM-IP>:8025
```

<details>
<summary>📸 <strong>Click to view Screenshot - Email Notification</strong></summary>

<img width="1842" height="974" alt="image" src="https://github.com/user-attachments/assets/c0471fe5-8d8a-4518-9155-5257faf1852f" />

</details>

### Open Slack 

<details>
<summary>📸 <strong>Click to view Screenshot - Slack Notification</strong></summary>

<img width="1842" height="974" alt="image" src="https://github.com/user-attachments/assets/0efd8277-b453-472b-afd7-56df762b7de2" />

</details>

---

<a id="9-expected-results"></a>

## 9. Expected Results

| Component                   | Expected Result                           |
| --------------------------- | ----------------------------------------- |
| MailHog SMTP                | Emails visible in MailHog UI              |
| Slack Webhook               | Notifications delivered to Slack channel  |
| Python Notification Service | Notifications processed successfully      |
| ElasticSearch               | Notification records fetched successfully |

---

<a id="10-faqs"></a>

## 10. FAQs

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

<a id="11-contact-information"></a>

## 11. Contact Information

| Name         | ✉️ Contact                                                                        |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

<a id="12-references"></a>

## 12. References

| S.No | Description                 | Click to View                                                                                                                                                                       |
| ---- | --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | MailHog Documentation       | [![MailHog](https://img.shields.io/badge/MAILHOG-DOCUMENTATION-2F2F2F?style=flat-square\&logo=maildotru\&logoColor=white)](https://github.com/mailhog/MailHog)                      |
| 2    | Slack Incoming Webhooks     | [![Slack](https://img.shields.io/badge/SLACK-WEBHOOKS-3A3A3A?style=flat-square\&logo=slack\&logoColor=white)](https://api.slack.com/messaging/webhooks)                             |
| 3    | Python Emails Library       | [![Python Emails](https://img.shields.io/badge/PYTHON-EMAILS-404040?style=flat-square\&logo=python\&logoColor=white)](https://github.com/lavr/python-emails)                        |
| 4    | Python Requests Library     | [![Requests](https://img.shields.io/badge/PYTHON-REQUESTS-2B2B2B?style=flat-square\&logo=python\&logoColor=white)](https://requests.readthedocs.io/)                                |
| 5    | ElasticSearch Documentation | [![ElasticSearch](https://img.shields.io/badge/ELASTICSEARCH-DOCUMENTATION-1F1F1F?style=flat-square\&logo=elasticsearch\&logoColor=white)](https://www.elastic.co/guide/index.html) |
