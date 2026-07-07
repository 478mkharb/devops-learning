# Sprint-2 Interview & Implementation Handbook
## Part 2 of N — Epic: Application CI Design

> Covers: CI concept fundamentals, Mutable vs Immutable infra, Generic CI operations (Cred Scanning, License Scanning, Notification, AMI, Commit Sign-off), and the full CI-check matrix (Code Compilation, Static Analysis, Bugs Analysis, Unit Testing, DAST, Dependency Scanning) across React, Java, Python, and GoLang.

---

# TICKET 1: CI as a Concept

## Introduction

**What is it?**
Continuous Integration (CI) is the practice of merging every developer's code changes into a shared branch frequently (multiple times a day), with each merge automatically built, tested, and validated by an automated pipeline — rather than integrating large batches of work manually at the end of a cycle.

**Why do companies need it?**
Integration is one of the riskiest moments in software delivery — the longer code sits unintegrated, the larger and more painful the eventual merge conflicts and regressions become ("integration hell"). CI collapses that risk into small, frequent, cheap-to-fix increments.

**Key components:**
- **Trigger** (webhook on push/PR)
- **Build** (compile/transpile)
- **Test** (unit, sometimes integration)
- **Static analysis** (code quality/security)
- **Artifact generation** (the deployable output)
- **Feedback** (pass/fail signal back to the developer within minutes)

**Workflow:**
```
Commit → Push → Webhook → Pipeline Triggered → Build → Test →
Static Analysis → Quality Gate → Artifact → Notify Developer
```

**Benefits:** early bug detection, faster feedback loops, reduced integration risk, deployable artifacts at any time ("always releasable main"), and a documented, repeatable process that replaces tribal knowledge.

**Best practices:** keep the pipeline fast (developers stop trusting/waiting for a CI that takes 40+ minutes), fail fast (cheap checks like linting before expensive checks like DAST), and treat pipeline-as-code (Jenkinsfile in the repo, versioned alongside the app).

## Conclusion
CI is the foundation every other ticket in this epic builds on — static analysis, dependency scanning, and DAST are all *stages within* the CI concept, not separate systems.

## Contact Information
DevOps Team Lead — devops-lead@company.com

## References
- Martin Fowler, "Continuous Integration" (martinfowler.com)
- ThoughtWorks Technology Radar

---

# TICKET 2: Mutable vs Immutable Infrastructure

## Introduction

**What:** Mutable infrastructure is patched/updated in place (SSH into a server, run `apt upgrade`, edit a config file). Immutable infrastructure is never modified after creation — any change means building a brand-new instance/image and replacing the old one entirely.

