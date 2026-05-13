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

## Table of Contents

1. [Introduction](#1-introduction)
2. [DAST Characteristics](#2-dast-characteristics)
3. [Why DAST is Required](#3-why-dast-is-required)
4. [Flow Diagram](#4-flow-diagram)
5. [OWASP ZAP Scan Execution](#5-owasp-zap-scan-execution)
   - [5.1 Install OWASP ZAP](#51-install-owasp-zap)
   - [5.2 Baseline Scan](#52-baseline-scan)
   - [5.3 Full Active Scan](#53-full-active-scan)
   - [5.4 Generate Reports](#54-generate-reports)
   - [5.5 Viewing Reports](#55-viewing-reports)
6. [Various DAST Tools with Comparison](#6-various-dast-tools-with-comparison)
7. [Conclusion](#7-conclusion)
8. [Contact Info](#8-contact-info)
9. [References](#9-references)

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

<a id="3-why-dast-is-required"></a>

## 3. Why DAST is Required

| Requirement / Risk        | Description                                           |
| ------------------------- | ----------------------------------------------------- |
| XSS Detection             | Identifies malicious script injection vulnerabilities |
| API Security              | Protects backend APIs consumed by React apps          |
| Authentication Validation | Detects weak authentication/session issues            |
| Security Headers Check    | Finds missing [![CSP](https://img.shields.io/badge/CSP-2f2f2f)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) [![HSTS](https://img.shields.io/badge/HSTS-3a3a3a)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security) and header misconfigurations |
| Shift-Left Security       | Detects vulnerabilities earlier in CI pipeline        |
| Automated Validation      | Enables continuous security scanning on every build   |

---

<a id="4-flow-diagram"></a>

## 4. Flow Diagram of DAST Checks

><img width="800" height="500" alt="image" src="https://github.com/user-attachments/assets/66c17702-d678-4c89-97fc-c4efd59ce3fc" />


---

<a id="5-owasp-zap-scan-execution"></a>

## 5. OWASP ZAP Scan Execution

### 5.1 Install OWASP ZAP

```bash
sudo snap install zaproxy --classic
```
Verify installation:

```bash
zaproxy --version
```

### 5.2 Baseline Scan

```bash
zap-baseline.py -t http://<EC2-PUBLIC-IP>
```
### 5.3 Full Active Scan

```bash
zap-full-scan.py -t http://<EC2-PUBLIC-IP>
```

### 5.4 Generate Reports

```bash
zap-baseline.py -t http://<EC2-PUBLIC-IP> -r dast-report.html
zap-baseline.py -t http://<EC2-PUBLIC-IP> -J dast-report.json
```
### 5.5. Viewing Reports

```bash
xdg-open dast-report.html
cat dast-report.json
```

---

<a id="6-various-dast-tools"></a>

## 6. Various DAST Tools with Comparison

| <div align="center">Tool</div> | <div align="center">Type</div> | <div align="center">Highlight</div> | <div align="center">React CI Friendly</div> | <div align="center">API Scanning</div> | <div align="center">Automation</div> | <div align="center">Docker Support</div> | <div align="center">CI/CD Support</div> |
| --- | --- | --- | --- | --- | --- | --- | --- |
| <div align="center"><b>OWASP ZAP</b></div> | <div align="center">Open Source</div> | <div align="center">Most widely used automated DAST tool</div> | <div align="center">Excellent</div> | <div align="center">Yes</div> | <div align="center">High</div> | <div align="center">Yes</div> | <div align="center">Excellent</div> |
| <div align="center"><b>StackHawk</b></div> | <div align="center">Commercial</div> | <div align="center">Developer-friendly CI integration</div> | <div align="center">Excellent</div> | <div align="center">Yes</div> | <div align="center">Very High</div> | <div align="center">Yes</div> | <div align="center">Excellent</div> |
| <div align="center"><b>Burp Suite</b></div> | <div align="center">Commercial</div> | <div align="center">Advanced enterprise security testing</div> | <div align="center">Good</div> | <div align="center">Yes</div> | <div align="center">High</div> | <div align="center">Yes</div> | <div align="center">Excellent</div> |
| <div align="center"><b>Acunetix</b></div> | <div align="center">Commercial</div> | <div align="center">Fast vulnerability scanning</div> | <div align="center">Good</div> | <div align="center">Yes</div> | <div align="center">Medium</div> | <div align="center">Limited</div> | <div align="center">Good</div> |
| <div align="center"><b>Nikto</b></div> | <div align="center">Open Source</div> | <div align="center">Lightweight web scanner</div> | <div align="center">Moderate</div> | <div align="center">Limited</div> | <div align="center">Medium</div> | <div align="center">Yes</div> | <div align="center">Moderate</div> |

---

<a id="7-conclusion"></a>

## 7. Conclusion

OWASP ZAP is the most suitable DAST tool for React CI because it is open source, highly automated, Docker-friendly, and integrates easily with modern CI/CD pipelines.

---

<a id="8-contact-info"></a>

## 8. Contact Info

| Name | ✉️ Contact |
|---|---|
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |
---

<a id="9-references"></a>

## 9. References

| S.No | Description | Click to view |
|---|---|---|
| 1 | Cross-Site Scripting Vulnerability | [![XSS](https://img.shields.io/badge/XSS_SECURITY-DAST_PROTECTION-2F4F4F?style=flat-square)](https://owasp.org/www-community/attacks/xss/) |
| 2 | SQL Injection | [![SQL Injection](https://img.shields.io/badge/SQL_INJECTION-DAST_PROTECTION-2F4F4F?style=flat-square)](https://owasp.org/www-community/attacks/SQL_Injection) |
| 3 | Dynamic Application Security Testing | [![DAST](https://img.shields.io/badge/DAST_TESTING-SECURITY_SCANNING-2F4F4F?style=flat-square)](https://owasp.org/www-community/Vulnerability_Scanning_Tools) |
| 4 | OWASP ZAP Automated Scanner | [![OWASP ZAP](https://img.shields.io/badge/OWASP_ZAP-AUTOMATED_SCANNER-2F4F4F?style=flat-square)](https://www.zaproxy.org/) |
| 5 | Cross-Site Request Forgery Protection | [![CSRF](https://img.shields.io/badge/CSRF_SECURITY-REQUEST_PROTECTION-2F4F4F?style=flat-square)](https://owasp.org/www-community/attacks/csrf) |
| 6 | Secure HTTP Security Headers | [![Security Headers](https://img.shields.io/badge/SECURITY_HEADERS-HTTP_PROTECTION-2F4F4F?style=flat-square)](https://owasp.org/www-project-secure-headers/) |
