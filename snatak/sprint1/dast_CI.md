<h1 align="center">Documentation - React CI | DAST</h1>

<div align="center">

<img width="150" height="auto" alt="React-CI" src="https://upload.wikimedia.org/wikipedia/commons/a/a7/React-icon.svg" />

<p align="center">
  <a href="https://react.dev/"><img src="https://img.shields.io/badge/React-Documentation-61DAFB?style=for-the-badge&logo=react&logoColor=black" /></a>
  <a href="https://www.zaproxy.org/docs/"><img src="https://img.shields.io/badge/OWASP_ZAP-Docs-red?style=for-the-badge" /></a>
  <a href="https://portswigger.net/burp/documentation"><img src="https://img.shields.io/badge/Burp_Suite-Documentation-orange?style=for-the-badge" /></a>
  <a href="https://www.jenkins.io/doc/"><img src="https://img.shields.io/badge/Jenkins-Documentation-blue?style=for-the-badge&logo=jenkins&logoColor=white" /></a>
</p>

</div>

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
    <td align="center">11/05/2026</td>
    <td align="center">1.0</td>
    <td align="center">Mukesh Kharb</td>
    <td align="center">11/05/2026</td>
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
2. [DAST Characteristics](#2-dast-characteristics)
3. [Why DAST is Required in React CI](#3-why-dast-is-required-in-react-ci)
4. [Flow Diagram of DAST Checks](#4-flow-diagram-of-dast-checks)
5. [Various DAST Tools](#5-various-dast-tools)
6. [Comparison of DAST Tools](#6-comparison-of-dast-tools)
7. [Conclusion](#7-conclusion)
8. [Contact Info](#8-contact-info)
9. [References](#9-references)

---

<a id="1-introduction"></a>

## 1. Introduction

DAST (Dynamic Application Security Testing) helps identify runtime security vulnerabilities in running React applications. It is integrated into CI pipelines to automate security validation before deployment.

---

<a id="2-dast-characteristics"></a>

## 2. DAST Characteristics

| Feature              | Description                    |
| -------------------- | ------------------------------ |
| Testing Type         | Black-box security testing     |
| Scan Target          | Running application            |
| Source Code Required | No                             |
| Main Purpose         | Detect runtime vulnerabilities |
| Common Risks         | XSS, CSRF, SQL Injection       |
| CI/CD Support        | Yes                            |
| OWASP Coverage       | OWASP Top 10                   |

---

<a id="3-why-dast-is-required-in-react-ci"></a>

## 3. Why DAST is Required in React CI

| Requirement / Risk        | Description                                           |
| ------------------------- | ----------------------------------------------------- |
| XSS Detection             | Identifies malicious script injection vulnerabilities |
| API Security              | Protects backend APIs consumed by React apps          |
| Authentication Validation | Detects weak authentication/session issues            |
| Security Headers Check    | Finds missing CSP, HSTS, and header misconfigurations |
| Shift-Left Security       | Detects vulnerabilities earlier in CI pipeline        |
| Automated Validation      | Enables continuous security scanning on every build   |

---

<a id="4-flow-diagram-of-dast-checks"></a>

## 4. Flow Diagram of DAST Checks

```text
Developer Push
       │
       ▼
React CI Pipeline Triggered
       │
       ▼
Build React Application
       │
       ▼
Deploy Temporary Test Environment
       │
       ▼
Run DAST Scanner
       │
       ▼
Scan APIs + Frontend Routes
       │
       ▼
Generate Security Report
       │
       ▼
Pass / Fail Build
```

---

<a id="5-various-dast-tools"></a>

## 5. Various DAST Tools

| Tool       | Type        | Highlight                            | CI/CD Support |
| ---------- | ----------- | ------------------------------------ | ------------- |
| OWASP ZAP  | Open Source | Most widely used automated DAST tool | Excellent     |
| StackHawk  | Commercial  | Developer-friendly CI integration    | Excellent     |
| Burp Suite | Commercial  | Advanced enterprise security testing | Excellent     |
| Acunetix   | Commercial  | Fast vulnerability scanning          | Good          |
| Nikto      | Open Source | Lightweight web scanner              | Moderate      |

---

<a id="6-comparison-of-dast-tools"></a>

## 6. Comparison of DAST Tools

| Feature           | OWASP ZAP | StackHawk | Burp Suite | Acunetix |
| ----------------- | --------- | --------- | ---------- | -------- |
| Open Source       | Yes       | No        | No         | No       |
| React CI Friendly | Excellent | Excellent | Good       | Good     |
| API Scanning      | Yes       | Yes       | Yes        | Yes      |
| Automation        | High      | Very High | High       | Medium   |
| Docker Support    | Yes       | Yes       | Yes        | Limited  |

---

<a id="7-conclusion"></a>

## 7. Conclusion

OWASP ZAP is the most suitable DAST tool for React CI because it is open source, highly automated, Docker-friendly, and integrates easily with modern CI/CD pipelines.

---

<a id="8-contact-info"></a>

## 8. Contact Info

| Name         | Email                                                                             |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

<a id="9-references"></a>

## 9. References

| S.No | Resource                 | Link                                               |
| ---- | ------------------------ | -------------------------------------------------- |
| 1    | React Documentation      | [View](https://react.dev/)                         |
| 2    | OWASP ZAP Documentation  | [View](https://www.zaproxy.org/docs/)              |
| 3    | StackHawk Documentation  | [View](https://www.stackhawk.com/docs/)            |
| 4    | Burp Suite Documentation | [View](https://portswigger.net/burp/documentation) |
| 5    | Jenkins Documentation    | [View](https://www.jenkins.io/doc/)                |
