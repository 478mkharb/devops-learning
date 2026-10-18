# Terraform & Infrastructure as Code — 50 Advanced Interview Questions

**Audience:** Mid-level/L2 DevOps Engineer (3–5 years).
**Reference date:** 18 October 2026.

**Focus:** State, plans, drift, module design, provider management, testing, policy and safe changes

## How to practise
- First answer the **interview-ready answer** aloud in 60–90 seconds.
- Then explain the technical reasoning, trade-offs, and evidence you would collect.
- Treat commands as diagnostic starting points; replace placeholders and validate permissions.
- Do not run destructive or mutating commands in production without an approved change plan.
- Use the follow-up to test whether you understand the failure mechanism rather than memorizing a script.

## Questions

### Q1. A plan proposes replacing a production database. What do you do before applying?

**Difficulty:** Intermediate

**Interview-ready answer:** Stop the apply. Inspect the replacement marker and attribute, compare provider/module changes and drift, confirm backups and restore feasibility, then obtain review for a safe migration plan.

**Technical explanation:** Use a saved plan and inspect it. `prevent_destroy` can help but is not a universal safeguard; removing the resource from configuration may remove that protection.

**Production scenario:** Applying first and investigating afterward can cause irreversible data loss.

**Commands / checks:** ``terraform plan -out=tfplan`; `terraform show tfplan`; `terraform state show <address>``

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** What would you do if the plan shows replacement because an immutable identifier changed?

---

### Q2. Two CI jobs try to apply against the same S3 state. How do you protect state?

**Difficulty:** Intermediate

**Interview-ready answer:** Use a supported remote backend locking mechanism, restrict state access, enable S3 versioning and encryption, and serialize changes to the same state.

**Technical explanation:** For S3 backend, `use_lockfile = true` is supported by current Terraform. Confirm the team’s Terraform version and backend configuration before migration.

**Production scenario:** Force-unlocking while another operation is active can corrupt coordination.

**Commands / checks:** ``terraform plan`; S3 backend configuration; inspect active CI runs and lock owner`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** How do you safely recover a stale lock after confirming no active run exists?

---

### Q3. An engineer changes a security group manually in the console. How do you detect and resolve drift?

**Difficulty:** Advanced

**Interview-ready answer:** Run a reviewed plan and compare observed infrastructure with code. Decide whether to revert the manual change or codify the intended change; do not blindly apply if the change may be security-critical.

**Technical explanation:** Use refresh-aware plans, audit events, and drift detection. Coordinate emergency changes back into code after mitigation.

**Production scenario:** Automatically reverting all drift can undo a legitimate emergency mitigation.

**Commands / checks:** ``terraform plan`; cloud audit logs; `terraform plan -refresh-only``

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** How would you handle a manual change made during an active incident?

---

### Q4. A team needs to bring a manually created S3 bucket under Terraform. How do you do it safely?

**Difficulty:** Intermediate

**Interview-ready answer:** Write matching configuration, import the resource using the supported import workflow, inspect state and plan, and ensure the first apply does not unexpectedly replace or mutate it.

**Technical explanation:** Import associates existing infrastructure with state; it does not automatically generate complete, correct configuration.

**Production scenario:** Importing without matching configuration can create a destructive follow-up plan.

**Commands / checks:** ``terraform import` or `import` block; `terraform plan`; `terraform state show``

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** How do you confirm the imported bucket’s lifecycle and policy are represented accurately?

---

### Q5. An AWS provider upgrade shows many changes across resources. What is your process?

**Difficulty:** Advanced

**Interview-ready answer:** Review provider changelog and schema changes, pin the current version, test the upgrade in a branch/environment, classify expected and unexpected diffs, then roll out gradually.

**Technical explanation:** Use lock files and controlled `terraform init -upgrade`; avoid combining provider upgrades with unrelated infrastructure changes.

**Production scenario:** A broad plan diff is not safe to dismiss as cosmetic.

**Commands / checks:** ``terraform providers`; `terraform plan`; provider changelog`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** What evidence would justify postponing the provider upgrade?

