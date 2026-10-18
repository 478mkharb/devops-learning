# AWS (EC2, VPC, IAM, S3, EKS, etc.) — 50 Advanced Interview Questions

**Audience:** Mid-level/L2 DevOps Engineer (3–5 years).
**Reference date:** 18 October 2026.

**Focus:** AWS (EC2, VPC, IAM, S3, EKS, etc.) production operations, troubleshooting, security, and reliability

## How to practise
- First answer the **interview-ready answer** aloud in 60–90 seconds.
- Then explain the technical reasoning, trade-offs, and evidence you would collect.
- Treat commands as diagnostic starting points; replace placeholders and validate permissions.
- Do not run destructive or mutating commands in production without an approved change plan.
- Use the follow-up to test whether you understand the failure mechanism rather than memorizing a script.

## Questions

### Q1. An EC2 instance in a private subnet cannot download OS updates, while an instance in a public subnet can. Trace the likely network path and failure points.

**Difficulty:** Intermediate

**Interview-ready answer:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available. Trace traffic hop by hop: source subnet route, destination route, security group, NACL, DNS, and return path. Prefer private workloads and restrict egress intentionally. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Subnets, route tables, IGW, NAT, endpoints, security groups, NACLs Trace traffic hop by hop: source subnet route, destination route, security group, NACL, DNS, and return path. Prefer private workloads and restrict egress intentionally. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available.

**Production scenario:** EC2 has no outbound connectivity. Check default route, NAT/IGW placement, source/destination checks, DNS, SG egress, NACL ephemeral ports, and endpoint policies.

**Commands / checks:** ``aws ec2 describe-route-tables`; `describe-security-groups`; VPC Flow Logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q2. A route table looks correct, but outbound connections still time out. What security and return-path checks would you perform?

**Difficulty:** Intermediate

**Interview-ready answer:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available. Trace traffic hop by hop: source subnet route, destination route, security group, NACL, DNS, and return path. Prefer private workloads and restrict egress intentionally. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Subnets, route tables, IGW, NAT, endpoints, security groups, NACLs Trace traffic hop by hop: source subnet route, destination route, security group, NACL, DNS, and return path. Prefer private workloads and restrict egress intentionally. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available.

**Production scenario:** EC2 has no outbound connectivity. Check default route, NAT/IGW placement, source/destination checks, DNS, SG egress, NACL ephemeral ports, and endpoint policies.

**Commands / checks:** ``aws ec2 describe-route-tables`; `describe-security-groups`; VPC Flow Logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q3. An application receives S3 AccessDenied even though its IAM role allows GetObject. Which policy layers and encryption settings can still deny access?

**Difficulty:** Advanced

**Interview-ready answer:** For S3 access, evaluate the effective authorization path: caller identity policy, bucket/access-point policy, endpoint policy, organization controls, and KMS key policy if encryption is involved. Confirm the exact object ARN and request principal. Trace traffic hop by hop: source subnet route, destination route, security group, NACL, DNS, and return path. Prefer private workloads and restrict egress intentionally. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Subnets, route tables, IGW, NAT, endpoints, security groups, NACLs Trace traffic hop by hop: source subnet route, destination route, security group, NACL, DNS, and return path. Prefer private workloads and restrict egress intentionally. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** For S3 access, evaluate the effective authorization path: caller identity policy, bucket/access-point policy, endpoint policy, organization controls, and KMS key policy if encryption is involved. Confirm the exact object ARN and request principal.

**Production scenario:** EC2 has no outbound connectivity. Check default route, NAT/IGW placement, source/destination checks, DNS, SG egress, NACL ephemeral ports, and endpoint policies.

**Commands / checks:** ``aws ec2 describe-route-tables`; `describe-security-groups`; VPC Flow Logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q4. A workload role suddenly gains access to more buckets after a policy change. How would you detect and constrain the effective permissions?

**Difficulty:** Intermediate

**Interview-ready answer:** For S3 access, evaluate the effective authorization path: caller identity policy, bucket/access-point policy, endpoint policy, organization controls, and KMS key policy if encryption is involved. Confirm the exact object ARN and request principal. Trace traffic hop by hop: source subnet route, destination route, security group, NACL, DNS, and return path. Prefer private workloads and restrict egress intentionally. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Subnets, route tables, IGW, NAT, endpoints, security groups, NACLs Trace traffic hop by hop: source subnet route, destination route, security group, NACL, DNS, and return path. Prefer private workloads and restrict egress intentionally. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** For S3 access, evaluate the effective authorization path: caller identity policy, bucket/access-point policy, endpoint policy, organization controls, and KMS key policy if encryption is involved. Confirm the exact object ARN and request principal.

**Production scenario:** EC2 has no outbound connectivity. Check default route, NAT/IGW placement, source/destination checks, DNS, SG egress, NACL ephemeral ports, and endpoint policies.

**Commands / checks:** ``aws ec2 describe-route-tables`; `describe-security-groups`; VPC Flow Logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q5. An EC2 Auto Scaling group launches instances that never become healthy behind an ALB. How would you isolate bootstrap, target health, and networking failures?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect target health reason codes and Auto Scaling activity, then correlate bootstrap/user-data logs, instance profile, listener/target port, health-check path, and security rules. Trace traffic hop by hop: source subnet route, destination route, security group, NACL, DNS, and return path. Prefer private workloads and restrict egress intentionally. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Subnets, route tables, IGW, NAT, endpoints, security groups, NACLs Trace traffic hop by hop: source subnet route, destination route, security group, NACL, DNS, and return path. Prefer private workloads and restrict egress intentionally. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Inspect target health reason codes and Auto Scaling activity, then correlate bootstrap/user-data logs, instance profile, listener/target port, health-check path, and security rules.

**Production scenario:** EC2 has no outbound connectivity. Check default route, NAT/IGW placement, source/destination checks, DNS, SG egress, NACL ephemeral ports, and endpoint policies.

**Commands / checks:** ``aws ec2 describe-route-tables`; `describe-security-groups`; VPC Flow Logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q6. An AWS bill rises sharply after a new environment launches. How would you identify NAT, data-transfer, storage, and idle-compute costs?

**Difficulty:** Intermediate

**Interview-ready answer:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available. Trace traffic hop by hop: source subnet route, destination route, security group, NACL, DNS, and return path. Prefer private workloads and restrict egress intentionally. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Subnets, route tables, IGW, NAT, endpoints, security groups, NACLs Trace traffic hop by hop: source subnet route, destination route, security group, NACL, DNS, and return path. Prefer private workloads and restrict egress intentionally. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available.

**Production scenario:** EC2 has no outbound connectivity. Check default route, NAT/IGW placement, source/destination checks, DNS, SG egress, NACL ephemeral ports, and endpoint policies.

**Commands / checks:** ``aws ec2 describe-route-tables`; `describe-security-groups`; VPC Flow Logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q7. A team wants to store application credentials on EC2. Compare instance roles, Secrets Manager, and static credentials for this use case.

