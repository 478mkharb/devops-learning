# IAM

### Q1. What is IAM?

**Answer:** AWS Identity and Access Management (IAM) controls authentication and authorization for AWS resources. IAM manages identities such as users, groups, and roles, and uses policies to determine what actions those identities can perform.

---

### Q2. What is an IAM user?

**Answer:** An IAM user is an identity created in an AWS account for a person or application that needs AWS access. It can have credentials such as a password or access keys. For AWS workloads, IAM roles with temporary credentials are generally preferred.

---

### Q3. What is an IAM group?

**Answer:** An IAM group is a collection of IAM users. Policies attached to the group apply to its users and simplify permission management for users with similar responsibilities.

---

### Q4. What is an IAM role?

**Answer:** An IAM role is an identity that trusted principals can assume to obtain temporary security credentials. Roles are commonly used by EC2, Lambda, ECS tasks, federated users, applications, and cross-account access.

---

### Q5. When should an application use an IAM role instead of access keys?

**Answer:** An application should use an IAM role whenever the AWS service supports role-based temporary credentials. This avoids storing long-lived access keys and allows AWS to provide short-lived credentials.

---

### Q6. What is the difference between authentication and authorization?

**Answer:** Authentication determines **who the principal is**. Authorization determines **what that principal is allowed to do**.

```text
Authentication → Who are you?
Authorization  → What can you do?
```

---

### Q7. What is an IAM policy?

**Answer:** An IAM policy is a JSON permissions document that defines whether actions are allowed or denied. Common elements are `Effect`, `Action`, `Resource`, and `Condition`. Resource-based policies and trust policies can also use `Principal`.

---

### Q8. What is an identity-based policy?

**Answer:** An identity-based policy is attached to an IAM user, group, or role and defines what actions that identity can perform on specified resources.

---

### Q9. What is a resource-based policy?

**Answer:** A resource-based policy is attached to a supported AWS resource and defines which principals can access that resource and under what conditions. An S3 bucket policy is a common example.

---

### Q10. What is an inline policy?

**Answer:** An inline policy is embedded directly into a single IAM user, group, or role. It has a one-to-one relationship with that identity and is deleted when the identity is deleted.

Inline policies are useful when permissions are intentionally specific to one identity and should not be reused independently.

---

### Q11. What is a managed policy?

**Answer:** A managed policy is a standalone IAM policy that can be attached to supported IAM identities. Managed policies are independent policy objects and can be reused.

There are two types:

```text
Managed Policy
├── AWS managed policy
└── Customer managed policy
```

---

### Q12. What is an AWS managed policy?

**Answer:** An AWS managed policy is created and maintained by AWS. AWS can update its permissions as services and recommended permissions evolve.

Example:

```text
AmazonS3ReadOnlyAccess
```

AWS managed policies are convenient but can be broader than a custom least-privilege policy.

---

### Q13. What is a customer managed policy?

**Answer:** A customer managed policy is a standalone IAM policy created and controlled by the customer. It provides control over actions, resources, conditions, policy versions, and reuse across identities.

---

### Q14. What is the difference between inline and managed policies?

**Answer:**

| Inline Policy | Managed Policy |
|---|---|
| Embedded in one identity | Standalone policy |
| One-to-one with an identity | Can be attached to multiple identities |
| Not independently reusable | Designed for reuse |
| Deleted with the identity | Exists independently |

---

### Q15. What is the difference between AWS managed and customer managed policies?

**Answer:**

| AWS Managed | Customer Managed |
|---|---|
| Created by AWS | Created by customer |
| Maintained by AWS | Maintained by customer |
| AWS controls updates | Customer controls updates |
| Convenient for common permissions | Better for customized least privilege |

---

### Q16. What is the difference between identity-based and resource-based policies?

**Answer:**

| Identity-Based | Resource-Based |
|---|---|
| Attached to user, group, or role | Attached to supported resource |
| Defines what the identity can access | Defines who can access the resource |
| Common example: role policy | Common example: S3 bucket policy |

---

### Q17. Can an IAM user or role have both inline and managed policies?