---

### Q6. A root module requires AWS provider v6 while a child module constrains v5. How do you fix it?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect the dependency graph and constraints. Upgrade or refactor the child module to a compatible range, or deliberately use compatible module/provider versions; test the resulting plan.

**Technical explanation:** Provider constraints across modules must resolve to a compatible version. Provider aliases and explicit provider passing may be needed for multi-account designs.

**Production scenario:** Widening version constraints without testing can hide incompatibilities.

**Commands / checks:** ``terraform providers`; `terraform init`; inspect module `required_providers``

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** When should a module declare a minimum version versus a narrow tested range?

---

### Q7. How do you protect Terraform state that may contain passwords or tokens?

**Difficulty:** Intermediate

**Interview-ready answer:** Treat state as sensitive. Restrict backend IAM, encrypt at rest and in transit, enable versioning and audit access, avoid outputs that expose secrets, and rotate secrets if state leaks.

**Technical explanation:** Marking a value `sensitive` suppresses some CLI display but does not guarantee it is absent from state.

**Production scenario:** A sensitive flag is not encryption and does not remove data from state.

**Commands / checks:** `S3 bucket policy; IAM audit; state access logs; secret rotation`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** How would you respond if a state file was accidentally committed to Git?

---

### Q8. Would you use workspaces or separate directories/backends for dev and prod?

**Difficulty:** Advanced

**Interview-ready answer:** Choose based on isolation, access boundaries, and operational workflow. Production generally needs distinct permissions and state isolation; workspaces alone are not a security boundary.

**Technical explanation:** Use separate state keys/backends or accounts when blast-radius and access separation require it.

**Production scenario:** Selecting a workspace is not equivalent to changing AWS credentials or enforcing authorization.

**Commands / checks:** `Backend config; workspace list; IAM policies; CI environment mapping`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** How would you prevent a dev pipeline from applying to production?

---

### Q9. You must deploy a central logging bucket in one account and app resources in another. How would you model it?

**Difficulty:** Intermediate

**Interview-ready answer:** Use separate provider configurations with explicit aliases and role assumptions, pass the correct provider to modules, and test resource placement and permissions.

**Technical explanation:** Keep account IDs/roles explicit, use temporary credentials, and validate the plan’s target account before apply.

**Production scenario:** A missing provider alias can place resources in the default account.

**Commands / checks:** ``aws sts get-caller-identity`; provider alias config; plan review`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** What CI guard would reject a plan targeting the wrong account?

---

### Q10. You rename a module or move a resource into a module. Terraform plans to destroy and recreate it. How do you prevent that?

**Difficulty:** Advanced

**Interview-ready answer:** Use a `moved` block or a carefully planned state move so Terraform understands the address changed but the real resource did not.

**Technical explanation:** Review the plan after the refactor and retain migration history in code.

**Production scenario:** Changing a resource address without a migration can look like delete-and-create.

**Commands / checks:** ``terraform state list`; `moved` block; `terraform plan``

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** When is a state move unsafe or insufficient?

---

### Q11. How do you ensure the exact plan reviewed by an approver is the plan applied?

**Difficulty:** Intermediate

**Interview-ready answer:** Save the plan artifact, review it, protect it and its metadata, and apply that saved plan in the approved workflow rather than generating a new plan later.

**Technical explanation:** Plan artifacts may contain sensitive data and must be stored securely with limited retention.

**Production scenario:** A fresh plan at apply time may differ from the reviewed proposal.

**Commands / checks:** ``terraform plan -out=tfplan`; `terraform show tfplan`; `terraform apply tfplan``

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** What should happen if infrastructure changes after approval but before apply?

---

### Q12. How do you migrate state from local storage to an S3 backend?

**Difficulty:** Intermediate

**Interview-ready answer:** Back up the current state, configure the remote backend, run `terraform init` migration carefully, verify state and resource mapping, and confirm locking/versioning and team access.

**Technical explanation:** Perform in a controlled window with one operator, protect the backup, and ensure CI points to the same backend.