**Difficulty:** Intermediate

**Interview-ready answer:** Separate role trust (who can assume it) from permission policy (what it can do), then check explicit denies from SCPs, permissions boundaries, session policies, and resource policies. Confirm the actual caller with STS. Trace traffic hop by hop: source subnet route, destination route, security group, NACL, DNS, and return path. Prefer private workloads and restrict egress intentionally. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Subnets, route tables, IGW, NAT, endpoints, security groups, NACLs Trace traffic hop by hop: source subnet route, destination route, security group, NACL, DNS, and return path. Prefer private workloads and restrict egress intentionally. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Separate role trust (who can assume it) from permission policy (what it can do), then check explicit denies from SCPs, permissions boundaries, session policies, and resource policies. Confirm the actual caller with STS.

**Production scenario:** EC2 has no outbound connectivity. Check default route, NAT/IGW placement, source/destination checks, DNS, SG egress, NACL ephemeral ports, and endpoint policies.

**Commands / checks:** ``aws ec2 describe-route-tables`; `describe-security-groups`; VPC Flow Logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q8. A backup job reports success, but a restored application cannot connect to its database. What should a restore test include?

**Difficulty:** Advanced

**Interview-ready answer:** Treat recovery as an end-to-end test: restore data, decrypt it, recreate dependencies and network access, validate application consistency, and measure achieved RTO/RPO. A successful backup job alone is insufficient. Trace traffic hop by hop: source subnet route, destination route, security group, NACL, DNS, and return path. Prefer private workloads and restrict egress intentionally. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Subnets, route tables, IGW, NAT, endpoints, security groups, NACLs Trace traffic hop by hop: source subnet route, destination route, security group, NACL, DNS, and return path. Prefer private workloads and restrict egress intentionally. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Treat recovery as an end-to-end test: restore data, decrypt it, recreate dependencies and network access, validate application consistency, and measure achieved RTO/RPO. A successful backup job alone is insufficient.

**Production scenario:** EC2 has no outbound connectivity. Check default route, NAT/IGW placement, source/destination checks, DNS, SG egress, NACL ephemeral ports, and endpoint policies.

**Commands / checks:** ``aws ec2 describe-route-tables`; `describe-security-groups`; VPC Flow Logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q9. An EBS volume is encrypted with a customer-managed KMS key and restore fails in a recovery account. What permissions and key policies do you inspect?

**Difficulty:** Intermediate

**Interview-ready answer:** For S3 access, evaluate the effective authorization path: caller identity policy, bucket/access-point policy, endpoint policy, organization controls, and KMS key policy if encryption is involved. Confirm the exact object ARN and request principal. Trace traffic hop by hop: source subnet route, destination route, security group, NACL, DNS, and return path. Prefer private workloads and restrict egress intentionally. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Subnets, route tables, IGW, NAT, endpoints, security groups, NACLs Trace traffic hop by hop: source subnet route, destination route, security group, NACL, DNS, and return path. Prefer private workloads and restrict egress intentionally. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** For S3 access, evaluate the effective authorization path: caller identity policy, bucket/access-point policy, endpoint policy, organization controls, and KMS key policy if encryption is involved. Confirm the exact object ARN and request principal.

**Production scenario:** EC2 has no outbound connectivity. Check default route, NAT/IGW placement, source/destination checks, DNS, SG egress, NACL ephemeral ports, and endpoint policies.

**Commands / checks:** ``aws ec2 describe-route-tables`; `describe-security-groups`; VPC Flow Logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q10. Design a multi-AZ web tier with private application instances and controlled outbound access. Explain the routing and security boundaries.

**Difficulty:** Advanced

**Interview-ready answer:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available. Trace traffic hop by hop: source subnet route, destination route, security group, NACL, DNS, and return path. Prefer private workloads and restrict egress intentionally. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Subnets, route tables, IGW, NAT, endpoints, security groups, NACLs Trace traffic hop by hop: source subnet route, destination route, security group, NACL, DNS, and return path. Prefer private workloads and restrict egress intentionally. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available.

**Production scenario:** EC2 has no outbound connectivity. Check default route, NAT/IGW placement, source/destination checks, DNS, SG egress, NACL ephemeral ports, and endpoint policies.

**Commands / checks:** ``aws ec2 describe-route-tables`; `describe-security-groups`; VPC Flow Logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q11. CloudTrail shows repeated AssumeRole failures from a CI workload. How do you distinguish a trust-policy failure from missing permissions?

**Difficulty:** Intermediate

**Interview-ready answer:** Separate role trust (who can assume it) from permission policy (what it can do), then check explicit denies from SCPs, permissions boundaries, session policies, and resource policies. Confirm the actual caller with STS. Use roles and temporary credentials, separate trust from permissions, scope actions and resources, and validate effective access. Avoid long-lived access keys and broad wildcard grants. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Roles, trust policies, permission policies, STS, least privilege Use roles and temporary credentials, separate trust from permissions, scope actions and resources, and validate effective access. Avoid long-lived access keys and broad wildcard grants. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Separate role trust (who can assume it) from permission policy (what it can do), then check explicit denies from SCPs, permissions boundaries, session policies, and resource policies. Confirm the actual caller with STS.

**Production scenario:** An app gets AccessDenied despite an Allow policy. Check trust, session policy, SCP, permissions boundary, resource policy, KMS policy, and explicit denies.

**Commands / checks:** ``aws sts get-caller-identity`; IAM policy simulator; CloudTrail event history`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q12. A security group permits inbound HTTPS, but clients cannot connect. What listener, target, subnet, NACL, and health checks would you inspect?

**Difficulty:** Intermediate

**Interview-ready answer:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available. Use roles and temporary credentials, separate trust from permissions, scope actions and resources, and validate effective access. Avoid long-lived access keys and broad wildcard grants. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Roles, trust policies, permission policies, STS, least privilege Use roles and temporary credentials, separate trust from permissions, scope actions and resources, and validate effective access. Avoid long-lived access keys and broad wildcard grants. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available.

**Production scenario:** An app gets AccessDenied despite an Allow policy. Check trust, session policy, SCP, permissions boundary, resource policy, KMS policy, and explicit denies.

**Commands / checks:** ``aws sts get-caller-identity`; IAM policy simulator; CloudTrail event history`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q13. An instance has no public IP but must access S3 without traversing a NAT Gateway. What architecture would you consider and what policies must align?

