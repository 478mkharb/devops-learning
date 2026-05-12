<h1 align="center">Documentation - React CI | DAST </h1>

<div align="center">
<img width="100" alt="React" src="https://upload.wikimedia.org/wikipedia/commons/a/a7/React-icon.svg" />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="200" alt="OWASP ZAP" src="https://www.zaproxy.org/img/zap-by-checkmarx.svg" />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
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
5. [Various DAST Tools with Comparison](#5-various-dast-tools-with-comparison)
6. [Conclusion](#6-conclusion)
7. [Contact Info](#7-contact-info)
8. [References](#8-references)

---

<a id="1-introduction"></a>

## 1. Introduction

DAST (Dynamic Application Security Testing) helps identify runtime security vulnerabilities (e.g., XSS, injection flaws) in running React applications and their backend APIs. It is integrated into CI pipelines to automate security validation against an ephemeral or staging environment, ensuring no critical flaws reach production deployment.

---

<a id="2-dast-characteristics"></a>

<table>
<tr>

## 2. DAST Characteristics

| Feature | Description |
|---|---|
| Testing Type | Black-box security testing |
| Scan Target | Running application |
| Source Code Required | No |
| Main Purpose | Detect runtime vulnerabilities |
| Common Risks | [![XSS](https://img.shields.io/badge/XSS-2f2f2f)](https://owasp.org/www-community/attacks/xss/) [![CSRF](https://img.shields.io/badge/CSRF-3a3a3a)](https://owasp.org/www-community/attacks/csrf) [![SQL Injection](https://img.shields.io/badge/SQL_Injection-4a4a4a)](https://owasp.org/www-community/attacks/SQL_Injection) |
| CI/CD Support | Yes |
| OWASP Coverage | [![OWASP Top 10](https://img.shields.io/badge/OWASP_Top_10-2025-2f2f2f)](https://owasp.org/Top10/2025/) |
---

<a id="3-why-dast-is-required-in-react-ci"></a>

## 3. Why DAST is Required in React CI

| Requirement / Risk        | Description                                           |
| ------------------------- | ----------------------------------------------------- |
| XSS Detection             | Identifies malicious script injection vulnerabilities |
| API Security              | Protects backend APIs consumed by React apps          |
| Authentication Validation | Detects weak authentication/session issues            |
| Security Headers Check    | Finds missing [![CSP](https://img.shields.io/badge/CSP-2f2f2f)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) [![HSTS](https://img.shields.io/badge/HSTS-3a3a3a)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security) and header misconfigurations |
| Shift-Left Security       | Detects vulnerabilities earlier in CI pipeline        |
| Automated Validation      | Enables continuous security scanning on every build   |

---

<a id="4-flow-diagram-of-dast-checks"></a>

## 4. Flow Diagram of DAST Checks

><img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/9e5fadf7-9087-46b4-9e77-7f01fbd8e485" />

---

<a id="5-various-dast-tools"></a>

## 5. Various DAST Tools with Comparison

| Tool       | Type        | Highlight                            | React CI Friendly | API Scanning | Automation | Docker Support | CI/CD Support |
| ---------- | ----------- | ------------------------------------ | ----------------- | ------------ | ---------- | -------------- | ------------- |
| OWASP ZAP  | Open Source | Most widely used automated DAST tool | Excellent         | Yes          | High       | Yes            | Excellent     |
| StackHawk  | Commercial  | Developer-friendly CI integration    | Excellent         | Yes          | Very High  | Yes            | Excellent     |
| Burp Suite | Commercial  | Advanced enterprise security testing | Good              | Yes          | High       | Yes            | Excellent     |
| Acunetix   | Commercial  | Fast vulnerability scanning          | Good              | Yes          | Medium     | Limited        | Good          |
| Nikto      | Open Source | Lightweight web scanner              | Moderate          | Limited      | Medium     | Yes            | Moderate      |


---

<a id="6-conclusion"></a>

## 6. Conclusion

OWASP ZAP is the most suitable DAST tool for React CI because it is open source, highly automated, Docker-friendly, and integrates easily with modern CI/CD pipelines.

---

<a id="7-contact-info"></a>

## 7. Contact Info

| Name | ✉️ Contact |
|---|---|
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |
---

<a id="8-references"></a>

## 8. References

| S.No | Description | Click to view |
|---|---|---|
| 1 | Cross-Site Scripting Vulnerability | [![XSS](https://img.shields.io/badge/XSS_SECURITY-DAST_PROTECTION-2F4F4F?style=flat-square)](https://owasp.org/www-community/attacks/xss/) |
| 2 | SQL Injection | [![SQL Injection](https://img.shields.io/badge/SQL_INJECTION-DAST_PROTECTION-2F4F4F?style=flat-square)](https://owasp.org/www-community/attacks/SQL_Injection) |
| 3 | Dynamic Application Security Testing | [![DAST](https://img.shields.io/badge/DAST_TESTING-SECURITY_SCANNING-2F4F4F?style=flat-square)](https://owasp.org/www-community/Vulnerability_Scanning_Tools) |
| 4 | OWASP ZAP Automated Scanner | [![OWASP ZAP](https://img.shields.io/badge/OWASP_ZAP-AUTOMATED_SCANNER-2F4F4F?style=flat-square)](https://www.zaproxy.org/) |
| 5 | Cross-Site Request Forgery Protection | [![CSRF](https://img.shields.io/badge/CSRF_SECURITY-REQUEST_PROTECTION-2F4F4F?style=flat-square)](https://owasp.org/www-community/attacks/csrf) |
| 6 | Secure HTTP Security Headers | [![Security Headers](https://img.shields.io/badge/SECURITY_HEADERS-HTTP_PROTECTION-2F4F4F?style=flat-square)](https://owasp.org/www-project-secure-headers/) |

