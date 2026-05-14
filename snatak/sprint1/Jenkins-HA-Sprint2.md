<h1 align="center">Documentation - Jenkins High Availability (HA)</h1>
<div align="center">
<img width="120" alt="Jenkins" src="https://www.jenkins.io/images/logos/jenkins/jenkins.svg" />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
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
    <td align="center">14/05/2026</td>
    <td align="center">1.0</td>
    <td align="center">Mukesh Kharb</td>
    <td align="center">14/05/2026</td>
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
2. [What is Jenkins HA](#2-what-is-jenkins-ha)  
3. [Why Jenkins HA is Required](#3-why-jenkins-ha-is-required)  
4. [Methods of Jenkins High Availability](#4-methods-of-jenkins-high-availability)  
   4.1 [Active-Passive HA](#41-active-passive-ha)  
   4.2 [Active-Active HA](#42-active-active-ha)  
5. [Comparison of HA Methods](#5-comparison-of-ha-methods)  
6. [Best Practices](#6-best-practices)  
7. [Monitoring and Health Checks](#8-monitoring-and-health-checks)  
8. [Conclusion](#9-conclusion)  
9. [Contact Information](#9-contact-information)  
10. [References](#10-references)

---

<a id="1-introduction"></a>

## 1. Introduction

Jenkins is a widely used CI/CD automation platform used for building, testing, and deploying applications.

>In enterprise DevOps environments, Jenkins becomes a critical service. Any downtime in Jenkins directly impacts build pipelines, deployments, release timelines, and operational stability.

High Availability (HA) architecture ensures Jenkins remains operational even during infrastructure, storage, or node failures.


---

<a id="2-what-is-jenkins-ha"></a>

## 2. What is Jenkins HA

Jenkins High Availability (HA) is a setup that keeps Jenkins
running with minimal downtime during failures.

>It uses multiple Jenkins controllers, failover mechanisms,
and shared storage to keep CI/CD pipelines running smoothly.
>Load balancers distribute traffic efficiently, while backups
help in faster recovery and service restoration.

---

<a id="3-why-jenkins-ha-is-required"></a>

## 3. Why Jenkins HA is Required

| Requirement / Risk    | Description                    |
| --------------------- | ------------------------------ |
| Jenkins Failure       | CI/CD pipelines stop           |
| Infrastructure Outage | Build and deployment downtime  |
| Disk Corruption       | Loss of Jenkins configurations |
| Heavy Build Load      | Performance degradation        |
| Plugin Failure        | Interrupted pipeline execution |
| Business Continuity   | Continuous software delivery   |

---

<a id="4-methods-of-jenkins-high-availability"></a>

## 4. Methods of Jenkins High Availability

> Jenkins HA strategies are commonly implemented using cloud-managed infrastructure services such as AWS EC2, EFS, ALB, Route53, Auto Scaling, and CloudWatch.

### &nbsp;&nbsp;&nbsp;&nbsp;4.1 Active-Passive HA

- &nbsp;&nbsp;&nbsp;&nbsp;Active-Passive HA uses one active Jenkins controller and one standby controller for failover support.  
- &nbsp;&nbsp;&nbsp;&nbsp;It provides simpler management, lower operational complexity, and cost-efficient high availability.

### &nbsp;&nbsp;&nbsp;&nbsp;Flow Diagram
><img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/d797e9d9-1aa1-4ed9-8470-3f1909503e95" />

---

### &nbsp;&nbsp;&nbsp;&nbsp;4.2 Active-Active HA

- &nbsp;&nbsp;&nbsp;&nbsp;Active-Active HA uses multiple active Jenkins controllers handling requests simultaneously.  
- &nbsp;&nbsp;&nbsp;&nbsp;It offers higher scalability and faster failover but introduces greater synchronization and management complexity.
### &nbsp;&nbsp;&nbsp;&nbsp;Flow Diagram
><img width="1536" height="1024" alt="ChatGPT Image May 14, 2026, 02_38_57 PM" src="https://github.com/user-attachments/assets/e7d5cf11-8d70-4d1f-aa84-b4cf906d933c" />



---

<a id="5-comparison-of-ha-methods"></a>

## 5. Comparison of HA Methods

| HA Method        | Complexity | Cost     | Scalability | Failover Speed | Best Use Case      |
| ---------------- | ---------- | -------- | ----------- | -------------- | ------------------ |
| Active-Passive   | Medium     | Moderate | Medium      | Fast           | Small-Medium Teams |
| Active-Active    | High       | High     | High        | Very Fast      | Enterprise Scale   |

---

<a id="6-best-practices"></a>

## 6. Best Practices

| Best Practice                        | Purpose                  |
| ------------------------------------ | ------------------------ |
| Enable Automated Backups             | Faster recovery          |
| Store Jenkins Home on Shared Storage | Persistent data          |
| Use Dynamic Agents                   | Better scalability       |
| Enable Centralized Logging           | Easier troubleshooting   |
| Monitor Jenkins Health               | Early issue detection    |
| Use RBAC                             | Better security          |
| Regular Failover Testing             | Validate HA setup        |
| Minimize Plugin Usage                | Reduce instability       |
| Secure Secrets Properly              | Improve security posture |

---

<a id="7-monitoring-and-health-checks"></a>

## 7. Monitoring and Health Checks

| Tool                      | Purpose                              |
| ------------------------- | ------------------------------------ |
| CloudWatch                | Infrastructure monitoring and alerts |
| Jenkins Monitoring Plugin | Jenkins metrics and health checks    |



---

### Important Health Checks

| Health Check         | Parameters        | Healthy Figure |
| -------------------- | ----------------- | -------------- |
| CPU Usage            | CPU %             | < 70%          |
| Memory Usage         | RAM %, Heap Usage | < 75%          |
| Disk Usage           | Disk %            | < 70%          |
| Queue Size           | Pending Jobs      | < 10 Jobs      |
| Agent Connectivity   | Online Agents     | 100% Online    |
| API Health Endpoint  | HTTP Status       | 200 OK         |
| Network Connectivity | Latency           | < 50 ms        |
| Service Status       | Jenkins Status    | Running        |

---

<a id="8-conclusion"></a>

## 8. Conclusion

>Jenkins High Availability improves CI/CD reliability, minimizes downtime, and supports continuous software delivery.

A *cloud-managed Active-Passive Jenkins HA architecture* is recommended because it provides better operational stability, 
simpler failover management, lower infrastructure complexity, and cost-efficient scalability for medium-scale CI/CD workloads.

---

<a id="9-contact-information"></a>

## 9. Contact Information

| Name         | ✉️ Contact                                                                        |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

<a id="10-references"></a>

## 10. References

| S.No | Description              | Click to View                                                                                                                                                                  |
| ---- | ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1    | Jenkins Documentation    | [![Jenkins Docs](https://img.shields.io/badge/Jenkins-Documentation-2F4F4F?style=flat-square\&logo=jenkins\&logoColor=white)](https://www.jenkins.io/doc/)                     |
| 2    | Jenkins Scaling          | [![Jenkins Scaling](https://img.shields.io/badge/Jenkins-Scaling-3A3A3A?style=flat-square\&logo=jenkins\&logoColor=white)](https://www.jenkins.io/doc/book/scaling/)           |
| 3    | AWS EFS Documentation    | [![AWS EFS](https://img.shields.io/badge/AWS-EFS-2B2B2B?style=flat-square\&logo=amazonaws\&logoColor=white)](https://docs.aws.amazon.com/efs/)                                 |