**Difficulty:** Advanced

**Interview-ready answer:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available. Use roles and temporary credentials, separate trust from permissions, scope actions and resources, and validate effective access. Avoid long-lived access keys and broad wildcard grants. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Roles, trust policies, permission policies, STS, least privilege Use roles and temporary credentials, separate trust from permissions, scope actions and resources, and validate effective access. Avoid long-lived access keys and broad wildcard grants. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available.

**Production scenario:** An app gets AccessDenied despite an Allow policy. Check trust, session policy, SCP, permissions boundary, resource policy, KMS policy, and explicit denies.

**Commands / checks:** ``aws sts get-caller-identity`; IAM policy simulator; CloudTrail event history`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q14. An application works in one Availability Zone but fails in another. How would you investigate subnet routes, endpoints, DNS, and zonal dependencies?

**Difficulty:** Intermediate

**Interview-ready answer:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available. Use roles and temporary credentials, separate trust from permissions, scope actions and resources, and validate effective access. Avoid long-lived access keys and broad wildcard grants. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Roles, trust policies, permission policies, STS, least privilege Use roles and temporary credentials, separate trust from permissions, scope actions and resources, and validate effective access. Avoid long-lived access keys and broad wildcard grants. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available.

**Production scenario:** An app gets AccessDenied despite an Allow policy. Check trust, session policy, SCP, permissions boundary, resource policy, KMS policy, and explicit denies.

**Commands / checks:** ``aws sts get-caller-identity`; IAM policy simulator; CloudTrail event history`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q15. An IAM policy uses Resource: '*'. How would you scope it down without breaking legitimate operations?

**Difficulty:** Advanced

**Interview-ready answer:** Separate role trust (who can assume it) from permission policy (what it can do), then check explicit denies from SCPs, permissions boundaries, session policies, and resource policies. Confirm the actual caller with STS. Use roles and temporary credentials, separate trust from permissions, scope actions and resources, and validate effective access. Avoid long-lived access keys and broad wildcard grants. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Roles, trust policies, permission policies, STS, least privilege Use roles and temporary credentials, separate trust from permissions, scope actions and resources, and validate effective access. Avoid long-lived access keys and broad wildcard grants. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Separate role trust (who can assume it) from permission policy (what it can do), then check explicit denies from SCPs, permissions boundaries, session policies, and resource policies. Confirm the actual caller with STS.

**Production scenario:** An app gets AccessDenied despite an Allow policy. Check trust, session policy, SCP, permissions boundary, resource policy, KMS policy, and explicit denies.

**Commands / checks:** ``aws sts get-caller-identity`; IAM policy simulator; CloudTrail event history`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q16. A private EC2 instance cannot be reached through Session Manager. What IAM, SSM agent, endpoint/NAT, and instance-profile checks are needed?

**Difficulty:** Intermediate

**Interview-ready answer:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available. Use roles and temporary credentials, separate trust from permissions, scope actions and resources, and validate effective access. Avoid long-lived access keys and broad wildcard grants. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Roles, trust policies, permission policies, STS, least privilege Use roles and temporary credentials, separate trust from permissions, scope actions and resources, and validate effective access. Avoid long-lived access keys and broad wildcard grants. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available.

**Production scenario:** An app gets AccessDenied despite an Allow policy. Check trust, session policy, SCP, permissions boundary, resource policy, KMS policy, and explicit denies.

**Commands / checks:** ``aws sts get-caller-identity`; IAM policy simulator; CloudTrail event history`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q17. A database restore is required after accidental deletion. How do you choose between snapshots, point-in-time recovery, and a broader disaster-recovery plan?

**Difficulty:** Intermediate

**Interview-ready answer:** Treat recovery as an end-to-end test: restore data, decrypt it, recreate dependencies and network access, validate application consistency, and measure achieved RTO/RPO. A successful backup job alone is insufficient. Use roles and temporary credentials, separate trust from permissions, scope actions and resources, and validate effective access. Avoid long-lived access keys and broad wildcard grants. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Roles, trust policies, permission policies, STS, least privilege Use roles and temporary credentials, separate trust from permissions, scope actions and resources, and validate effective access. Avoid long-lived access keys and broad wildcard grants. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Treat recovery as an end-to-end test: restore data, decrypt it, recreate dependencies and network access, validate application consistency, and measure achieved RTO/RPO. A successful backup job alone is insufficient.

**Production scenario:** An app gets AccessDenied despite an Allow policy. Check trust, session policy, SCP, permissions boundary, resource policy, KMS policy, and explicit denies.

**Commands / checks:** ``aws sts get-caller-identity`; IAM policy simulator; CloudTrail event history`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q18. An ALB returns 503 while EC2 instances appear running. Which target-health and application checks narrow the cause?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect target health reason codes and Auto Scaling activity, then correlate bootstrap/user-data logs, instance profile, listener/target port, health-check path, and security rules. Use roles and temporary credentials, separate trust from permissions, scope actions and resources, and validate effective access. Avoid long-lived access keys and broad wildcard grants. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Roles, trust policies, permission policies, STS, least privilege Use roles and temporary credentials, separate trust from permissions, scope actions and resources, and validate effective access. Avoid long-lived access keys and broad wildcard grants. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Inspect target health reason codes and Auto Scaling activity, then correlate bootstrap/user-data logs, instance profile, listener/target port, health-check path, and security rules.

**Production scenario:** An app gets AccessDenied despite an Allow policy. Check trust, session policy, SCP, permissions boundary, resource policy, KMS policy, and explicit denies.

**Commands / checks:** ``aws sts get-caller-identity`; IAM policy simulator; CloudTrail event history`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q19. A bucket policy denies requests from a VPC endpoint unexpectedly. How do you inspect endpoint policy conditions, principal, and request context?

**Difficulty:** Intermediate

**Interview-ready answer:** For S3 access, evaluate the effective authorization path: caller identity policy, bucket/access-point policy, endpoint policy, organization controls, and KMS key policy if encryption is involved. Confirm the exact object ARN and request principal. Use roles and temporary credentials, separate trust from permissions, scope actions and resources, and validate effective access. Avoid long-lived access keys and broad wildcard grants. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Roles, trust policies, permission policies, STS, least privilege Use roles and temporary credentials, separate trust from permissions, scope actions and resources, and validate effective access. Avoid long-lived access keys and broad wildcard grants. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** For S3 access, evaluate the effective authorization path: caller identity policy, bucket/access-point policy, endpoint policy, organization controls, and KMS key policy if encryption is involved. Confirm the exact object ARN and request principal.

