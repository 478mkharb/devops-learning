# SSL/TLS Interview Notes (Part 3)

## Table of Contents

* [25. How SSL/TLS is Implemented](#25-how-ssltls-is-implemented)
* [26. Generating SSL Certificates Using OpenSSL](#26-generating-ssl-certificates-using-openssl)
* [27. Self-Signed vs CA-Signed Certificates](#27-self-signed-vs-ca-signed-certificates)
* [28. SSL Implementation in Nginx](#28-ssl-implementation-in-nginx)
* [29. SSL Implementation in Apache](#29-ssl-implementation-in-apache)
* [30. SSL Implementation in Spring Boot](#30-ssl-implementation-in-spring-boot)
* [31. SSL in Kubernetes](#31-ssl-in-kubernetes)
* [32. SSL Termination](#32-ssl-termination)
* [33. Mutual TLS (mTLS)](#33-mutual-tls-mtls)
* [34. SSL in AWS](#34-ssl-in-aws)
* [35. SSL in OT-Microservices](#35-ssl-in-ot-microservices)
* [36. Common OpenSSL Commands](#36-common-openssl-commands)
* [37. Frequently Asked Interview Questions (Part 3)](#37-frequently-asked-interview-questions-part-3)

---

<a id="25-how-ssltls-is-implemented"></a>

# 25. How SSL/TLS is Implemented

Implementing SSL involves obtaining a certificate and configuring it on the server.

---

## Complete SSL Implementation Flow

```text
Generate Private Key
        │
        ▼
Generate CSR
        │
        ▼
Send CSR to Certificate Authority
        │
        ▼
CA Verifies Domain Ownership
        │
        ▼
Certificate Issued
        │
        ▼
Install Certificate on Web Server
        │
        ▼
Enable HTTPS (Port 443)
        │
        ▼
Clients Access Website Securely
```

---

## SSL Implementation Steps

### Step 1

Generate Private Key

↓

### Step 2

Generate CSR (Certificate Signing Request)

↓

### Step 3

Submit CSR to CA

↓

### Step 4

CA verifies ownership

↓

### Step 5

Receive Certificate

↓

### Step 6

Install Certificate

↓

### Step 7

Restart Web Server

↓

HTTPS Enabled

---

<a id="26-generating-ssl-certificates-using-openssl"></a>

# 26. Generating SSL Certificates Using OpenSSL

## Generate Private Key

```bash
openssl genrsa -out private.key 2048
```

Creates a 2048-bit RSA private key.

---

## Generate CSR

```bash
openssl req \
-new \
-key private.key \
-out server.csr
```

You'll be asked:

* Country
* State
* Organization
* Common Name (Domain)
* Email

---

## Generate Self-Signed Certificate

```bash
openssl req \
-x509 \
-newkey rsa:2048 \
-keyout private.key \
-out server.crt \
-days 365
```

---

## View Certificate

```bash
openssl x509 \
-in server.crt \
-text \
-noout
```

Displays:

* Issuer
* Subject
* Validity
* Public Key
* Signature Algorithm

---

## Verify Certificate

```bash
openssl verify server.crt
```

---

<a id="27-self-signed-vs-ca-signed-certificates"></a>

# 27. Self-Signed vs CA-Signed Certificates

| Feature               | Self-Signed | CA-Signed                            |
| --------------------- | ----------- | ------------------------------------ |
| Issued By             | Yourself    | Certificate Authority                |
| Browser Trust         | ❌ Warning   | ✅ Trusted                            |
| Cost                  | Free        | Usually Paid (Let's Encrypt is Free) |
| Development           | ✅ Yes       | Limited                              |
| Production            | ❌ No        | ✅ Yes                                |
| Identity Verification | None        | Verified                             |

---

## When to Use Self-Signed

* Local Development
* Internal Testing
* Lab Environments

---

## When to Use CA-Signed

* Production Websites
* Banking Applications
* Public APIs
* Cloud Applications

---

<a id="28-ssl-implementation-in-nginx"></a>

# 28. SSL Implementation in Nginx

### Example Configuration

```nginx
server {

    listen 443 ssl;

    server_name example.com;

    ssl_certificate /etc/nginx/ssl/server.crt;

    ssl_certificate_key /etc/nginx/ssl/private.key;

    ssl_protocols TLSv1.2 TLSv1.3;

    ssl_ciphers HIGH:!aNULL:!MD5;

}
```

Restart Nginx

```bash
sudo systemctl restart nginx
```

---

<a id="29-ssl-implementation-in-apache"></a>

# 29. SSL Implementation in Apache

```apache
<VirtualHost *:443>

SSLEngine on

SSLCertificateFile /etc/ssl/server.crt

SSLCertificateKeyFile /etc/ssl/private.key

</VirtualHost>
```

Restart Apache

```bash
sudo systemctl restart apache2
```

---

<a id="30-ssl-implementation-in-spring-boot"></a>

# 30. SSL Implementation in Spring Boot

Spring Boot typically uses a Java KeyStore (JKS) or PKCS12 file.

### application.properties

```properties
server.port=8443

server.ssl.enabled=true

server.ssl.key-store=classpath:keystore.p12

server.ssl.key-store-password=password

server.ssl.key-store-type=PKCS12
```

Application starts on HTTPS.

```
https://localhost:8443
```

---

<a id="31-ssl-in-kubernetes"></a>

# 31. SSL in Kubernetes

SSL is generally terminated at the **Ingress Controller**.

Example

```text
Browser

↓

HTTPS

↓

Ingress Controller

↓

HTTP

↓

Service

↓

Pod
```

Ingress Controllers

* NGINX Ingress
* Traefik
* HAProxy
* AWS ALB Ingress

Certificates are usually stored as Kubernetes Secrets.

Example

```bash
kubectl create secret tls app-cert \
--cert=server.crt \
--key=private.key
```

---

<a id="32-ssl-termination"></a>

# 32. SSL Termination

SSL Termination means HTTPS ends at a proxy or load balancer.

Backend services receive HTTP.

---

## Flow

```text
Browser

↓

HTTPS

↓

Nginx

↓

HTTP

↓

Spring Boot
```

Advantages

* Backend services don't manage certificates.
* Reduced CPU usage.
* Easier certificate renewal.

---

## SSL Passthrough

```text
Browser

↓

HTTPS

↓

Load Balancer

↓

HTTPS

↓

Application
```

Here, encryption continues all the way to the application.

---

<a id="33-mutual-tls-mtls"></a>

# 33. Mutual TLS (mTLS)

Normal TLS

```text
Client

↓

Verify Server

↓

Server
```

Only the server is authenticated.

---

Mutual TLS

```text
Client

⇄

Server
```

Both verify each other's certificates.

---

Used In

* Banking
* Internal APIs
* Kubernetes Service Mesh
* Financial Systems
* Enterprise Microservices

---

<a id="34-ssl-in-aws"></a>

# 34. SSL in AWS

Common AWS services using SSL

* Application Load Balancer (ALB)
* Network Load Balancer
* API Gateway
* CloudFront
* Elastic Beanstalk

Certificates are managed using:

**AWS Certificate Manager (ACM)**

---

Example

```text
User

↓

HTTPS

↓

AWS ALB

↓

HTTP

↓

EC2

↓

Spring Boot
```

Here,

ALB performs SSL Termination.

---

<a id="35-ssl-in-ot-microservices"></a>

# 35. SSL in OT-Microservices

Example Architecture

```text
Browser
    │
HTTPS (443)
    │
    ▼
NGINX Reverse Proxy
    │
    ├───────────────┐
    ▼               ▼
Employee API    Attendance API
    │               │
    ▼               ▼
ScyllaDB      PostgreSQL

        │
        ▼
Salary API

        │
        ▼
Notification API
```

In most deployments:

* NGINX handles TLS.
* Internal APIs communicate over the private network.
* Backend services don't require individual certificates unless mTLS is implemented.

---

<a id="36-common-openssl-commands"></a>

# 36. Common OpenSSL Commands

## Generate RSA Private Key

```bash
openssl genrsa -out private.key 2048
```

---

## Generate CSR

```bash
openssl req -new \
-key private.key \
-out server.csr
```

---

## Generate Self-Signed Certificate

```bash
openssl req \
-x509 \
-newkey rsa:2048 \
-keyout private.key \
-out server.crt \
-days 365
```

---

## View Certificate

```bash
openssl x509 \
-in server.crt \
-text \
-noout
```

---

## Verify Certificate

```bash
openssl verify server.crt
```

---

## Test HTTPS Connection

```bash
openssl s_client \
-connect example.com:443
```

---

## Display Certificate Expiry

```bash
openssl x509 \
-enddate \
-noout \
-in server.crt
```

---

## Check Private Key

```bash
openssl rsa \
-in private.key \
-check
```

---

## Convert PEM to PKCS12

```bash
openssl pkcs12 \
-export \
-out keystore.p12 \
-inkey private.key \
-in server.crt
```

---

<a id="37-frequently-asked-interview-questions-part-3"></a>

# 37. Frequently Asked Interview Questions (Part 3)

| Question                                                     | Detailed Answer                                                                                                                                                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **How do you implement SSL on a web server?**                | Generate a private key, create a CSR, obtain a certificate from a Certificate Authority, install the certificate and private key on the web server, configure HTTPS (port 443), and restart the server. |
| **What is a CSR?**                                           | A Certificate Signing Request containing the server's public key and identity information. It is submitted to a Certificate Authority to obtain a certificate.                                          |
| **Can the private key be shared with the CA?**               | **No.** The private key must remain secret and never leave the server. Only the CSR (which contains the public key) is sent to the CA.                                                                  |
| **Why is the private key important?**                        | It decrypts the session key exchanged during the TLS handshake and proves ownership of the certificate. If compromised, attackers can impersonate the server.                                           |
| **What is SSL Termination?**                                 | SSL/TLS is decrypted at a reverse proxy or load balancer (e.g., Nginx or AWS ALB). Backend applications communicate using HTTP or optionally another TLS connection.                                    |
| **What is SSL Passthrough?**                                 | The load balancer forwards encrypted traffic to the backend without decrypting it. The backend server performs TLS termination.                                                                         |
| **What is Mutual TLS (mTLS)?**                               | mTLS authenticates both the client and the server using certificates. It is commonly used for internal microservice communication and highly secure enterprise systems.                                 |
| **Why do Kubernetes Ingress Controllers use SSL?**           | They terminate TLS at the cluster edge, simplifying certificate management and allowing backend services to receive traffic over HTTP or HTTPS as configured.                                           |
| **What is AWS Certificate Manager (ACM)?**                   | ACM is an AWS service that provisions, manages, and automatically renews SSL/TLS certificates for supported AWS services like ALBs, API Gateway, and CloudFront.                                        |
| **What is the difference between SSL Termination and mTLS?** | SSL Termination describes where encrypted traffic is decrypted. mTLS is an authentication mechanism where both the client and server present certificates to verify each other.                         |
