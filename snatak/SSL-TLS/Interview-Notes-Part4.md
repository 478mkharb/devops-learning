# SSL/TLS Interview Notes (Part 4)

## Table of Contents

- [38. Common SSL Errors](#38-common-ssl-errors)
- [39. SSL Certificate Renewal](#39-ssl-certificate-renewal)
- [40. OCSP & CRL](#40-ocsp--crl)
- [41. SSL Best Practices](#41-ssl-best-practices)
- [42. SSL in DevOps](#42-ssl-in-devops)
- [43. SSL in CI/CD Pipelines](#43-ssl-in-cicd-pipelines)
- [44. SSL in OT-Microservices Architecture](#44-ssl-in-ot-microservices-architecture)
- [45. Frequently Asked Interview Questions](#45-frequently-asked-interview-questions)
- [46. Quick Revision Cheat Sheet](#46-quick-revision-cheat-sheet)

---

# 38. Common SSL Errors

During SSL implementation, you may encounter several common errors.

---

## 1. NET::ERR_CERT_AUTHORITY_INVALID

### Cause

The certificate is self-signed or issued by an unknown Certificate Authority.

```
Browser

↓

Certificate

↓

Unknown CA

↓

Not Secure
```

### Solution

- Use Let's Encrypt or another trusted CA.
- Import your internal Root CA into the browser.

---

## 2. NET::ERR_CERT_COMMON_NAME_INVALID

### Cause

The certificate does not match the hostname or IP address.

Example

```
Certificate

CN = example.com

↓

Browser

https://192.168.122.167

↓

Mismatch
```

### Solution

Generate the certificate using the correct **Common Name (CN)** and **Subject Alternative Name (SAN)**.

---

## 3. Certificate Expired

```
Certificate

↓

Expiry Date Passed

↓

Browser Rejects Certificate
```

### Solution

Renew the certificate.

---

## 4. Missing Intermediate Certificate

```
Server Certificate

↓

Intermediate Missing

↓

Browser Cannot Verify Chain
```

### Solution

Install the **full certificate chain**.

---

## 5. Private Key Mismatch

```
Certificate

↓

Doesn't Match

↓

Private Key

↓

TLS Handshake Fails
```

Verify

```bash
openssl x509 -noout -modulus -in server.crt | openssl md5

openssl rsa -noout -modulus -in server.key | openssl md5
```

The hashes must match.

---

## 6. Wrong System Time

Incorrect server or client time causes certificate validation failures.

Always synchronize time using NTP.

---

# 39. SSL Certificate Renewal

Certificates expire.

Example

```
Certificate

Valid

365 Days

↓

Expires

↓

Renew
```

---

## Let's Encrypt Renewal

Manual

```bash
sudo certbot renew
```

Dry Run

```bash
sudo certbot renew --dry-run
```

Automatic Renewal

```bash
systemctl status certbot.timer
```

---

# 40. OCSP & CRL

Sometimes certificates must be revoked before expiration.

Example

- Private key leaked
- Certificate stolen
- Company compromised

---

## CRL (Certificate Revocation List)

Certificate Authority publishes a list of revoked certificates.

```
Browser

↓

Download CRL

↓

Certificate Revoked?

↓

Yes

↓

Reject
```

---

## OCSP (Online Certificate Status Protocol)

Instead of downloading a huge CRL,

the browser asks the CA directly.

```
Browser

↓

OCSP Server

↓

Is Certificate Valid?

↓

YES / NO
```

OCSP is faster than CRL.

---

# 41. SSL Best Practices

Always follow these recommendations.

| Best Practice | Reason |
|--------------|--------|
| Use TLS 1.2 or TLS 1.3 | Modern secure protocols |
| Disable SSL 2.0 & SSL 3.0 | Vulnerable |
| Disable TLS 1.0 & TLS 1.1 | Deprecated |
| Protect Private Keys | Prevent impersonation |
| Use Strong Cipher Suites | Better encryption |
| Enable HTTP → HTTPS Redirect | Force secure access |
| Renew Certificates Before Expiry | Prevent downtime |
| Use HSTS | Force browsers to use HTTPS |
| Rotate Certificates Periodically | Improve security |

---

## Enable HSTS

Example

```nginx
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
```

---

# 42. SSL in DevOps

SSL is used almost everywhere.

```
Developer

↓

GitHub

↓

Jenkins

↓

SonarQube

↓

Artifact Repository

↓

Kubernetes

↓

Application
```

Every communication should use HTTPS.

---

Examples

| Tool | HTTPS |
|------|-------|
| Jenkins | Yes |
| GitHub | Yes |
| GitLab | Yes |
| SonarQube | Yes |
| Nexus | Yes |
| Grafana | Yes |
| Prometheus | Yes |
| ArgoCD | Yes |
| Kubernetes Dashboard | Yes |

---

# 43. SSL in CI/CD Pipelines

Example Pipeline

```
Developer

↓

Git Push

↓

Jenkins

↓

HTTPS

↓

GitHub

↓

HTTPS

↓

SonarQube

↓

HTTPS

↓

Nexus

↓

HTTPS

↓

Deploy
```

SSL protects

- Credentials
- Git Tokens
- API Keys
- Build Artifacts

---

# 44. SSL in OT-Microservices Architecture

Recommended Architecture

```
                Browser
                   │
              HTTPS (443)
                   │
                   ▼
          NGINX Reverse Proxy
                   │
      SSL Termination Happens Here
                   │
     ┌─────────────┼─────────────┐
     ▼             ▼             ▼
 Employee API   Attendance API  Salary API
   :8080            :8081         :8082
      │               │             │
      ▼               ▼             ▼
  ScyllaDB       PostgreSQL     ScyllaDB
                      │
                      ▼
             Notification API
                   :8085
```

### Flow

```
Browser

↓

TLS Handshake

↓

Nginx

↓

Decrypt HTTPS

↓

Forward HTTP

↓

Backend APIs

↓

Database

↓

Response

↓

Nginx Encrypts

↓

Browser
```

---

## Why SSL Termination at Nginx?

Advantages

- One certificate to manage
- Backend applications remain unchanged
- Lower CPU usage
- Easier certificate renewal
- Centralized security

---

# 45. Frequently Asked Interview Questions

| Question | Detailed Answer |
|----------|-----------------|
| What is SSL? | SSL (Secure Sockets Layer) is the predecessor of TLS. Modern applications use TLS to encrypt communication between clients and servers. |
| Why do we still call it an SSL certificate? | Although SSL is deprecated, the industry continues to use the term "SSL certificate" to refer to TLS certificates. |
| What is TLS Handshake? | The process where the client and server authenticate, negotiate encryption algorithms, exchange a session key, and establish an encrypted connection. |
| Why is Public Key encryption not used for all communication? | It is computationally expensive. TLS uses it only to exchange a session key, after which symmetric encryption is used. |
| Why is HTTPS faster after the handshake? | Because it uses symmetric encryption (AES or ChaCha20), which is much faster than asymmetric encryption. |
| What is a Certificate Authority? | A trusted organization that validates identities and signs certificates trusted by browsers. |
| Why does Chrome show "Not Secure" for a self-signed certificate? | Because the certificate is not signed by a trusted Certificate Authority, even though the traffic is encrypted. |
| What is SSL Termination? | Decrypting HTTPS at a reverse proxy or load balancer before forwarding traffic to backend services. |
| What is mTLS? | Mutual TLS authenticates both the client and the server using certificates. |
| What is the difference between Self-Signed and CA-Signed Certificates? | Self-signed certificates are suitable for testing and require manual trust. CA-signed certificates are trusted by browsers and used in production. |
| What is a CSR? | A Certificate Signing Request containing the server's public key and identity information. |
| Can the Private Key be shared? | No. The private key must remain secret. Only the public key is shared. |
| What happens if the Private Key is leaked? | Attackers can impersonate the server and decrypt future TLS handshakes that rely on that key (unless using mechanisms like Perfect Forward Secrecy for past sessions). |
| Which port is used by HTTPS? | Port **443**. HTTP uses port **80**. |
| Why do we redirect HTTP to HTTPS? | To ensure all client communication is encrypted and prevent accidental insecure access. |

---

# 46. Quick Revision Cheat Sheet

| Topic | Key Point |
|--------|-----------|
| SSL | Deprecated Security Protocol |
| TLS | Modern Secure Protocol |
| HTTPS | HTTP + TLS |
| Port | 443 |
| HTTP Port | 80 |
| Certificate | Proves Server Identity |
| Public Key | Encrypts Data |
| Private Key | Decrypts Data |
| CA | Issues Certificates |
| CSR | Certificate Signing Request |
| Session Key | Symmetric Key Generated During Handshake |
| TLS Handshake | Establishes Secure Connection |
| Self-Signed | Development Only |
| CA-Signed | Production |
| SSL Termination | HTTPS Ends at Nginx/ALB |
| mTLS | Client and Server Authenticate Each Other |
| HSTS | Forces HTTPS |
| OCSP | Checks Certificate Revocation Online |
| CRL | List of Revoked Certificates |
| Let's Encrypt | Free Trusted Certificate Authority |
| Certbot | Automatically Issues and Renews Certificates |

---

# Complete SSL/TLS Flow (Interview Revision)

```
Client
   │
HTTPS Request
   │
   ▼
TCP Connection
   │
   ▼
TLS Handshake
   │
   ├── Client Hello
   ├── Server Hello
   ├── Certificate
   ├── Certificate Verification
   ├── Session Key Exchange
   └── Secure Connection Established
   │
   ▼
Encrypted HTTP Request
   │
   ▼
Nginx (SSL Termination)
   │
   ▼
Backend APIs
   │
   ▼
Database
   │
   ▼
Encrypted HTTP Response
   │
   ▼
Client
```

---

# One-Line Interview Summary

> **TLS (commonly called SSL) secures communication by authenticating the server with a digital certificate, using asymmetric cryptography to establish a shared symmetric session key during the TLS handshake, and then encrypting all subsequent data with that session key to provide confidentiality, integrity, and authentication.**