**Production scenario:** An app gets AccessDenied despite an Allow policy. Check trust, session policy, SCP, permissions boundary, resource policy, KMS policy, and explicit denies.

**Commands / checks:** ``aws sts get-caller-identity`; IAM policy simulator; CloudTrail event history`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q20. How would you separate dev and production AWS accounts while still centralizing logs and security findings?

**Difficulty:** Advanced

**Interview-ready answer:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident. Use roles and temporary credentials, separate trust from permissions, scope actions and resources, and validate effective access. Avoid long-lived access keys and broad wildcard grants. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Roles, trust policies, permission policies, STS, least privilege Use roles and temporary credentials, separate trust from permissions, scope actions and resources, and validate effective access. Avoid long-lived access keys and broad wildcard grants. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident.

**Production scenario:** An app gets AccessDenied despite an Allow policy. Check trust, session policy, SCP, permissions boundary, resource policy, KMS policy, and explicit denies.

**Commands / checks:** ``aws sts get-caller-identity`; IAM policy simulator; CloudTrail event history`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q21. An Auto Scaling group repeatedly replaces instances after deployment. How do you correlate lifecycle hooks, health checks, user data, and application startup?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect target health reason codes and Auto Scaling activity, then correlate bootstrap/user-data logs, instance profile, listener/target port, health-check path, and security rules. Use immutable images where practical, launch templates, health checks, and Auto Scaling. Secure IMDS, patch images, and validate capacity and AZ distribution. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Instance health, AMIs, Auto Scaling, launch templates, metadata Use immutable images where practical, launch templates, health checks, and Auto Scaling. Secure IMDS, patch images, and validate capacity and AZ distribution. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Inspect target health reason codes and Auto Scaling activity, then correlate bootstrap/user-data logs, instance profile, listener/target port, health-check path, and security rules.

**Production scenario:** Instances launch but fail health checks. Inspect user data, IAM instance profile, network paths, app startup, target health, and scaling activity.

**Commands / checks:** ``aws ec2 describe-instance-status`; Auto Scaling activity history; `curl` IMDSv2 token flow`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q22. An EC2 instance reaches CPU saturation during a traffic spike. How do you decide between vertical scaling, horizontal scaling, and application optimization?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect target health reason codes and Auto Scaling activity, then correlate bootstrap/user-data logs, instance profile, listener/target port, health-check path, and security rules. Use immutable images where practical, launch templates, health checks, and Auto Scaling. Secure IMDS, patch images, and validate capacity and AZ distribution. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Instance health, AMIs, Auto Scaling, launch templates, metadata Use immutable images where practical, launch templates, health checks, and Auto Scaling. Secure IMDS, patch images, and validate capacity and AZ distribution. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Inspect target health reason codes and Auto Scaling activity, then correlate bootstrap/user-data logs, instance profile, listener/target port, health-check path, and security rules.

**Production scenario:** Instances launch but fail health checks. Inspect user data, IAM instance profile, network paths, app startup, target health, and scaling activity.

**Commands / checks:** ``aws ec2 describe-instance-status`; Auto Scaling activity history; `curl` IMDSv2 token flow`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q23. A team plans to copy backups to another account and Region. Which encryption, access, retention, and restore considerations matter?

**Difficulty:** Advanced

**Interview-ready answer:** Treat recovery as an end-to-end test: restore data, decrypt it, recreate dependencies and network access, validate application consistency, and measure achieved RTO/RPO. A successful backup job alone is insufficient. Use immutable images where practical, launch templates, health checks, and Auto Scaling. Secure IMDS, patch images, and validate capacity and AZ distribution. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Instance health, AMIs, Auto Scaling, launch templates, metadata Use immutable images where practical, launch templates, health checks, and Auto Scaling. Secure IMDS, patch images, and validate capacity and AZ distribution. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Treat recovery as an end-to-end test: restore data, decrypt it, recreate dependencies and network access, validate application consistency, and measure achieved RTO/RPO. A successful backup job alone is insufficient.

**Production scenario:** Instances launch but fail health checks. Inspect user data, IAM instance profile, network paths, app startup, target health, and scaling activity.

**Commands / checks:** ``aws ec2 describe-instance-status`; Auto Scaling activity history; `curl` IMDSv2 token flow`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q24. An application can reach an AWS API from a laptop but not from EKS. How would you compare identity, DNS, network routes, and egress controls?

**Difficulty:** Intermediate

**Interview-ready answer:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available. Use immutable images where practical, launch templates, health checks, and Auto Scaling. Secure IMDS, patch images, and validate capacity and AZ distribution. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Instance health, AMIs, Auto Scaling, launch templates, metadata Use immutable images where practical, launch templates, health checks, and Auto Scaling. Secure IMDS, patch images, and validate capacity and AZ distribution. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available.

**Production scenario:** Instances launch but fail health checks. Inspect user data, IAM instance profile, network paths, app startup, target health, and scaling activity.

**Commands / checks:** ``aws ec2 describe-instance-status`; Auto Scaling activity history; `curl` IMDSv2 token flow`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q25. A role policy allows an action, but an AWS Organizations SCP blocks it. How would you prove which policy layer is responsible?

**Difficulty:** Advanced

**Interview-ready answer:** Separate role trust (who can assume it) from permission policy (what it can do), then check explicit denies from SCPs, permissions boundaries, session policies, and resource policies. Confirm the actual caller with STS. Use immutable images where practical, launch templates, health checks, and Auto Scaling. Secure IMDS, patch images, and validate capacity and AZ distribution. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Instance health, AMIs, Auto Scaling, launch templates, metadata Use immutable images where practical, launch templates, health checks, and Auto Scaling. Secure IMDS, patch images, and validate capacity and AZ distribution. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Separate role trust (who can assume it) from permission policy (what it can do), then check explicit denies from SCPs, permissions boundaries, session policies, and resource policies. Confirm the actual caller with STS.

**Production scenario:** Instances launch but fail health checks. Inspect user data, IAM instance profile, network paths, app startup, target health, and scaling activity.

**Commands / checks:** ``aws ec2 describe-instance-status`; Auto Scaling activity history; `curl` IMDSv2 token flow`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q26. Design secure S3 access for an EKS workload without long-lived access keys. Compare EKS Pod Identity and IRSA at a high level.

**Difficulty:** Intermediate

