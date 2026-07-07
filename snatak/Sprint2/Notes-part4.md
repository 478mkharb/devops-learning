# Sprint-2 Interview & Implementation Handbook
## Part 4 of N — OWASP ZAP Deep Dive, Python CI Specifics, Cloud Infra Design, Cost Optimization

---

# SECTION A: OWASP ZAP Deep Dive

## Introduction
**What:** OWASP ZAP (Zed Attack Proxy) is an open-source DAST (Dynamic Application Security Testing) tool that attacks a **running** application over HTTP to find real, exploitable vulnerabilities (XSS, SQL injection, security misconfigurations) — as opposed to SAST, which reads source code without executing it.
**Why:** SAST can miss vulnerabilities that only manifest at runtime (misconfigured headers, auth bypass through actual request manipulation, session handling flaws) — DAST validates the deployed, working system the way a real attacker would.

## Core Components

### Spider
Crawls the target application by following links/forms, building a full site map of discoverable endpoints before any attack begins — you can't attack what you don't know exists. ZAP also has an **AJAX Spider** (uses a real browser engine) for JS-heavy SPAs like the React frontend, since a plain HTTP spider can't execute JavaScript to discover client-rendered routes.

### Passive Scan
Analyzes traffic (requests/responses) that's already flowing through the proxy **without sending any additional attack traffic** — flags issues like missing security headers, cookies without `Secure`/`HttpOnly` flags, information disclosure in responses. Runs automatically and safely against any environment, including production, since it never sends malicious payloads.

### Active Scan
Actively injects attack payloads (SQLi strings, XSS payloads, path traversal attempts) into every discovered parameter and observes the response for signs of a vulnerability. **Never run against production** — this is why DAST always targets a dedicated staging/test environment in the pipeline, never the live app.

## How ZAP Attacks a Running Application (Internal Mechanism)
1. ZAP starts as a **local proxy** (or in headless/daemon mode for CI) sitting between the "browser"/spider and the target application.
2. The Spider issues real HTTP requests to the target, parsing HTML/JS responses to discover new links/forms, building an internal site tree.
3. Passive Scan rules run against every request/response pair as they pass through the proxy — implemented as rule plugins checking headers, cookies, and response bodies against known-bad patterns.
4. Active Scan iterates through every discovered parameter (query string, form field, header) and, for each registered attack rule (SQLi, XSS, etc.), substitutes a crafted payload, sends the request, and inspects the response for signatures indicating success (e.g., a SQL error message leaking, a reflected script tag surviving unescaped, or a timing difference indicating a blind injection).
5. Findings are classified by risk (High/Medium/Low/Informational) and confidence, and exposed via ZAP's REST API for the automation layer (Jenkins) to consume.

## Daemon Mode / Automation / REST API
For CI integration, ZAP runs headless via `zap.sh -daemon -host 0.0.0.0 -port 8080 -config api.key=<key>`. Jenkins then drives it entirely through ZAP's **REST API** (not the GUI): triggering `/JSON/spider/action/scan/`, polling `/JSON/spider/view/status/` until 100%, triggering `/JSON/ascan/action/scan/`, polling similarly, then pulling `/OTHER/core/other/htmlreport/` for the final report. This API-driven model is what makes ZAP genuinely CI-native rather than a manual pentester's desktop tool only.

## HTML Reports
ZAP generates a self-contained HTML report (also available as JSON/XML/Markdown) listing every finding with risk level, affected URL/parameter, a description of the vulnerability class, and remediation guidance — archived as a Jenkins build artifact per run so trend history is preserved across builds.

## False Positives & Limitations
DAST tools inherently generate some false positives (e.g., flagging a header as missing when a CDN/WAF actually adds it at the edge, not at the app itself) — findings need a triage step, typically via ZAP's "alert filters" to permanently suppress confirmed false positives per rule+URL combination rather than re-triaging the same finding every single build. **Limitation:** ZAP can only find what it can reach/discover — it won't test business logic flaws requiring domain understanding, and authenticated areas of the app need explicit auth-context configuration (login sequence scripting) or ZAP will only ever test the unauthenticated surface.

