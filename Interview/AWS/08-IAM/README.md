# AWS IAM — Senior DevOps Notes

## 1. IAM at a Glance

| Concept | Meaning | DevOps relevance |
|---|---|---|
| IAM User | Long-lived identity for a person or legacy workload | Avoid for CI/CD and EC2 workloads |
| IAM Group | Collection of users | Simplifies human permission administration |
| IAM Role | Identity assumed by a trusted principal | Preferred for EC2, Jenkins, Terraform and cross-account access |
| IAM Policy | JSON document defining permissions | Controls what an identity or resource can do |
| Authentication | Verifies who/what is requesting access | Password, MFA, federation or temporary credentials |
| Authorization | Determines whether the request is allowed | Evaluated through IAM policies and guardrails |
| STS | AWS Security Token Service | Issues temporary credentials through role assumption |

**Senior principle:** authenticate with the strongest available identity mechanism, authorize with least privilege, and prefer short-lived credentials over static access keys.

---

## 2. IAM Policy Types

| Policy type | Attached to | Main purpose |
|---|---|---|
| Identity-based | User, group or role | Defines what the identity can do |
| Resource-based | Supported AWS resource | Defines who can access the resource |
| AWS-managed | Managed by AWS | Convenient but may grant broader permissions than required |
| Customer-managed | Managed by your organization | Reusable and version-controlled least-privilege policy |
| Inline | Embedded in one identity/resource | One-to-one policy; harder to reuse and govern |
| Permissions boundary | User or role | Maximum permissions an identity can receive |
| Session policy | Role session/federated session | Further restricts permissions for that session |
| SCP | AWS Organizations account/OU | Maximum permissions available in an account; does not grant access |
| Resource control policy | Supported AWS resources | Organization-level resource guardrail where supported |

### Policy design recommendation

| Prefer | Avoid |
|---|---|
| Customer-managed policies for reusable permissions | Large inline policies scattered across identities |
| Resource-level ARNs | `Resource: "*"` when a narrower ARN is possible |
| Explicit conditions | Unrestricted actions and principals |
| Separate read, write and administrative permissions | One policy granting full administration |
| Version-controlled policy documents | Manual console-only policy changes |

---

## 3. IAM Policy Evaluation

A request is allowed only when AWS finds an applicable **Allow** and no applicable **explicit Deny**.

| Evaluation factor | Effect |
|---|---|
| Identity-based Allow | Can grant permission |
| Resource-based Allow | Can grant permission where supported |
| Permissions boundary | Limits identity permissions |
| SCP | Limits account/OU permissions |
| Session policy | Limits the active session |
| Explicit Deny | Overrides applicable Allows |
| Missing Allow | Request is denied by default |

### Simplified evaluation flow

```text
Request
  ↓
Authenticate principal
  ↓
Collect identity/resource policies
  ↓
Apply SCPs, boundaries and session policies
  ↓
Check explicit Deny
  ↓
Applicable Allow + no Deny = Allowed
Otherwise = Denied
```

**Important:** a permissions boundary or SCP does not grant permissions. They only restrict what could otherwise be granted.

---

## 4. Trust Policy vs Permissions Policy

| Policy | Answers | Attached to | Example |
|---|---|---|---|
| Trust policy | Who may assume this role? | Role | EC2 service, Jenkins role, another AWS account |
| Permissions policy | What may the role do after assumption? | Role/user/group | `ec2:DescribeInstances`, `s3:GetObject` |

A role requires both:

1. A trust policy that permits the caller to assume it.
2. Permissions policies that authorize actions after assumption.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "Service": "ec2.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

---

## 5. IAM Roles, STS and Temporary Credentials

| Concept | Explanation |
|---|---|
| AssumeRole | Requests temporary credentials for a role |
| Temporary credentials | Access key, secret key and session token with an expiry |
| Role session | The temporary security context created after assumption |
| Role chaining | Assuming another role using temporary credentials; session duration is restricted |
| External ID | Helps protect cross-account third-party access against confused-deputy attacks |
| Session tags | Pass context into a role session for authorization and auditing |

### Why temporary credentials are preferred

- No permanent secret stored in source code or Jenkins credentials.
- Automatic expiry limits the impact of credential leakage.
- Access can be centrally controlled through role trust policies.
- CloudTrail records the assumed-role session identity.

---

## 6. EC2 Instance Profiles

An EC2 instance should obtain AWS permissions through an **IAM role attached via an instance profile**.

