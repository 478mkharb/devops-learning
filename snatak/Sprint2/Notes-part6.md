# Sprint-2 Interview & Implementation Handbook
## Part 6 of 6 — Consolidated Mock Interview & Final Revision Set

> This closes the series. Structured as: a full mock interview across categories (Beginner → Leadership), then the layered revision materials (1-page, 5-page, 20-page, cheat sheet).

---

# PART I: MOCK INTERVIEW

## Section 1 — Beginner (VCS & Git Fundamentals)

1. What's the difference between Git and GitHub? — *Git is the distributed VCS tool; GitHub is a hosting platform built on top of Git adding collaboration/UI/CI.*
2. What is a commit? — *An immutable snapshot of the repo's tracked files plus metadata (author, message, parent commit SHA), hashed into a unique ID.*
3. What's the difference between `git fetch` and `git pull`? — *Fetch downloads remote changes without merging; pull fetches then merges/rebases automatically.*
4. What is a branch, internally? — *A lightweight, movable pointer to a commit — not a copy of the codebase.*
5. Why do we protect the `main` branch? — *Prevents unreviewed/unbuilt code reaching the branch everything else and production deploys are based on.*
6. What's a PR/MR? — *A request to merge one branch into another, gated by review and automated checks before merge is allowed.*
7. What does `.gitignore` do? — *Tells Git which untracked files/patterns to never stage, keeping build artifacts/secrets out of version control.*
8. What is a webhook? — *An HTTP callback the VCS platform fires automatically on specified events (push, PR opened) to a configured URL.*
9. What's SSO, in plain terms? — *Logging in once at a central identity provider, which then federates that identity to every connected tool.*
10. What is CI in one sentence? — *Automatically building and testing every code change as soon as it's pushed, to catch problems early.*
11. What's a Jenkinsfile? — *The pipeline definition, written as code and stored in the repo, describing every CI/CD stage.*
12. What's the difference between a Jenkins Declarative and Scripted pipeline? — *Declarative is structured/opinionated YAML-like syntax with built-in validation; Scripted is full Groovy for maximum flexibility.*
13. What's an artifact in a CI context? — *The build output (jar, docker image, zip) produced by a pipeline and stored for deployment/reference.*
14. What is static code analysis? — *Examining source code without executing it, to flag bugs, vulnerabilities, and quality issues.*
15. What's the difference between SAST and DAST? — *SAST analyzes source code at rest; DAST attacks the running, deployed application.*
16. What is a Quality Gate? — *A pass/fail threshold on code metrics (coverage, bugs, vulnerabilities) that blocks a build if not met.*
17. What's an AMI? — *A template capturing an EC2 instance's OS, config, and installed software, used to launch new identical instances.*
18. What's the difference between mutable and immutable infrastructure? — *Mutable is patched in place; immutable is replaced entirely with a new image on any change.*
19. What does `npm ci` do differently from `npm install`? — *`ci` installs exactly what's in the lockfile and fails if it's out of sync with `package.json`, rather than resolving/updating like `install` can.*
20. What is a pre-commit hook? — *A local script Git runs before finalizing a commit, used here to enforce JIRA-ID commit message format.*

## Section 2 — Intermediate

21. Why does `waitForQualityGate` in Jenkins not block synchronously on a simple HTTP call? — *SonarQube analysis runs asynchronously on the server (Compute Engine); Jenkins polls a task ID until the async processing completes, then checks the gate result.*
22. Why is `vm.max_map_count` tuning required for SonarQube specifically? — *Its embedded Elasticsearch needs a higher OS-level memory-mapped area limit than the Linux default, or it fails to start.*
23. Explain the difference between `git merge` and `git rebase` and when each is appropriate. — *Merge preserves true history with a merge commit, safe for shared branches; rebase rewrites commit history for a linear log, unsafe on branches others have based work on.*
24. Why must `poetry.lock` be committed to the repo? — *It pins exact resolved versions and hashes, guaranteeing reproducible installs across every machine and CI run.*
25. What's the actual mechanism that enforces "Lead-only merge" beyond passing checks? — *A separate branch protection restriction ("restrict who can merge") layered on top of the required-checks gate — passing checks alone doesn't enable the merge button for non-Leads.*
26. How does OWASP ZAP discover authenticated pages? — *It requires an explicitly configured auth context/login sequence script; without it, ZAP only crawls the public, unauthenticated surface.*
27. Why is Flake8 the blocking gate while Pylint is informational-only? — *Flake8 is fast and low-noise, suited to a blocking fail-fast stage; Pylint is deeper but slower/noisier, better suited to non-blocking design feedback.*
28. What's the difference between a Golden AMI and a regular AMI? — *A Golden AMI is a pre-approved, security-hardened baseline all other application AMIs must derive from, enforcing a consistent org-wide security standard.*
29. Why is Molecule's idempotence check the most important test for an Ansible role? — *A role that reports "changed" on a second identical run indicates unstable, non-declarative logic that will cause unwanted drift/restarts in real environments.*
30. What's the difference between SCP and IAM policy? — *SCPs are organization-level guardrails that set the maximum permissions even account admins can't exceed via IAM; IAM policies grant permissions within whatever the SCP allows.*
31. Why run `npm audit` before considering a paid tool like Snyk? — *Establishes a free baseline dependency-vulnerability gate immediately; Snyk is evaluated as a future upgrade once budget/deeper remediation guidance is justified.*
32. What does `archiveArtifacts` actually do inside a Jenkins pipeline? — *Copies specified files from the agent's workspace into the Jenkins controller's build record storage, making them downloadable/referenceable after the workspace is wiped.*
33. Why does `pip-audit` check against the PyPA advisory DB specifically? — *It's the official, Python-ecosystem-specific vulnerability feed maintained by the Python Packaging Authority, more precisely scoped than a generic cross-language CVE database.*
34. What's the difference between a Security Group and a NACL? — *Security Groups are stateful and instance-level (return traffic auto-allowed); NACLs are stateless and subnet-level (return traffic must be explicitly allowed too).*
35. Why is a dynamic inventory (`aws_ec2` plugin) preferred over a static inventory file for Ansible CD? — *Prevents drift between the inventory and actual running infrastructure — new/terminated instances are picked up automatically via tags instead of manual file edits.*

