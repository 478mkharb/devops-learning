<h1 align="center">Documentation: Implementation of SSL</h1>

<p align="center">
  <img src="https://img.shields.io/badge/Certbot-SSL_Automation-green?style=for-the-badge&logo=letsencrypt&logoColor=white" />
</p>

<p align="center">
  <a href="https://letsencrypt.org/"><img src="https://img.shields.io/badge/Let's_Encrypt-Free_Certificates-blue?style=for-the-badge" /></a>
  <a href="https://nginx.org/"><img src="https://img.shields.io/badge/NGINX-Reverse_Proxy-brightgreen?style=for-the-badge" /></a>
  <a href="https://minvya.com/"><img src="https://img.shields.io/badge/Domain-minvya.com-orange?style=for-the-badge" /></a>
</p>

---

| Author       | Created On | Version | Last Updated By | Last Edited On | Pre Reviewer | L0 Reviewer | L1 Reviewer | L2 Reviewer  |
| ------------ | ---------- | ------- | --------------- | -------------- | ------------ | ----------- | ----------- | ------------ |
| Mukesh Kharb | 27/04/2026 | 1.0     | Mukesh Kharb    | 27/04/2026     | Team         | Mohit Kumar | Faisal Khan | Mahesh Kumar |

---

## Table of Contents

* [Introduction](#introduction)
* [Prerequisites](#prerequisites)
* [Implementation Steps](#implementation-steps)

  * [Step 1 - Verify DNS](#step-1---verify-dns)
  * [Step 2 - Verify Website Access](#step-2---verify-website-access)
  * [Step 3 - Install Certbot](#step-3---install-certbot)
  * [Step 4 - Generate SSL Certificate](#step-4---generate-ssl-certificate)
  * [Step 5 - Verify HTTPS](#step-5---verify-https)
  * [Step 6 - Auto Renewal](#step-6---auto-renewal)
* [Benefits](#benefits)
* [FAQs](#faqs)
* [Contact Information](#contact-information)
* [References](#references)

---

## Introduction

SSL (Secure Sockets Layer) enables encrypted communication between users and the web server. It converts HTTP traffic into HTTPS and improves security, trust, and SEO ranking.

This document explains SSL implementation using **NGINX + Certbot + Let's Encrypt**.

---

## Prerequisites

| Requirement         | Status     |
| ------------------- | ---------- |
| Domain configured   | minvya.com |
| DNS pointing to EC2 | Completed  |
| NGINX installed     | Completed  |
| Port 80 open        | Required   |
| Port 443 open       | Required   |

> 📸 **Screenshot of Security Group**
><img width="1527" height="714" alt="image" src="https://github.com/user-attachments/assets/0469e37e-da99-407f-b7a0-d410162a114b" />

## Implementation Steps
---

## Step 1 - Verify DNS

```bash
ping minvya.com
```

> 📸 **Screenshot of Ping Output Showing Domain IP** 
><img width="1390" height="415" alt="image" src="https://github.com/user-attachments/assets/7ccd6ef3-09c1-48ea-a01d-b4936d950243" />

---

## Step 2 - Verify Website Access

```bash
curl -I http://minvya.com
```

> 📸 **Screenshot of curl Output with HTTP 200 OK**
><img width="1111" height="439" alt="image" src="https://github.com/user-attachments/assets/9abc99e6-3981-4509-9bdb-058e6d23643c" />

---

## Step 3 - Install Certbot

```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx -y
```

> 📸 **Screenshot of Certbot Installation Output**
><img width="1417" height="817" alt="image" src="https://github.com/user-attachments/assets/47ef8cd5-3d02-4c84-a89a-6eb719e1db4c" />

---

## Step 4 - Generate SSL Certificate

```bash
sudo certbot --nginx -d minvya.com -d www.minvya.com
```

Choose redirect option:

```text
2: Redirect HTTP to HTTPS
```
> 📸 **Screenshot of Certbot Successful Certificate Generation** 
><img width="1251" height="719" alt="image" src="https://github.com/user-attachments/assets/79f42e3a-6b57-4f43-8df1-c658dd068170" />

---

## Step 5 - Verify HTTPS

Open browser:

```text
https://minvya.com
```

Or run:

```bash
curl -I https://minvya.com
```
><img width="1080" height="402" alt="image" src="https://github.com/user-attachments/assets/c6862575-8238-42b5-9b1a-10b8c1beb7b0" />

Expected result:

```text
HTTP/2 200
```

> 📸 **Screenshot of Browser HTTPS Lock Icon**
<img width="1853" height="1023" alt="image" src="https://github.com/user-attachments/assets/c355180a-254b-4ced-8c75-6870c84280d0" />

---

## Step 6 - Auto Renewal

```bash
sudo systemctl status certbot.timer
```
> 📸 **Screenshot of certbot.timer Service Status**
> <img width="1080" height="402" alt="image" src="https://github.com/user-attachments/assets/dd64355d-437c-4dad-85c1-f289522b8b37" />

---

## Benefits

| Benefit    | Description              |
| ---------- | ------------------------ |
| Security   | Encrypts traffic         |
| Trust      | Browser lock icon        |
| SEO        | Better search ranking    |
| Compliance | Meets security standards |
| Automation | Auto renewal supported   |

---

## FAQs

### 1. Is SSL free using Let's Encrypt?

> Yes, completely free.

### 2. How long is certificate valid?

> Usually 90 days with auto renewal.

### 3. Will HTTP still work?

> It redirects automatically to HTTPS.

### 4. Does SSL affect performance?

> Minimal overhead, usually negligible.

### 5. Is domain ownership required?

> Yes, DNS must point to your server.

---

## Contact Information

| Name         | Email                                                                             |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

## References

| Resource      | Link                                                                 |
| ------------- | -------------------------------------------------------------------- |
| Certbot       | [https://certbot.eff.org/](https://certbot.eff.org/)                 |
| Let's Encrypt | [https://letsencrypt.org/](https://letsencrypt.org/)                 |
| NGINX         | [https://nginx.org/](https://nginx.org/)                             |
| SSL Labs Test | [https://www.ssllabs.com/ssltest/](https://www.ssllabs.com/ssltest/) |