**Production scenario:** Multiple operators using local and remote state can create divergent state histories.

**Commands / checks:** ``terraform state list`; `terraform init -migrate-state`; verify remote state`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** How do you recover if the migration is interrupted?

---

### Q13. How would you protect a critical RDS instance from accidental destruction?

**Difficulty:** Advanced

**Interview-ready answer:** Use `prevent_destroy` where appropriate, restrictive IAM, separate state and approvals, backup/restore testing, and policy checks for destructive plans.

**Technical explanation:** No single guardrail is sufficient. Review how configuration removal and state changes affect lifecycle protection.

**Production scenario:** `prevent_destroy` does not replace backup strategy or access controls.

**Commands / checks:** `Lifecycle block; plan policy; IAM permissions; backup test`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** What happens if the resource block is removed entirely?

---

### Q14. What testing layers would you use for a reusable VPC module?

**Difficulty:** Intermediate

**Interview-ready answer:** Use formatting and validation, static policy scans, module tests, plan inspection, and integration tests in an isolated account or environment.

**Technical explanation:** Test network invariants such as private subnet routes, restricted public access, and expected outputs. Use disposable test accounts where possible.

**Production scenario:** A syntactically valid module can still create insecure or unusable networking.

**Commands / checks:** ``terraform fmt -check`; `terraform validate`; `terraform test`; Checkov`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** Which tests can run without creating cloud resources?

---

### Q15. A security policy blocks a valid resource exception. How should the team handle it?

**Difficulty:** Advanced

**Interview-ready answer:** Document the risk and business reason, use a scoped and time-bound exception with owner and expiry, and retain compensating controls. Improve the policy if the false positive is systematic.

**Technical explanation:** Do not disable the entire scanner or add broad suppressions.

**Production scenario:** Permanent global bypasses turn security gates into decoration.

**Commands / checks:** `Policy report; exception record; CI gate config`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** How would you measure whether a policy is too noisy?

---

### Q16. Terraform cost estimation is much lower than the AWS bill. What do you investigate?

**Difficulty:** Intermediate

**Interview-ready answer:** Check unmodeled services, data transfer, NAT, logging, snapshots, idle resources, usage assumptions, and pricing coverage. Compare estimates with billing and cost allocation tags.

**Technical explanation:** Cost tools estimate modeled resources; actual usage, discounts, and variable charges may differ.

**Production scenario:** A clean plan does not guarantee a predictable bill.

**Commands / checks:** `Cost Explorer; CUR; resource tags; cost estimate report`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** How would you set a budget alert for a newly deployed environment?

---

### Q17. A secret is passed as a Terraform variable. Is it safe if marked sensitive?

**Difficulty:** Intermediate

**Interview-ready answer:** Not necessarily. Sensitive flags mask output but may not keep the value out of state or logs. Prefer references to a secrets manager and carefully manage provider/resource behavior.

**Technical explanation:** Use ephemeral/write-only features only when supported by the relevant Terraform/provider versions; verify state behavior explicitly.

**Production scenario:** Assuming `sensitive = true` means “never stored” is incorrect.

**Commands / checks:** `State inspection in a safe test; secret manager references; log review`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** How would you rotate a secret that is already present in historical state versions?

---

### Q18. A `count`-based list changes order and plans to recreate resources. How would you redesign it?

**Difficulty:** Advanced

**Interview-ready answer:** Use stable `for_each` keys tied to meaningful identities where appropriate, and migrate resource addresses deliberately to avoid unintended replacement.

**Technical explanation:** Stable keys make identity clearer; changing from `count` to `for_each` may need moved blocks or state moves.

**Production scenario:** Index-based identity can turn reordering into infrastructure churn.

**Commands / checks:** ``terraform plan`; resource address list; `for_each` map keys`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** When is `count` still a reasonable choice?

---

### Q19. Would you run Terraform applies from developer laptops or a controlled CI runner?

**Difficulty:** Intermediate

