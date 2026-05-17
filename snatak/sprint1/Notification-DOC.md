# <h1 align="center">Documentation - Slack & Email Notification Integration </h1>

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
2. [What are Slack and Email Notifications](#2-what-are-slack-and-email-notifications)
3. [Why Notification Integration is Required](#3-why-notification-integration-is-required)
4. [Notification Components](#4-notification-components)
5. [Advantages](#5-advantages)
6. [Best Practices](#6-best-practices)
7. [Recommendation / Conclusion](#7-recommendation--conclusion)
8. [FAQs](#8-faqs)
9. [Contact Information](#8-contact-information)
10. [References](#9-references)

---

<a id="1-introduction"></a>

## 1. Introduction

Slack and Email Notifications help OT-Microservices provide centralized communication for salary notifications and operational alerts. The existing Notification API uses ElasticSearch, a scheduled Python worker, MailHog SMTP, and Slack Webhooks for automated notification delivery.

> [!NOTE]
> The implementation uses the existing `notification.py` service with minimal code changes to support both Slack and Email notification delivery.

[![Click Here for POC](https://img.shields.io/badge/CLICK_HERE-FOR_POC-2F2F2F?style=flat-square&logo=github&logoColor=white)](https://github.com/Snaatak-Infra-Titans/Documentations/blob/SCRUM-141-mukesh/Application_CI_Design/Generic_CI_Operations/Notifications/POC/README.md)
---

<a id="2-what-are-slack-and-email-notifications"></a>

## 2. What are Slack and Email Notifications

Slack and Email Notifications are automated communication mechanisms used to notify users about application events.

### Email and Slack Notification Flow
<img width="1000" height="350" alt="notification" src="https://github.com/user-attachments/assets/7fbefad6-2045-479a-b281-0ca80edb1649" />


### Common Notification Use Cases

| Notification Type     | Example                                |
| --------------------- | -------------------------------------- |
| Salary Notification   | Salary slip generated successfully     |
| Employee Notification | Employee onboarding email              |


---

<a id="3-why-notification-integration-is-required"></a>

## 3. Why Notification Integration is Required

| Requirement               | Description                               |
| ------------------------- | ----------------------------------------- |
| Centralized Communication | Standardizes notification handling        |
| Real-time Visibility      | Slack alerts appear instantly             |
| Automated Email Delivery  | Eliminates manual communication           |
| Asynchronous Processing   | Notification worker runs independently    |
| Better Monitoring         | Notification logs improve observability   |
| Easy Testing              | MailHog simplifies SMTP validation        |
| Scalable Architecture     | Supports future notification integrations |

---

<a id="4-notification-components"></a>

## 4. Notification Components

| Component         | Purpose                             |
| ----------------- | ----------------------------------- |
| notification.py   | Main scheduled notification worker  |
| ElasticSearch     | Stores employee records             |
| MailHog           | SMTP testing service                |
| Slack Webhook     | Sends Slack channel notifications   |
| config.yaml       | Stores notification configuration   |
| Scheduler         | Executes notifications periodically |
| Logging Framework | Tracks notification execution       |

---

<a id="5-advantages"></a>

## 5. Advantages

| Advantage                  | Description                                      |
| -------------------------- | ------------------------------------------------ |
| Minimal Code Changes       | Reuses existing microservice architecture        |
| Easy Email Testing         | MailHog enables safe SMTP testing                |
| Instant Notifications      | Slack alerts are delivered in real time          |
| Decoupled Processing       | Notifications run independently from core APIs   |
| Centralized Notification   | Single worker manages all notification channels  |

---

<a id="6-best-practices"></a>

## 6. Best Practices

| Best Practice               | Description                          |
| --------------------------- | ------------------------------------ |
| Use Environment Variables   | Store configuration securely         |
| Avoid Hardcoded Secrets     | Protect Slack webhook URLs           |
| Centralize Logging          | Maintain notification logs           |
| Use Scheduled Workers       | Prevent blocking operations          |
| Validate SMTP Connectivity  | Ensure MailHog is running            |
| Use Separate Slack Channels | Organize notifications properly      |
| Handle Exceptions Properly  | Prevent notification worker failures |

---

<a id="7-recommendation--conclusion"></a>

## 7. Recommendation / Conclusion

The recommended approach is to extend the existing `notification.py` service with Slack Webhooks and MailHog SMTP support instead of creating a separate notification system. This keeps the architecture simple, centralized, scalable, and suitable for both POC and future production implementations.

---

## 8. FAQs

### Q1. Why is MailHog used in the notification system?

**Answer:**
MailHog provides SMTP email testing without using real email services during POC validation.

### Q2. Why are Slack Webhooks used?

**Answer:**
Slack Webhooks enable real-time notification delivery directly to Slack channels.

### Q3. Were major architectural changes required for Slack integration?

**Answer:**
No. The existing notification worker, scheduler, logging, and ElasticSearch integration were reused.

### Q4. Why is MailHog used instead of a real email service in POC?

**Answer:**
MailHog allows safe local email testing without sending emails to real users or external SMTP providers.

### Q5. What is the role of ElasticSearch in the notification workflow?

**Answer:**
ElasticSearch stores employee records which are fetched by the notification worker.

### Q6. Can this implementation be extended for production environments?

**Answer:**
Yes. MailHog can later be replaced with enterprise SMTP services while keeping the same notification architecture.

---

<a id="8-contact-information"></a>

## 8. Contact Information

| Name         | ✉️ Contact                                                                        |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

<a id="9-references"></a>

## 9. References

| S.No | Description           | Click to View                                                                                                                                                                       |
| ---- | --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | Python Documentation  | [![PYTHON DOCS](https://img.shields.io/badge/PYTHON-DOCUMENTATION-2F2F2F?style=flat-square\&logo=python\&logoColor=white)](https://docs.python.org/3/)                              |
| 2    | Slack Webhooks        | [![SLACK WEBHOOKS](https://img.shields.io/badge/SLACK-WEBHOOKS-2F2F2F?style=flat-square\&logo=slack\&logoColor=white)](https://api.slack.com/messaging/webhooks)                    |
| 3    | MailHog Repository    | [![MAILHOG REPO](https://img.shields.io/badge/MAILHOG-REPOSITORY-2F2F2F?style=flat-square\&logo=maildotru\&logoColor=white)](https://github.com/mailhog/MailHog)                    |
| 4    | ElasticSearch Docs    | [![ELASTICSEARCH](https://img.shields.io/badge/ELASTICSEARCH-DOCUMENTATION-2F2F2F?style=flat-square\&logo=elasticsearch\&logoColor=white)](https://www.elastic.co/guide/index.html) |
| 5    | Python Emails Library | [![PYTHON EMAILS](https://img.shields.io/badge/PYTHON-EMAILS_LIBRARY-2F2F2F?style=flat-square\&logo=python\&logoColor=white)](https://github.com/lavr/python-emails)                |
| 6    | Slack API Docs        | [![SLACK API DOCS](https://img.shields.io/badge/SLACK-API_DOCUMENTATION-2F2F2F?style=flat-square\&logo=slack\&logoColor=white)](https://api.slack.com/)                             |
| 7    | MailHog SMTP Tool     | [![MAILHOG SMTP](https://img.shields.io/badge/MAILHOG-SMTP_TOOL-2F2F2F?style=flat-square\&logo=maildotru\&logoColor=white)](https://github.com/mailhog/MailHog)                     |
