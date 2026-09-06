# KMS

[⬅️ Back to AWS Topics](../README.md)

## 🔑 Keywords

🔑 KMS | Key Policy | CMK | AWS Managed | Envelope Encryption | Data Key | Rotation | Grant | Encryption Context

## 🧠 Core Memory

🧠 **Remember:** **KMS manages keys; it does not replace your application's data store.**

---

## ❓ Interview Questions

### 📌 Core

#### Q1. What is AWS KMS?

**💡 Answer:** AWS KMS is a managed key service used to create and control cryptographic keys for encrypting data and protecting other secrets. It integrates with many AWS services.

**🔑 Keywords:** `KMS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q2. What is a KMS key?

**💡 Answer:** AWS KMS is a managed key service used to create and control cryptographic keys for encrypting data and protecting other secrets. It integrates with many AWS services.

**🔑 Keywords:** `KMS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q3. What is a customer managed key?

**💡 Answer:** A customer managed KMS key is a key that the customer creates, controls through key policy/IAM/grants, and can configure for lifecycle and rotation behavior.

**🔑 Keywords:** `KMS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q4. What is an AWS managed key?

**💡 Answer:** An AWS managed KMS key is created and managed by an AWS service in the customer's account for that service's encryption needs. The service controls much of the key lifecycle.

**🔑 Keywords:** `KMS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q5. What is an AWS owned key?

**💡 Answer:** An AWS owned key is managed entirely by AWS and used by AWS services for supported encryption scenarios. It is not visible as a customer-managed key in the same way.

**🔑 Keywords:** `KMS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Encryption

#### Q6. What is envelope encryption?

**💡 Answer:** Envelope encryption encrypts data with a data key and then encrypts that data key with a KMS key. This avoids sending large data payloads through KMS and scales encryption efficiently.

**🔑 Keywords:** `KMS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q7. What is data key encryption?

**💡 Answer:** A KMS data key is generated for application-side or envelope encryption. The plaintext data key can encrypt data locally while its encrypted form can be stored with the ciphertext.

**🔑 Keywords:** `KMS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q8. What is key rotation?

**💡 Answer:** KMS key rotation changes the underlying key material used for new encryptions while keeping the key ID stable. AWS managed and customer managed keys have different rotation options and defaults.

**🔑 Keywords:** `KMS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q9. What is an encryption context?

**💡 Answer:** An encryption context is additional authenticated data supplied with supported KMS cryptographic operations. It can strengthen authorization conditions and must match when decrypting ciphertext created with that context.

**🔑 Keywords:** `KMS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q10. What is KMS grants?

**💡 Answer:** AWS KMS is a managed key service used to create and control cryptographic keys for encrypting data and protecting other secrets. It integrates with many AWS services.

**🔑 Keywords:** `KMS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Policy

#### Q11. What is a KMS key policy?

**💡 Answer:** AWS KMS is a managed key service used to create and control cryptographic keys for encrypting data and protecting other secrets. It integrates with many AWS services.

**🔑 Keywords:** `KMS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q12. How does IAM interact with KMS authorization?

**💡 Answer:** AWS IAM controls authentication and authorization to AWS resources. It includes identities such as users, groups, and roles and policies that determine allowed actions.

**🔑 Keywords:** `KMS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q13. Why is kms:Decrypt sensitive?

**💡 Answer:** AWS KMS is a managed key service used to create and control cryptographic keys for encrypting data and protecting other secrets. It integrates with many AWS services.

**🔑 Keywords:** `KMS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q14. How can KMS integrate with S3?

**💡 Answer:** AWS KMS is a managed key service used to create and control cryptographic keys for encrypting data and protecting other secrets. It integrates with many AWS services.

**🔑 Keywords:** `KMS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q15. How can KMS integrate with EBS and RDS?

**💡 Answer:** Amazon EBS provides persistent block storage for EC2. EBS volumes can be used for boot disks, filesystems, and databases, and snapshots can provide backup and recovery.

**🔑 Keywords:** `KMS` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

## 🚀 Last-Minute Revision

> 🧠 **Remember:** **KMS manages keys; it does not replace your application's data store.**

[⬆️ Back to top](#kms)

[⬅️ Back to AWS Topics](../README.md)