## Pipeline Integration
```
Deploy to Staging (post-build stage) → 
Jenkins stage: start ZAP daemon → 
Configure auth context (if testing authenticated routes) → 
Run Spider → Run Active Scan → 
Generate report → archiveArtifacts → 
Fail build if High-risk findings exceed threshold (configurable, since 0-tolerance on Medium/Low would be unrealistically strict for early adoption)
```

## Troubleshooting
- Scan times out/never completes → usually the Spider getting stuck on an infinite pagination loop or logout link — configure scan scope exclusions.
- Zero findings reported → check ZAP is actually proxying traffic to the *test* environment URL, not accidentally hitting a health-check endpoint with no real app logic.
- Authenticated scan finds almost nothing → login sequence likely isn't configured, so ZAP is only crawling the public/unauthenticated pages.

## ZAP vs Burp Suite

| Factor | OWASP ZAP | Burp Suite (Pro) |
|---|---|---|
| Cost | Free, open-source | Paid license per user |
| CI/Automation | Native REST API, daemon mode, built for pipeline integration | CLI/automation exists (Burp Suite Enterprise) but the flagship product is manual-pentester-focused |
| Extensibility | Plugin marketplace (Zest scripting, add-ons) | Extensive BApp Store, generally considered more polished for manual testing workflows |
| Best fit | CI/CD-embedded automated DAST at no license cost | Deep manual penetration testing engagements |

**Recommendation:** OWASP ZAP for CI-embedded automated DAST (this project's use case) — Burp Suite Pro remains valuable for periodic manual penetration testing engagements by a security team, but that's a complementary activity, not a CI pipeline stage.

---

# SECTION B: Python CI Deep Dive (Attendance & Notification Services)

## Introduction
This section covers the Python-specific toolchain in depth, since both the Attendance API and the Notification service are Python-based and share the same CI approach, differing mainly in runtime dependencies (Gunicorn as the WSGI server, Liquibase for DB migrations where applicable).

## Poetry (Dependency Management)

### What/Why
Poetry manages dependencies, virtual environments, and packaging through a single `pyproject.toml` + `poetry.lock`, replacing the older `requirements.txt` + manual `venv` workflow with fully reproducible, hash-locked dependency resolution.

### How Poetry Resolves Dependencies (Internal Mechanism)
1. Reads `pyproject.toml`'s declared dependency constraints (e.g., `flask = "^2.0"`).
2. Builds a full dependency graph including transitive dependencies, using a SAT-like backtracking resolver to find a version set satisfying every package's constraints simultaneously (this is genuinely more rigorous than pip's older resolver, which historically could install conflicting versions silently).
3. Writes the exact resolved versions **and their file hashes** into `poetry.lock` — this lockfile is what guarantees byte-for-byte identical installs across every developer machine and CI run, not just "a version that satisfies the range."
4. `poetry install` reads the lockfile (not `pyproject.toml` directly, once a lock exists) and installs exactly those pinned, hash-verified versions.

### Commands
- `poetry install` — installs from lockfile; in CI, always run with `--no-root` (skip installing the project package itself if only testing) or `--only main` to exclude dev-only dependency groups from a production build.
- `poetry update` — re-resolves and updates the lockfile against current constraint ranges; **never run this in CI** — it should only ever happen deliberately in a dev branch/PR, since running it in CI would make builds non-reproducible (a "passing" build today could silently install different versions tomorrow).
- `poetry run <cmd>` — executes a command inside the resolved virtual environment without needing to manually `source venv/bin/activate`.

## venv
Python's built-in lightweight virtual environment tool — Poetry uses a venv internally under the hood by default, isolating the project's dependencies from system Python. Interview-relevant distinction: **venv vs Docker** — venv isolates Python packages only (still shares the OS, system libraries, Python interpreter version installed on the host), while Docker isolates the entire OS/runtime — venv is sufficient for pure dependency isolation in CI, but Docker is used when the deployment target itself needs full environment reproducibility (system-level libs, exact OS version).