## Section 3 — Advanced / Architecture

36. Design the HA strategy for SonarQube Community Edition (no clustering license) and justify it. — *Active-passive via ASG self-healing for the app tier + RDS Multi-AZ for the DB tier; true active-active requires Data Center Edition licensing, explicitly flagged as a cost decision, not silently accepted as a limitation.*
37. Why is Jenkins controller HA architecturally harder than agent HA? — *The controller holds critical persistent state (JENKINS_HOME: job configs, build history, encrypted credentials store) making it inherently stateful, unlike disposable/ephemeral agents.*
38. Explain the full internal lifecycle of a Jenkins pipeline triggered by a GitHub push, from webhook to artifact. — *(See Part 1/Part 3 architecture diagrams — webhook signature validation → job match → agent allocation/workspace creation → Jenkinsfile parse → stage execution → SonarQube scan submission → async Quality Gate poll → archiveArtifacts → post-build notification.)*
39. Why did we choose multi-account (AWS Organizations) over a single account with IAM-only environment separation? — *Hard security/billing isolation — a Dev misconfiguration cannot reach Prod resources at all, versus relying entirely on IAM policy correctness within one account.*
40. Justify Jenkins over GitLab CI/CD for this specific project. — *GitHub is already the VCS-of-record (decoupling CI from GitLab's native advantage); a genuinely heterogeneous toolchain (SonarQube, ZAP, Ansible, 4 languages) benefits from Jenkins' plugin flexibility over GitLab's more opinionated built-in model.*
41. Design the DR approach for both Jenkins and SonarQube and justify the chosen RTO/RPO tier. — *Pilot-light for both: cross-region DB replica/snapshot kept warm/available, app tier rebuildable via Ansible + Golden AMI on demand — balances cost against acceptable recovery time for tools that gate CI but aren't customer-facing production systems.*
42. Why does the credentials store living inside `JENKINS_HOME` complicate both HA and DR simultaneously? — *It's the single hardest piece of state to safely replicate/restore — any HA/DR design must account for keeping the encrypted credential data consistent and available without exposing it insecurely in transit/backup.*
43. Explain why License Finder was chosen over Snyk/FOSSA for license scanning specifically, distinct from the vulnerability-scanning tool choices. — *Free, single-CLI coverage across all four project ecosystems, sufficient for the allow/deny policy this project needs; FOSSA/Snyk's added legal-workflow polish isn't yet justified against their licensing cost.*
44. How would you evolve the current single-team Jenkins setup if the org grew 10x? — *Consider Buildpiper-style self-service abstraction, sharded/federated Jenkins controllers per business unit, and mono-repo tooling investment (Bazel/Nx) reconsideration if repo count/coupling grows unmanageable under micro-repo.*
45. Explain how `govulncheck`'s reachability analysis reduces false positives compared to a naive CVE-matching dependency scanner. — *It checks whether the vulnerable function in a flagged dependency is actually reachable from the project's own call graph, rather than flagging every CVE for every version match regardless of whether the vulnerable code path is ever invoked.*

## Section 4 — Scenario / Production / Debugging

46. Jenkins controller disk hit 100% at 2am — walk through your response end to end. — *(See Part 3 scenario answer: monitoring alert → identify largest consumers under JENKINS_HOME/jobs → apply retention/log-rotation → restore → postmortem.)*
47. SonarQube's Elasticsearch status is red — what's your triage order? — *`es.log` first → `vm.max_map_count` (most common cause) → disk space on data volume → JVM heap allocated to embedded ES.*
48. A ZAP active scan against an authenticated app reports zero findings — is that reassuring? — *No — almost certainly the auth/login sequence isn't configured, so ZAP only crawled the public surface; always cross-check discovered-URL count alongside finding count.*
49. Dev environment AWS spend spiked 3x overnight with no deployment — investigate. — *Cost Anomaly Detection alert as trigger → Cost Explorer grouped by tag to isolate team/project → CloudTrail to find the resource and actor → check for an SCP gap that allowed an untagged/oversized resource.*
50. A secret was committed and pushed to `main` — what's the actual first step, and why isn't it "rewrite history"? — *Rotate/revoke the secret immediately — it may already be cached/fetched elsewhere; history rewriting is a secondary cleanup step, not the mitigation for the exposure itself.*
51. A role passed all Molecule tests in CI but broke a real Prod server — how is that possible? — *Molecule's test container likely doesn't fully replicate Prod's actual OS/AMI specifics; recommend aligning the test image closer to the real target and treating Staging's promotion gate as the real validation Molecule can't fully replace.*
52. Two reviewers approved a PR, but a third person's later comment flagged a real issue that got missed at merge — what process gap does this reveal? — *Comments alone don't block merge unless tied to a formal "changes requested" review state, or approvals aren't dismissed on new pushes/comments — discuss "dismiss stale approvals" and whether comment-only feedback should gate merge.*
53. The Jenkins-to-VCS webhook silently stopped firing for two days — how do you detect and debug it? — *Check the VCS platform's webhook delivery log (most platforms show delivery attempts/response codes) → verify Jenkins endpoint reachability/firewall/cert validity → check for payload signature validation failures being silently dropped.*
54. A dependency scan flags a transitive dependency's CVE with no available patched version — what do you actually do? — *Assess real exploitability/reachability (is the vulnerable code path ever called), document a time-bound risk acceptance, monitor for a fix, don't just ignore or block indefinitely without a decision trail.*
55. The CD pipeline ran an Ansible playbook against Prod but missed two newly launched instances — why, and how do you prevent recurrence? — *Likely dynamic inventory caching/refresh lag or instances not tagged atomically at launch; fix by tagging at launch-template level and shortening inventory cache TTL relative to deployment cadence.*

## Section 5 — Leadership / Cross-Questions

56. How would you convince a skeptical team lead that immutable infrastructure is worth the slower iteration on small config tweaks? — *Frame around MTTR and reproducibility: a rollback is "redeploy a known-good image" versus manually reverse-engineering what changed on a live server; the slower iteration cost is paid once per change, the debugging cost of drift is paid repeatedly and unpredictably.*
57. A developer wants to bypass the pre-commit hook regularly using `--no-verify` — how do you handle it as a process owner, not just a tool owner? — *Acknowledge the local hook is convenience/fast-feedback only; the actual enforcement boundary is the server-side/CI check — reiterate that boundary rather than trying to technically prevent `--no-verify`, and address the underlying friction causing them to skip it.*
58. How do you decide when a "custom Quality Gate" is justified versus just tuning the default? — *Tie the decision to risk profile of the specific service (e.g., stricter gate for auth-handling services) rather than applying one gate uniformly or one-off per developer preference — document the rationale so it's an intentional policy, not ad-hoc.*
59. If budget were cut in half tomorrow, which of today's HA/DR choices would you revisit first, and why? — *Likely candidates: warm-standby-adjacent choices already flagged as "future upgrade" (Data Center Edition SonarQube HA, CloudBees Jenkins HA) were never funded in the first place; the pilot-light DR approach already reflects a cost-conscious baseline, so the next cut would more likely come from environment sizing (Dev/Staging instance classes) than from these guardrails.*
60. How do you keep four different language ecosystems' CI checks consistent without forcing an identical tool per language? — *Standardize on the outcome (a Quality Gate result in SonarQube) rather than the specific linter/tool per language — each ecosystem's best-fit tool (ESLint, Flake8, golangci-lint, Checkstyle) feeds the same central gate, giving consistency at the decision layer while respecting each ecosystem's native tooling.*

---

# PART II: FINAL REVISION SET

## One-Page Revision Sheet

- **VCS:** GitHub (SaaS) chosen; micro-repo; SSO/SAML for authn, group-mapped teams for authz; PR requires 2 reviewer + Jenkins sign-off + Lead-only merge; pre-commit hook enforces JIRA-ID commit format.
- **CI concept:** frequent small integrations + automated build/test/scan, fail fast, pipeline-as-code.
- **Immutable infra:** AMI-based, Packer-baked Golden AMIs, ASG rolling replace; mutable only for legacy exceptions.
- **Generic CI ops:** Gitleaks (cred scan), License Finder (license scan), Slack/email via Jenkins post-block, Golden AMI standardization, GPG commit signing.
- **Language CI matrix:** compile → lint/SonarQube → bug analysis → unit test+coverage → dependency scan (npm audit/Dependency-Check/pip-audit/govulncheck) → ZAP DAST.
- **CI orchestrator:** Jenkins (over GitLab CI/Buildpiper) — flexibility + heterogeneous toolchain fit.
- **SonarQube & Jenkins:** SSO authn + RBAC authz; AWS: ALB → app EC2 → RDS Multi-AZ; Ansible role documented (JCasC for Jenkins), POC manual this sprint; HA = active-passive ASG + Multi-AZ RDS; DR = pilot-light; monitored via Prometheus/CloudWatch with defined thresholds; distinct log files per subsystem.
- **ZAP:** Spider → Passive (safe anywhere) → Active (staging only) → REST API automation in Jenkins.
- **Python CI:** Poetry (lockfile+hash resolution) → pytest/coverage → Flake8 blocking/Pylint informational → pip-audit; Gunicorn for prod WSGI; Liquibase for DB migrations.
- **Cloud Infra:** multi-account AWS Organizations (Tooling/Dev/Staging/Prod); Dev VPC with public/private/DB subnet tiers, tight SG boundaries.
- **Cost:** mandatory tags enforced via Config/SCP, right-sizing + Spot for Jenkins agents, Budgets + Cost Anomaly Detection, Cost Explorer grouped by tag.
- **Ansible:** Role CI via Molecule (create/converge/idempotence/verify/destroy); Playbook CI via syntax-check/lint/`--check` dry-run; CD via dynamic inventory + Vault/Secrets Manager, promotion-gated Dev→Staging→Prod.

## Five-Page Quick Notes
*(Condensed section headers — expand any one on request for full depth):*
1. VCS Implementation (platform choice, repo strategy, authn/authz, PR workflow, notifications, hooks)
2. Application CI Design (CI concept, mutable/immutable, cred/license scanning, AMI, sign-off, 4-language check matrix)
3. CI Orchestration + SonarQube + Jenkins (tool choice, authn/authz, infra, Ansible, HA/DR, monitoring, logs, quality gates)
4. ZAP + Python CI + Cloud Infra + Cost (DAST internals, Python toolchain, AWS architecture, cost governance)
5. Ansible Role/Playbook CI/CD (Molecule, dry-run testing, inventory/secrets, promotion gates)

## Twenty-Page Revision Notes
This maps directly to **Parts 1–5** of this handbook series in full — each part is roughly a 4-page equivalent at full depth. Use Parts 1–5 as the twenty-page revision set directly; this Part 6 is the condensed layer on top of them.

## Cheat Sheet — Things Reviewers Ask Most
- "Why this tool over the alternative?" (asked for nearly every tool choice — always have the *rejected* alternatives and the specific reason ready, not just the winner).
- "Walk me through what happens internally" (never stop at a definition — always be ready to go one level deeper).
- "How is this enforced, not just documented?" (the recurring theme across PR workflow, quality gates, SCPs — mechanism over policy).
- "What's your HA/DR story?" — active-passive/pilot-light pattern repeats across Jenkins and SonarQube; know it once, apply it twice.

## Cheat Sheet — Things Interviewers Ask Most
- Scenario/debugging questions almost always want **triage order**, not just the eventual root cause — state the sequence of checks, not just the answer.
- Comparison questions (SAST vs DAST, mutable vs immutable, mono vs micro-repo) — always close with a recommendation, not just a neutral table.
- "How would this change at 10x scale?" — a very common way to test whether a decision was reasoned or just copied from a template.

## Common Mistakes to Avoid in the Actual Interview/Review
- Describing a tool without saying *why it was chosen over its alternatives*.
- Explaining a security control (SCP, branch protection, Quality Gate) as a "best practice" without explaining the concrete mechanism that actually enforces it.
- Treating HA and DR as the same thing — always distinguish single-node/AZ failure (HA) from full-region/catastrophic failure (DR).

## Golden Tips
- Anchor every answer back to *this specific project's* decisions (GitHub, Jenkins, micro-repo, multi-account AWS) rather than generic textbook answers — reviewers are checking you understand *why we* chose this, not just DevOps theory in the abstract.
- When asked "why," have a second-level "why" ready — almost every ticket in this handbook was built with that exact follow-up in mind.

---

*This completes the full 6-part Sprint-2 Interview & Implementation Handbook. Parts 1–5 hold the full ticket-by-ticket depth; this Part 6 is your fast-recall layer before a review or interview. Good luck with the demo.*
