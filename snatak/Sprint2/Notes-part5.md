# Sprint-2 Interview & Implementation Handbook
## Part 5 of N — Ansible Role CI/CD & Ansible Playbook CI/CD

---

# SECTION A: Ansible Role CI/CD

## A1. Introduction — What & Why Ansible Role CI

**What:** Ansible Roles are the reusable, self-contained units of automation (tasks, handlers, templates, defaults, vars) used across this project — e.g., the `sonarqube_setup` and `jenkins_controller_setup` roles documented in Part 3. "CI for an Ansible role" means the role's code itself goes through the same rigor as application code: linting, syntax validation, and automated testing **before** it's ever applied against a real environment.

**Why:** an Ansible role is executable infrastructure logic — a bug in a role doesn't just break "code," it can misconfigure or take down real running infrastructure (Jenkins, SonarQube, app servers). Treating roles as untested scripts that get "tried live" is a common and expensive anti-pattern; CI on the role catches syntax errors, bad variable defaults, and idempotency breaks *before* they ever touch an instance.

## A2. CI Documentation

### Prerequisites
- Python + `ansible-core` and `ansible-lint` installed in the CI agent/container image.
- Molecule (with a Docker or Podman driver) for actual convergence testing — the industry-standard framework for testing Ansible roles in isolation.
- A pinned `requirements.yml` for any role/collection dependencies, resolved the same way application dependencies are (see Part 2/4's dependency-management philosophy).

### Code Linting & Testing Steps
```
Checkout → 
Stage 1: Syntax Check (ansible-playbook --syntax-check) → 
Stage 2: Lint (ansible-lint against role directory) → 
Stage 3: Molecule Test (create → converge → idempotence → verify → destroy) → 
Stage 4: Publish results (JUnit-format test report if using Molecule's test reporting)
```

### Pipeline Structure (Jenkinsfile stages, described)
1. **Lint stage:** `ansible-lint roles/sonarqube_setup/` — catches style violations and known-bad patterns (e.g., using `shell` where a proper module exists, missing `become: true` where required, hardcoded paths).
2. **Molecule test stage:** `molecule test` — this single command internally runs a full lifecycle:
   - **create:** spins up a disposable Docker container as the test target.
   - **converge:** runs the role against that container (the actual "does it work" test).
   - **idempotence:** runs the role a **second time** against the same container and asserts **zero changes reported** — this is the single most important test for infrastructure-as-code, since a non-idempotent role (one that reports "changed" every single run) is a sign of a poorly written task that will cause unnecessary drift/restarts in production.
   - **verify:** runs assertions (via Testinfra or Ansible's own `assert` module) confirming the end state is actually correct (e.g., "is the SonarQube systemd service active," "is port 9000 listening").
   - **destroy:** tears down the test container, leaving no residue.
3. **Report stage:** archive Molecule's test output as a build artifact; fail the build on any non-idempotent result or failed verification.

### Tool Versions
Ansible-core version and `ansible-lint` version pinned explicitly in a `requirements.txt`/CI image, matching the version that will actually run the role in production (Ansible behavior has historically had breaking changes across major versions — testing against a different version than production uses defeats the purpose of the test).

### Error Handling
- Syntax errors fail fast at Stage 1 (cheapest possible check, seconds not minutes).
- A non-idempotent role fails the Molecule stage explicitly with a clear message identifying which task reported "changed" on the second run — pointing the developer directly at the offending task rather than a generic failure.
- Molecule's `destroy` step is configured to run even on failure (`molecule test --destroy=always` semantics) so failed CI runs don't leave orphaned test containers accumulating on the CI agent.

### Working Example of Automated Validation
For the `sonarqube_setup` role specifically: Molecule's `verify.yml` would assert the SonarQube service is running, port 9000 is reachable, `vm.max_map_count` was actually set to the required value, and the templated `sonar.properties` file contains the expected DB connection string pattern (without asserting on the literal secret value, which shouldn't appear in a test fixture at all — a synthetic test DB credential is used instead).

## A3. CD Documentation — How the Role Is Deployed to Target Environments

### What & Why Ansible Role's CD
Once a role passes CI (lint + Molecule), CD is the process of actually **applying** that role against real target environments (Dev → Staging → Prod) in a controlled, auditable, promotion-based way — not simply "run the playbook manually whenever someone remembers to."

### Steps
```
Role passes CI on merge to main → 
Tag/version the role (e.g., git tag matching semantic version) → 
CD Pipeline Stage: Deploy to Dev (automatic on merge) → 
Manual approval gate → 
Deploy to Staging → 
Manual approval gate (with change-ticket reference) → 
Deploy to Prod
```

### Environment Requirements
- A per-environment inventory file (or dynamic inventory sourced from AWS EC2 tags) so the same role runs against the correct target hosts per environment without hardcoding IPs.
- Environment-specific variables (`group_vars/dev.yml`, `group_vars/prod.yml`) overriding role defaults — e.g., different instance sizing hints, different DB endpoints — kept out of the role itself so the role stays environment-agnostic and reusable.
- Ansible Vault (or an external secrets backend like AWS Secrets Manager via a lookup plugin) for any environment-specific secrets, never plaintext in `group_vars`.

### Best Practices
Promotion-based deployment (Dev → Staging → Prod, never skipping straight to Prod), always run `--check` (dry-run) mode as a pre-Prod-deploy safety stage showing a diff of intended changes for human review, tag/version roles so a specific "known-good" version can be pinned/rolled back to explicitly rather than always deploying "whatever main currently is."

---

# SECTION B: Ansible Playbook CI/CD

## B1. Introduction — What & Why Ansible Playbook CI

**What:** a Playbook is the top-level orchestration file that composes one or more Roles against a target inventory to achieve an end-to-end outcome (e.g., "provision and fully configure the Jenkins controller environment"). Playbook CI validates that this composition — the orchestration logic itself, not just the individual roles — is syntactically correct and safe before execution.

**Why:** even if every individual role is well-tested in isolation (Section A), the playbook that stitches them together can still have bugs — wrong role execution order, missing `become` at the play level, incorrect inventory group targeting — that only manifest when they're combined. Playbook-level CI catches composition errors specifically.

### How the Playbook Is Tested
- **Syntax check:** `ansible-playbook site.yml --syntax-check` — parses the YAML and validates structural correctness (task keys, module names exist) without executing anything.
- **Linting:** `ansible-lint site.yml` — flags anti-patterns at the playbook level (e.g., using `hosts: all` where a specific group was clearly intended, missing tags for selective execution).
- **Dry-run (`--check` + `--diff`):** executes the playbook in check mode against a real (or staging) inventory, reporting what *would* change without actually changing it — this is the playbook-level equivalent of Molecule's idempotence check, and is often the most practical test for playbooks (versus full Molecule scenarios, which are more commonly used at the individual-role level).

## B2. CD Documentation — Automatic Execution with Inventory & Secrets Management

### What & Why
CD for a playbook means it executes against target systems automatically (or via a controlled manual trigger) as part of the deployment pipeline, with the correct inventory and without secrets ever touching the pipeline definition itself.

### Inventory Management
Dynamic inventory sourced from the AWS EC2 plugin (`aws_ec2` inventory plugin) filtering on instance tags (`Environment=prod`, `Role=jenkins-controller`) — this avoids maintaining a static, manually updated inventory file that drifts out of sync with actual running infrastructure (a very common real-world source of "the playbook ran against the wrong/old servers").

### Secrets Management
- **Ansible Vault** for at-rest encrypted variables committed to the repo (vault password itself injected into the CD pipeline via Jenkins Credentials, never committed).
- Alternatively, secrets pulled at runtime via a lookup plugin against **AWS Secrets Manager/SSM Parameter Store**, so the secret never lives in the Ansible codebase at all, encrypted or not — generally the stronger pattern for anything rotated frequently or shared across multiple tools (since the SonarQube DB password, for example, would then have one source of truth rather than being duplicated into an Ansible Vault file *and* wherever else it's needed).

### CD Pipeline Flow
```
Playbook passes CI (syntax + lint + dry-run) → 
Merge to main → 
CD Stage: dynamic inventory lookup for target environment → 
Fetch secrets from Vault/Secrets Manager → 
Execute playbook (ansible-playbook site.yml -i aws_ec2.yml --limit dev) → 
Manual approval → repeat --limit staging → --limit prod
```

## B3. Implementation — POC

### Ansible Role CI POC
This sprint's POC: a working Molecule test suite set up against one representative role (e.g., a simplified version of `sonarqube_setup`), demonstrated end-to-end in Jenkins — lint stage passing, a deliberately introduced non-idempotent task causing the Molecule idempotence check to correctly fail the build, then the fix causing the same pipeline to pass, shown live to the reviewer as the demo.

### Ansible Playbook CI POC
Similarly, a POC playbook composing 2 roles, demonstrating `--syntax-check`, `ansible-lint`, and a `--check --diff` dry-run against a test inventory, showing the diff output for review before any real execution — validating the CI mechanics work before full role/playbook implementation is completed in a later sprint.

### CD POC (Role & Playbook)
A POC deployment run against the Dev environment only (Staging/Prod promotion gates documented but not yet exercised this sprint, consistent with "implementation will be done in the next sprint" scoping already set for the SonarQube/Jenkins Ansible roles in Part 3) — demonstrating dynamic inventory correctly targeting only Dev-tagged instances and Vault-encrypted variables being decrypted and applied without ever appearing in plaintext in Jenkins console output (a common review question: Ansible automatically redacts variables marked via Vault or `no_log: true` from console output specifically to prevent secret leakage into build logs).

---

# Reviewer & Interview Questions — Ansible Epic

**Reviewer questions:**
1. Why is the idempotence check specifically the most important part of Molecule's test lifecycle for infrastructure code?
2. Walk through what happens differently in `--check` mode versus a real run — what can and can't `--check` mode actually validate?
3. Why use dynamic inventory (`aws_ec2` plugin) instead of a static inventory file?
4. Why should Ansible Vault's password itself never be committed alongside the vault-encrypted file it protects?
5. What's the practical difference between testing at the Role level (Molecule) versus the Playbook level (dry-run/lint) — why do we need both?
6. Why does promotion-based deployment (Dev → Staging → Prod with gates) matter for a playbook that's already passed CI?
7. How does Ansible prevent a Vault-encrypted secret from leaking into Jenkins console logs?
8. Why is pinning the `ansible-core` version in CI important, given Ansible has had breaking changes across versions?
9. What's a concrete example of a bug that only shows up at the Playbook composition level, not within any individual Role's own tests?
10. Why is `requirements.yml` for role/collection dependencies treated with the same rigor as `poetry.lock` in Part 4's Python dependency discussion?

**Scenario:**
1. "A role passed Molecule tests in CI but broke a real Prod server on deploy — how is that possible, and how do you prevent it next time?" → Molecule's test container may not fully replicate Prod's actual OS/config specifics (a common gap); recommend aligning the Molecule test image as closely as possible to the real target AMI, and treat the Staging environment's promotion gate as the real-world validation step Molecule can't fully replace.
2. "The CD pipeline ran the playbook against Prod using an inventory that hadn't picked up two newly launched instances — why, and how do you fix it?" → likely a caching/refresh issue with the dynamic inventory plugin or missing tags on the new instances at launch time; fix involves ensuring instance tagging happens atomically at launch (e.g., via the launch template itself) and that the inventory plugin's cache TTL is short enough for the deployment cadence.

---

*End of Part 5 — this closes out the ticket-by-ticket deep dives across all epics. Only Part 6 remains: the consolidated 300-question Mock Interview (Beginner/Intermediate/Advanced/Scenario/Architecture/Production/Debugging/Leadership/Reviewer/Cross-questions) plus the Final Revision set (one-pager, 5-page notes, 20-page notes, and the full cheat sheet). Ready when you are.*
