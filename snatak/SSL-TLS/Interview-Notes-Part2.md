# SSL/TLS Interview Notes (Part 2)

## Table of Contents

* [14. SSL/TLS Handshake](#14-ssltls-handshake)
* [15. Detailed SSL Handshake Flow](#15-detailed-ssl-handshake-flow)
* [16. Session Key](#16-session-key)
* [17. HTTPS Request Flow](#17-https-request-flow)
* [18. Certificate Signing Request (CSR)](#18-certificate-signing-request-csr)
* [19. Certificate Chain](#19-certificate-chain)
* [20. Trust Store](#20-trust-store)
* [21. Digital Signature](#21-digital-signature)
* [22. Hashing in SSL/TLS](#22-hashing-in-ssltls)
* [23. Perfect Forward Secrecy (PFS)](#23-perfect-forward-secrecy-pfs)
* [24. Frequently Asked Interview Questions (Part 2)](#24-frequently-asked-interview-questions-part-2)

---

<a id="14-ssltls-handshake"></a>

# 14. SSL/TLS Handshake

The **SSL/TLS Handshake** is the process that establishes a **secure connection** between the client and the server before any application data is exchanged.

Its objectives are:

* Authenticate the server
* Negotiate the TLS version
* Select encryption algorithms
* Exchange a secure session key
* Begin encrypted communication

---

## SSL Handshake Overview

```text
                Browser                           Server

                   │                                 │
                   │ -------- Client Hello --------> │
                   │                                 │
                   │ <------ Server Hello ---------- │
                   │ <----- Certificate ------------ │
                   │ <----- Public Key ------------- │
                   │                                 │
Verify Certificate │                                 │
Generate Session Key│                                │
                   │                                 │
Encrypt Session Key│                                 │
with Public Key    │                                 │
                   │ -------- Encrypted Key -------> │
                   │                                 │
                   │      Decrypt using              │
                   │      Private Key               │
                   │                                 │
================ Secure TLS Connection Established ================
                   │                                 │
          Encrypted Communication Starts
```

---

<a id="15-detailed-ssl-handshake-flow"></a>

# 15. Detailed SSL Handshake Flow

## Step 1 – Client Hello

The browser initiates the connection.

It sends:

* Supported TLS versions
* Supported Cipher Suites
* Random Number
* Compression Methods

```text
Browser

↓

Client Hello

↓

TLS 1.3

Cipher Suites

Random Number
```

---

## Step 2 – Server Hello

The server responds with:

* Selected TLS Version
* Selected Cipher Suite
* Server Random Number

```text
Server

↓

TLS 1.3

AES-256

Random Number
```

---

## Step 3 – Server Certificate

The server sends:

* SSL Certificate
* Public Key

```text
Server

↓

Certificate

↓

Public Key
```

---

## Step 4 – Certificate Verification

The browser verifies:

* Certificate is trusted
* Certificate is not expired
* Domain matches
* Certificate signed by trusted CA

```text
Certificate

↓

Trusted CA?

↓

YES

↓

Continue
```

If verification fails:

```text
Your Connection is Not Private
```

---

## Step 5 – Session Key Generation

The client generates a **random session key**.

Example

```text
Session Key

A7D91KX8PQ
```

---

## Step 6 – Encrypt Session Key

The browser encrypts the session key using the server's Public Key.

```text
Session Key

↓

Encrypt

↓

Server Public Key

↓

Encrypted Session Key
```

---

## Step 7 – Server Decrypts

Only the server can decrypt it.

```text
Encrypted Session Key

↓

Private Key

↓

Session Key
```

Now both client and server know the same session key.

---

## Step 8 – Secure Communication Begins

From this point onward,

all communication uses

**Symmetric Encryption**

```text
Browser

↓

AES Session Key

↓

HTTPS

↓

Server
```

---

<a id="16-session-key"></a>

# 16. Session Key

A **Session Key** is a temporary symmetric key generated during the SSL handshake.

It is used to encrypt all communication after the handshake.

---

## Why Not Use Public Key Every Time?

Public Key encryption is slow.

Example

```text
1000 HTTPS Requests
```

Using RSA for every request would be very expensive.

Instead

```text
Handshake

↓

Exchange Session Key

↓

AES Encryption

↓

Fast Communication
```

---

## Session Key Lifetime

The Session Key exists only for the duration of the connection.

When the connection closes,

the session key is discarded.

A new handshake generates a new session key.

---

<a id="17-https-request-flow"></a>

# 17. HTTPS Request Flow

```text
Browser

↓

DNS Lookup

↓

TCP Connection

↓

TLS Handshake

↓

HTTPS GET Request

↓

Server

↓

Encrypted Response

↓

Browser
```

---

### Complete HTTPS Flow

```text
Client
   │
DNS Lookup
   │
TCP 3-Way Handshake
   │
TLS Handshake
   │
Encrypted HTTP Request
   │
Web Server
   │
Application
   │
Database
   │
Encrypted HTTP Response
   │
Client
```

---

<a id="18-certificate-signing-request-csr"></a>

# 18. Certificate Signing Request (CSR)

A **CSR (Certificate Signing Request)** is a file generated before requesting an SSL certificate.

It contains:

* Domain Name
* Organization
* Country
* Public Key

It **does NOT contain the Private Key**.

---

## CSR Generation Flow

```text
Generate Private Key

↓

Generate CSR

↓

Send CSR to CA

↓

CA Verification

↓

Certificate Issued
```

---

## Why CSR?

The Certificate Authority uses the CSR to create the certificate.

---

<a id="19-certificate-chain"></a>

# 19. Certificate Chain

When a browser receives a certificate,

it doesn't trust it directly.

It verifies the entire certificate chain.

```text
Website Certificate

↓

Intermediate CA

↓

Root CA

↓

Trusted
```

---

## Certificate Chain Example

```text
example.com

↓

DigiCert Intermediate CA

↓

DigiCert Root CA

↓

Browser Trust Store
```

---

## Why Use Intermediate CA?

Root Certificates remain highly protected.

Intermediate CAs issue certificates on behalf of the Root CA.

This improves security.

---

<a id="20-trust-store"></a>

# 20. Trust Store

Every operating system and browser maintains a list of trusted Root CAs.

Examples:

* Windows Certificate Store
* Linux CA Bundle
* macOS Keychain
* Firefox Trust Store

When the server sends a certificate,

the browser checks:

```text
Certificate

↓

Root CA Found?

↓

YES

↓

Trusted
```

Otherwise

```text
Certificate Not Trusted
```

---

<a id="21-digital-signature"></a>

# 21. Digital Signature

A Digital Signature guarantees:

* Authenticity
* Integrity
* Non-Repudiation

Certificate Authorities digitally sign certificates.

---

Example

```text
Certificate

↓

Hash

↓

Encrypted using CA Private Key

↓

Digital Signature
```

Browser

↓

Decrypts using CA Public Key

↓

Verifies Hash

↓

Certificate Valid

---

<a id="22-hashing-in-ssltls"></a>

# 22. Hashing in SSL/TLS

Hashing ensures that data has not been modified.

Common algorithms

* SHA-256
* SHA-384

Example

```text
Original Data

↓

SHA256

↓

Hash Value

↓

Send

↓

Receiver Calculates Hash

↓

Compare

↓

Match

↓

Integrity Verified
```

If hashes differ,

the data was modified.

---

<a id="23-perfect-forward-secrecy-pfs"></a>

# 23. Perfect Forward Secrecy (PFS)

Modern TLS versions use **Ephemeral Keys**.

This feature is called

**Perfect Forward Secrecy (PFS).**

---

Without PFS

```text
Private Key Stolen

↓

Old Traffic

↓

Can Be Decrypted
```

---

With PFS

```text
Private Key Stolen

↓

Old Session Keys

↓

Not Recoverable

↓

Old Traffic Safe
```

This is why modern TLS prefers:

* ECDHE
* DHE

instead of static RSA key exchange.

---

<a id="24-frequently-asked-interview-questions-part-2"></a>

# 24. Frequently Asked Interview Questions (Part 2)

| **Question**                                                 | **Detailed Answer**                                                                                                                                                                                                                                                         |
| ------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **What is an SSL/TLS Handshake?**                            | The SSL/TLS Handshake is the process of establishing a secure connection between a client and server. During the handshake, the server authenticates itself, encryption algorithms are negotiated, a session key is securely exchanged, and encrypted communication begins. |
| **What is a Session Key?**                                   | A Session Key is a temporary symmetric encryption key generated during the TLS handshake. It encrypts all communication after the handshake because symmetric encryption is much faster than asymmetric encryption.                                                         |
| **What is a CSR?**                                           | A Certificate Signing Request (CSR) is a file sent to a Certificate Authority when requesting an SSL certificate. It contains the public key and identity information but never includes the private key.                                                                   |
| **What is a Certificate Chain?**                             | A Certificate Chain is the sequence of trust from the server certificate through one or more Intermediate CAs to a trusted Root CA. Browsers verify this chain before trusting the server.                                                                                  |
| **What is a Trust Store?**                                   | A Trust Store is a collection of trusted Root Certificate Authorities maintained by browsers and operating systems. Certificates signed by these trusted CAs are automatically accepted.                                                                                    |
| **What is a Digital Signature?**                             | A Digital Signature is created by encrypting a hash with the issuer's private key. It proves authenticity and integrity. Browsers verify it using the issuer's public key.                                                                                                  |
| **Why is hashing used in TLS?**                              | Hashing verifies that transmitted data has not been modified. If the calculated hash matches the received hash, data integrity is confirmed.                                                                                                                                |
| **What is Perfect Forward Secrecy (PFS)?**                   | PFS ensures that each TLS session uses a unique ephemeral session key. Even if the server's private key is compromised later, previously recorded sessions cannot be decrypted.                                                                                             |
| **Why is the Session Key encrypted using the Public Key?**   | Because only the server's private key can decrypt it, allowing both parties to securely establish a shared secret over an untrusted network.                                                                                                                                |
| **Why is a new Session Key generated for every connection?** | To improve security. If one session key is compromised, it does not affect previous or future TLS sessions.                                                                                                                                                                 |
