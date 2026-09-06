# IAM

## Interview Questions & Answers

### Q1. What is IAM?

**Answer:** AWS IAM controls authentication and authorization to AWS resources. It includes identities such as users, groups, and roles and policies that determine allowed actions.

---

### Q2. What is an IAM user?

**Answer:** AWS IAM controls authentication and authorization to AWS resources. It includes identities such as users, groups, and roles and policies that determine allowed actions.

---

### Q3. What is an IAM group?

**Answer:** AWS IAM controls authentication and authorization to AWS resources. It includes identities such as users, groups, and roles and policies that determine allowed actions.

---

### Q4. What is an IAM role?

**Answer:** AWS IAM controls authentication and authorization to AWS resources. It includes identities such as users, groups, and roles and policies that determine allowed actions.

---

### Q5. When should an application use an IAM role instead of access keys?

**Answer:** AWS IAM controls authentication and authorization to AWS resources. It includes identities such as users, groups, and roles and policies that determine allowed actions.

---

### Q6. What is the difference between authentication and authorization?

**Answer:** Explain the identity/policy object, where it is attached, how AWS evaluates it, and the least-privilege/security implication. Remember that an explicit Deny overrides an Allow.

---

### Q7. What is an IAM policy?

**Answer:** AWS IAM controls authentication and authorization to AWS resources. It includes identities such as users, groups, and roles and policies that determine allowed actions.

---

### Q8. What is an identity-based policy?

**Answer:** An identity-based policy is attached to an IAM identity such as a user, group, or role and defines what actions that identity may perform on specified resources.

---

### Q9. What is a resource-based policy?

**Answer:** A resource-based policy is attached to a supported resource and specifies which principals can access the resource and under what conditions. S3 bucket policies and KMS key policies are common examples.

---

### Q10. What is a permissions boundary?

**Answer:** A permissions boundary sets the maximum permissions an IAM user or role can receive from identity-based policies. A boundary does not itself grant permissions.

---

### Q11. What is a Service Control Policy?

**Answer:** An SCP in AWS Organizations defines the maximum available permissions for principals in member accounts. It does not grant permissions; an IAM policy must still allow the requested action.

---

### Q12. What is a Resource Control Policy?

**Answer:** An RCP is an AWS Organizations policy that can place organization-level restrictions on access to supported resources. It acts as a guardrail rather than a grant of permissions.

---

### Q13. What is a session policy?

**Answer:** A session policy can further restrict permissions for a role session or federated session. It cannot grant permissions beyond what the underlying identity and other policy controls allow.

---

### Q14. What is a VPC endpoint policy?

**Answer:** A VPC is a logically isolated virtual network in AWS. It contains subnets, route tables, network interfaces, and security controls and can connect to the Internet, other VPCs, on-premises networks, or AWS services.

---

### Q15. What are ACLs in the IAM policy evaluation context?

**Answer:** AWS IAM controls authentication and authorization to AWS resources. It includes identities such as users, groups, and roles and policies that determine allowed actions.

---

### Q16. Does a permissions boundary grant permissions?

**Answer:** A permissions boundary sets the maximum permissions an IAM user or role can receive from identity-based policies. A boundary does not itself grant permissions.

---

### Q17. What is explicit deny?

**Answer:** An explicit Deny overrides an Allow during AWS policy evaluation. This is a fundamental IAM rule and is why organization guardrails and deny policies can prevent otherwise allowed actions.

---

### Q18. What is the effect of an explicit Deny?

**Answer:** An explicit Deny overrides an Allow during AWS policy evaluation. This is a fundamental IAM rule and is why organization guardrails and deny policies can prevent otherwise allowed actions.

---

### Q19. How are identity and resource policies evaluated?

**Answer:** A Terraform resource represents infrastructure that Terraform manages, such as an EC2 instance, security group, S3 bucket, or load balancer.

---

### Q20. How do SCPs constrain permissions?

**Answer:** A Service Control Policy is an AWS Organizations guardrail that limits the maximum permissions available in member accounts. It does not grant permissions by itself.

---

### Q21. What is least privilege?

**Answer:** Least privilege means granting only the permissions required to perform a task. In AWS this includes restricting actions, resources, conditions, principals, and credential lifetime where practical.

---

### Q22. What is policy inheritance?

**Answer:** Explain the identity/policy object, where it is attached, how AWS evaluates it, and the least-privilege/security implication. Remember that an explicit Deny overrides an Allow.

---

### Q23. What is MFA?

**Answer:** Multi-factor authentication requires an additional authentication factor beyond a password. It is especially important for privileged identities and account-root protection.

---

### Q24. What is an IAM access key?

**Answer:** AWS IAM controls authentication and authorization to AWS resources. It includes identities such as users, groups, and roles and policies that determine allowed actions.

---

### Q25. What is IAM Access Analyzer?

**Answer:** AWS IAM controls authentication and authorization to AWS resources. It includes identities such as users, groups, and roles and policies that determine allowed actions.

---

### Q26. What is STS?

**Answer:** AWS STS issues temporary security credentials for sessions and role assumption. Temporary credentials are time-limited and reduce the risks associated with long-lived keys.

---

### Q27. What is AssumeRole?

**Answer:** Explain the identity/policy object, where it is attached, how AWS evaluates it, and the least-privilege/security implication. Remember that an explicit Deny overrides an Allow.

---

### Q28. What is a trust policy?

**Answer:** A role trust policy is a resource-based policy that defines which principals are allowed to assume the role. It is different from the role's permissions policy, which defines allowed AWS actions after assumption.

---

### Q29. What is the difference between a trust policy and a permissions policy?

**Answer:** A role trust policy is a resource-based policy that defines which principals are allowed to assume the role. It is different from the role's permissions policy, which defines allowed AWS actions after assumption.

---