**Interview-ready answer:** Separate role trust (who can assume it) from permission policy (what it can do), then check explicit denies from SCPs, permissions boundaries, session policies, and resource policies. Confirm the actual caller with STS. Use immutable images where practical, launch templates, health checks, and Auto Scaling. Secure IMDS, patch images, and validate capacity and AZ distribution. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Instance health, AMIs, Auto Scaling, launch templates, metadata Use immutable images where practical, launch templates, health checks, and Auto Scaling. Secure IMDS, patch images, and validate capacity and AZ distribution. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Separate role trust (who can assume it) from permission policy (what it can do), then check explicit denies from SCPs, permissions boundaries, session policies, and resource policies. Confirm the actual caller with STS.

**Production scenario:** Instances launch but fail health checks. Inspect user data, IAM instance profile, network paths, app startup, target health, and scaling activity.

**Commands / checks:** ``aws ec2 describe-instance-status`; Auto Scaling activity history; `curl` IMDSv2 token flow`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q27. An instance's user-data script fails silently on first boot. What evidence would you inspect and how would you make bootstrap idempotent?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect target health reason codes and Auto Scaling activity, then correlate bootstrap/user-data logs, instance profile, listener/target port, health-check path, and security rules. Use immutable images where practical, launch templates, health checks, and Auto Scaling. Secure IMDS, patch images, and validate capacity and AZ distribution. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Instance health, AMIs, Auto Scaling, launch templates, metadata Use immutable images where practical, launch templates, health checks, and Auto Scaling. Secure IMDS, patch images, and validate capacity and AZ distribution. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Inspect target health reason codes and Auto Scaling activity, then correlate bootstrap/user-data logs, instance profile, listener/target port, health-check path, and security rules.

**Production scenario:** Instances launch but fail health checks. Inspect user data, IAM instance profile, network paths, app startup, target health, and scaling activity.

**Commands / checks:** ``aws ec2 describe-instance-status`; Auto Scaling activity history; `curl` IMDSv2 token flow`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q28. A cross-account role can be assumed but cannot decrypt an object. What KMS key-policy and grant issues might explain this?

**Difficulty:** Advanced

**Interview-ready answer:** For S3 access, evaluate the effective authorization path: caller identity policy, bucket/access-point policy, endpoint policy, organization controls, and KMS key policy if encryption is involved. Confirm the exact object ARN and request principal. Use immutable images where practical, launch templates, health checks, and Auto Scaling. Secure IMDS, patch images, and validate capacity and AZ distribution. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Instance health, AMIs, Auto Scaling, launch templates, metadata Use immutable images where practical, launch templates, health checks, and Auto Scaling. Secure IMDS, patch images, and validate capacity and AZ distribution. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** For S3 access, evaluate the effective authorization path: caller identity policy, bucket/access-point policy, endpoint policy, organization controls, and KMS key policy if encryption is involved. Confirm the exact object ARN and request principal.

**Production scenario:** Instances launch but fail health checks. Inspect user data, IAM instance profile, network paths, app startup, target health, and scaling activity.

**Commands / checks:** ``aws ec2 describe-instance-status`; Auto Scaling activity history; `curl` IMDSv2 token flow`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q29. An AWS Backup vault is accessible to the same administrator who can delete production resources. How would you reduce correlated risk?

**Difficulty:** Intermediate

**Interview-ready answer:** Treat recovery as an end-to-end test: restore data, decrypt it, recreate dependencies and network access, validate application consistency, and measure achieved RTO/RPO. A successful backup job alone is insufficient. Use immutable images where practical, launch templates, health checks, and Auto Scaling. Secure IMDS, patch images, and validate capacity and AZ distribution. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Instance health, AMIs, Auto Scaling, launch templates, metadata Use immutable images where practical, launch templates, health checks, and Auto Scaling. Secure IMDS, patch images, and validate capacity and AZ distribution. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Treat recovery as an end-to-end test: restore data, decrypt it, recreate dependencies and network access, validate application consistency, and measure achieved RTO/RPO. A successful backup job alone is insufficient.

**Production scenario:** Instances launch but fail health checks. Inspect user data, IAM instance profile, network paths, app startup, target health, and scaling activity.

**Commands / checks:** ``aws ec2 describe-instance-status`; Auto Scaling activity history; `curl` IMDSv2 token flow`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q30. Describe a safe process for rotating an IAM access key that may have leaked, including impact assessment and verification.

**Difficulty:** Advanced

**Interview-ready answer:** Separate role trust (who can assume it) from permission policy (what it can do), then check explicit denies from SCPs, permissions boundaries, session policies, and resource policies. Confirm the actual caller with STS. Use immutable images where practical, launch templates, health checks, and Auto Scaling. Secure IMDS, patch images, and validate capacity and AZ distribution. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Instance health, AMIs, Auto Scaling, launch templates, metadata Use immutable images where practical, launch templates, health checks, and Auto Scaling. Secure IMDS, patch images, and validate capacity and AZ distribution. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Separate role trust (who can assume it) from permission policy (what it can do), then check explicit denies from SCPs, permissions boundaries, session policies, and resource policies. Confirm the actual caller with STS.

**Production scenario:** Instances launch but fail health checks. Inspect user data, IAM instance profile, network paths, app startup, target health, and scaling activity.

**Commands / checks:** ``aws ec2 describe-instance-status`; Auto Scaling activity history; `curl` IMDSv2 token flow`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q31. Route 53 resolves an internal service name to an unexpected address. How would you trace private hosted-zone association and resolver behavior?

**Difficulty:** Intermediate

**Interview-ready answer:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available. Use least-privilege bucket policies, block public access, enable encryption and versioning where appropriate, and test recovery. Distinguish object storage from block and shared file storage. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Bucket policy, encryption, versioning, lifecycle, access points, EBS/EFS Use least-privilege bucket policies, block public access, enable encryption and versioning where appropriate, and test recovery. Distinguish object storage from block and shared file storage. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available.

**Production scenario:** An application gets S3 403. Verify identity, bucket policy, object ARN, KMS permissions, endpoint policy, and organization controls.

**Commands / checks:** ``aws s3api get-public-access-block`; `get-bucket-versioning`; `head-object``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q32. An EC2 app reports TLS errors when calling an AWS endpoint. What DNS, clock, CA trust, proxy, and endpoint checks would you perform?

**Difficulty:** Intermediate

**Interview-ready answer:** Compare authoritative records, private hosted-zone associations, resolver rules, TTL/cache behavior, and the caller's VPC. Test from the affected subnet rather than only from a laptop. Use least-privilege bucket policies, block public access, enable encryption and versioning where appropriate, and test recovery. Distinguish object storage from block and shared file storage. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Bucket policy, encryption, versioning, lifecycle, access points, EBS/EFS Use least-privilege bucket policies, block public access, enable encryption and versioning where appropriate, and test recovery. Distinguish object storage from block and shared file storage. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Compare authoritative records, private hosted-zone associations, resolver rules, TTL/cache behavior, and the caller's VPC. Test from the affected subnet rather than only from a laptop.

