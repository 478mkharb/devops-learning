<h1 align="center">Documentation - React CI | DAST POC</h1>

<div align="center">
<img width="100" alt="React" src="https://upload.wikimedia.org/wikipedia/commons/a/a7/React-icon.svg" />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="200" alt="OWASP ZAP" src="https://www.zaproxy.org/img/zap-by-checkmarx.svg" />
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
  </tr>

  <tr>
    <td align="center">Mukesh Kharb</td>
    <td align="center">13/05/2026</td>
    <td align="center">1.0</td>
    <td align="center">Mukesh Kharb</td>
    <td align="center">13/05/2026</td>
  </tr>
</table>

</div>

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Running React Frontend](#2-running-react-frontend)
3. [Install OWASP ZAP](#3-install-owasp-zap)
4. [DAST Scan Execution](#4-dast-scan-execution)
5. [Generate Reports](#5-generate-reports)
6. [Common Findings](#6-common-findings)
7. [Contact Information](#7-contact-information)
8. [References](#8-references)

---

<a id="1-introduction"></a>

## 1. Introduction

This POC demonstrates Dynamic Application Security Testing (DAST) for the React frontend of the OT-Microservices platform using OWASP ZAP.

The scan validates:

* Runtime vulnerabilities
* Missing security headers
* Browser-side security issues
* Frontend route exposure

---

---

<a id="2-running-react-frontend"></a>

## 2. Running React Frontend

Install dependencies:

```bash
npm install
```

Start frontend:

```bash
npm start
```

Frontend URL:

```text
http://localhost:3000
```

---

## Screenshot Placeholder — Frontend Running

```text
[ PLACE SCREENSHOT HERE - REACT FRONTEND RUNNING ]
```

---

<a id="3-install-owasp-zap"></a>

## 3. Install OWASP ZAP

Install OWASP ZAP:

```bash
sudo snap install zaproxy --classic
```

Verify installation:

```bash
zaproxy --version
```

---

## Screenshot Placeholder — OWASP ZAP Installation

```text
[ PLACE SCREENSHOT HERE - OWASP ZAP INSTALLATION ]
```

---

<a id="4-dast-scan-execution"></a>

## 4. DAST Scan Execution

### Baseline Scan

```bash
zap-baseline.py -t http://localhost:3000
```

---

## Screenshot Placeholder — Baseline Scan

```text
[ PLACE SCREENSHOT HERE - BASELINE SCAN EXECUTION ]
```

---

### Full Active Scan

```bash
zap-full-scan.py -t http://localhost:3000
```

---

## Screenshot Placeholder — Active Scan

```text
[ PLACE SCREENSHOT HERE - ACTIVE SCAN EXECUTION ]
```

---

<a id="5-generate-reports"></a>

## 5. Generate Reports

Generate HTML report:

```bash
zap-baseline.py -t http://localhost:3000 -r dast-report.html
```

Generate JSON report:

```bash
zap-baseline.py -t http://localhost:3000 -J dast-report.json
```

View reports:

```bash
xdg-open dast-report.html
cat dast-report.json
```

---

## Screenshot Placeholder — HTML Report

```text
[ PLACE SCREENSHOT HERE - HTML REPORT ]
```

---

## Screenshot Placeholder — JSON Report

```text
[ PLACE SCREENSHOT HERE - JSON REPORT ]
```

---

<a id="6-common-findings"></a>

## 6. Common Findings

| Finding                 | Description                       |
| ----------------------- | --------------------------------- |
| Missing CSP             | Content Security Policy missing   |
| Missing HSTS            | Strict Transport Security missing |
| Missing X-Frame-Options | Clickjacking protection missing   |
| Cookie Security Issues  | Missing Secure/HttpOnly flags     |
| Information Disclosure  | Browser/server details exposed    |

---

## Screenshot Placeholder — OWASP ZAP Findings

```text
[ PLACE SCREENSHOT HERE - OWASP ZAP ALERTS ]
```

---

<a id="7-contact-information"></a>

## 7. Contact Information

| Name         | Contact                                                                           |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

<a id="8-references"></a>

## 8. References

| S.No | Description         | Click to View                                                                                                                                                              |
| ---- | ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | OWASP ZAP           | [https://www.zaproxy.org/](https://www.zaproxy.org/)                                                                                                                       |
| 2    | OWASP Top 10        | [https://owasp.org/Top10/](https://owasp.org/Top10/)                                                                                                                       |
| 3    | React Documentation | [https://react.dev/](https://react.dev/)                                                                                                                                   |
| 4    | CSP Headers         | [https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)                                                             |
| 5    | HSTS Headers        | [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security) |

---

## Important Notes

* This ticket scope is limited to React frontend DAST validation only.
* Backend API scanning is excluded from this POC.
* OWASP ZAP is installed on the same EC2 instance hosting the React frontend.
* The scan validates frontend runtime behavior and browser-side security findings.
