# IAM

### Q1. What is IAM?

**Answer:** AWS Identity and Access Management (IAM) controls authentication and authorization for AWS resources. It includes identities such as users, groups, and roles and policies that determine what actions are allowed or denied.

---

### Q2. What is an IAM user?

**Answer:** An IAM user is an AWS identity with long-term credentials and permissions. For workloads, IAM roles and temporary credentials are generally preferred over long-lived user access keys.

---

### Q3. What is an IAM group?

**Answer:** An IAM group is a collection of IAM users. Policies attached to the group can grant permissions to all users in that group.

---

### Q4. What is an IAM role?

**Answer:** An IAM role is an identity that trusted principals can assume to obtain temporary credentials. Roles are commonly used by EC2, Lambda, federated users, and cross-account access.

---

### Q5. When should an application use an IAM role instead of access keys?

**Answer:** An application should use an IAM role whenever it runs on an AWS service that supports role-based temporary credentials, such as EC2 or Lambda. This avoids storing long-lived access keys and allows AWS to provide short-lived credentials through the service.

---

### Q6. What is the difference between authentication and authorization?

**Answer:** Authentication establishes who the principal is. Authorization determines what that authenticated principal is allowed to do. In AWS, IAM policies provide authorization while identities, federation, and credentials are used for authentication.

---

### Q7. What is an IAM policy?

**Answer:** An IAM policy is a JSON permissions document. It commonly contains Effect, Action, Resource, and optional Condition elements, and resource-based policies can also contain Principal. AWS evaluates applicable policies to determine whether an action is allowed.

---

### Q8. What is an identity-based policy?

**Answer:** An identity-based policy is attached to a user, group, or role and defines what actions that identity is allowed or denied to perform on specified resources.

---

### Q9. What is a resource-based policy?

**Answer:** A resource-based policy is attached to a supported AWS resource and specifies which principals can access the resource and under what conditions. S3 bucket policies are a common example.

---

### Q10. What is a permissions boundary?

**Answer:** A permissions boundary sets the maximum permissions an IAM user or role can receive through identity-based policies. A boundary does not grant permissions by itself.

---

### Q11. What is a Service Control Policy?

**Answer:** An SCP is an AWS Organizations policy that sets the maximum available permissions for principals in member accounts. It does not grant permissions; IAM policies must still allow the action.

---

### Q12. What is a Resource Control Policy?

**Answer:** An RCP is an AWS Organizations resource-policy guardrail for supported resources. It limits what resource-based access can achieve and does not itself grant access.

---

### Q13. What is a session policy?

**Answer:** A session policy can further restrict permissions for a role or federated session. It cannot grant permissions beyond the permissions already available to the session.

---

### Q14. What is a VPC endpoint policy?

**Answer:** A VPC endpoint policy controls which principals and actions can use a VPC endpoint to access supported AWS services. It is an additional resource-policy control.

---

### Q15. What are ACLs in the IAM policy evaluation context?

**Answer:** ACLs are access-control mechanisms used by certain AWS resources, most notably S3. They are separate from IAM identity policies and provide another layer of resource access control. Network ACLs are also separate from IAM and operate at the subnet boundary.

---

### Q16. Does a permissions boundary grant permissions?

**Answer:** No. A permissions boundary does not grant permissions. It defines the maximum permissions an identity-based policy can grant to the user or role; an applicable identity policy must still allow the action.

---

### Q17. What is explicit deny?

**Answer:** An explicit Deny overrides an Allow during AWS authorization evaluation. This is a fundamental IAM rule and is used by security guardrails to prevent otherwise permitted actions.

---

### Q18. What is the effect of an explicit Deny?

**Answer:** An explicit Deny overrides any Allow that might otherwise grant the action. This applies even when an identity policy, resource policy, or another mechanism allows the request.

---

### Q19. How are identity and resource policies evaluated?

**Answer:** Identity-based policies grant or deny permissions to principals, while resource-based policies are attached to supported resources and can grant access to specified principals. AWS evaluates all applicable policies together; an explicit Deny overrides any Allow.

---

### Q20. How do SCPs constrain permissions?

**Answer:** SCPs define the maximum permissions available to principals in member accounts. An IAM policy must still grant the action, and an action denied by an SCP cannot be performed even if an IAM policy allows it.

---

### Q21. What is least privilege?

**Answer:** Least privilege means granting only the permissions required for a task. In AWS this means restricting actions, resources, conditions, principals, and credential lifetime as appropriate.

---

### Q22. What is policy inheritance?

**Answer:** AWS IAM does not use traditional policy inheritance like an object-oriented hierarchy. Effective permissions are determined by the applicable identity, resource, session, permissions-boundary, SCP/RCP, endpoint-policy, and other authorization controls, with explicit Deny taking precedence.

---

### Q23. What is MFA?

**Answer:** Multi-factor authentication requires an additional authentication factor beyond a password. It is especially important for privileged identities and protecting the AWS account root user.

---

### Q24. What is an IAM access key?

**Answer:** An IAM access key is a long-term programmatic credential consisting of an access-key ID and secret access key. Temporary role credentials are preferred for AWS workloads whenever possible.

---

### Q25. What is IAM Access Analyzer?

**Answer:** IAM Access Analyzer helps identify unintended external access and validate policies against policy grammar and best practices. It supports least-privilege and resource-sharing reviews.

---

### Q26. What is STS?

**Answer:** AWS Security Token Service (STS) issues temporary security credentials for authenticated sessions and role assumption. Temporary credentials reduce the need for long-lived access keys.

---

### Q27. What is AssumeRole?

**Answer:** AssumeRole is an AWS STS operation that allows a trusted principal to obtain temporary credentials for an IAM role. The role trust policy determines who can assume it, while the role's permissions policies determine what the resulting session can do.

---

### Q28. What is a trust policy?

**Answer:** A role trust policy is a resource-based policy that specifies which principals are allowed to assume the role. It is separate from the role's permissions policy.

---

### Q29. What is the difference between a trust policy and a permissions policy?

**Answer:** A role trust policy controls who or what is allowed to assume the role. The role's permissions policy controls what actions the principal can perform after assuming it. In short: trust policy = who can assume; permissions policy = what they can do.

---
