# <h1 align="center">Implementation - AWS Service Control Policies (SCPs)</h1>

<div align="center">
<img width="100" alt="AWS" src="https://upload.wikimedia.org/wikipedia/commons/9/93/Amazon_Web_Services_Logo.svg" />
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
    <td align="center">16/05/2026</td>
    <td align="center">1.0</td>
    <td align="center">Mukesh Kharb</td>
    <td align="center">16/05/2026</td>
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
2. [AWS SCP Overview](#2-aws-scp-overview)
3. [Prerequisites](#3-prerequisites)
4. [Create AWS Organization](#4-create-aws-organization)
5. [Create Organizational Units](#5-create-organizational-units)
6. [Create SCP Policy](#6-create-scp-policy)
7. [Attach SCP to Organizational Unit](#7-attach-scp-to-organizational-unit)
8. [Validate SCP Restrictions](#8-validate-scp-restrictions)
9. [Benefits of SCPs](#9-benefits-of-scps)
10. [Best Practices](#10-best-practices)
11. [FAQs](#11-faqs)
12. [Conclusion](#12-conclusion)
13. [Contact Information](#13-contact-information)
14. [References](#14-references)

---

<a id="1-introduction"></a>

## 1. Introduction

This POC demonstrates implementation of AWS Organizations Service Control Policies (SCPs) to enforce centralized governance and security restrictions across AWS accounts. SCPs help restrict unauthorized AWS actions, control resource provisioning, and standardize security controls for cloud infrastructure.

---

<a id="2-aws-scp-overview"></a>

## 2. AWS SCP Overview

| Component                | Description                                       |
| ------------------------ | ------------------------------------------------- |
| AWS Organizations        | Centralized multi-account management service      |
| SCP                      | Policy used to define maximum allowed permissions |
| Organizational Unit (OU) | Logical grouping of AWS accounts                  |
| Root Account             | Top-level organization container                  |
| IAM Policies             | Grant permissions inside accounts                 |
| SCP Enforcement          | Restricts IAM permissions at organization level   |

---

<a id="3-prerequisites"></a>

## 3. Prerequisites

| Requirement               | Version / Details                                   |
| ------------------------- | --------------------------------------------------- |
| AWS Account               | Active AWS Account with billing enabled             |
| AWS Organizations         | Enabled in Management Account                       |
| IAM Permissions           | `AWSOrganizationsFullAccess` or AdministratorAccess |
| Service Control Policies  | SCP feature enabled in AWS Organizations            |

---

<a id="4-create-aws-organization"></a>

## 4. Create AWS Organization

### Steps

| Step | Action                                      |
| ---- | ------------------------------------------- |
| 1    | Login to AWS Management Console             |
| 2    | Search for AWS Organizations                |
| 3    | Click Create an Organization                |
| 4    | Select Enable All Features                  |
| 5    | Verify organization creation from dashboard |

<details>
<summary>📸 <strong>Click to view Screenshot - AWS Organization Creation</strong></summary>
<img width="1838" height="950" alt="image" src="https://github.com/user-attachments/assets/b54d3b71-1966-481a-a6cd-0a0691f20c7c" />


</details>

---

<a id="5-create-organizational-units"></a>

## 5. Create Organizational Units

### Steps

| Step | Action                                                 |
| ---- | ------------------------------------------------------ |
| 1    | Open AWS Organizations                                 |
| 2    | Navigate to Organizational Units                       |
| 3    | Click Create new organizational unit                   |
| 4    | Enter OU names such as Development, QA, and Production |
| 5    | Select Parent Root                                     |
| 6    | Click Create organizational unit                       |

<details>
<summary>📸 <strong>Click to view Screenshot - Organizational Units</strong></summary>

<img width="1838" height="950" alt="image" src="https://github.com/user-attachments/assets/46d5bb96-9390-4242-8461-34dfbe279536" />


</details>

---

<a id="6-create-scp-policy"></a>

## 6. Create SCP Policy

### Steps

| Step | Action                            |
| ---- | --------------------------------- |
| 1    | Open AWS Organizations            |
| 2    | Navigate to Policies              |
| 3    | Select Service Control Policies   |
| 4    | Click Create Policy               |
| 5    | Enter policy name and description |
| 6    | Paste SCP JSON policy             |
| 7    | Click Create Policy               |

<details>
<summary>📸 <strong>Click to view Screenshot - Create SCP Policy</strong></summary>

<img width="1838" height="950" alt="image" src="https://github.com/user-attachments/assets/98251877-4c27-4373-8370-c67de31a18e4" />
<img width="1838" height="950" alt="image" src="https://github.com/user-attachments/assets/9163bab6-1a5a-4f66-801c-63a10c0c6624" />
<img width="1838" height="950" alt="image" src="https://github.com/user-attachments/assets/f906a926-4c83-4d13-97dd-7fe705c1e5d1" />
<img width="1838" height="950" alt="image" src="https://github.com/user-attachments/assets/a8711763-1418-4785-bd34-40a28830aef3" />


</details>

---

<a id="7-attach-scp-to-organizational-unit"></a>

## 7. Attach SCP to Organizational Unit

### Steps

| Step | Action                                      |
| ---- | ------------------------------------------- |
| 1    | Login to AWS Management Console             |
| 2    | Open AWS Organizations                      |
| 3    | Navigate to Organizational Units            |
| 4    | Select required Organizational Unit         |
| 5    | Open Policies tab                           |
| 6    | Under Service Control Policies click Attach |
| 7    | Select required SCP                         |
| 8    | Click Attach Policy                         |
| 9    | Verify SCP under attached policies          |

> [!NOTE]
> SCPs attached to Organizational Units automatically apply to all AWS accounts under that OU.

<details>
<summary>📸 <strong>Click to view Screenshot - Attach SCP to Organizational Unit</strong></summary>

<img width="1838" height="950" alt="image" src="https://github.com/user-attachments/assets/4625f1a2-bbfb-43a3-a12d-421f0dc15f9e" />



</details>

---

<a id="8-validate-scp-restrictions"></a>

## 8. Validate SCP Restrictions

### Steps

| Validation                            | Expected Result       |
| ------------------------------------- | --------------------- |
| Launch allowed EC2 instance type      | Successful deployment |
| Launch restricted EC2 instance type   | Access denied error   |
| Create resources in restricted region | Request denied        |
| Modify restricted AWS resources       | Operation blocked     |

<details>
<summary>📸 <strong>Click to view Screenshot - SCP Restriction Validation</strong></summary>
<img width="1838" height="950" alt="image" src="https://github.com/user-attachments/assets/5b6bfedd-8293-49f5-8458-b043d2222294" />
<img width="1691" height="930" alt="image" src="https://github.com/user-attachments/assets/0418d1e5-d28f-469b-ae30-c6a3e5d9987a" />
</details>

---

<a id="9-benefits-of-scps"></a>

## 9. Benefits of SCPs

| Benefit                | Description                                     |
| ---------------------- | ----------------------------------------------- |
| Centralized Governance | Manage permissions across multiple AWS accounts |
| Security Enforcement   | Prevent unauthorized actions                    |
| Compliance Alignment   | Standardize security controls                   |
| Reduced Risk           | Prevent accidental misconfigurations            |
| Multi-Account Control  | Apply consistent restrictions organization-wide |

---

<a id="10-best-practices"></a>

## 10. Best Practices

| Best Practice            | Description                                  |
| ------------------------ | -------------------------------------------- |
| Use Least Privilege      | Restrict only required actions               |
| Test SCPs Carefully      | Validate in non-production OUs first         |
| Avoid Full Deny Policies | Prevent accidental account lockout           |
| Separate OU Policies     | Apply environment-specific governance        |
| Monitor CloudTrail       | Track denied API calls and policy violations |

---

<a id="11-faqs"></a>

## 11. FAQs

### Q1. What is the primary purpose of AWS SCPs?

**Answer:**
SCPs define the maximum permissions allowed for AWS accounts within an organization.

---

### Q2. Do SCPs grant permissions to IAM users?

**Answer:**
No. SCPs only restrict permissions. IAM policies are still required to grant access.

---

### Q3. What happens if an SCP denies an action?

**Answer:**
The action is blocked even if IAM policies explicitly allow it.

---

### Q4. Why should SCPs be tested in development OUs first?

**Answer:**
Improper SCP configurations can block critical AWS operations across accounts.

---

### Q5. Can SCPs restrict AWS regions?

**Answer:**
Yes. SCPs can deny access to unsupported or unauthorized AWS regions.

---

### Q6. What is the benefit of SCPs in AWS Infrastructure?

**Answer:**
They enforce centralized governance and secure infrastructure standards across all AWS environments.

---

<a id="12-conclusion"></a>

## 12. Conclusion

AWS Organizations SCPs provide strong governance and centralized security control for AWS Infrastructure environments. By implementing SCP guardrails, organizations can reduce operational risk, standardize permissions, enforce compliance requirements, and improve infrastructure security across all AWS accounts.

---

<a id="13-contact-information"></a>

## 13. Contact Information

| Name         | ✉️ Contact                                                                        |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

<a id="14-references"></a>

## 14. References

| S.No | Description                          | Click to View                                                                                                                                                                                                 |
| ---- | ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | AWS Organizations Documentation      | [![AWS Organizations](https://img.shields.io/badge/AWS-Organizations-2F2F2F?style=flat-square\&logo=amazonaws\&logoColor=white)](https://docs.aws.amazon.com/organizations/)                                  |
| 2    | AWS SCP Documentation                | [![AWS SCP](https://img.shields.io/badge/AWS-SCP-2F2F2F?style=flat-square\&logo=amazonaws\&logoColor=white)](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)       |
| 3    | AWS Management Console Documentation | [![AWS Console](https://img.shields.io/badge/AWS-Console-2F2F2F?style=flat-square\&logo=amazonaws\&logoColor=white)](https://aws.amazon.com/console/)                                                         |
| 4    | IAM Policy Elements                  | [![IAM Policies](https://img.shields.io/badge/AWS-IAM_Policies-2F2F2F?style=flat-square\&logo=amazonaws\&logoColor=white)](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements.html) |
| 5    | AWS Security Best Practices          | [![AWS Security](https://img.shields.io/badge/AWS-Security-2F2F2F?style=flat-square\&logo=amazonaws\&logoColor=white)](https://aws.amazon.com/architecture/security-identity-compliance/)                     |