**Interview-ready answer:** For production, use a controlled, auditable workflow with short-lived credentials, reviewed plans, policy gates, and clear approvals. Local use may be acceptable for isolated development.

**Technical explanation:** Protect plan/state artifacts and restrict who can assume apply roles.

**Production scenario:** A centralized runner is not secure if its credentials or workflow definitions are broadly writable.

**Commands / checks:** `CI permissions; IAM trust policy; audit logs`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** How would you allow emergency changes without losing auditability?

---

### Q20. A data source intermittently fails because a resource is not yet created. What would you change?

**Difficulty:** Advanced

**Interview-ready answer:** Understand Terraform’s dependency graph and use explicit references where possible. Avoid using arbitrary sleeps as the primary fix; handle eventual consistency with supported mechanisms and staged workflows.

**Technical explanation:** Separate independently managed resources where useful and design for eventual consistency.

**Production scenario:** A hard-coded delay may fail under a slower environment and waste time otherwise.

**Commands / checks:** `Plan graph; provider logs; dependency references`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** When is `depends_on` appropriate for a data source?

---

### Q21. A team uses `ignore_changes = all` to stop noisy diffs. Is that acceptable?

**Difficulty:** Intermediate

**Interview-ready answer:** Usually not without a clear ownership reason. Ignore only attributes intentionally managed by another controller and document why; broad ignores hide drift and configuration errors.

**Technical explanation:** Use narrowly scoped ignore rules and monitor externally managed fields.

**Production scenario:** Silencing a diff does not resolve the underlying ownership conflict.

**Commands / checks:** `Lifecycle config; plan output; controller ownership docs`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** Give an example where a narrow `ignore_changes` is justified.

---

### Q22. Different engineers use different Terraform versions. What risks does that create?

**Difficulty:** Intermediate

**Interview-ready answer:** Provider resolution, syntax support, state behavior, and plan output can differ. Pin the CLI version, commit the provider lock file, and validate in CI.

**Technical explanation:** Use version constraints and a consistent toolchain; upgrade deliberately.

**Production scenario:** The provider lock file does not pin the Terraform CLI version itself.

**Commands / checks:** ``terraform version`; `.terraform.lock.hcl`; CI tool image`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** How do you upgrade the CLI and provider without mixing unrelated changes?

---

### Q23. How would you detect console changes without applying them automatically?

**Difficulty:** Advanced

**Interview-ready answer:** Run scheduled plans or managed drift detection, alert on meaningful changes, classify emergency versus unauthorized drift, and reconcile through reviewed code.

**Technical explanation:** Use read-only credentials for detection and separate permissions for apply.

**Production scenario:** Automatically applying every drift can undo a legitimate emergency fix.

**Commands / checks:** `Scheduled plan output; cloud audit events; drift reports`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** What makes drift alert actionable rather than noisy?

---

### Q24. A pipeline cannot acquire the state lock. What is the safe procedure?

**Difficulty:** Intermediate

**Interview-ready answer:** Check whether another apply is running, identify lock owner and operation, wait if active, and recover a stale lock only after confirming it is abandoned.

**Technical explanation:** Use pipeline concurrency controls to reduce collisions and preserve lock metadata for diagnosis.

**Production scenario:** Force-unlocking a live operation can damage state coordination.

**Commands / checks:** `CI run status; lock metadata; backend logs`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** What evidence is enough to declare a lock stale?

---

### Q25. How should a team version and publish shared Terraform modules?

**Difficulty:** Advanced

**Interview-ready answer:** Use semantic versioning where appropriate, immutable tagged releases, changelogs, compatibility tests, and controlled adoption in consumer repositories.

**Technical explanation:** Avoid relying on a moving branch for production modules; automate consumer validation.

**Production scenario:** A tag convention without compatibility tests does not guarantee safe upgrades.

**Commands / checks:** `Module registry/tag; changelog; integration tests`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** How would you introduce a breaking module input change?

---

### Q26. A team still uses DynamoDB locking for the S3 backend. What is the migration plan?

**Difficulty:** Intermediate