**Production scenario:** An application gets S3 403. Verify identity, bucket policy, object ARN, KMS permissions, endpoint policy, and organization controls.

**Commands / checks:** ``aws s3api get-public-access-block`; `get-bucket-versioning`; `head-object``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q33. A NAT Gateway cost is unexpectedly high. How would you determine whether architecture changes or data-transfer patterns caused it?

**Difficulty:** Advanced

**Interview-ready answer:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available. Use least-privilege bucket policies, block public access, enable encryption and versioning where appropriate, and test recovery. Distinguish object storage from block and shared file storage. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Bucket policy, encryption, versioning, lifecycle, access points, EBS/EFS Use least-privilege bucket policies, block public access, enable encryption and versioning where appropriate, and test recovery. Distinguish object storage from block and shared file storage. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available.

**Production scenario:** An application gets S3 403. Verify identity, bucket policy, object ARN, KMS permissions, endpoint policy, and organization controls.

**Commands / checks:** ``aws s3api get-public-access-block`; `get-bucket-versioning`; `head-object``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q34. An EBS volume is nearly full even though the application deleted old files. What Linux-level checks would you pair with AWS volume metrics?

**Difficulty:** Intermediate

**Interview-ready answer:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident. Use least-privilege bucket policies, block public access, enable encryption and versioning where appropriate, and test recovery. Distinguish object storage from block and shared file storage. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Bucket policy, encryption, versioning, lifecycle, access points, EBS/EFS Use least-privilege bucket policies, block public access, enable encryption and versioning where appropriate, and test recovery. Distinguish object storage from block and shared file storage. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident.

**Production scenario:** An application gets S3 403. Verify identity, bucket policy, object ARN, KMS permissions, endpoint policy, and organization controls.

**Commands / checks:** ``aws s3api get-public-access-block`; `get-bucket-versioning`; `head-object``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q35. An S3 lifecycle rule appears not to transition objects. What object-versioning, prefix, age, and storage-class conditions would you inspect?

**Difficulty:** Advanced

**Interview-ready answer:** For S3 access, evaluate the effective authorization path: caller identity policy, bucket/access-point policy, endpoint policy, organization controls, and KMS key policy if encryption is involved. Confirm the exact object ARN and request principal. Use least-privilege bucket policies, block public access, enable encryption and versioning where appropriate, and test recovery. Distinguish object storage from block and shared file storage. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Bucket policy, encryption, versioning, lifecycle, access points, EBS/EFS Use least-privilege bucket policies, block public access, enable encryption and versioning where appropriate, and test recovery. Distinguish object storage from block and shared file storage. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** For S3 access, evaluate the effective authorization path: caller identity policy, bucket/access-point policy, endpoint policy, organization controls, and KMS key policy if encryption is involved. Confirm the exact object ARN and request principal.

**Production scenario:** An application gets S3 403. Verify identity, bucket policy, object ARN, KMS permissions, endpoint policy, and organization controls.

**Commands / checks:** ``aws s3api get-public-access-block`; `get-bucket-versioning`; `head-object``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q36. An EKS node joins the cluster but cannot pull images from ECR. Which IAM, DNS, endpoint, and network checks come first?

**Difficulty:** Intermediate

**Interview-ready answer:** Separate role trust (who can assume it) from permission policy (what it can do), then check explicit denies from SCPs, permissions boundaries, session policies, and resource policies. Confirm the actual caller with STS. Use least-privilege bucket policies, block public access, enable encryption and versioning where appropriate, and test recovery. Distinguish object storage from block and shared file storage. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Bucket policy, encryption, versioning, lifecycle, access points, EBS/EFS Use least-privilege bucket policies, block public access, enable encryption and versioning where appropriate, and test recovery. Distinguish object storage from block and shared file storage. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Separate role trust (who can assume it) from permission policy (what it can do), then check explicit denies from SCPs, permissions boundaries, session policies, and resource policies. Confirm the actual caller with STS.

**Production scenario:** An application gets S3 403. Verify identity, bucket policy, object ARN, KMS permissions, endpoint policy, and organization controls.

**Commands / checks:** ``aws s3api get-public-access-block`; `get-bucket-versioning`; `head-object``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q37. A production subnet has an accidental route to an Internet Gateway. What is the safe response and what preventive policy would you add?

**Difficulty:** Intermediate

**Interview-ready answer:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available. Use least-privilege bucket policies, block public access, enable encryption and versioning where appropriate, and test recovery. Distinguish object storage from block and shared file storage. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Bucket policy, encryption, versioning, lifecycle, access points, EBS/EFS Use least-privilege bucket policies, block public access, enable encryption and versioning where appropriate, and test recovery. Distinguish object storage from block and shared file storage. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available.

**Production scenario:** An application gets S3 403. Verify identity, bucket policy, object ARN, KMS permissions, endpoint policy, and organization controls.

**Commands / checks:** ``aws s3api get-public-access-block`; `get-bucket-versioning`; `head-object``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q38. How would you design CloudWatch alarms for an application so they detect user impact rather than merely high CPU?

**Difficulty:** Advanced

**Interview-ready answer:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident. Use least-privilege bucket policies, block public access, enable encryption and versioning where appropriate, and test recovery. Distinguish object storage from block and shared file storage. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Bucket policy, encryption, versioning, lifecycle, access points, EBS/EFS Use least-privilege bucket policies, block public access, enable encryption and versioning where appropriate, and test recovery. Distinguish object storage from block and shared file storage. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident.

**Production scenario:** An application gets S3 403. Verify identity, bucket policy, object ARN, KMS permissions, endpoint policy, and organization controls.

**Commands / checks:** ``aws s3api get-public-access-block`; `get-bucket-versioning`; `head-object``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q39. A recovery point exists, but the KMS key is disabled. What operational and governance controls would have prevented this failure?

**Difficulty:** Intermediate

**Interview-ready answer:** For S3 access, evaluate the effective authorization path: caller identity policy, bucket/access-point policy, endpoint policy, organization controls, and KMS key policy if encryption is involved. Confirm the exact object ARN and request principal. Use least-privilege bucket policies, block public access, enable encryption and versioning where appropriate, and test recovery. Distinguish object storage from block and shared file storage. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Bucket policy, encryption, versioning, lifecycle, access points, EBS/EFS Use least-privilege bucket policies, block public access, enable encryption and versioning where appropriate, and test recovery. Distinguish object storage from block and shared file storage. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** For S3 access, evaluate the effective authorization path: caller identity policy, bucket/access-point policy, endpoint policy, organization controls, and KMS key policy if encryption is involved. Confirm the exact object ARN and request principal.

