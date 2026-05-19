# <h1 align="center">Documentation - VCS Notifications | Code Events & Branch Merge Alerts </h1>

<div align="center">
<img width="100" alt="GitHub" src="https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png" />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="80" alt="Slack" src="https://cdn-icons-png.flaticon.com/512/2111/2111615.png" />
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
    <td align="center">19/05/2026</td>
    <td align="center">1.0</td>
    <td align="center">Mukesh Kharb</td>
    <td align="center">19/05/2026</td>
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

   * [6.1 Configure Repository Webhooks](#61-configure-repository-webhooks)
   * [6.2 Configure Pull Request / Merge Request Notifications](#62-configure-pull-request--merge-request-notifications)
   * [6.3 Configure Commit Push Notifications](#63-configure-commit-push-notifications)
   * [6.4 Configure Branch Create/Delete Notifications](#64-configure-branch-createdelete-notifications)
   * [6.5 Configure Branch Merge Notifications](#65-configure-branch-merge-notifications)
   * [6.6 Configure Slack Channel Integration](#66-configure-slack-channel-integration)
7. [Notification Event Examples](#7-notification-event-examples)
8. [Advantages](#8-advantages)
9. [Best Practices](#9-best-practices)
10. [Conclusion](#10-conclusion)
11. [Contact Information](#11-contact-information)
12. [References](#12-references)

---

<a id="1-introduction"></a>

# 1. Introduction

Version Control System (VCS) notifications provide real-time visibility into repository activities such as Pull Request creation, Merge Request updates, code pushes, branch operations, and branch merges.

These notifications help DevOps, Development, and QA teams monitor repository activities continuously and improve collaboration across CI/CD workflows.

Notifications can be integrated with communication platforms such as Slack, Microsoft Teams, Email, or Discord using repository webhooks.

---

<a id="2-purpose-of-vcs-notifications"></a>

# 2. Purpose of VCS Notifications

| Purpose              | Description                                   |
| -------------------- | --------------------------------------------- |
| Real-time Monitoring | Tracks repository activities instantly        |
| Faster Collaboration | Helps teams review changes quickly            |
| Deployment Awareness | Alerts teams about production merges          |
| Audit Visibility     | Maintains visibility of branch operations     |
| CI/CD Integration    | Enables automation based on repository events |
| Incident Tracking    | Helps identify unauthorized changes           |

---

<a id="3-supported-notification-events"></a>

# 3. Supported Notification Events

| Event Type             | Description                                                        |
| ---------------------- | ------------------------------------------------------------------ |
| Pull Request Created   | Trigger notification when PR is opened                             |
| Pull Request Updated   | Trigger notification when commits are added                        |
| Pull Request Commented | Trigger notification when review comments are added                |
| Commit Push            | Trigger notification when commits are pushed to important branches |
| Branch Created         | Notify when a new branch is created                                |
| Branch Deleted         | Notify when a branch is removed                                    |
| Branch Merge           | Notify when PR/MR is merged                                        |
| Release Tag Push       | Notify when release tags are pushed                                |

---

<a id="4-architecture-workflow"></a>

# 4. Architecture Workflow

> <img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/41158008-f3a1-487a-8ce1-f646c9cfaec9" />


---

<a id="5-prerequisites"></a>

# 5. Prerequisites

| Requirement                | Description                              |
| -------------------------- | ---------------------------------------- |
| GitHub / GitLab Repository | Repository access with admin permissions |
| Slack Workspace            | Slack channel for notifications          |
| Webhook URL                | Slack Incoming Webhook URL               |

---

<a id="6-step-by-step-setup-guide"></a>

# 6. Step-by-step Setup Guide

---

<a id="61-configure-repository-webhooks"></a>

## 6.1 Configure Repository Webhooks

### GitHub Webhook Configuration

Navigate to:

```text
Repository → Settings → Webhooks → Add webhook
```

Configure:

| Field            | Value                      |
| ---------------- | -------------------------- |
| Payload URL      | Slack Incoming Webhook URL |
| Content Type     | application/json           |
| Secret           | Optional                   |
| SSL Verification | Enable                     |
| Trigger Events   | Select individual events   |

Click:

```text
Add webhook
```

---

### GitLab Webhook Configuration

Navigate to:

```text
Project → Settings → Webhooks
```

Configure:

| Field            | Value                               |
| ---------------- | ----------------------------------- |
| URL              | Slack Incoming Webhook URL          |
| Trigger          | Push Events, Merge Events, Comments |
| SSL Verification | Enable                              |

Save webhook configuration.

---

<a id="62-configure-pull-request--merge-request-notifications"></a>

## 6.2 Configure Pull Request / Merge Request Notifications

Enable the following webhook events:

| Event                    | Description        |
| ------------------------ | ------------------ |
| Pull Request Opened      | New PR created     |
| Pull Request Synchronize | New commits pushed |
| Pull Request Review      | Reviewer activity  |
| Issue Comment            | PR comments        |

### GitHub Event Selection

```text
Pull requests
Issue comments
Pull request reviews
```

### GitLab Event Selection

```text
Merge request events
Comments
```

---

<a id="63-configure-commit-push-notifications"></a>

## 6.3 Configure Commit Push Notifications

Configure push event monitoring for important branches.

### Recommended Branches

| Branch    | Purpose                 |
| --------- | ----------------------- |
| main      | Production              |
| develop   | Development integration |
| release/* | Release pipelines       |
| hotfix/*  | Emergency fixes         |

### GitHub Push Event

Enable:

```text
Pushes
```

### Example Notification Payload

```json
{
  "branch": "main",
  "author": "developer1",
  "commit_message": "Fixed authentication issue"
}
```

---

<a id="64-configure-branch-createdelete-notifications"></a>

## 6.4 Configure Branch Create/Delete Notifications

Enable branch activity monitoring.

### GitHub Events

```text
Create
Delete
```

### GitLab Events

```text
Push Events
```

Branch creation and deletion events help teams monitor repository hygiene and unauthorized branch activity.

---

<a id="65-configure-branch-merge-notifications"></a>

## 6.5 Configure Branch Merge Notifications

Branch merge notifications are triggered whenever Pull Requests or Merge Requests are merged into important branches.

### Important Merge Targets

| Branch    | Description                  |
| --------- | ---------------------------- |
| main      | Production deployment branch |
| develop   | Shared integration branch    |
| release/* | Release branches             |

### GitHub Merge Notification Flow

```text
Pull Request → Review Approved → Merge → Webhook Triggered → Slack Notification
```

### GitLab Merge Notification Flow

```text
Merge Request → Approval → Merge → Notification Triggered
```

### Example Merge Notification

```text
PR #142 merged successfully into main branch by Mukesh Kharb
```

---

<a id="66-configure-slack-channel-integration"></a>

## 6.6 Configure Slack Channel Integration

### Create Slack Incoming Webhook

Navigate to:

```text
Slack API → Apps → Incoming Webhooks
```

Enable:

```text
Activate Incoming Webhooks
```

Create webhook for required channel.

Example:

```text
https://hooks.slack.com/services/XXXX/YYYY/ZZZZ
```

---

### Test Notification

Example CURL command:

```bash
curl -X POST -H 'Content-type: application/json' \
--data '{"text":"VCS Notification Test Successful"}' \
https://hooks.slack.com/services/XXXX/YYYY/ZZZZ
```

---

<a id="7-notification-event-examples"></a>

# 7. Notification Event Examples

| Event           | Example Notification             |
| --------------- | -------------------------------- |
| PR Created      | PR #25 created by developer1     |
| PR Updated      | New commits pushed to PR #25     |
| PR Comment      | Reviewer commented on PR #25     |
| Commit Push     | Commit pushed to develop branch  |
| Branch Created  | feature/login-api branch created |
| Branch Deleted  | feature/test branch deleted      |
| Merge Completed | PR #25 merged into main          |

---

<a id="8-advantages"></a>

# 8. Advantages

| Advantage                 | Description                            |
| ------------------------- | -------------------------------------- |
| Faster Reviews            | Immediate visibility of PR activity    |
| Better Collaboration      | Teams stay informed continuously       |
| Improved Security         | Detects unauthorized branch operations |
| CI/CD Awareness           | Provides deployment visibility         |
| Reduced Communication Gap | Centralized repository alerts          |
| Faster Incident Response  | Teams react quickly to risky changes   |

---

<a id="9-best-practices"></a>

# 9. Best Practices

| Best Practice                | Description                             |
| ---------------------------- | --------------------------------------- |
| Monitor Critical Branches    | Focus on production-impacting branches  |
| Use Dedicated Channels       | Separate CI/CD alerts from general chat |
| Enable SSL Verification      | Secure webhook communication            |
| Avoid Notification Spam      | Filter unnecessary branch events        |
| Restrict Webhook Access      | Use repository admin permissions        |
| Audit Webhook Configurations | Periodically validate webhook URLs      |

---

<a id="10-conclusion"></a>

# 10. Conclusion

VCS notifications improve collaboration, operational visibility, and deployment awareness by providing real-time alerts for repository activities. Integrating GitHub or GitLab with Slack using webhooks enables centralized monitoring of Pull Requests, commits, branch operations, and merge activities.

Properly configured notifications reduce response time, improve review efficiency, and strengthen CI/CD governance across development workflows.

---

<a id="11-contact-information"></a>

# 11. Contact Information

| Name         | ✉️ Contact                                                                        |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

<a id="12-references"></a>

# 12. References

| S.No | Description                   | Click to View                                                                                                                                                                                             |
| ---- | ----------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | GitHub Webhooks Documentation | [![GitHub Webhooks](https://img.shields.io/badge/GitHub-WEBHOOKS-2F2F2F?style=flat-square\&logo=github\&logoColor=white)](https://docs.github.com/en/webhooks)                                            |
| 2    | GitLab Webhooks Documentation | [![GitLab Webhooks](https://img.shields.io/badge/GitLab-WEBHOOKS-3A3A3A?style=flat-square\&logo=gitlab\&logoColor=white)](https://docs.gitlab.com/ee/user/project/integrations/webhooks.html)             |
| 3    | Slack Incoming Webhooks       | [![Slack Webhooks](https://img.shields.io/badge/Slack-INCOMING_WEBHOOKS-1F1F1F?style=flat-square\&logo=slack\&logoColor=white)](https://api.slack.com/messaging/webhooks)                                 |
| 4    | GitHub Pull Request Events    | [![GitHub PR Events](https://img.shields.io/badge/GitHub-PR_EVENTS-404040?style=flat-square\&logo=github\&logoColor=white)](https://docs.github.com/en/webhooks/webhook-events-and-payloads#pull_request) |
| 5    | GitLab Merge Request Events   | [![GitLab MR Events](https://img.shields.io/badge/GitLab-MR_EVENTS-2B2B2B?style=flat-square\&logo=gitlab\&logoColor=white)](https://docs.gitlab.com/ee/user/project/integrations/webhook_events.html)     |