**Interview-ready answer:** Review current Terraform support and team version, back up state, plan a maintenance window, migrate configuration to S3 lock files if appropriate, and test concurrent operation behavior.

**Technical explanation:** Do not remove old locking until all clients and CI jobs use the new supported configuration.

**Production scenario:** Mixed clients using different lock strategies may not coordinate as expected.

**Commands / checks:** `Terraform S3 backend docs; version inventory; concurrent plan test`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** How do you inventory every runner that uses the state?

---

### Q27. A VPC module upgrade makes a subnet route to an Internet Gateway. What do you do?

**Difficulty:** Intermediate

**Interview-ready answer:** Stop the apply, inspect route table and module defaults, compare plan/provider changes, and confirm the desired public/private design before proceeding.

**Technical explanation:** Add policy/tests that assert private subnets do not have unintended public routes or public IP assignment.

**Production scenario:** A subnet name containing `private` is not proof of its route behavior.

**Commands / checks:** `Plan diff; route table; subnet attributes; IaC policy scan`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** What specific route and IP settings define the exposure?

---

### Q28. Two independent pipelines change resources in a shared state. How do you reduce collision risk?

**Difficulty:** Advanced

**Interview-ready answer:** Coordinate applies for the same state, split state only along clear ownership boundaries, and avoid splitting merely to bypass locks.

**Technical explanation:** State boundaries should reflect dependencies and blast radius; cross-state references need deliberate contracts.

**Production scenario:** More state files can increase coordination complexity and create hidden dependencies.

**Commands / checks:** `CI concurrency settings; backend key design; dependency graph`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** When should resources share a state versus separate states?

---

### Q29. Kubernetes or autoscaling changes an attribute Terraform manages, causing perpetual diffs. How do you resolve ownership?

**Difficulty:** Intermediate

**Interview-ready answer:** Decide which controller owns the attribute. Model stable desired state in Terraform and avoid conflicting ownership; narrowly ignore controller-managed fields only when justified.

**Technical explanation:** Document ownership boundaries and monitor the controller’s effective configuration.

**Production scenario:** Two controllers managing the same field create drift loops.

**Commands / checks:** `Plan diff; controller config; lifecycle rules`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** How would you handle an autoscaled desired capacity field?

---

### Q30. Terraform reports a dependency cycle after module refactoring. How do you debug it?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect references and `depends_on`, identify the circular dependency, and redesign module interfaces or split resources so the graph is acyclic.

**Technical explanation:** Avoid broad module dependencies; pass outputs as explicit inputs where possible.

**Production scenario:** Adding more `depends_on` statements cannot fix a real cycle.

**Commands / checks:** ``terraform plan`; graph output where useful; module reference review`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** How do you distinguish a genuine dependency from a sequencing workaround?

---

### Q31. A pipeline archives `terraform output -json`. What risk does this create?

**Difficulty:** Intermediate

**Interview-ready answer:** The output may contain sensitive values. Restrict access and retention, avoid archiving unnecessary outputs, and use secret references instead of exposing secret values.

**Technical explanation:** Review artifacts and logs as data stores with access control and retention.

**Production scenario:** Marking output sensitive does not guarantee all downstream artifact handling is safe.

**Commands / checks:** ``terraform output`; artifact ACLs; pipeline logs`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** How would you audit historical pipeline artifacts for exposure?

---

### Q32. A cleanup job plans to destroy resources in an unexpected environment. How do you stop it?

**Difficulty:** Intermediate

**Interview-ready answer:** Fail closed on account, workspace, backend, and environment mismatch. Require explicit environment selection and approvals for destructive plans.

**Technical explanation:** Use dedicated credentials and policy checks that reject unexpected destroy actions.

**Production scenario:** A workspace name alone is not an authorization boundary.

**Commands / checks:** ``terraform workspace show`; `aws sts get-caller-identity`; plan policy`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** What checks should run before any production destroy?

---

### Q33. A resource type requires replacement but stores important data. How do you plan the change?

