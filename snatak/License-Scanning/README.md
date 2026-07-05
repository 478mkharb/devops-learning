# License Scanning - DevOps Interview Notes

## 1. What is License Scanning?

**License Scanning** is the process of identifying the licenses of third-party open-source dependencies used in an application and verifying whether they comply with an organization's legal and compliance policies.

**Purpose**
- Ensure legal compliance
- Avoid restricted licenses
- Reduce legal risks
- Enforce organizational OSS policies

---

# 2. Why is License Scanning Important?

- Prevents legal and compliance issues
- Identifies prohibited licenses
- Ensures safe use of open-source software
- Supports software governance
- Helps maintain license compliance before release

---

# 3. Difference Between License Scanning and Vulnerability Scanning

| License Scanning | Vulnerability Scanning |
|------------------|------------------------|
| Checks software licenses | Checks security vulnerabilities (CVEs) |
| Focuses on legal compliance | Focuses on security risks |
| Detects GPL, MIT, Apache, BSD, etc. | Detects CVEs and known exploits |
| Ensures policy compliance | Ensures application security |

---

# 4. Types of License Scanning

| Type | Purpose |
|------|---------|
| Dependency License Scanning | Scans third-party libraries (Maven, npm, pip, Go Modules) |
| Source Code License Scanning | Detects license headers and copied source code |
| Container/Image License Scanning | Scans OS packages and software inside container images |
| Binary/Artifact License Scanning | Scans compiled artifacts like JAR, WAR, DLL, EXE |

> **Interview Tip:** The most commonly used type in DevOps pipelines is **Dependency License Scanning**.

---

# 5. Types of Open Source Licenses

| License Category | Examples | Enterprise Friendly |
|------------------|----------|---------------------|
| Permissive | MIT, Apache 2.0, BSD, ISC | ✅ Yes |
| Weak Copyleft | LGPL, MPL | ⚠ Depends on policy |
| Strong Copyleft | GPLv2, GPLv3, AGPL | ❌ Usually Restricted |

| License  | Full Form                         | Type            | Commercial Use | Key Rule                                                                                               | Risk Level   |
| -------- | --------------------------------- | --------------- | -------------- | ------------------------------------------------------------------------------------------------------ | ------------ |
| **GPL**  | GNU General Public License        | Strong Copyleft | ⚠️ Limited     | If you distribute software using GPL code, you must release your source code under GPL.                | 🔴 High      |
| **LGPL** | GNU Lesser General Public License | Weak Copyleft   | ✅ Yes          | You can use it in proprietary software, but modifications to the LGPL library must remain open source. | 🟡 Medium    |
| **MPL**  | Mozilla Public License            | Weak Copyleft   | ✅ Yes          | Only modified MPL files must be open source; the rest of your application can remain proprietary.      | 🟡 Medium    |
| **AGPL** | GNU Affero General Public License | Strong Copyleft | ⚠️ Limited     | Even if software is used over a network (SaaS), the source code must be made available to users.       | 🔴 Very High |

## Easy Analogy

Imagine someone gives you a recipe.

### Permissive License (MIT/Apache)

"Take my recipe. Modify it. Sell it. You don't have to tell anyone your changes."

### Copyleft License (GPL)

"Take my recipe. Modify it. Sell it if you want. But if you share or sell it, you must also share your modified recipe."

---

# 6. Common License Examples

| License | Commercial Use | Modification | Patent Grant |
|----------|----------------|--------------|--------------|
| MIT | ✅ | ✅ | ❌ |
| Apache 2.0 | ✅ | ✅ | ✅ |
| BSD | ✅ | ✅ | ❌ |
| LGPL | ✅ | Limited | ❌ |
| GPL | Limited | Open Source Required | ❌ |
| AGPL | Limited | Network Source Disclosure Required | ❌ |

---

# 7. Commonly Allowed and Restricted Licenses

## Allowed

- MIT
- Apache 2.0
- BSD
- ISC

## Usually Restricted

- GPLv2
- GPLv3
- AGPL

> **Note:** Actual policies vary by organization.

---

# 8. Popular License Scanning Tools

| Tool | Open Source | License Scanning | Vulnerability Scanning |
|------|-------------|------------------|-------------------------|
| Black Duck | ❌ | ✅ | ✅ |
| Mend (WhiteSource) | ❌ | ✅ | ✅ |
| FOSSA | ❌ | ✅ | Limited |
| Snyk | Freemium | ✅ | ✅ |
| Trivy | ✅ | ✅ | ✅ |
| ScanCode Toolkit | ✅ | ✅ | ❌ |
| FOSSology | ✅ | ✅ | ❌ |
| Syft | ✅ | ✅ | ❌ |
| ORT | ✅ | ✅ | Limited |

---

# 9. Which Tool is Best?

| Requirement | Best Tool |
|------------|-----------|
| Enterprise License Compliance | **Black Duck** |
| Enterprise DevSecOps | **Mend (WhiteSource)** |
| Container License Scanning | **Trivy** |
| Source Code License Analysis | **ScanCode Toolkit** |
| Legal Compliance | **FOSSology** |
| SBOM Generation | **Syft** |
| Developer-Friendly | **Snyk** |

---

# 10. License Scanning in CI/CD Pipeline

```text
Developer
     │
     ▼
Git Push
     │
     ▼
Build
     │
     ▼
Dependency Installation
     │
     ▼
License Scanning
     │
     ▼
Dependency (SCA) Scanning
     │
     ▼
SAST
     │
     ▼
Unit Testing
     │
     ▼
Build Artifact
     │
     ▼
Container Scanning
     │
     ▼
Deploy
```

---

# 11. Frequently Asked Interview Questions

| Question | Short Answer |
|----------|--------------|
| What is License Scanning? | Identifies software licenses of OSS dependencies and checks compliance. |
| Why is License Scanning important? | Prevents legal and compliance risks. |
| What is scanned? | Third-party libraries, source code, containers, and binaries. |
| Which type is most common? | Dependency License Scanning. |
| Difference between License Scanning and SCA? | License Scanning checks licenses; SCA checks vulnerabilities and licenses. |
| Difference between License Scanning and SAST? | License Scanning checks legal compliance; SAST analyzes source code for security issues. |
| Which licenses are usually allowed? | MIT, Apache 2.0, BSD, ISC. |
| Which licenses are usually restricted? | GPL, AGPL. |
| Which tool is best for enterprise? | Black Duck or Mend (WhiteSource). |
| Which tool is best for containers? | Trivy. |
| Which tool generates SBOM? | Syft. |
| Can License Scanning fail a pipeline? | Yes, if a prohibited license violates organizational policy. |

---

# 12. One-Line Interview Answers

| Question | Answer |
|----------|--------|
| What is License Scanning? | It identifies open-source licenses used in an application and verifies compliance with organizational policies. |
| What is the goal? | To avoid legal risks and ensure license compliance. |
| Most common scanning type? | Dependency License Scanning. |
| Most enterprise-friendly license? | Apache 2.0. |
| Most commonly restricted license? | GPL/AGPL. |
| Best enterprise tool? | Black Duck. |
| Best open-source tool? | Trivy. |
| Best source code license scanner? | ScanCode Toolkit. |
| Best SBOM tool? | Syft. |

---

# Interview Tip

For DevOps interviews, remember these **5 keywords**:

- **Compliance**
- **Open Source Licenses**
- **Dependency Scanning**
- **Policy Enforcement**
- **Black Duck / Trivy**
