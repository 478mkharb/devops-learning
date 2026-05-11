<h1 align="center">Documentation - SonarQube Disaster Recovery on AWS EC2</h1>

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
2. [SonarQube PostgreSQL Integration](#2-sonarqube-postgresql-integration)
3. [Important Directories](#3-important-directories)
4. [Backup & Recovery Flow](#4-backup--recovery-flow)
5. [Backup Strategy](#5-backup-strategy)
6. [Recovery Strategy](#6-recovery-strategy)
7. [MTTR](#7-mttr)
8. [Best Practices](#8-best-practices)
9. [Troubleshooting](#9-troubleshooting)
10. [Contact Information](#10-contact-information)
11. [References](#11-references)

---

## 1. Introduction

This document explains how Disaster Recovery (DR) can be implemented for SonarQube deployed on AWS EC2 servers. It covers backup, recovery, and restoration strategies using PostgreSQL backups, EC2 AMIs, and EBS snapshots to minimize downtime during failures.

---

## 2. SonarQube PostgreSQL Integration

SonarQube uses PostgreSQL as the primary database for storing projects, analysis history, quality gates, users, issues, and metrics.

The database connectivity is configured in:

```bash
/opt/sonarqube/conf/sonar.properties
```

### PostgreSQL Configuration Example

```properties
sonar.jdbc.username=sonar
sonar.jdbc.password=********
sonar.jdbc.url=jdbc:postgresql://<db-host>:5432/sonarDB
```

### Why Configuration Backup is Required

| Component            | Purpose                                    |
| -------------------- | ------------------------------------------ |
| PostgreSQL Database  | Stores SonarQube analysis data             |
| sonar.properties     | Stores database connectivity configuration |
| Extensions / Plugins | Maintains plugin compatibility             |
| EC2 / EBS Backup     | Speeds up infrastructure recovery          |

Without the `sonar.properties` backup, SonarQube cannot reconnect to PostgreSQL after recovery.

---

## 3. Important Directories

| Directory / File                     | Description                          |
| ------------------------------------ | ------------------------------------ |
| /opt/sonarqube/conf/sonar.properties | Database and SonarQube configuration |
| /opt/sonarqube/data                  | SonarQube application data           |
| /opt/sonarqube/extensions            | SonarQube plugins and extensions     |
| /opt/sonarqube/logs                  | SonarQube logs                       |

---

## 4. Backup Flow

><img width="1536" height="645" alt="sonar" src="https://github.com/user-attachments/assets/67f759a8-bfeb-49e0-9532-d3f5caa10be6" />



---

## 5. Backup Strategy

SonarQube stores analysis data inside PostgreSQL, while application configuration and plugins are stored separately on the server filesystem. Therefore, both database and filesystem-level backups are required for complete disaster recovery.

| Component           | What It Contains                              | Backup Method        | Automation | Frequency      |
| ------------------- | --------------------------------------------- | -------------------- | ---------- | -------------- |
| PostgreSQL Database | Projects, issues, users, analysis history     | pg_dump              | Cron Job   | Daily          |
| sonar.properties    | Database connectivity configuration           | tar backup + S3 sync | Cron Job   | Weekly         |
| EC2 / EBS           | Server and application disk including plugins | AMI + Snapshot       | AWS Backup | Weekly / Daily |

### PostgreSQL Backup

```bash
pg_dump -U sonar sonarDB > sonarqube_backup.sql
aws s3 cp sonarqube_backup.sql s3://sonarqube-dr-backup/
```

### Automated PostgreSQL Backup Using Cron

```bash
0 1 * * * pg_dump -U sonar sonarDB > /backup/sonar_$(date +\%F).sql
```

The above cron job automatically takes PostgreSQL backup daily at 1 AM.

### SonarQube Configuration Backup

The following file is important during recovery because it stores PostgreSQL connectivity configuration:

```bash
/opt/sonarqube/conf/sonar.properties
```

```bash
tar -czvf sonarqube-config.tar.gz \
/opt/sonarqube/conf \
/opt/sonarqube/extensions
```

### EC2 AMI Backup

```bash
aws ec2 create-image \
--instance-id i-xxxxxxxxxxxx \
--name "sonarqube-dr-backup"
```

### EBS Snapshot Backup

EBS snapshots help recover the complete SonarQube server filesystem, including SonarQube binaries, logs, plugins, and configuration files.

```bash
aws ec2 create-snapshot \
--volume-id vol-xxxxxxxx \
--description "SonarQube EBS Backup"
```

EBS snapshots are generally automated using AWS Backup Policies or Lambda scheduled jobs.

---

## 6. Recovery Strategy

### Database Recovery

```bash
systemctl stop sonarqube
psql -U sonar sonarDB < sonarqube_backup.sql
systemctl start sonarqube
```

### Recovery Implementation

| Step                   | Production Implementation            |
| ---------------------- | ------------------------------------ |
| EC2 Provisioning       | Automated using AMI or Terraform     |
| Volume Recovery        | Automated EBS snapshot restoration   |
| Database Recovery      | Automated PostgreSQL restore scripts |
| Configuration Recovery | Retrieved from S3 or Git repository  |
| Validation             | Health checks and service monitoring |

---

## 7. MTTR

| Term    | Description                       |
| ------- | --------------------------------- |
| MTTR    | Mean Time To Recovery             |
| Purpose | Measures service restoration time |

```math
MTTR = Total Recovery Time / Number of Incidents
```

| Incident              | Recovery Time |
| --------------------- | ------------- |
| Database Failure      | 30 mins       |
| EC2 Failure           | 45 mins       |
| Configuration Failure | 15 mins       |

Average MTTR = 30 mins

---

## 8. Best Practices

| Practice            | Description                       |
| ------------------- | --------------------------------- |
| Automated Backups   | Schedule regular backups          |
| S3 Backup Storage   | Store backups remotely            |
| Backup Encryption   | Secure backup files               |
| Recovery Testing    | Test restore procedures regularly |
| Snapshot Monitoring | Verify snapshot creation          |
| IAM Security        | Restrict backup permissions       |

---

## 9. Troubleshooting

| Issue           | Cause               | Solution                  |
| --------------- | ------------------- | ------------------------- |
| Backup failed   | S3 permission issue | Check IAM role            |
| Snapshot failed | Invalid volume ID   | Verify EBS volume         |
| Restore failed  | Corrupted backup    | Validate backup integrity |

---

## 10. Contact Information

| Name         | Email                                                                             |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

## 11. References

| S.No | Resource                       | Link                                           |
| ---- | ------------------------------ | ---------------------------------------------- |
| 1    | SonarQube Documentation        | [Click Here](https://docs.sonarsource.com/)    |
| 2    | PostgreSQL Documentation       | [Click Here](https://www.postgresql.org/docs/) |
| 3    | AWS EC2 Documentation          | [Click Here](https://docs.aws.amazon.com/ec2/) |
| 4    | AWS EBS Snapshot Documentation | [Click Here](https://docs.aws.amazon.com/ebs/) |