**Difficulty:** Advanced

**Interview-ready answer:** Treat it as a migration: create a compatible target if possible, copy/replicate data, validate consistency, switch traffic, and retain a tested recovery path.

**Technical explanation:** The correct method depends on the service’s replication and consistency semantics.

**Production scenario:** Service-specific migration docs; backup/restore test; cutover metrics

**Commands / checks:** `Inspect relevant logs, configuration, health metrics, and recent change history; use the topic guide's diagnostic commands as a starting point.`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** Terraform cannot automatically make a data migration safe just because it can replace a resource.

---

### Q34. A policy engine cannot evaluate some plan values because they are unknown until apply. What do you do?

**Difficulty:** Intermediate

**Interview-ready answer:** Use policies that handle unknowns safely, split workflows where necessary, and require runtime/post-deployment verification for properties unavailable at plan time.

**Technical explanation:** Avoid treating unknown as automatically compliant; define explicit fail/approval behavior.

**Production scenario:** Failing open on unknown security-critical values creates a bypass.

**Commands / checks:** `Plan JSON; policy evaluation output; post-apply checks`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** Which checks must be known before apply, and which can be verified afterward?

---

### Q35. A large apply fails with cloud API throttling. How do you improve it?

**Difficulty:** Advanced

**Interview-ready answer:** Reduce unnecessary parallelism if evidence supports it, split changes into logical batches, retry transient throttles safely, and coordinate with service quotas.

**Technical explanation:** Balance speed with API limits; avoid changing global parallelism without measuring impact.

**Production scenario:** Terraform logs; cloud API throttling metrics; apply duration

**Commands / checks:** `Inspect relevant logs, configuration, health metrics, and recent change history; use the topic guide's diagnostic commands as a starting point.`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** More parallelism can worsen throttling and make failures harder to recover from.

---

### Q36. A team member can read production state but should only manage dev. How do you fix it?

**Difficulty:** Intermediate

**Interview-ready answer:** Separate state access permissions and roles by environment, audit state readers, and rotate any secrets exposed through unauthorized access.

**Technical explanation:** State access can reveal sensitive infrastructure data even without apply permissions.

**Production scenario:** S3 bucket policy; IAM policy simulation; CloudTrail

**Commands / checks:** `Inspect relevant logs, configuration, health metrics, and recent change history; use the topic guide's diagnostic commands as a starting point.`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** Read-only state access can still be a security incident.

---

### Q37. Why might declarative import blocks be preferable to a one-off CLI import?

**Difficulty:** Intermediate

**Interview-ready answer:** They make the import intent reviewable in version control and repeatable in the normal plan/apply workflow. Still inspect the resulting state and plan carefully.

**Technical explanation:** Choose the workflow supported by the Terraform version and team process.

**Production scenario:** A declarative import does not automatically produce complete configuration.

**Commands / checks:** `Import block; `terraform plan`; state inspection`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** How would you ensure the first post-import apply is non-destructive?

---

### Q38. A module’s output CIDR is wrong but all resources were created successfully. How do you catch it?

**Difficulty:** Advanced

**Interview-ready answer:** Test output values and invariants, not just successful resource creation. Validate CIDR ranges, subnet counts, route associations, and expected security boundaries.

**Technical explanation:** Use unit/plan tests where supported and integration checks for real cloud behavior.

**Production scenario:** `terraform test`; output assertions; route table inspection

**Commands / checks:** `Inspect relevant logs, configuration, health metrics, and recent change history; use the topic guide's diagnostic commands as a starting point.`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** Successful provisioning does not prove the architecture matches requirements.

---

### Q39. What should a reviewer look for beyond additions and deletions?

**Difficulty:** Intermediate

**Interview-ready answer:** Review replacements, IAM permission expansion, public exposure, encryption, network routes, data retention, cost, dependency ordering, and sensitive changes.

**Technical explanation:** Use structured plan output and policy checks; ask for context on high-risk diffs.

**Production scenario:** `terraform show tfplan`; JSON plan policy; security scan