## pip-audit
### What/Why
Official PyPA tool cross-referencing installed package versions against the **Python Packaging Advisory Database (PyPI's own vulnerability feed)** and OSV, identifying known CVEs in dependencies.
### How it identifies CVEs internally
Reads the resolved environment (or `poetry.lock`/`requirements.txt` directly via `--requirement`), extracts package name + exact version for every dependency, queries the vulnerability database (either online or an offline cached copy) for matching advisories by package+version range, and reports any match with CVE ID, severity, and (if available) the fixed version to upgrade to.

## pytest
### How pytest Discovers Tests (Internal Mechanism)
Pytest doesn't require explicit test registration — at collection time it recursively walks the specified directory (or repo root by default), identifying any file matching `test_*.py` or `*_test.py`, then within those files any function prefixed `test_` or class prefixed `Test` (with methods prefixed `test_`), building an internal collection tree before executing any of them — this convention-over-configuration discovery is why pytest requires near-zero boilerplate compared to `unittest`'s explicit `TestLoader` registration.

### coverage / coverage xml
`coverage run -m pytest` wraps test execution with Python's trace-hook mechanism, recording which lines actually executed during the test run; `coverage xml` exports this into Cobertura-format XML, which is the format SonarQube's scanner expects to import for the "coverage on new code" Quality Gate metric — this is the concrete mechanical link between "we ran tests" and "SonarQube's coverage number."

## flake8 / pylint
Already compared in Part 2 — reiterated here in the Python-specific pipeline context: Flake8 runs first (fast, blocking), Pylint runs as a non-blocking informational stage feeding into code review culture rather than gating merges.

## Gunicorn
### What/Why
Gunicorn is a production-grade WSGI HTTP server used to actually run the Python web application (Flask/Django) in a deployed environment — Python's built-in development server (`app.run()`) is single-threaded and explicitly not designed for production traffic. Gunicorn spawns multiple **worker processes** (pre-fork model) so concurrent requests are handled in parallel across OS processes, sidestepping Python's Global Interpreter Lock (GIL) limitation on true multi-threading within a single process.
### CI relevance
CI doesn't typically "test" Gunicorn directly, but a smoke-test stage after build (starting the app under Gunicorn briefly and hitting a health-check endpoint) validates that the WSGI entrypoint is correctly wired before deployment — catching a whole class of "works with the dev server, breaks under Gunicorn" configuration bugs.

## Liquibase
### What/Why
Liquibase manages database schema changes as version-controlled, ordered "changesets," tracked in a `DATABASECHANGELOG` table in the target DB itself — so the CI/CD pipeline can safely, repeatably, and idempotently apply only the migrations a given environment hasn't yet received, instead of manually running ad-hoc SQL scripts.
### Pipeline relevance
A dedicated CD-stage (not CI) step runs `liquibase update` against the target environment's DB as part of deployment — critically, this is validated in CI first via `liquibase validate` and a dry-run (`updateSQL`) against a disposable test DB, catching a broken migration before it ever reaches a real environment.

## Attendance API — CI Considerations Specific to This Service
Given it's a Python service with a real database dependency, its CI pipeline needs an **ephemeral test database** (e.g., a throwaway Postgres container spun up just for the pipeline run) so unit/integration tests against actual queries run against a real (if temporary) schema rather than mocks alone — this is where Liquibase's `updateSQL` dry-run validation plugs in directly ahead of the test stage.

---

# SECTION C: Cloud Infra Design

## C1. Cloud Infra Design — 30,000-Feet View

### Introduction
This document captures the overall cloud architecture at a level appropriate for leadership/architecture review — not implementation-level detail (that's the Dev-environment-specific document below), but the shape of the whole system.

### Pre-requisites
AWS account structure decided (single account vs multi-account via AWS Organizations), network CIDR plan agreed to avoid future VPC-peering conflicts, environment strategy decided (Dev/Staging/Prod separation — by account, by VPC, or by namespace).

### System Development Approaches
Chosen approach: **multi-account via AWS Organizations** (separate accounts for Dev, Staging, Prod, and a shared Tooling account hosting Jenkins/SonarQube) — this gives hard security/billing isolation between environments (a Dev misconfiguration literally cannot touch Prod resources, versus relying on IAM policy alone within a single account) and clean cost attribution per environment.

### 30,000-Feet Diagram
```
                     ┌─────────────────────────┐
                     │   AWS Organizations       │
                     │   (Management Account)    │
                     └────────────┬─────────────┘
        ┌───────────────┬─────────┼─────────┬───────────────┐
        ▼               ▼         ▼         ▼               ▼
   ┌─────────┐    ┌─────────┐ ┌─────────┐ ┌─────────┐  ┌─────────┐
   │ Tooling  │    │  Dev     │ │ Staging  │ │  Prod    │  │ Shared   │
   │ Account  │    │ Account  │ │ Account  │ │ Account  │  │ Services │
   │(Jenkins, │    │(App infra│ │(App infra│ │(App infra│  │(Logging, │
   │ SonarQube)│    │ per svc) │ │ per svc) │ │ per svc) │  │ Security)│
   └─────────┘    └─────────┘ └─────────┘ └─────────┘  └─────────┘
```

### Resource Requirements (documented explicitly per requirement)
Per environment account: VPC with public/private subnets across 2+ AZs, NAT Gateway(s) for private subnet egress, RDS instance(s) sized per service, EC2/ASG capacity for app + Jenkins agents, S3 for artifacts/logs, IAM roles scoped per service (least privilege) — sized and costed separately per environment since Dev/Staging can run smaller instance classes than Prod.

## C2. Cloud Infra Design — Dev Environment

### Introduction
This document details the Dev account/VPC specifically, at implementation-ready detail.

### Infra Diagram
```
Dev VPC (10.10.0.0/16)
 ├── Public Subnet AZ-a (10.10.1.0/24) — ALB, NAT Gateway
 ├── Public Subnet AZ-b (10.10.2.0/24) — ALB (multi-AZ)
 ├── Private Subnet AZ-a (10.10.11.0/24) — App EC2/ECS, Jenkins agents
 ├── Private Subnet AZ-b (10.10.12.0/24) — App EC2/ECS, Jenkins agents
 └── Private DB Subnet group (10.10.21.0/24, 10.10.22.0/24) — RDS
```

### Security Group & NACL Details
- **ALB SG:** inbound 443 from corporate VPN CIDR only; outbound to App SG.
- **App SG:** inbound from ALB SG only on app port; outbound to DB SG (5432/3306) and internet via NAT for dependency pulls.
- **DB SG:** inbound from App SG only, no other source; no public accessibility flag on RDS at all.
- **NACLs:** default-allow within the VPC's own CIDR (defense-in-depth is primarily handled at the SG layer, which is stateful and per-instance); NACL used mainly to explicitly deny known-bad CIDR ranges if ever required, since NACLs are stateless and coarser-grained than SGs.

### Resource Requirements
Dev sized deliberately smaller than Prod (e.g., `t3.medium` app instances vs Prod's `m5.large`+, single-AZ RDS in Dev vs Multi-AZ in Prod) — documented explicitly so cost differences between environments are a visible, intentional decision rather than accidental under-provisioning of Prod or over-spending in Dev.

---

# SECTION D: Cost Optimization

## D1. AWS Cost Allocation Tags

### Setup Guide (per last sprint's documentation)
1. Define a mandatory tagging standard: `Environment` (Dev/Staging/Prod), `Team`, `Project`, `CostCenter`.
2. Enforce tag presence via **AWS Config rule** (`required-tags`) or a Service Control Policy denying resource creation without required tags (ties directly into the SCP ticket below).
3. Activate the tags as **user-defined cost allocation tags** in the Billing Console so they become available as a Cost Explorer filter/group-by dimension.
4. Demo to Pre & L0 reviewers: show a live resource creation blocked by missing tags, then show it succeed once tags are supplied, then show that resource appearing correctly grouped in Cost Explorer.

### Right-Sizing (called out explicitly as a prerequisite before setup)
Before finalizing instance types/sizes across the Dev environment, run AWS Compute Optimizer recommendations against current (even if only briefly running) workloads to right-size instance classes — avoiding the common anti-pattern of copying a "safe" oversized instance type from a template without validating actual utilization.

### Spot Instances (explored per requirement)
For non-critical, interruption-tolerant workloads (Jenkins ephemeral build agents are the textbook use case — a build that gets interrupted simply retries on a new agent), Spot Instances offer substantial cost savings (commonly 60–90% off on-demand pricing) — explicitly explored for Jenkins agent capacity as part of this ticket, with a fallback to on-demand agents via EC2 Fleet's mixed-instance-policy if spot capacity is unavailable at request time.

## D2. AWS Service Control Policies (SCPs)
SCPs (applied at the AWS Organizations OU level) enforce hard guardrails that even account admins cannot override via IAM alone — e.g., denying instance launches without required cost-allocation tags, restricting allowed instance types/regions per environment to prevent accidental expensive resource creation, denying deletion of the AWS Config/CloudTrail logging setup (compliance guardrail).

## D3. Budget & Cost Anomaly Alerts
**AWS Budgets** configured per account/tag-dimension with threshold alerts (e.g., 80%/100%/120% of monthly budget) routed to Slack/email. **Cost Anomaly Detection** uses AWS's own ML-based service to learn normal spend patterns per linked account/service and alert on statistically unusual spend spikes even *before* a budget threshold is technically crossed — catching, for example, a runaway forgotten dev resource within a day rather than waiting for the monthly budget alert to fire.

## D4. Cost Tag Reports via Cost Explorer
Cost Explorer reports built with **Group By: Tag (Team/Project/Environment)** as the primary dimension, saved as recurring reports, giving each team visibility into their own actual spend against their tagged resources — this is the direct payoff of the tagging enforcement in D1: without consistent tags, Cost Explorer's "group by tag" view is simply incomplete/inaccurate.

---

# Reviewer & Interview Questions — Part 4

**Reviewer questions:**
1. Why must Active Scan never run against production, but Passive Scan is safe to?
2. Walk through how ZAP discovers authenticated-only pages it otherwise couldn't reach.
3. Why does `poetry update` never belong in a CI pipeline?
4. How does `poetry.lock`'s hash-pinning actually prevent a "works on my machine" dependency drift issue?
5. Why does the Attendance API's CI pipeline need an ephemeral database rather than mocking everything?
6. Why multi-account (AWS Organizations) instead of a single account with IAM boundaries for Dev/Staging/Prod separation?
7. Why is Spot particularly well-suited to Jenkins build agents specifically, more than to the application tier?
8. What's the actual mechanism that makes a Service Control Policy stronger than an IAM Deny policy at the account level?
9. Why is Cost Anomaly Detection meaningfully different from a simple Budget threshold alert?
10. Why does Gunicorn's worker-process model matter given Python's GIL?

**Scenario:**
1. "A ZAP active scan is reporting zero vulnerabilities on a login-protected app — is that good news?" → almost certainly means the auth context/login sequence isn't configured and ZAP is only crawling the public surface, not a genuinely clean bill of health — always sanity-check scan coverage (number of URLs discovered) alongside the finding count.
2. "Dev environment costs spiked 3x overnight with no deployment — how do you investigate?" → Cost Anomaly Detection alert as the trigger, Cost Explorer grouped by tag to isolate which team/project, CloudTrail to identify what resource was created and by whom, and check whether an SCP gap allowed an oversized/untagged resource through.

---

*End of Part 4. Remaining in the series: Ansible Role & Playbook CI/CD deep dive (Part 5), then the consolidated 300-question Mock Interview and the Final Revision materials (one-pager, 5-page notes, 20-page notes, cheat sheet) as a closing Part 6. Say the word for Part 5.*
