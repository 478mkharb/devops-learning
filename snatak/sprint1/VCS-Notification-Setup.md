# <h1 align="center">Documentation - VCS Notifications | Slack & Email Alerts </h1>

<div align="center">
<img width="80" alt="GitHub" src="https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png" />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="80" alt="Slack" src="https://cdn-icons-png.flaticon.com/512/2111/2111615.png" />
<img width="80" alt="Email" src="https://cdn-icons-png.flaticon.com/512/732/732200.png" />
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
    <td align="center">21/05/2026</td>
    <td align="center">1.0</td>
    <td align="center">Mukesh Kharb</td>
    <td align="center">21/05/2026</td>
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
2. [Purpose of VCS Notifications](#2-purpose-of-vcs-notifications)
3. [Supported Notification Events](#3-supported-notification-events)
4. [Architecture Workflow](#4-architecture-workflow)
5. [Prerequisites](#5-prerequisites)
6. [Step-by-step Setup Guide](#6-step-by-step-setup-guide)
   * [6.1 Install GitHub Slack App](#61-install-github-slack-app)
   * [6.2 Connect GitHub Account](#62-connect-github-account)
   * [6.3 Invite GitHub Bot to Slack Channel](#63-invite-github-bot-to-slack-channel)
   * [6.4 Configure Repository Notifications](#64-configure-repository-notifications)
   * [6.5 Configure Email Notifications](#65-configure-email-notifications)
   * [6.6 Verify Notifications](#66-verify-notifications)
7. [Advantages](#7-advantages)
8. [Best Practices](#8-best-practices)
9. [Conclusion](#9-conclusion)
10. [Contact Information](#10-contact-information)
11. [References](#11-references)

---

<a id="1-introduction"></a>

## 1. Introduction

Version Control System (VCS) notifications provide real-time visibility into repository activities such as Pull Request creation, commits, branch operations, and merge events.

GitHub notifications can be integrated with Slack channels and Email alerts to improve collaboration, CI/CD monitoring, and deployment visibility.

---

<a id="2-purpose-of-vcs-notifications"></a>

## 2. Purpose of VCS Notifications

| Purpose              | Description                              |
| -------------------- | ---------------------------------------- |
| Real-time Monitoring | Tracks repository activities instantly   |
| Faster Collaboration | Helps teams review changes quickly       |
| Deployment Awareness | Alerts teams about production merges     |
| Audit Visibility     | Maintains repository activity visibility |
| CI/CD Integration    | Enables automated workflow awareness     |
| Incident Tracking    | Helps identify risky changes quickly     |

---

<a id="3-supported-notification-events"></a>

## 3. Supported Notification Events

| Event                  | Description                            |
| ---------------------- | -------------------------------------- |
| Pull Request Opened    | PR creation notification               |
| Pull Request Updated   | New commits pushed to PR               |
| Pull Request Commented | Reviewer comments                      |
| Pull Request Merged    | Merge completion notification          |
| Commit Push            | Push notifications                     |
| Branch Created         | Branch creation alerts                 |
| Branch Deleted         | Branch deletion alerts                 |
| Email Notification     | Email alerts for repository activities |

---

<a id="4-architecture-workflow"></a>

## 4. Architecture Workflow

> <img width="1000" alt="workflow" src="https://github.com/user-attachments/assets/6f8f21c7-1a84-4b54-b4b8-6f9f6bce6f90" />

---

<a id="5-prerequisites"></a>

## 5. Prerequisites

| Requirement       | Description                              |
| ----------------- | ---------------------------------------- |
| GitHub Repository | Repository access with admin permissions |
| Slack Workspace   | Slack channel for notifications          |
| Internet Access   | Required for GitHub-Slack communication  |
| GitHub Account    | Required for repository subscriptions    |
| Email Access      | Required for GitHub email notifications  |

---

<a id="6-step-by-step-setup-guide"></a>

## 6. Step-by-step Setup Guide

---

<a id="61-install-github-slack-app"></a>

### 6.1 Install GitHub Slack App

Open:

```text
https://slack.github.com/
```

Click:

```text
Add to Slack
```

Authorize:

* Slack Workspace
* GitHub Account

<img width="1853" height="985" alt="image" src="https://github.com/user-attachments/assets/6db6b508-8953-4dc7-bc9f-143982e216d2" />

---

<a id="62-connect-github-account"></a>

### 6.2 Connect GitHub Account

Authenticate GitHub account with Slack:

```bash
/github signin
```

Authorize the required GitHub account in browser.

<img width="1837" height="985" alt="image" src="https://github.com/user-attachments/assets/a6282347-7e8f-4757-b558-27f4a3bd555e" />


---

<a id="63-invite-github-bot-to-slack-channel"></a>

### 6.3 Invite GitHub Bot to Slack Channel

Open Slack channel:

```text
#ot-micro-infra-titans
```

Run:

```bash
/invite @GitHub
```

Verify:

```text
@GitHub joined the channel
```
<img width="1837" height="985" alt="image" src="https://github.com/user-attachments/assets/b7e7ac74-b8e5-48d7-bc40-462ec6575a1b" />

---

<a id="64-configure-repository-notifications"></a>

### 6.4 Configure Repository Notifications

Subscribe repository:

```bash
/github subscribe 478mkharb/Infra-Titans
```

Enable repository notifications:

```bash
/github subscribe 478mkharb/Infra-Titans pulls commits comments branches merges
```

<img width="1837" height="985" alt="image" src="https://github.com/user-attachments/assets/00aa1e7c-091d-4510-a6f9-5f643fe49c98" />


### Recommended Branches

| Branch    | Purpose          |
| --------- | ---------------- |
| main      | Production       |
| develop   | Development      |
| release/* | Release branches |
| hotfix/*  | Emergency fixes  |

---

<a id="65-configure-email-notifications"></a>

### 6.5 Configure Email Notifications

Navigate to:

```text
GitHub Repository → Settings → Notifications
```

Enable:

* Pull Request notifications
* Commit notifications
* Review comments
* Branch activity alerts
* Merge notifications

Team members can subscribe to repository activities using GitHub Watch Notifications to receive email alerts for Pull Requests, reviews, commits, and merges.

<img width="1837" height="985" alt="image" src="https://github.com/user-attachments/assets/9470e36c-4f41-4c28-bd6d-63ae2e768bf1" />

---

<a id="66-verify-notifications"></a>

### 6.6 Verify Notifications

Create test branch:

```bash
git checkout -b feature/slack-pr-test
```

Create test file and Commit changes:

```bash
touch demo.txt
git add .
git commit -m "Testing PR notifications"
```

Push branch:

```bash
git push origin feature/slack-pr-test
```

Slack channel and configured email accounts will receive notifications.

<img width="1451" height="938" alt="image" src="https://github.com/user-attachments/assets/d56d1886-37d4-455f-963e-7ec9b5241160" />

<img width="1837" height="985" alt="image" src="https://github.com/user-attachments/assets/7ecb261b-a684-40a5-bc58-189dd3d81c66" />

<img width="1693" height="952" alt="image" src="https://github.com/user-attachments/assets/6b05d406-a8c9-41b9-a92d-d79fb131891a" />

---

<a id="7-advantages"></a>

## 7. Advantages

| Advantage                 | Description                              |
| ------------------------- | ---------------------------------------- |
| Faster Reviews            | Immediate PR visibility                  |
| Better Collaboration      | Teams stay informed continuously         |
| Improved Security         | Detects unauthorized repository activity |
| CI/CD Awareness           | Provides deployment visibility           |
| Email Backup Alerts       | Notifications remain accessible offline  |
| Reduced Communication Gap | Centralized repository alerts            |

---

<a id="8-best-practices"></a>

## 8. Best Practices

| Best Practice               | Description                             |
| --------------------------- | --------------------------------------- |
| Monitor Critical Branches   | Focus on production-impacting branches  |
| Use Dedicated Channels      | Separate CI/CD alerts from general chat |
| Avoid Notification Spam     | Filter unnecessary branch events        |
| Restrict Repository Access  | Use proper GitHub permissions           |
| Audit Notification Settings | Periodically validate subscriptions     |
| Enable Email Alerts         | Maintain offline visibility             |

---

<a id="9-conclusion"></a>

## 9. Conclusion

GitHub notifications integrated with Slack and Email improve collaboration, operational visibility, and deployment awareness. Using the GitHub Slack App simplifies repository monitoring without requiring custom webhooks or middleware.

Properly configured notifications help teams respond quickly to Pull Requests, commits, branch operations, and merge activities.

---

<a id="10-contact-information"></a>

## 10. Contact Information

| Name         | ✉️ Contact                                                                        |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

<a id="11-references"></a>

## 11. References

| S.No | Description                        | Click to View                                                                                                                                                                                   |
| ---- | ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | GitHub Slack Integration           | [![GitHub Slack](https://img.shields.io/badge/GitHub-Slack_Integration-2F2F2F?style=flat-square\&logo=github\&logoColor=white)](https://slack.github.com/)                                      |
| 2    | GitHub Notifications Documentation | [![GitHub Notifications](https://img.shields.io/badge/GitHub-NOTIFICATIONS-3A3A3A?style=flat-square\&logo=github\&logoColor=white)](https://docs.github.com/en/subscriptions-and-notifications) |
| 3    | Slack Documentation                | [![Slack Docs](https://img.shields.io/badge/Slack-DOCUMENTATION-1F1F1F?style=flat-square\&logo=slack\&logoColor=white)](https://slack.com/help)                                                 |
| 4    | GitHub Repository Settings         | [![GitHub Settings](https://img.shields.io/badge/GitHub-REPOSITORY_SETTINGS-404040?style=flat-square\&logo=github\&logoColor=white)](https://docs.github.com/en/repositories)                   |