**Commands / checks:** `Inspect relevant logs, configuration, health metrics, and recent change history; use the topic guide's diagnostic commands as a starting point.`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** A small line count can hide a large privilege or exposure change.

---

### Q40. How should Terraform authenticate to its remote backend in CI?

**Difficulty:** Advanced

**Interview-ready answer:** Use temporary workload identity or role assumption, scoped to the required state bucket and lock operations; avoid long-lived static keys.

**Technical explanation:** Separate backend access from provider permissions where feasible and audit both.

**Production scenario:** CI identity; STS identity; backend bucket policy

**Commands / checks:** `Inspect relevant logs, configuration, health metrics, and recent change history; use the topic guide's diagnostic commands as a starting point.`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** Provider credentials and backend credentials may be initialized and managed differently.

---

### Q41. An operator deletes the remote state object. What is your response?

**Difficulty:** Intermediate

**Interview-ready answer:** Stop all Terraform runs, recover a known-good versioned state object, validate resource mappings, and run a plan before any apply. If no valid backup exists, carefully reconstruct state/import resources.

**Technical explanation:** Use S3 versioning and test recovery; coordinate a single operator during repair.

**Production scenario:** S3 object versions; state backup; `terraform state list`; plan review

**Commands / checks:** `Inspect relevant logs, configuration, health metrics, and recent change history; use the topic guide's diagnostic commands as a starting point.`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** Creating a new empty state and applying can create duplicates or destructive changes.

---

### Q42. How do you organize Terraform for several applications and environments in one repository?

**Difficulty:** Intermediate

**Interview-ready answer:** Separate reusable modules from environment roots, define clear state boundaries, pin module versions, and give each environment explicit credentials and pipelines.

**Technical explanation:** Keep shared module changes tested against consumers and use code ownership for sensitive roots.

**Production scenario:** Repository layout; backend keys; CI path filters

**Commands / checks:** `Inspect relevant logs, configuration, health metrics, and recent change history; use the topic guide's diagnostic commands as a starting point.`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** A directory structure alone does not isolate IAM permissions or state.

---

### Q43. A production plan requires a temporary public endpoint for migration. How do you govern it?

**Difficulty:** Advanced

**Interview-ready answer:** Require documented justification, scope the exposure narrowly, time-limit the exception, add compensating controls and automatic expiry, and verify closure.

**Technical explanation:** Policy exceptions should be auditable and tied to a change ticket.

**Production scenario:** Exception record; security group rules; expiry automation

**Commands / checks:** `Inspect relevant logs, configuration, health metrics, and recent change history; use the topic guide's diagnostic commands as a starting point.`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** A temporary exception without an expiry can become permanent exposure.

---

### Q44. A provider upgrade, console drift, and a state-lock error occur on the same day. What order do you follow?

**Difficulty:** Intermediate

**Interview-ready answer:** Freeze applies, confirm whether an operation is active, preserve state versions, inspect drift and provider changes separately, then reconcile in a controlled sequence with reviewed plans.

**Technical explanation:** Avoid mixing unrelated changes and keep a timeline of state and infrastructure actions.

**Production scenario:** CI runs; lock metadata; state versions; plan output; audit logs

**Commands / checks:** `Inspect relevant logs, configuration, health metrics, and recent change history; use the topic guide's diagnostic commands as a starting point.`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** Trying to fix all three simultaneously obscures cause and increases blast radius.

---

### Q45. A CI runner unexpectedly selects a different provider version from a developer laptop. How do you make provider selection reproducible?

**Difficulty:** Advanced

**Interview-ready answer:** Commit `.terraform.lock.hcl`, constrain provider versions, use a consistent Terraform CLI version, and review deliberate lock-file updates.

**Technical explanation:** `terraform version`; inspect `.terraform.lock.hcl`; `terraform providers`; `terraform init -lockfile=readonly`

**Production scenario:** A provider lock file does not pin the Terraform CLI itself.

**Commands / checks:** `Inspect relevant logs, configuration, health metrics, and recent change history; use the topic guide's diagnostic commands as a starting point.`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** How would you safely roll out a provider update across several roots?