**Answer:** Yes. An IAM user, group, or role can have applicable AWS managed policies, customer managed policies, and inline policies. AWS evaluates applicable policies together when authorizing a request.

---

### Q18. What is a permissions boundary?

**Answer:** A permissions boundary defines the maximum permissions an IAM user or role can receive through identity-based policies.

A permissions boundary **does not grant permissions by itself**.

---

### Q19. Does a permissions boundary grant permissions?

**Answer:** No. An identity-based policy must grant the permission, and the permissions boundary limits the maximum permissions available to that identity.

---

### Q20. What is an explicit deny?

**Answer:** An explicit `Deny` prevents an action even if another applicable policy contains an `Allow`.

```text
Allow + Deny → Deny
```

Explicit Deny takes precedence over Allow.

---

### Q21. What is least privilege?

**Answer:** Least privilege means granting only the permissions required for a task. Policies should restrict actions, resources, conditions, and principals as appropriate.

---

### Q22. What is a trust policy?

**Answer:** A trust policy is a resource-based policy attached to an IAM role. It specifies which principals are allowed to assume the role.

```text
Trust Policy → Who can assume the role?
```

It is separate from the role's permissions policies.

---

### Q23. What is the difference between a trust policy and a permissions policy?

**Answer:**

| Trust Policy | Permissions Policy |
|---|---|
| Controls who can assume the role | Controls what the role can do |
| Defines trusted principals | Defines allowed/denied actions |
| Used during role assumption | Used when the role session accesses resources |

Memory:

> **Trust policy = Who can assume? Permissions policy = What can they do?**

---

### Q24. What is AWS STS?

**Answer:** AWS Security Token Service (STS) provides temporary security credentials for AWS access. It is commonly used for role assumption, federation, and cross-account access.

Temporary credentials contain an access key ID, secret access key, and session token.

---

### Q25. What is AssumeRole?

**Answer:** `AssumeRole` is an STS operation that allows a trusted principal to obtain temporary credentials for an IAM role.

```text
Principal
   ↓
STS AssumeRole
   ↓
IAM Role
   ↓
Temporary Credentials
```

The trust policy controls who can assume the role; permissions policies control what the resulting session can do.

---

### Q26. What is an instance profile?

**Answer:** An instance profile is a container for an IAM role that allows the role to be associated with an EC2 instance.

```text
EC2
 ↓
Instance Profile
 ↓
IAM Role
 ↓
Temporary Credentials
```

---

### Q27. How does an EC2 instance obtain IAM role credentials?

**Answer:** An EC2 instance associated with an IAM role through an instance profile can obtain temporary credentials through the EC2 Instance Metadata Service (IMDS). Applications can use these credentials without storing long-term access keys.

---

### Q28. What is an IAM access key?

**Answer:** An IAM access key is a long-term programmatic credential consisting of an access key ID and secret access key. Long-term access keys require lifecycle management and should generally be avoided for AWS workloads when temporary role credentials are available.

---

### Q29. What is the difference between long-term and temporary credentials?

**Answer:**

| Long-Term Credentials | Temporary Credentials |
|---|---|
| IAM user access keys | STS/role credentials |
| Remain valid until rotated/deactivated | Automatically expire |
| Require credential management | Short-lived |
| Generally less preferred for workloads | Preferred for AWS workloads |

---

### Q30. What is IAM policy evaluation?

**Answer:** AWS evaluates applicable policies to determine whether a request is authorized.

At a high level:

```text
Request
  ↓
Applicable policies
  ↓
Explicit Deny?
 ├── Yes → Deny
 └── No
      ↓
Applicable Allow?
 ├── Yes → Allow
 └── No  → Deny
```

An explicit Deny overrides an Allow.

---

### Q31. How do identity-based and resource-based policies work together?

**Answer:** AWS evaluates applicable identity-based and resource-based policies together. A request must satisfy the applicable authorization rules, and an explicit Deny overrides an Allow.

This is particularly important for cross-account access, where the appropriate permissions and trust/resource policies must be configured.

---

### Q32. What is IAM managed policy versioning?

**Answer:** IAM managed policies can have multiple policy versions. One version is designated as the **default version**, which is used for permissions evaluation.