**Why it matters:** mutable infra accumulates "configuration drift" — over months, servers that started identical diverge because different people patched them differently, at different times, leaving unreproducible snapshots of state ("it works on this server but not that one" even though they're supposedly the same).

## Comparison Table

| Factor | Mutable | Immutable |
|---|---|---|
| Change process | In-place edit (SSH, config management run) | Rebuild new image, replace instance |
| Configuration drift | High risk over time | Effectively zero — each instance is byte-identical to its image |
| Rollback | Hard — must manually revert changes | Trivial — redeploy the previous image/AMI |
| Debugging reproducibility | Hard (state differs per server) | Easy (image is versioned and identical everywhere) |
| Startup time | Fast (small config change) | Slower (new instance boot, though pre-baked AMIs mitigate this) |
| Tooling | Ansible/Puppet/Chef applied repeatedly | Packer/AMI baking + Terraform/ASG replacement |
| Best for | Legacy stateful systems, quick hotfixes | Cloud-native, auto-scaled, stateless workloads |

## Reference Diagram — Mutable

```
[Running Server v1] --SSH patch--> [Same Server, now v1.1] --SSH patch--> [Same Server, now v1.2, drifted]
```

## Reference Diagram — Immutable

```
[Base AMI] --bake v1--> [AMI v1] --deploy--> [Instance A, Instance B (identical)]
                │
                └--bake v2 (new change)--> [AMI v2] --replace ASG instances--> [Instance A', B' (all identical, old ones terminated)]
```

## Advantages / Disadvantages

**Immutable advantages:** predictable rollbacks, easier auditing (an AMI ID *is* the audit record of what's running), safer horizontal scaling (new instances are guaranteed identical), and fewer "works on my server" incidents.

**Immutable disadvantages:** slower iteration for tiny config tweaks, requires investment in image-baking pipelines (Packer + CI), larger storage cost for image versions.

## Best Practice / Conclusion
Enterprise standard for cloud-native workloads (this project's EC2/AMI-based deployment) is immutable infrastructure, baked via Packer into Golden AMIs, deployed via Auto Scaling Groups with rolling replacement — mutable patching is reserved only for legitimate exceptions (e.g., a stateful DB host where full replacement is impractical without careful data migration).

---

# TICKET 3: Generic CI Operations

## 3a. Credential Scanning

### Introduction
**What:** scanning source code (and its full Git history) for accidentally committed secrets — API keys, passwords, private keys, tokens.
**Why:** a secret committed even once and later deleted still exists in Git history unless history is rewritten — attackers actively scan public (and sometimes breached private) repos for exactly this pattern.

### Workflow Diagram
```
Commit → Pre-commit hook (fast, local scan) → Push → CI Pipeline stage: full-repo/history scan → 
   Pass → continue pipeline
   Fail → block build, alert security channel
```

### Tools Compared

| Tool | Scope | Strengths | Weaknesses |
|---|---|---|---|
| **Gitleaks** | Repo + history, regex + entropy-based | Fast, open-source, easy CI integration, highly configurable rules | Regex-based → some false positives/negatives |
| **TruffleHog** | Repo + history, entropy + verified-secret checking | Can *verify* if a found secret is still live (calls the provider API) | Slower due to verification calls |
| **GitHub Secret Scanning** (native) | GitHub-hosted repos | Zero setup, partner program auto-revokes some leaked keys | Only works if hosted on GitHub, limited on self-hosted |
| **git-secrets** (AWS) | Regex-based, pre-commit focused | Simple, AWS-pattern aware out of the box | Less actively maintained, narrower detection |

### POC Result / Recommendation
**Chosen tool: Gitleaks** — integrated as both a pre-commit hook (fast local feedback) and a required Jenkins pipeline stage scanning full history on first setup, then incremental diffs per PR thereafter. Rationale: open-source, no licensing cost, fast enough not to slow the pipeline, and has a mature Jenkins/GitHub Actions integration story. TruffleHog's verification feature is valuable but adds pipeline latency deemed unnecessary at this project's scale — noted as a future enhancement, not this sprint's choice.

### Best Practices
Never rely on scanning alone as prevention — pair with pre-commit hooks (shift-left) and secret-management tools (Vault/AWS Secrets Manager) so secrets never need to touch source in the first place.

---

## 3b. License Scanning

### Introduction
**What:** analyzing all direct and transitive dependencies for their software licenses (MIT, Apache-2.0, GPL, AGPL, etc.) to catch legally incompatible or restrictive licenses before they ship in a commercial product.
**Why:** a single GPL/AGPL dependency pulled in transitively can legally obligate a company to open-source proprietary code it's linked against — a business/legal risk, not just a technical one.

### Workflow Diagram
```
Dependency Manifest (package.json/pom.xml/requirements.txt) → 
License Scanner reads resolved dependency tree → 
Cross-reference each license against an allow/deny policy → 
Report + Fail build on denied license
```

### Tools Compared

| Tool | Ecosystem coverage | Strengths | Weaknesses |
|---|---|---|---|
| **FOSSA** | Multi-language, SaaS | Very mature policy engine, legal-team-friendly reports | Paid/commercial |
| **License Finder** | Multi-language, OSS | Free, CLI-based, CI-friendly | Less polished reporting, more manual policy setup |
| **Snyk (License feature)** | Multi-language | Combines with vuln scanning already in use elsewhere | License scanning is a paid-tier add-on |
| **npm-license-checker / pip-licenses / license-maven-plugin** | Single-ecosystem native tools | Free, simple, zero extra infra | No unified cross-language view; must be stitched together per language |

### POC / Recommendation
**Chosen: License Finder** — free, supports all four of this project's ecosystems (npm, Maven, pip/Poetry, Go modules) under one CLI, and integrates cleanly as a Jenkins stage that fails the build on any license outside an allow-list (MIT, Apache-2.0, BSD-family) and flags anything GPL/AGPL for manual legal review rather than an automatic hard fail (since context sometimes matters, e.g., dev-only tooling vs shipped code).

---

## 3c. Notification (Slack/Email from CI)

### Introduction
Pipeline notifications close the loop for build/test/deploy outcomes, distinct from the VCS-level PR/merge notifications covered in the VCS epic — this is CI-stage-specific (build broke, quality gate failed, deployment succeeded).

### Workflow Diagram
```
Jenkins Pipeline Stage completes (success/failure) → 
Post-build block (`post { success {} failure {} }`) → 
Jenkins Slack Plugin / Email-ext Plugin → 
Formats message with build number, branch, failing stage, log link → 
Sends to Slack channel / email distribution list
```

### Internal mechanism
The Jenkins Slack Notification plugin makes an authenticated HTTPS POST to Slack's `chat.postMessage` API (or an incoming webhook URL) using a bot token stored in Jenkins Credentials — never hardcoded in the Jenkinsfile. The `post` block in a Declarative Pipeline is evaluated after all stages regardless of outcome, which is why it's the correct place for notification logic rather than embedding it inside individual stages.

### Best Practices
Notify on failure and on deployment to production always; notify on success only for production deploys (not every dev-branch build, to avoid noise); always link directly to the failing stage's console log, not just "build failed."

---

## 3d. AMI (Amazon Machine Image)

### Introduction
**What:** an AMI is a template capturing an EC2 instance's root volume, launch permissions, and block device mappings — essentially a snapshotted, bootable OS+application image.
**Why:** it's the concrete mechanism that *implements* immutable infrastructure on AWS.

### Workflow Diagram
```
Base OS AMI (e.g., Amazon Linux 2023) → 
Packer template runs provisioning (install runtime, agents, harden OS, bake app or leave app deploy for boot-time) → 
New "Golden AMI" registered → 
Auto Scaling Group launch template references new AMI ID → 
Rolling instance refresh replaces old instances
```

### Security Benefits of AMI / Golden AMI
- **Standardization:** every instance boots from the exact same hardened baseline — no drift between "prod-server-3" and "prod-server-7."
- **Faster patching at scale:** patch once at the image level, then trigger a fleet-wide rolling replace, rather than patching N servers individually (which also means faster, more consistent CVE remediation timelines for compliance).
- **Golden AMI concept:** a pre-approved, security-hardened, pre-scanned (via tools like Amazon Inspector or Trivy against the image) base image that all application AMIs must derive from — enforces a security baseline org-wide (CIS benchmark hardening, mandatory agents like SSM/CloudWatch agent pre-installed, no default credentials).
- **Patching impact:** instead of an in-place patch-and-pray cycle with downtime risk on live instances, patching becomes: bake new Golden AMI monthly (or on critical CVE) → automated pipeline rebuilds all derived app AMIs → rolling deploy — patch compliance becomes measurable and enforceable (e.g., "no instance older than 30 days" as an AWS Config rule).

### Best Practices
Version and tag AMIs with build metadata (git SHA, build number, base AMI version) for traceability; deregister and clean up old AMIs/snapshots on a lifecycle policy to control storage cost; never bake secrets into an AMI — inject at boot via IMDS/Secrets Manager instead.

---

## 3e. Commit Sign-off (DCO / GPG Signing)

### Introduction
**What:** cryptographically or procedurally attesting that a commit's author is who they claim to be and/or that they have the right to contribute the code (Developer Certificate of Origin).
**Why:** protects against impersonation (someone commits code under another engineer's identity) and provides non-repudiation for compliance/audit — critical in regulated industries and for open-source contribution provenance.

### Workflow Diagram
```
Developer generates GPG key → Registers public key with VCS profile → 
git commit -S (signs commit with private key) → Push → 
VCS verifies signature against registered public key → 
Displays "Verified" badge; branch protection can require "signed commits only"
```

### Advantages
Strong identity assurance beyond just "whoever had the VCS account's password," protection against commit spoofing (`git commit --author="fake name"` doesn't require any credential and is trivially fakeable without signing), audit-grade compliance evidence.

### Best Practices
Enforce via branch protection ("Require signed commits") on protected branches, distribute GPG key setup via onboarding automation (not manual tribal knowledge), integrate key-expiry monitoring so stale keys don't silently fail verification.

---

# TICKET 4–7: CI Check Matrix (React / Java / Python / GoLang)

Rather than repeating identical structure four times over, here is the matrix once, followed by ecosystem-specific tool detail and the recommendation for each.

## Universal Workflow (applies to every language)

```
Checkout → Install/Resolve Dependencies → Compile/Transpile → 
Static Analysis (lint + SonarQube) → Bug Analysis (SAST-adjacent) → 
Unit Tests + Coverage → Dependency Scan (SCA) → 
[Deploy to test env] → DAST → Quality Gate → Artifact
```

### Code Compilation

| Language | Tool | Why chosen |
|---|---|---|
| React (JS/TS) | `npm run build` (Webpack/Vite under Create-React-App or custom config), `tsc` for type-checking | Standard toolchain, zero extra licensing, TypeScript catches type errors at "compile" time even though JS is interpreted |
| Java | Maven (`mvn compile`) | Industry-standard build tool with mature dependency management; chosen over Gradle for this project due to team familiarity and simpler declarative XML config vs Gradle's Groovy/Kotlin DSL learning curve |
| Python | N/A (interpreted) — "compilation" check = `python -m py_compile` / import validation | Python has no true compile step; this is really a syntax-validity gate |
| Go | `go build` | Go's compiler is fast, statically typed, produces a single static binary — compilation itself doubles as a strong correctness check given Go's strict typing |

### Static Code Analysis

| Language | Tools compared | Chosen |
|---|---|---|
| React | ESLint vs Prettier (formatting only, not analysis) vs SonarQube JS plugin | **ESLint + SonarQube** — ESLint for fast local/CI linting, SonarQube for deeper cross-file quality metrics and the org-wide Quality Gate |
| Java | Checkstyle vs PMD vs SonarQube | **SonarQube** (with Checkstyle rules imported) — single pane of glass across all languages, which is why SonarQube also appears in every other language row here |
| Python | Flake8 vs Pylint vs SonarQube | **Flake8** for fast style/error checking in CI (Pylint is more thorough but noticeably slower and more opinionated/noisy by default) + SonarQube for org-wide gate |
| Go | `go vet` vs `golangci-lint` vs SonarQube | **golangci-lint** (aggregates `go vet`, `staticcheck`, `errcheck`, and more into one fast parallelized runner) + SonarQube |

**Flake8 vs Pylint (called out specifically since it's a required comparison):**

| Factor | Flake8 | Pylint |
|---|---|---|
| Speed | Fast | Slower (deeper analysis) |
| Depth of analysis | Style + basic errors (wraps pyflakes + pycodestyle + mccabe complexity) | Much deeper — type inference, design checks, more categories |
| Noise/false positives | Low, minimal config needed | Higher out-of-the-box noise, needs tuning |
| CI fit | Excellent for fast fail-fast stage | Better as a periodic deep-analysis job, not blocking every PR |

Recommendation: Flake8 as the blocking CI gate (fast, low-noise), Pylint run non-blocking/informational for deeper design feedback.

### Bug Analysis
This is SonarQube's "Bugs" category specifically (distinct from Code Smells/Vulnerabilities) — SonarQube's rule engine flags patterns statistically correlated with runtime defects (null-pointer risks, resource leaks, off-by-one patterns) via AST-level pattern matching per language plugin. All four ecosystems funnel into the same SonarQube instance for this, which is precisely why SonarQube gets its own full section later in the series (Part 3).

### Unit Testing

| Language | Tool | Why |
|---|---|---|
| React | Jest (+ React Testing Library) | De facto standard for React, built-in mocking/snapshot testing, fast |
| Java | JUnit 5 + Maven Surefire plugin | Industry standard, deeply integrated with Maven lifecycle |
| Python | Pytest + `coverage` | More expressive fixtures/parametrization than unittest, huge plugin ecosystem |
| Go | Built-in `go test` | Testing is a first-class part of the Go toolchain — no third-party framework needed for basics |

### Dependency Scanning (SCA)

| Language | Tool | Why |
|---|---|---|
| React | `npm audit` (baseline, free) vs Snyk (deeper DB, remediation guidance) | `npm audit` for the free baseline gate; Snyk considered for future upgrade if budget allows |
| Java | OWASP Dependency-Check (Maven plugin) | Free, cross-references NVD CVE database against Maven dependency tree |
| Python | `pip-audit` | Official PyPA tool, checks against the Python Packaging Advisory Database (PyPI's own vuln feed) |
| Go | `govulncheck` | Official Go team tool, uses Go's own vulnerability database, understands actual call-graph reachability (fewer false positives than "package X has *a* CVE" — it checks if your code actually calls the vulnerable function) |

### DAST (Dynamic Application Security Testing)
Same tool across all four languages since DAST attacks the **running application** over HTTP, not the source code — language is irrelevant to the attack surface.
**Chosen: OWASP ZAP** in daemon/automation mode, run against a deployed test environment as a pipeline stage after deployment to a test/staging target. (Full ZAP internals covered in Part 3.)

---

## Recommendation Summary (stated explicitly per requirement)

- **Cred scanning:** Gitleaks
- **License scanning:** License Finder
- **Static analysis (org-wide):** SonarQube, backed by ESLint/Flake8/golangci-lint per language for fast local feedback
- **Dependency scanning:** `npm audit` (React), OWASP Dependency-Check (Java), `pip-audit` (Python), `govulncheck` (Go)
- **DAST:** OWASP ZAP for all
- **CI orchestrator:** deferred to the dedicated CI Orchestration Tools ticket (Jenkins vs GitLab vs Buildpiper) — covered in Part 3

---

# Reviewer & Interview Questions — Application CI Design Epic

**Reviewer questions:**
1. Why Gitleaks over TruffleHog for credential scanning, given TruffleHog can verify live secrets?
2. Walk through exactly how a Golden AMI improves patch compliance versus patching live servers.
3. Why is DAST language-agnostic while SAST/static analysis isn't?
4. Why Flake8 as the blocking gate but Pylint only informational?
5. What's the actual difference between "Bug Analysis" and "Static Code Analysis" in SonarQube's model?
6. Why govulncheck over a generic CVE database checker for Go specifically?
7. What legal risk does license scanning actually protect against — walk through a concrete GPL scenario.
8. Why is commit signing valuable if the VCS already requires SSO login to push?
9. Immutable infra sounds slower to iterate — how do you address that critique from a developer who wants a quick config tweak?
10. Why run `npm audit` instead of jumping straight to Snyk?

**Cross-questions / scenario:**
1. "A dependency scan flags a transitive dependency's CVE with no available fix — what do you do?" → assess actual exploitability/reachability, consider `govulncheck`-style call-graph reasoning even outside Go, document risk acceptance with expiry date, monitor for a patched release.
2. "SonarQube's Bug Analysis and your team's manual bug analysis disagree — who wins?" → discuss static analysis's inherent false-positive rate, human review as final arbiter, tuning rule severity over time rather than blindly trusting or ignoring the tool.

---

*End of Part 2. Part 3 will cover: CI Orchestration Tools (Jenkins vs GitLab vs Buildpiper + recommendation), the full SonarQube deep dive (authn/authz, infra, HA, DR, monitoring, logs, quality gates), and the full Jenkins deep dive (same structure). Say the word and I'll continue.*