---

### Q46. A production pipeline initializes the backend successfully but then targets the wrong AWS account. What guardrails would you add?

**Difficulty:** Intermediate

**Interview-ready answer:** Verify backend identity and provider identity separately, use account-scoped roles, and fail the pipeline if the expected account/region does not match.

**Technical explanation:** `aws sts get-caller-identity`; inspect backend config and provider aliases; plan policy

**Production scenario:** Backend access and provider resource permissions are separate concerns.

**Commands / checks:** `Inspect relevant logs, configuration, health metrics, and recent change history; use the topic guide's diagnostic commands as a starting point.`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** Where would you enforce an account allowlist so it cannot be bypassed by a developer?

---

### Q47. A Terraform plan uses an old resource attribute after an external change. How do you reason about refresh and dependencies?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect when the data source is evaluated, refresh behavior, dependencies, and eventual consistency. Avoid relying on arbitrary sleeps; redesign dependencies or split stages if needed.

**Technical explanation:** `terraform plan`; inspect dependency references and provider logs

**Production scenario:** Refresh is not a guarantee that every remote API immediately returns strongly consistent data.

**Commands / checks:** `Inspect relevant logs, configuration, health metrics, and recent change history; use the topic guide's diagnostic commands as a starting point.`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** When is an explicit dependency appropriate for a data source?

---

### Q48. A deployment job accidentally applies to the default workspace instead of production. How do you prevent this?

**Difficulty:** Advanced

**Interview-ready answer:** Make environment and backend selection explicit, assert workspace/account/region before plan and apply, and separate production credentials and state access.

**Technical explanation:** `terraform workspace show`; `aws sts get-caller-identity`; CI environment assertions

**Production scenario:** Workspace selection alone is not an authorization boundary.

**Commands / checks:** `Inspect relevant logs, configuration, health metrics, and recent change history; use the topic guide's diagnostic commands as a starting point.`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** What should the pipeline do when the environment assertion fails?

---

### Q49. A Terraform plan changes 400 resources after a refactor. How do you make review safe?

**Difficulty:** Intermediate

**Interview-ready answer:** Stop and split the change. Use plan JSON or grouped diffs to classify replacements, permission changes, exposure, and unrelated churn; validate smaller batches in staging.

**Technical explanation:** `terraform show tfplan`; plan JSON policy; module diff

**Production scenario:** A large plan is not necessarily wrong, but it is difficult to review reliably as one change.

**Commands / checks:** `Inspect relevant logs, configuration, health metrics, and recent change history; use the topic guide's diagnostic commands as a starting point.`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** Which classes of changes should require separate approval?

---

### Q50. Terraform repeatedly fails with API throttling during a large apply. How do you reduce failures?

**Difficulty:** Advanced

**Interview-ready answer:** Confirm throttling from logs, reduce concurrency if appropriate, batch changes, and use supported retry behavior. Coordinate with quotas and avoid repeating unsafe partial operations.

**Technical explanation:** Terraform logs; cloud API throttling metrics; `-parallelism` after measurement

**Production scenario:** Increasing parallelism can make throttling worse.

**Commands / checks:** `Inspect relevant logs, configuration, health metrics, and recent change history; use the topic guide's diagnostic commands as a starting point.`

**Common mistake:** Changing several variables at once, relying on one green status, or making an irreversible change before verifying the failure domain.

**Follow-up:** How do you determine whether throttling is caused by Terraform concurrency or other automation?

---

## Official references

- [Terraform releases](https://github.com/hashicorp/terraform/releases)
- [S3 backend and lock files](https://developer.hashicorp.com/terraform/language/backend/s3)
- [Terraform tests](https://developer.hashicorp.com/terraform/language/tests)

## Version and safety note

This guide is dated 18 October 2026. Release status, feature maturity, cloud-region availability, plugins, and supported versions change. Check the linked official documentation against the exact versions and architecture you use before applying any production recommendation.