**Production scenario:** An application gets S3 403. Verify identity, bucket policy, object ARN, KMS permissions, endpoint policy, and organization controls.

**Commands / checks:** ``aws s3api get-public-access-block`; `get-bucket-versioning`; `head-object``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q40. How would you use tags and cost allocation to identify the owner and business purpose of unallocated AWS spend?

**Difficulty:** Advanced

**Interview-ready answer:** Treat recovery as an end-to-end test: restore data, decrypt it, recreate dependencies and network access, validate application consistency, and measure achieved RTO/RPO. A successful backup job alone is insufficient. Use least-privilege bucket policies, block public access, enable encryption and versioning where appropriate, and test recovery. Distinguish object storage from block and shared file storage. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Bucket policy, encryption, versioning, lifecycle, access points, EBS/EFS Use least-privilege bucket policies, block public access, enable encryption and versioning where appropriate, and test recovery. Distinguish object storage from block and shared file storage. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Treat recovery as an end-to-end test: restore data, decrypt it, recreate dependencies and network access, validate application consistency, and measure achieved RTO/RPO. A successful backup job alone is insufficient.

**Production scenario:** An application gets S3 403. Verify identity, bucket policy, object ARN, KMS permissions, endpoint policy, and organization controls.

**Commands / checks:** ``aws s3api get-public-access-block`; `get-bucket-versioning`; `head-object``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q41. A production region is unavailable. Explain how you would use RTO and RPO to decide whether to restore, fail over, or wait.

**Difficulty:** Intermediate

**Interview-ready answer:** Treat recovery as an end-to-end test: restore data, decrypt it, recreate dependencies and network access, validate application consistency, and measure achieved RTO/RPO. A successful backup job alone is insufficient. Design recovery around business-defined RTO and RPO. Backups must be isolated and restore-tested; use multi-AZ for availability and cross-account/Region copies for selected disaster scenarios. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Multi-AZ, AWS Backup, RTO/RPO, CloudWatch, budgets, DR testing Design recovery around business-defined RTO and RPO. Backups must be isolated and restore-tested; use multi-AZ for availability and cross-account/Region copies for selected disaster scenarios. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Treat recovery as an end-to-end test: restore data, decrypt it, recreate dependencies and network access, validate application consistency, and measure achieved RTO/RPO. A successful backup job alone is insufficient.

**Production scenario:** A backup job succeeds but restore fails. Test full restore workflows, permissions, dependencies, encryption keys, networking, and application consistency.

**Commands / checks:** ``aws backup list-backup-jobs`; `aws backup list-recovery-points-by-backup-vault`; restore-test records`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q42. An engineer proposes one broad IAM role for every application. What risks does this create and how would you refactor it?

**Difficulty:** Intermediate

**Interview-ready answer:** Separate role trust (who can assume it) from permission policy (what it can do), then check explicit denies from SCPs, permissions boundaries, session policies, and resource policies. Confirm the actual caller with STS. Design recovery around business-defined RTO and RPO. Backups must be isolated and restore-tested; use multi-AZ for availability and cross-account/Region copies for selected disaster scenarios. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Multi-AZ, AWS Backup, RTO/RPO, CloudWatch, budgets, DR testing Design recovery around business-defined RTO and RPO. Backups must be isolated and restore-tested; use multi-AZ for availability and cross-account/Region copies for selected disaster scenarios. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Separate role trust (who can assume it) from permission policy (what it can do), then check explicit denies from SCPs, permissions boundaries, session policies, and resource policies. Confirm the actual caller with STS.

**Production scenario:** A backup job succeeds but restore fails. Test full restore workflows, permissions, dependencies, encryption keys, networking, and application consistency.

**Commands / checks:** ``aws backup list-backup-jobs`; `aws backup list-recovery-points-by-backup-vault`; restore-test records`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q43. An S3 bucket is not public, yet an external account can read an object. What policies, access points, ACLs, and sharing mechanisms would you audit?

**Difficulty:** Advanced

**Interview-ready answer:** For S3 access, evaluate the effective authorization path: caller identity policy, bucket/access-point policy, endpoint policy, organization controls, and KMS key policy if encryption is involved. Confirm the exact object ARN and request principal. Design recovery around business-defined RTO and RPO. Backups must be isolated and restore-tested; use multi-AZ for availability and cross-account/Region copies for selected disaster scenarios. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Multi-AZ, AWS Backup, RTO/RPO, CloudWatch, budgets, DR testing Design recovery around business-defined RTO and RPO. Backups must be isolated and restore-tested; use multi-AZ for availability and cross-account/Region copies for selected disaster scenarios. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** For S3 access, evaluate the effective authorization path: caller identity policy, bucket/access-point policy, endpoint policy, organization controls, and KMS key policy if encryption is involved. Confirm the exact object ARN and request principal.

**Production scenario:** A backup job succeeds but restore fails. Test full restore workflows, permissions, dependencies, encryption keys, networking, and application consistency.

**Commands / checks:** ``aws backup list-backup-jobs`; `aws backup list-recovery-points-by-backup-vault`; restore-test records`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q44. An ALB health check passes but users receive application errors. Why can health checks give a false sense of readiness?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect target health reason codes and Auto Scaling activity, then correlate bootstrap/user-data logs, instance profile, listener/target port, health-check path, and security rules. Design recovery around business-defined RTO and RPO. Backups must be isolated and restore-tested; use multi-AZ for availability and cross-account/Region copies for selected disaster scenarios. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Multi-AZ, AWS Backup, RTO/RPO, CloudWatch, budgets, DR testing Design recovery around business-defined RTO and RPO. Backups must be isolated and restore-tested; use multi-AZ for availability and cross-account/Region copies for selected disaster scenarios. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Inspect target health reason codes and Auto Scaling activity, then correlate bootstrap/user-data logs, instance profile, listener/target port, health-check path, and security rules.

**Production scenario:** A backup job succeeds but restore fails. Test full restore workflows, permissions, dependencies, encryption keys, networking, and application consistency.

**Commands / checks:** ``aws backup list-backup-jobs`; `aws backup list-recovery-points-by-backup-vault`; restore-test records`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q45. A security group change causes intermittent failures rather than a complete outage. How would you use flow logs and connection evidence to diagnose it?

**Difficulty:** Advanced