| Component | Purpose |
|---|---|
| IAM role | Defines trust and permissions |
| Instance profile | EC2 container for the role |
| EC2 metadata service | Supplies temporary credentials to the instance |
| IMDSv2 | Token-based metadata access; recommended for production |

### Recommended pattern

```text
EC2 instance
   ↓
Instance profile
   ↓
IAM role
   ↓
Temporary credentials from IMDSv2
   ↓
AWS API access
```

Do not place access keys in `/etc/environment`, application files, AMIs or user-data scripts.

---

## 7. Jenkins and Terraform Authentication

| Workload | Preferred authentication |
|---|---|
| Jenkins on EC2 | Instance profile attached to the Jenkins host, or workload federation where supported |
| Jenkins external to AWS | OIDC/federation or a tightly scoped role assumption flow |
| Terraform on EC2 | Instance profile with only required infrastructure permissions |
| Terraform from CI platform | OIDC federation into an AWS role |
| Human administrator | IAM Identity Center/federated identity with MFA |
| Cross-account deployment | Assume a deployment role in the target account |

### CI/CD separation of duties

| Role | Typical permissions |
|---|---|
| Terraform plan role | Read state, inspect resources, calculate changes |
| Terraform apply role | Required create/update/delete permissions for the stack |
| Deployment role | Deploy application artifacts and restart/update services |
| Read-only audit role | Read-only access and security investigation |

Avoid giving Jenkins unrestricted `AdministratorAccess` merely because Terraform is used.

---

## 8. `iam:PassRole`

`iam:PassRole` is required when a principal passes an IAM role to an AWS service, such as:

- EC2 instance profile
- Lambda execution role
- ECS task role
- CloudFormation service role
- Auto Scaling launch template role

| Permission | Meaning |
|---|---|
| `iam:PassRole` | Allows passing a specific role to a service |
| `sts:AssumeRole` | Allows assuming a role and receiving temporary credentials |

These are different permissions. Restrict `iam:PassRole` to approved role ARNs and, where possible, use `iam:PassedToService` conditions.

---

## 9. Cross-Account Access

```text
Account A: caller role/user
        │
        │ sts:AssumeRole
        ▼
Account B: target role
        │
        ▼
Target-account permissions
```

Both sides must be configured correctly:

| Location | Requirement |
|---|---|
| Target role trust policy | Trusts the source account or source role |
| Source identity policy | Allows `sts:AssumeRole` on the target role ARN |
| Target role permissions | Grants required actions on target resources |
| SCPs/boundaries | Must not block the request |

Use an explicit role ARN, external ID for third parties, session tags where useful, and short session durations.

---

## 10. SCP vs IAM Policy vs Permissions Boundary

| Control | Scope | Grants access? | Main use |
|---|---|---:|---|
| IAM policy | Identity/resource | Yes | Normal authorization |
| Permissions boundary | Individual user/role | No | Limit delegated administrators/workloads |
| SCP | Account or OU | No | Organization-wide guardrails |
| Session policy | One session | No | Restrict temporary session permissions |

**Example:** An IAM role may allow `ec2:RunInstances`, but an SCP denying launches outside approved regions still blocks the request.

---

## 11. IAM Conditions

Conditions make policies context-aware and reduce broad permissions.

| Condition key | Typical use |
|---|---|
| `aws:RequestedRegion` | Restrict actions to approved regions |
| `aws:SourceIp` | Restrict access from approved networks |
| `aws:MultiFactorAuthPresent` | Require MFA for sensitive actions |
| `aws:SecureTransport` | Deny non-TLS requests to supported services |
| `aws:PrincipalArn` | Restrict trusted principals |
| `aws:SourceVpce` | Restrict access through a VPC endpoint |
| `aws:ResourceTag/<key>` | Control access based on resource tags |
| `aws:RequestTag/<key>` | Require tags during resource creation |
| `iam:PassedToService` | Limit role passing to a service |

Use conditions for defense in depth, not as a substitute for least-privilege actions and resources.

---

## 12. Production IAM Security Practices

| Practice | Production expectation |
|---|---|
| Root account | No routine use; enable MFA and secure recovery controls |
| Human access | Federation/IAM Identity Center with MFA |
| Workloads | Roles and temporary credentials |
| Permissions | Least privilege and resource-level scoping |
| Secrets | Never commit keys; use Secrets Manager/Parameter Store or federation |
| Policy changes | Code review, version control and audit trail |
| Access review | Remove unused users, roles, policies and permissions |
| Detection | CloudTrail, Access Analyzer and security monitoring |
| Break-glass access | Separate, tightly controlled emergency role |
| Region control | Restrict deployments with SCPs and conditions |
| Tag governance | Enforce required tags for ownership and cost control |