Policy versions allow controlled policy changes and rollback to a previous version.

---

### Q33. Do inline policies have independent policy versions?

**Answer:** No. Managed policies have independent policy versions. Inline policies are embedded in their IAM identity and do not have standalone managed-policy versions.

---

### Q34. What is MFA in IAM?

**Answer:** Multi-factor authentication (MFA) requires an additional authentication factor beyond the primary credential. MFA is especially important for privileged access and protecting the AWS account root user.

IAM policy conditions can also require MFA for specific actions.

---

### Q35. What is IAM Access Analyzer?

**Answer:** IAM Access Analyzer helps identify unintended external access to supported resources and helps validate IAM policies. It can support least-privilege policy development and resource-sharing reviews.

---

### Q36. What is a service-linked role?

**Answer:** A service-linked role is a special IAM role linked directly to an AWS service. The service uses the role to perform required actions in the customer's account.

The permissions required by a service-linked role are defined by the associated AWS service.

---

### Q37. What is the difference between an IAM role and a service-linked role?

**Answer:**

| IAM Role | Service-Linked Role |
|---|---|
| General-purpose identity | Specifically linked to an AWS service |
| Used by applications, users, cross-account access, etc. | Used by the associated AWS service |
| Customer configures its trust and permissions within AWS rules | Permissions are defined by the linked service |

---

### Q38. What is cross-account IAM role access?

**Answer:** Cross-account access allows a principal in one AWS account to assume a role in another AWS account and receive temporary credentials.

```text
Account A
Principal
   ↓
STS AssumeRole
   ↓
Account B
IAM Role
   ↓
AWS Resources
```

The target role must trust the appropriate principal, and the resulting session must have the required permissions.

---

### Q39. Why are IAM roles preferred for cross-account access?

**Answer:** Roles provide temporary credentials instead of requiring long-term access keys to be shared between accounts. This reduces credential-management risk and provides a controlled trust relationship.

---

### Q40. What is the difference between an IAM user, group, and role?

**Answer:**

| User | Group | Role |
|---|---|---|
| Individual IAM identity | Collection of users | Assumable identity |
| Can have long-term credentials | Has no credentials of its own | Provides temporary credentials when assumed |
| Can have policies | Policies apply to member users | Has trust and permissions policies |
| Used for specific identity access | Simplifies user permissions | Preferred for AWS workloads and temporary/cross-account access |

---

### Q41. What are `Effect`, `Action`, `Resource`, and `Condition` in an IAM policy?

**Answer:**

| Element | Purpose |
|---|---|
| `Effect` | `Allow` or `Deny` |
| `Action` | AWS API actions |
| `Resource` | Resources affected |
| `Condition` | Conditions under which the statement applies |

Example:

```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::my-bucket/*"
}
```

---

### Q42. What is the Principal element in an IAM policy?

**Answer:** `Principal` identifies the AWS principal to which a resource-based policy grants or denies access.

It is also central to role trust policies because the trust policy identifies who or what can assume the role.

---

### Q43. What is an IAM policy condition?

**Answer:** A `Condition` restricts when a policy statement applies. Conditions can use request context such as source IP, AWS Region, MFA status, VPC endpoint, resource tags, or principal attributes.

Example:

```json
"Condition": {
  "Bool": {
    "aws:MultiFactorAuthPresent": "true"
  }
}
```

---

### Q44. What is IAM Identity Center?

**Answer:** IAM Identity Center provides centralized workforce access to AWS accounts and applications. It supports identity federation and centralized permission assignment for users and groups.

It is commonly used instead of creating individual IAM users for human workforce access in multi-account environments.

---

### Q45. What is the recommended IAM approach for an EC2 application that needs S3 access?

**Answer:** Create an IAM role with only the required S3 permissions, associate it with the EC2 instance through an instance profile, and let the application obtain temporary credentials through IMDS.

```text
EC2
 ↓
Instance Profile
 ↓
IAM Role
 ↓
Least-Privilege Policy
 ↓
Specific S3 Actions
 ↓
Specific S3 Resources
```

Do not store long-term AWS access keys on the EC2 instance when role-based credentials are available.