**Interview-ready answer:** Check listener rules, target registration and health, subnet routing, security groups, NACL ephemeral ports, and the application’s actual listening port. Do not infer root cause from one status code. Design recovery around business-defined RTO and RPO. Backups must be isolated and restore-tested; use multi-AZ for availability and cross-account/Region copies for selected disaster scenarios. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Multi-AZ, AWS Backup, RTO/RPO, CloudWatch, budgets, DR testing Design recovery around business-defined RTO and RPO. Backups must be isolated and restore-tested; use multi-AZ for availability and cross-account/Region copies for selected disaster scenarios. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Check listener rules, target registration and health, subnet routing, security groups, NACL ephemeral ports, and the application’s actual listening port. Do not infer root cause from one status code.

**Production scenario:** A backup job succeeds but restore fails. Test full restore workflows, permissions, dependencies, encryption keys, networking, and application consistency.

**Commands / checks:** ``aws backup list-backup-jobs`; `aws backup list-recovery-points-by-backup-vault`; restore-test records`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q46. An application needs outbound access to a third-party API but must not have unrestricted internet egress. What controls would you design?

**Difficulty:** Intermediate

**Interview-ready answer:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available. Design recovery around business-defined RTO and RPO. Backups must be isolated and restore-tested; use multi-AZ for availability and cross-account/Region copies for selected disaster scenarios. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Multi-AZ, AWS Backup, RTO/RPO, CloudWatch, budgets, DR testing Design recovery around business-defined RTO and RPO. Backups must be isolated and restore-tested; use multi-AZ for availability and cross-account/Region copies for selected disaster scenarios. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Trace the full packet path in both directions: subnet route table, NAT/Internet Gateway or VPC endpoint, security-group statefulness, NACL rules, DNS, and destination response route. Use Flow Logs or Reachability Analyzer where available.

**Production scenario:** A backup job succeeds but restore fails. Test full restore workflows, permissions, dependencies, encryption keys, networking, and application consistency.

**Commands / checks:** ``aws backup list-backup-jobs`; `aws backup list-recovery-points-by-backup-vault`; restore-test records`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q47. How do you validate that a backup copy in another account is isolated from production administrators?

**Difficulty:** Intermediate

**Interview-ready answer:** Treat recovery as an end-to-end test: restore data, decrypt it, recreate dependencies and network access, validate application consistency, and measure achieved RTO/RPO. A successful backup job alone is insufficient. Design recovery around business-defined RTO and RPO. Backups must be isolated and restore-tested; use multi-AZ for availability and cross-account/Region copies for selected disaster scenarios. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Multi-AZ, AWS Backup, RTO/RPO, CloudWatch, budgets, DR testing Design recovery around business-defined RTO and RPO. Backups must be isolated and restore-tested; use multi-AZ for availability and cross-account/Region copies for selected disaster scenarios. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Treat recovery as an end-to-end test: restore data, decrypt it, recreate dependencies and network access, validate application consistency, and measure achieved RTO/RPO. A successful backup job alone is insufficient.

**Production scenario:** A backup job succeeds but restore fails. Test full restore workflows, permissions, dependencies, encryption keys, networking, and application consistency.

**Commands / checks:** ``aws backup list-backup-jobs`; `aws backup list-recovery-points-by-backup-vault`; restore-test records`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q48. A team is considering Spot Instances for a stateless service. What interruption handling and capacity diversification are required?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect target health reason codes and Auto Scaling activity, then correlate bootstrap/user-data logs, instance profile, listener/target port, health-check path, and security rules. Design recovery around business-defined RTO and RPO. Backups must be isolated and restore-tested; use multi-AZ for availability and cross-account/Region copies for selected disaster scenarios. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Multi-AZ, AWS Backup, RTO/RPO, CloudWatch, budgets, DR testing Design recovery around business-defined RTO and RPO. Backups must be isolated and restore-tested; use multi-AZ for availability and cross-account/Region copies for selected disaster scenarios. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Inspect target health reason codes and Auto Scaling activity, then correlate bootstrap/user-data logs, instance profile, listener/target port, health-check path, and security rules.

**Production scenario:** A backup job succeeds but restore fails. Test full restore workflows, permissions, dependencies, encryption keys, networking, and application consistency.

**Commands / checks:** ``aws backup list-backup-jobs`; `aws backup list-recovery-points-by-backup-vault`; restore-test records`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q49. An EC2 role can access metadata credentials from a container unexpectedly. What hardening and workload isolation controls would you review?

**Difficulty:** Intermediate

**Interview-ready answer:** Separate role trust (who can assume it) from permission policy (what it can do), then check explicit denies from SCPs, permissions boundaries, session policies, and resource policies. Confirm the actual caller with STS. Design recovery around business-defined RTO and RPO. Backups must be isolated and restore-tested; use multi-AZ for availability and cross-account/Region copies for selected disaster scenarios. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Multi-AZ, AWS Backup, RTO/RPO, CloudWatch, budgets, DR testing Design recovery around business-defined RTO and RPO. Backups must be isolated and restore-tested; use multi-AZ for availability and cross-account/Region copies for selected disaster scenarios. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Separate role trust (who can assume it) from permission policy (what it can do), then check explicit denies from SCPs, permissions boundaries, session policies, and resource policies. Confirm the actual caller with STS.

**Production scenario:** A backup job succeeds but restore fails. Test full restore workflows, permissions, dependencies, encryption keys, networking, and application consistency.

**Commands / checks:** ``aws backup list-backup-jobs`; `aws backup list-recovery-points-by-backup-vault`; restore-test records`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q50. After a cloud incident, what evidence would you preserve from CloudTrail, VPC Flow Logs, and workload logs before remediation?

**Difficulty:** Advanced

**Interview-ready answer:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident. Design recovery around business-defined RTO and RPO. Backups must be isolated and restore-tested; use multi-AZ for availability and cross-account/Region copies for selected disaster scenarios. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Multi-AZ, AWS Backup, RTO/RPO, CloudWatch, budgets, DR testing Design recovery around business-defined RTO and RPO. Backups must be isolated and restore-tested; use multi-AZ for availability and cross-account/Region copies for selected disaster scenarios. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident.

**Production scenario:** A backup job succeeds but restore fails. Test full restore workflows, permissions, dependencies, encryption keys, networking, and application consistency.

**Commands / checks:** ``aws backup list-backup-jobs`; `aws backup list-recovery-points-by-backup-vault`; restore-test records`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

## Official references

- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
- [IAM best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [AWS Backup developer guide](https://docs.aws.amazon.com/aws-backup/latest/devguide/)

## Version and safety note

This guide is dated 18 October 2026. Release status, feature maturity, cloud-region availability, plugins, and supported versions change. Check the linked official documentation against the exact versions and architecture you use before applying any production recommendation.
