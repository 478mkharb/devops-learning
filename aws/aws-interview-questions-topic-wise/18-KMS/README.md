# KMS

### Q1. What is AWS KMS?

**Answer:** AWS KMS is a managed service for creating and controlling cryptographic keys used to protect data. It integrates with many AWS services and provides authorization and audit controls.

---

### Q2. What is a KMS key?

**Answer:** A KMS key is a logical cryptographic key resource managed by AWS KMS. Its key policy, IAM permissions, grants, and related controls determine how it can be used.

---

### Q3. What is a customer managed key?

**Answer:** A customer managed KMS key is created and controlled by the customer, including its key policy and lifecycle settings. It provides more direct control than an AWS managed key.

---

### Q4. What is an AWS managed key?

**Answer:** An AWS managed KMS key is created and managed by an AWS service in the customer's account for that service's encryption needs. The customer has less lifecycle control than with a customer managed key.

---

### Q5. What is an AWS owned key?

**Answer:** An AWS owned key is managed entirely by AWS and is used by AWS services for supported encryption scenarios. It is not a customer-managed key resource.

---

### Q6. What is envelope encryption?

**Answer:** Envelope encryption encrypts data with a data key and then encrypts that data key with a KMS key. This allows large amounts of data to be encrypted efficiently without sending the entire payload through KMS.

---

### Q7. What is data key encryption?

**Answer:** A KMS data key is used for envelope encryption. The plaintext data key can encrypt application data locally, while the encrypted data key can be stored with the ciphertext.

---

### Q8. What is key rotation?

**Answer:** KMS key rotation changes the cryptographic key material used for new encryption operations while retaining the logical KMS key identity. Rotation behavior depends on the key type and configuration.

---

### Q9. What is an encryption context?

**Answer:** An encryption context is additional authenticated data supplied to supported KMS cryptographic operations. It can be used in authorization conditions and must match during decryption.

---

### Q10. What is KMS grants?

**Answer:** A KMS grant is a delegation mechanism that provides a principal with permission to use a KMS key for supported operations without requiring a permanent key-policy statement for every authorization scenario.

---

### Q11. What is a KMS key policy?

**Answer:** A KMS key policy is the primary resource-based policy for a KMS key. It controls which principals can administer or use the key, together with IAM policies and grants where applicable.

---

### Q12. How does IAM interact with KMS authorization?

**Answer:** KMS authorization can involve the KMS key policy, IAM policies, grants, and conditions. An IAM Allow alone is not sufficient in cases where the key policy does not permit the account/principal model required for access. The effective permission is the result of all applicable controls, with explicit Deny taking precedence.

---

### Q13. Why is kms:Decrypt sensitive?

**Answer:** `kms:Decrypt` can turn encrypted ciphertext into plaintext. If granted too broadly, a principal may be able to read sensitive application data protected by the key, so access should be restricted to the required key, principals, resources, and conditions.

---

### Q14. How can KMS integrate with S3?

**Answer:** S3 can use KMS keys for SSE-KMS encryption. When an authorized caller accesses an encrypted object, S3 uses KMS as part of the encryption/decryption process, and the caller needs both the appropriate S3 permissions and KMS permissions.

---

### Q15. How can KMS integrate with EBS and RDS?

**Answer:** Amazon RDS is a managed relational database service. AWS manages much of the underlying infrastructure, provisioning, backups, patching, and high-availability plumbing while the customer manages database data, schema, and configuration.

---
