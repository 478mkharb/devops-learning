<h1 align="center">Documentation - SonarQube Disaster Recovery</h1>

<p align="center">
  <img width="300" height="auto" alt="DV-SonarQube" src="https://github.com/user-attachments/assets/36f08d50-e4ca-4704-9020-f0c5c8cb18dc" />
</p>

<p align="center">
  <a href="#"><img src="https://img.shields.io/badge/SonarQube-Disaster_Recovery-blue?style=for-the-badge" /></a>
  <a href="#"><img src="https://img.shields.io/badge/AWS-EC2-orange?style=for-the-badge" /></a>
  <a href="#"><img src="https://img.shields.io/badge/PostgreSQL-Backup_&_Recovery-blue?style=for-the-badge" /></a>
</p>

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
    <td align="center">22/04/2026</td>
    <td align="center">1.0</td>
    <td align="center">Mukesh Kharb</td>
    <td align="center">22/04/2026</td>
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
2. [Disaster Recovery Strategies](#2-disaster-recovery-strategies)
3. [Backup & Recovery Flow](#3-backup--recovery-flow)
4. [MTTR Calculation](#4-mttr-calculation)
5. [Best Practices](#5-best-practices)
6. [Troubleshooting](#6-troubleshooting)
7. [Conclusion](#7-conclusion)
8. [Contact Information](#8-contact-information)
9. [References](#9-references)

---

## 1. Introduction

This document explains Disaster Recovery (DR) strategies for SonarQube deployed on AWS EC2 servers and identifies the most practical production-ready approach for minimizing downtime and simplifying recovery.

---

## 2. Disaster Recovery Strategies

|            DR Strategy            |                  Description                   |   MTTR      | Complexity | Production Suitability |
|:---------------------------------:|:----------------------------------------------:|:-----------:|:----------:|:----------------------:|
| PostgreSQL Backup Only | Restores database using pg_dump backup | High | Low | Low |
| AMI Backup Only | Restores complete EC2 environment from AMI | Low | Low | Medium |
| AMI + EBS Snapshot | Restores EC2 and storage snapshots | Very Low | Medium | High |
| RDS Snapshot + AMI | Uses RDS automated snapshots with EC2 recovery | Very Low | Medium | Enterprise |
| Multi-AZ / HA Setup | High availability deployment across zones | Lowest | High | Enterprise |

> [!Note]
> SonarQube uses PostgreSQL as the primary database for storing projects, analysis history, quality gates, users, issues, and metrics.
>
> PostgreSQL connectivity is configured in:
>
> ```bash
> /opt/sonarqube/conf/sonar.properties
> ```

### Recommended Strategy

The most practical production-ready strategy for a single EC2 SonarQube deployment is:

```text
AMI Backup Strategy
```

| Advantage              | Description                                         |
| ---------------------- | --------------------------------------------------- |
| Low MTTR               | Faster server restoration                           |
| Simplicity             | Single recovery artifact                            |
| Lower Complexity       | No separate DB restore process                      |
| Faster Recovery        | SonarQube, PostgreSQL, and configs already included |
| Operational Simplicity | Easier automation and maintenance                   |
| Cross Region DR        | Backup available in secondary AWS region            |

---

## 3. Backup & Recovery Flow

><img width="1692" height="929" alt="image" src="https://github.com/user-attachments/assets/3e3d3f2e-7c60-41f1-a31b-ed3177ac91be" />

---

## 4. MTTR Calculation

 - MTTR   -  Mean Time To Recovery
 - Purpose  - Measures the average recovery time after failure 

### *MTTR = Total Recovery Time / Number of Incidents*


| Recovery Activity          | Estimated Recovery Time |
| -------------------------- | ----------------------- |
| Launch EC2 from AMI        | 10 mins                 |
| Start & Validate SonarQube | 5 mins                  |
| Total Estimated MTTR       | 15 mins                 |

---

## 5. Best Practices

| Practice          | Description                                                     |
| ----------------- | --------------------------------------------------------------- |
| Automated Backups | Schedule AMI backups using AWS Backup policies                  |
| Backup Monitoring | Verify backup completion                                        |
| Recovery Testing  | Periodically validate AMI restoration                           |
| IAM Security      | Restrict backup permissions                                     |
| Backup Retention  | Maintain backup retention and cross-region replication policies |

---

## 6. Troubleshooting

| Scenario | Possible Cause | Resolution |
|---|---|---|
| AMI backup not getting created | AWS Backup policy or IAM role misconfiguration | Verify AWS Backup schedules, IAM permissions, and EC2 backup assignment |
| Cross-region AMI replication failed | Missing destination region permissions or quota limits | Validate cross-region copy permissions and available snapshot quotas |
| SonarQube service not accessible after recovery | SonarQube service not started or port blocked | Verify SonarQube service status and EC2 security group rules |
| PostgreSQL connection failure after restore | Incorrect database configuration in sonar.properties | Validate PostgreSQL hostname, port, username, and connectivity |
| Recovery taking longer than expected | Large EBS volume initialization delay | Enable fast snapshot restore or optimize EBS sizing |

---
## 7. Conclusion

For production environments, an AMI-based disaster recovery strategy with cross-region replication provides the best balance of low MTTR, operational simplicity, and faster infrastructure recovery.  
Automating AMI creation using AWS Backup policies further improves reliability, scalability, and disaster preparedness for SonarQube deployments.

---
## 8. Contact Information

| Name         | Email                                                                             |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

## 9. References

| S.No | Resource | Link |
|---|---|---|
| 1 | SonarQube Documentation | [Click to Open](https://docs.sonarsource.com/) |
| 2 | AWS EC2 Documentation | [Click to Open](https://docs.aws.amazon.com/ec2/) |
| 3 | AWS Backup Documentation | [Click to Open](https://docs.aws.amazon.com/aws-backup/) |
| 4 | AWS AMI Documentation | [Click to Open](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) |
| 5 | AWS Cross-Region Backup | [Click to Open](https://docs.aws.amazon.com/aws-backup/latest/devguide/cross-region-backup.html) |

