<h1 align="center"># POC - React CI | DAST using OWASP ZAP </h1>

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
    <td align="center">13/05/2026</td>
    <td align="center">1.0</td>
    <td align="center">Mukesh Kharb</td>
    <td align="center">13/05/2026</td>
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
2. [Running React Frontend](#2-running-react-frontend)
3. [Install Required Packages](#3-install-required-packages)
4. [Download OWASP ZAP Installer](#4-download-owasp-zap-installer)
5. [Install OWASP ZAP (Quiet Mode)](#5-install-owasp-zap-quiet-mode)
6. [Start ZAP in Daemon Mode](#6-start-zap-in-daemon-mode)
7. [Verify ZAP API and Scan Execution](#7-verify-zap-api-and-scan-execution)
8. [Generate HTML Report](#8-generate-html-report)
9. [OWASP ZAP Findings](#9-owasp-zap-findings)
10. [Contact Information](#10-contact-information)
11. [References](#11-references)

---


## 1. Introduction

This POC demonstrates Dynamic Application Security Testing (DAST) for the React frontend of the OT-Microservices platform using OWASP ZAP.

The scan validates:

* Runtime vulnerabilities
* Missing security headers
* Browser-side security issues
* Frontend route exposure
  
---

## 2. Running React Frontend

Clone repository:

```bash
git clone https://github.com/OT-MICROSERVICES/frontend.git
cd frontend
```
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

<details>
<summary>📸 <strong>Click to view Screenshot - Frontend </strong></summary>
<img width="1831" height="891" alt="image" src="https://github.com/user-attachments/assets/f75851ea-011b-4939-a610-7d99fda17e92" />
</details>

---

## 3. Install Required Packages

```bash
sudo apt update
sudo apt install openjdk-17-jdk wget unzip curl -y
java -version
```

<details>
<summary>📸 <strong>Click to view Screenshot - Installation of Install Required Packages</strong></summary>
<img width="1557" height="765" alt="image" src="https://github.com/user-attachments/assets/5d411b2a-8c86-4627-8a84-8c152e3604cc" />
</details>

---

## 4. Download OWASP ZAP Installer

```bash
wget https://github.com/zaproxy/zaproxy/releases/download/v2.16.1/ZAP_2_16_1_unix.sh
chmod +x ZAP_2_16_1_unix.sh
ls -l ZAP_2_16_1_unix.sh
```

<details>
<summary>📸 <strong>Click to view Screenshot - OWASP ZAP Installer</strong></summary>
<img width="1745" height="804" alt="image" src="https://github.com/user-attachments/assets/58d24372-87d4-47f6-bb6b-8c3885db3316" />
</details>

---

## 5. Install OWASP ZAP (Quiet Mode)

```bash
./ZAP_2_16_1_unix.sh -q
ls /opt/zaproxy
```

<details>
<summary>📸 <strong>Click to view Screenshot - Install ZAP</strong></summary>
<img width="1572" height="344" alt="image" src="https://github.com/user-attachments/assets/e906baed-f3ee-4b54-9785-f6653b2d8202" />
</details>

---
## 6. Start ZAP in Daemon Mode

- Open a new terminal
```bash
nohup /opt/zaproxy/zap.sh -daemon \
-host 127.0.0.1 \
-port 8090 \
-config api.disablekey=true \
-config api.addrs.addr.name=.* \
-config api.addrs.addr.regex=true \
> zap-daemon.log 2>&1 &
```
- Verify daemon
```bash
ps aux | grep zap
ss -tulpn | grep 8090
```

<details>
<summary>📸 <strong>Click to view Screenshot - Start ZAP in daemon Mode</strong></summary>
<img width="1549" height="422" alt="image" src="https://github.com/user-attachments/assets/551df27f-1165-437a-8843-009a01f652c3" />
</details>

---

## 7. Verify ZAP API and Scan Execution

- Verify ZAP API

```bash
curl "http://localhost:8090/JSON/core/view/version/"
```
- Run Spider Scan

```bash
curl "http://localhost:8090/JSON/spider/action/scan/?url=http://localhost:3000"
```
- Check Status

```bash
curl "http://localhost:8090/JSON/spider/view/status/?scanId=0"
```
- Run Active Scan

```bash
curl "http://localhost:8090/JSON/ascan/action/scan/?url=http://localhost:3000"
```
<details>
<summary>📸 <strong>Screenshot of ZAP API and Run Spider Scan Execution</strong></summary>
<img width="1488" height="373" alt="image" src="https://github.com/user-attachments/assets/a9b76848-09b4-4bfc-ada1-2b7c589c8b3e" />
</details>

> [!NOTE]
> - `{"scan":"0"}` → Scan ID generated by OWASP ZAP  
> - `{"status":"100"}` → Scan completed successfully

---

## 8. Generate HTML Report

```bash
curl "http://localhost:8090/OTHER/core/other/htmlreport/" -o dast-report.html
ls
```
- Run to open reports
  
```bash
python3 -m http.server 9000
```
- Open Browser and Access

```bash
http://<VM-IP>:9000/dast-report.html
```
<details>
<summary>📸 <strong>Screenshot of HTML Report</strong></summary>
<img width="1488" height="373" alt="image" src="https://github.com/user-attachments/assets/24b4430a-2920-4652-b01b-ba43848733d4" />
<img width="1823" height="955" alt="image" src="https://github.com/user-attachments/assets/ee3a5ecb-7432-4f3f-ba8f-f077542b7d27" />
</details>

---

## 9. OWASP ZAP Findings

| Finding | Risk Level | Count |
|---|---|---|
| CSP: Failure to Define Directive with No Fallback | 🟠 Medium | 2 |
| Content Security Policy (CSP) Header Not Set | 🟠 Medium | 1 |
| Missing Anti-clickjacking Header | 🟠 Medium | 1 |
| Private IP Disclosure | 🟡 Low | 1 |
| Server Leaks Information via `X-Powered-By` HTTP Response Header | 🟡 Low | 7 |
| X-Content-Type-Options Header Missing | 🟡 Low | 4 |
| ZAP is Out of Date | 🟡 Low | 1 |
| Information Disclosure - Suspicious Comments | 🔵 Informational | 3 |
| Modern Web Application | 🔵 Informational | 1 |

---

## 10. Contact Information

| Name         | Contact                                                                           |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

## 11. References

| S.No | Description | Click to View |
|---|---|---|
| 1 | OWASP ZAP | [![OWASP ZAP](https://img.shields.io/badge/OWASP-ZAP-2C2C2C?style=for-the-badge&logo=owasp&logoColor=white)](https://www.zaproxy.org/) |
| 2 | OWASP Top 10 | [![OWASP Top 10](https://img.shields.io/badge/OWASP-Top_10-3A3A3A?style=for-the-badge&logo=owasp&logoColor=white)](https://owasp.org/Top10/) |
| 3 | CSP Headers | [![CSP Headers](https://img.shields.io/badge/Security-CSP_Headers-1F1F1F?style=for-the-badge&logo=googlechrome&logoColor=white)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) |
| 4 | HSTS Headers | [![HSTS Headers](https://img.shields.io/badge/Security-HSTS_Headers-2B2B2B?style=for-the-badge&logo=shield&logoColor=white)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security) |