---

## 13. AccessDenied Troubleshooting

### Investigation sequence

1. Confirm the active identity:
   ```bash
   aws sts get-caller-identity
   ```
2. Confirm the AWS account and region.
3. Identify the exact denied action and resource ARN.
4. Inspect identity policies attached directly and through groups/roles.
5. Check the role trust policy if `AssumeRole` fails.
6. Check `iam:PassRole` if a service is receiving a role.
7. Check permissions boundaries, SCPs and session policies.
8. Check resource-based policies, KMS key policies and VPC endpoint policies.
9. Check explicit denies and condition keys.
10. Re-test using the smallest required action.

### Common symptoms

| Error pattern | Likely cause |
|---|---|
| `not authorized to perform sts:AssumeRole` | Trust policy or caller policy problem |
| `not authorized to perform iam:PassRole` | Missing or over-broad role-passing permission |
| `AccessDenied` despite an Allow | Explicit Deny, SCP, boundary, session policy or resource policy |
| EC2 cannot access S3 | Missing instance-role permission, wrong bucket policy or endpoint policy |
| Terraform works locally but not in Jenkins | Different identity, role, account or region |
| Role exists but cannot be assumed | Trust relationship does not trust the caller |

---

## 14. Useful CLI Commands

```bash
# Current identity
aws sts get-caller-identity

# List roles
aws iam list-roles

# Inspect a role and trust policy
aws iam get-role --role-name ROLE_NAME

# List policies attached to a role
aws iam list-attached-role-policies --role-name ROLE_NAME

# List inline policies
aws iam list-role-policies --role-name ROLE_NAME

# Inspect a managed policy version
aws iam get-policy --policy-arn POLICY_ARN
aws iam get-policy-version \
  --policy-arn POLICY_ARN \
  --version-id v1

# Assume a role
aws sts assume-role \
  --role-arn arn:aws:iam::ACCOUNT_ID:role/ROLE_NAME \
  --role-session-name devops-session

# Simulate permissions
aws iam simulate-principal-policy \
  --policy-source-arn ROLE_OR_USER_ARN \
  --action-names s3:GetObject
```

---

## 15. Senior DevOps Interview Questions

| Question | Strong answer |
|---|---|
| Why use roles instead of access keys? | Roles provide temporary, automatically rotated credentials and remove long-lived secrets from workloads. |
| Trust policy vs permissions policy? | Trust controls who can assume a role; permissions control what the assumed identity can do. |
| Does an SCP grant permissions? | No. It sets the maximum permissions available in an account or OU. |
| What overrides an Allow? | An applicable explicit Deny. |
| Why can Terraform fail only in Jenkins? | Jenkins may use a different role, account, region, session policy or credential source. |
| What is `iam:PassRole`? | Permission to pass a role to an AWS service; it is not the same as assuming the role. |
| What is an instance profile? | The EC2 wrapper that associates an IAM role with an instance. |
| How do you secure cross-account deployment? | Target role trust, source `sts:AssumeRole`, scoped target permissions, short sessions and guardrails. |
| How do you investigate AccessDenied? | Verify identity, action, resource, trust, policies, explicit denies, boundaries, SCPs, conditions and resource policies. |
| How do you prevent privilege escalation? | Least privilege, restricted `iam:PassRole`, boundaries, SCPs, permission review and audit monitoring. |

---

## 16. One-Line Revision

- **User:** Human or legacy identity; avoid for workloads.
- **Role:** Assumable identity with temporary credentials.
- **Policy:** JSON authorization document.
- **Trust policy:** Who can assume the role.
- **Permissions policy:** What the role can do.
- **STS:** Issues temporary credentials.
- **Instance profile:** Attaches a role to EC2.
- **`iam:PassRole`:** Passes a role to an AWS service.
- **SCP:** Organization/account permission guardrail.
- **Boundary:** Maximum permissions for one identity.
- **Explicit Deny:** Overrides Allow.
- **Least privilege:** Grant only required actions, resources and conditions.

## Canonical ownership

- EC2 identity usage: `01-EC2`
- Terraform execution: `22-Terraform-AWS-IaC`
- Secrets and managed access: `15-SSM-Secrets`
- Encryption key authorization: `16-KMS`
- Audit and detection: `14-CloudTrail` and `13-CloudWatch`
