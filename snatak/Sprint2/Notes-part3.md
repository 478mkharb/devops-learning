# Sprint-2 Interview & Implementation Handbook
## Part 3 of N — CI Orchestration Tools, SonarQube Deep Dive, Jenkins Deep Dive

---

# SECTION A: CI Orchestration Tools

## A1. Feature Document — Jenkins

### Introduction
**What:** Jenkins is an open-source, self-hosted automation server that orchestrates build/test/deploy pipelines via a plugin-based architecture. **Why:** it's the most extensible, longest-standing CI orchestrator with the largest plugin ecosystem (2000+), making it able to integrate with virtually any tool (SonarQube, ZAP, Slack, AWS, Ansible) without vendor lock-in.

### Workflow Diagram
```
Webhook (VCS) → Jenkins Controller receives event → 
Matches configured Multibranch Pipeline job → 
Allocates an Agent (executor) → 
Checks out Jenkinsfile from repo → 
Parses Declarative Pipeline stages → 
Executes each stage on the agent's workspace → 
Publishes results (JUnit reports, SonarQube results, artifacts) → 
Post-build actions (notify, archive)
```

### Advantages
Massive plugin ecosystem, full scripting flexibility (Groovy-based Scripted Pipelines for edge cases Declarative can't express), free/open-source (no per-seat licensing), works with virtually any VCS/cloud/tool, strong community and enterprise support options (CloudBees).

### Best Practices
Pipeline-as-code only (Jenkinsfile in repo, never manual "Freestyle" jobs for anything production-relevant), Shared Libraries for reusable pipeline logic (DRY across repos), agents as ephemeral containers/instances rather than long-lived pet servers, credentials only via the Credentials plugin (never hardcoded).

### Conclusion
Jenkins remains the most flexible, tool-agnostic CI orchestrator — the right choice when the org already uses a heterogeneous toolchain (which this project does: SonarQube + ZAP + Ansible + AWS + 4 different language ecosystems) rather than being locked into one vendor's opinionated pipeline model.

---

## A2. Feature Document — GitLab CI/CD

### Introduction
GitLab CI/CD is the native, built-in pipeline engine of the GitLab platform — pipelines are defined in `.gitlab-ci.yml` and executed by GitLab Runners.

### Workflow Diagram
```
git push → GitLab detects .gitlab-ci.yml → 
Creates a Pipeline made of Stages → 
Each Stage has Jobs → 
GitLab Runner (shared or self-hosted) picks up Jobs → 
Executes in isolated Docker containers by default → 
Reports back to GitLab UI (pipeline graph, test reports, security dashboards)
```

### Advantages
Zero integration friction since CI is native to the VCS platform (no separate webhook/plugin wiring), built-in SAST/DAST/dependency scanning/license scanning at the Ultimate tier (reduces the number of separate tools this project would otherwise need to wire together manually), excellent visual pipeline UI, Auto DevOps can bootstrap a working pipeline with minimal config.

### Best Practices
Use GitLab-native security scanning templates instead of hand-rolling equivalents, use `rules:` instead of legacy `only/except` for job conditions, cache dependencies between stages to speed up runs.

### Conclusion
GitLab CI/CD is the stronger choice when an org wants a single consolidated platform (VCS + CI + security scanning) and is willing to pay for the Ultimate tier to get built-in scanning — but this project already standardized on GitHub for VCS (Ticket 1, Epic 1), so adopting GitLab CI here would mean running CI disconnected from the VCS-of-record, adding complexity rather than removing it.

---

## A3. Feature Document — Buildpiper

### Introduction
Buildpiper is a DevOps orchestration/self-service platform (commonly used in Indian enterprise/BFSI contexts) that sits as a control-plane layer *above* underlying CI engines (often Jenkins) — providing a simplified UI/self-service portal for developers to trigger builds/deployments without needing to understand the underlying Jenkins/Kubernetes complexity directly.

### Workflow Diagram
```
Developer self-service portal (Buildpiper UI) → 
Buildpiper orchestration layer → 
Underlying Jenkins/K8s execution → 
Environment provisioning + deployment → 
Unified dashboard (build + deploy + environment status)
```

### Advantages
Simplifies developer experience for teams that don't want to touch Jenkinsfiles directly, strong environment/release management and self-service provisioning features, good fit for large enterprises standardizing developer self-service across many teams.

### Best Practices
Best deployed as a layer *on top of* an existing Jenkins investment rather than a full replacement, since it typically still needs an underlying execution engine.

### Conclusion
Buildpiper is compelling for very large organizations wanting to abstract Jenkins complexity from hundreds of developer teams — not yet justified for this project's current team size, where direct Jenkinsfile ownership by the DevOps team is still efficient and doesn't need an abstraction layer.

---

## A4. Comparison Table & Final Recommendation

| Factor | Jenkins | GitLab CI/CD | Buildpiper |
|---|---|---|---|
| Cost | Free (OSS) | Free tier limited; full security features need Ultimate (paid) | Commercial licensing |
| Flexibility | Highest (Groovy scripting, any plugin) | Good, YAML-based, less scriptable than Groovy | Depends on underlying engine |
| VCS coupling | Decoupled (works with any VCS) | Tightly coupled to GitLab | Decoupled, sits above another engine |
| Learning curve | Moderate–high (Groovy, plugin sprawl) | Low (YAML, native UI) | Low for end-users, hides complexity |
| Best fit | Heterogeneous toolchains, full control | All-in-one GitLab shops | Large orgs needing dev self-service |
| Maturity/ecosystem | Extremely mature, largest community | Very mature, growing fast | Niche, smaller community |

**Final recommendation: Jenkins.** This project already committed to GitHub for VCS (decoupled from GitLab's native CI), needs to integrate a genuinely heterogeneous toolchain (SonarQube, OWASP ZAP, Ansible, AWS, four different language ecosystems), and the team already has practical Jenkins implementation experience (per the prompt's premise) — Jenkins' flexibility and plugin ecosystem outweighs GitLab CI's convenience here, and Buildpiper's abstraction layer isn't justified at this team's current scale.

---

# SECTION B: SonarQube Deep Dive

## B1. Authentication & Authorization

### Introduction
SonarQube needs its own authn/authz layer since it holds sensitive information — source code quality data, security vulnerability findings — that shouldn't be visible org-wide.

### Types Compared

| Method | Description | Pros | Cons |
|---|---|---|---|
| Local SonarQube accounts | Username/password stored in SonarQube's own DB | Simple, no dependency | No central de-provisioning, password sprawl |
| SAML SSO | Federates to corporate IdP (Okta/Azure AD) | Central identity, MFA inherited from IdP, auto de-provisioning | Requires IdP admin coordination, more setup |
| LDAP/Active Directory | Direct bind to corporate directory | Works well in on-prem AD-heavy orgs | Less modern than SAML/OIDC, harder to federate cloud IdPs |

### Recommendation
**SAML SSO**, consistent with the SSO decision made for VCS access (Epic 1) — one identity source across all DevOps tooling, with group-claim mapping to SonarQube's built-in permission templates (`sonar-users`, `sonar-administrators`, and a custom `sonar-security-reviewers` group with access to the Security Hotspots review workflow specifically).

---

## B2. Infra Setup — AWS Architecture Diagram

```
                        ┌─────────────────────────┐
                        │      Route53 DNS         │
                        │  sonarqube.company.com   │
                        └────────────┬─────────────┘
                                     │
                        ┌────────────▼─────────────┐
                        │   Application Load        │
                        │   Balancer (ALB, HTTPS)    │
                        └────────────┬─────────────┘
                                     │
                  ┌──────────────────┼──────────────────┐
                  ▼                                      ▼
        ┌──────────────────┐                  ┌──────────────────┐
        │ EC2: SonarQube    │                  │ EC2: SonarQube    │
        │ App Node (AZ-a)   │                  │ App Node (AZ-b)   │  (only if
        │ (Elasticsearch     │                  │ (Elasticsearch     │   using DCE
        │  embedded)         │                  │  embedded)         │   licensed HA)
        └─────────┬─────────┘                  └─────────┬─────────┘
                  │                                        │
                  └───────────────────┬────────────────────┘
                                       ▼
                          ┌────────────────────────┐
                          │  RDS PostgreSQL         │
                          │  (Multi-AZ)             │
                          └────────────────────────┘
                                       │
                          ┌────────────────────────┐
                          │  EFS / EBS (persistent  │
                          │  data dir, plugins)     │
                          └────────────────────────┘

Security Groups:
- ALB SG: inbound 443 from corporate CIDR/VPN only
- App SG: inbound 9000 from ALB SG only; outbound to RDS SG (5432) and internet (plugin updates) via NAT
- RDS SG: inbound 5432 from App SG only, no public access
```

### Component Notes
SonarQube Community Edition is single-node only (no true active-active clustering) — the second app node in the diagram is only relevant if the org licenses **Data Center Edition**, which supports multiple search/application nodes behind a load balancer with shared state in Postgres/Elasticsearch. This distinction matters a great deal for the HA section below.

---

## B3. Software Configuration — Ansible Role (Documentation Only, POC via Manual Setup)

### Ansible Role Documentation (for implementation next sprint)

**Role name:** `sonarqube_setup`

**Intended structure:**
```
roles/sonarqube_setup/
├── tasks/
│   ├── main.yml          # orchestrates all included tasks
│   ├── prerequisites.yml # OS tuning: vm.max_map_count, ulimits (required by embedded Elasticsearch)
│   ├── install.yml       # download/unpack SonarQube distribution, create service user
│   ├── configure.yml     # templates sonar.properties (DB connection, ports)
│   └── service.yml       # systemd unit setup, enable/start service
├── templates/
│   └── sonar.properties.j2
├── defaults/main.yml      # default vars: version, install_dir, db_host, db_name
├── handlers/main.yml      # restart sonarqube handler
└── vars/main.yml
```

**Key documented requirements before implementation:**
- OS-level: `vm.max_map_count=262144` and `fs.file-max` tuning — SonarQube's embedded Elasticsearch will refuse to start without this, a very common real-world failure mode.
- Java runtime version pinned per SonarQube version compatibility matrix (checked in `prerequisites.yml`).
- Idempotency: the role must check current installed version before re-downloading/reinstalling (avoid unnecessary restarts on every playbook run).
- Secrets (DB password) pulled from Ansible Vault or an external secrets backend, never plaintext in `vars/main.yml`.

### POC (this sprint)
This sprint's POC is a **manual, normal installation** (not yet Ansible-automated) to validate the working configuration before it gets encoded into the role next sprint: provision an EC2 instance, install Java, download SonarQube, configure `sonar.properties` to point at a PostgreSQL RDS instance, tune `vm.max_map_count`, start via systemd, confirm UI reachable and a test project scan completes with a Quality Gate result.

---

## B4. High Availability (HA)

### Introduction
**What:** ensuring SonarQube remains available despite a single node/AZ failure. **Why:** SonarQube gates every build in the pipeline — if it's down, either all builds fail their quality gate stage or (worse) teams start skipping the gate, silently reintroducing the exact risk the tool exists to prevent.

### Methods Compared

| Method | Description | Applicability |
|---|---|---|
| **Data Center Edition clustering** | Sonar's official licensed HA: multiple app/search nodes behind an LB, shared Postgres | Only method for *true* active-active HA; requires paid license |
| **Active-Passive via ASG + AMI** | Single node, but Auto Scaling Group with min/max=1, health-check triggers automatic replacement from a Golden AMI on failure | Works with Community Edition; not true zero-downtime HA, but automates recovery (near-HA) |
| **RDS Multi-AZ for the DB tier only** | Database survives AZ failure even though the app node doesn't | Necessary regardless of which app-tier method is chosen — the DB should never be the single point of failure |

### Recommendation
Given Community Edition licensing (implied by cost constraints typical at this project stage), the pragmatic approach is **Active-Passive via Auto Scaling Group** (self-healing single node) combined with **RDS Multi-AZ** for the database tier — true Data Center Edition clustering is flagged as a future upgrade path once budget allows, and should be explicitly noted as a licensing/cost decision in the conclusion, not a technical limitation being silently accepted.

---

## B5. Disaster Recovery (DR)

### Introduction
DR covers recovery from a full region/catastrophic failure, distinct from HA's single-AZ/node failure scope.

### Backup & Recovery Strategy
- **Database:** RDS automated snapshots (daily) + point-in-time recovery via transaction logs (standard RDS capability) — this holds all project/quality-gate/issue history.
- **Application data directory** (plugins, non-DB config): backed up via EBS snapshot or synced to S3 on a schedule.
- **Configuration as code:** since the Ansible role (once implemented) fully defines the install, the *application tier* itself is disposable/reproducible — DR for the app tier is really "re-run the Ansible role against a new instance in a DR region," not "restore a server backup."

### DR Methods Compared
| Method | RTO | RPO | Cost |
|---|---|---|---|
| Backup & restore (cold DR) | Hours | Up to last snapshot interval | Lowest |
| Pilot light (DB replicated cross-region, app infra defined as code but not running) | Tens of minutes | Near-zero (DB replication lag only) | Moderate |
| Warm standby (scaled-down replica running in DR region) | Minutes | Near-zero | Higher |

### MTTR Consideration
Mean Time To Recovery is driven primarily by how quickly the app tier can be rebuilt — since this is Ansible-automated and AMI-backed, MTTR for the app tier itself can be under 15–20 minutes; the DB restore time from snapshot is typically the larger contributor to overall MTTR and should be measured explicitly during DR drills, not assumed.

### Recommendation
**Pilot light** approach: cross-region RDS read replica kept warm, Ansible role + Golden AMI ready to deploy on demand in the DR region — balances cost against an acceptable RTO/RPO for a tool that gates CI (important, but not a customer-facing production system requiring warm-standby-level spend).

---

## B6. Monitoring Metrics

### Introduction
Monitoring SonarQube ensures issues (capacity exhaustion, Elasticsearch problems, DB connection saturation) are caught before they cause an outage that silently disables the quality gate across all pipelines.

### Metrics to Monitor
| Category | Specific Metrics |
|---|---|
| **JVM health** | Heap usage, GC pause time, thread count |
| **Elasticsearch (embedded)** | Search node status (green/yellow/red), query latency, index size |
| **Database** | Active connections vs pool max, query latency, RDS CPU/storage |
| **Application** | Analysis queue length (jobs waiting to be processed — a growing queue = capacity problem), average analysis duration, Web API response time |
| **System** | Disk usage on data directory (fills up from analysis reports/logs), CPU/memory on the EC2 instance |

### How Monitored
CloudWatch agent on the EC2 instance for OS/JVM metrics; SonarQube exposes a `/api/system/health` and `/api/monitoring/metrics` endpoint (or a dedicated Prometheus exporter) that Prometheus/Grafana scrapes for the application-level metrics above; alerts routed via CloudWatch Alarms → SNS → Slack/PagerDuty.

### Thresholds & Alerts
- Analysis queue length > 10 pending for more than 5 minutes → warning (capacity concern).
- JVM heap usage > 85% sustained for 10 minutes → critical (risk of OOM).
- DB connection pool > 90% utilized → warning.
- Elasticsearch status = red → critical (search/analysis will fail outright).

---

## B7. Log Files

### Introduction
SonarQube produces several distinct logs, each covering a different subsystem — the failure mode is different for each, so knowing which file to check matters for fast troubleshooting.

### Log Files Identified
| Log file | Purpose |
|---|---|
| `logs/sonar.log` | Main application/wrapper log — startup issues, general app-level errors |
| `logs/web.log` | Web server process — HTTP request handling, API errors, authentication issues |
| `logs/ce.log` | Compute Engine — background processing of submitted analysis reports (this is where a stuck/failed analysis shows up) |
| `logs/es.log` | Embedded Elasticsearch — search index issues, the `vm.max_map_count` failure appears here specifically |
| `logs/access.log` (if enabled) | HTTP access log for auditing/troubleshooting web traffic patterns |

### Best Practice
Ship all of these to a centralized log aggregator (CloudWatch Logs or ELK) rather than relying on SSH-ing into the instance — critical for both DR (instance may not exist anymore) and for correlating an incident across the app, DB, and ES layers simultaneously.

---

## B8. Quality Gates

### Introduction
**What:** a Quality Gate is a set of conditions (thresholds on metrics like coverage, duplicated code, new bugs, new vulnerabilities) that a project's analysis must satisfy to be marked "Passed" — this is the mechanism that actually blocks a Jenkins pipeline (via `waitForQualityGate`) when code quality/security regresses.

### Default Quality Gate ("Sonar way")
Applies conditions primarily to **New Code** (not the whole codebase's historical debt, which would be unrealistic to fix all at once): 0 new bugs, 0 new vulnerabilities, security hotspots reviewed, coverage on new code ≥ 80%, duplicated lines on new code < 3%.

### Custom Quality Gates
This project defines a custom gate per language/repo type where the default doesn't fit — e.g., a stricter gate for security-sensitive services (backend Java/Python APIs handling auth) requiring 0 new *and* 0 pre-existing critical vulnerabilities, versus a slightly relaxed gate for the React frontend repo where coverage targets might differ due to heavy UI-component testing overhead.

### Internal Mechanism (tie-back to CI internals)
When Jenkins calls `waitForQualityGate()`, it doesn't poll synchronously in a tight loop — SonarQube's Compute Engine processes the submitted report asynchronously, and the scanner step embeds a **task ID** in its output; `waitForQualityGate` polls SonarQube's `/api/ce/task` endpoint using that task ID until status is `SUCCESS`, then checks the associated analysis's Quality Gate status via `/api/qualitygates/project_status` — if `ERROR`, the Jenkins step throws, which (combined with `pipeline { options { }}` or an explicit `error()` call) fails the stage and therefore the whole build.

---

# SECTION C: Jenkins Deep Dive

## C1. Authentication & Authorization

### Types Compared
| Method | Description | Pros | Cons |
|---|---|---|---|
| Jenkins' own user database | Local accounts | Simple | No central control, doesn't scale |
| LDAP | Bind to corporate directory | Central, mature plugin | On-prem AD focused |
| SAML SSO | Federated to Okta/Azure AD | Consistent with rest of toolchain, MFA inherited | Requires SAML plugin config + IdP coordination |
| Matrix-based / Role-based Authorization Strategy | Fine-grained per-job/per-folder permissions | Very granular | Can get complex to manage at scale without folders/naming conventions |

### Recommendation
**SAML SSO for authentication** (same IdP as VCS and SonarQube — one identity source org-wide) combined with the **Role-Based Authorization Strategy plugin** for authorization: global roles (Admin, Developer, Viewer) plus project/folder-scoped roles so a team only sees and can trigger jobs relevant to their own repos, not the entire Jenkins instance.

---

## C2. Infra Setup — AWS Architecture Diagram

```
                    ┌─────────────────────────┐
                    │      Route53 DNS         │
                    │  jenkins.company.com     │
                    └────────────┬─────────────┘
                                 │
                    ┌────────────▼─────────────┐
                    │   ALB (HTTPS, corp-only)   │
                    └────────────┬─────────────┘
                                 │
                    ┌────────────▼─────────────┐
                    │  EC2: Jenkins Controller   │
                    │  (single, stateful)        │
                    │  - JENKINS_HOME on EBS      │
                    └────────────┬─────────────┘
                                 │ SSH / JNLP over private subnet
              ┌──────────────────┼──────────────────┐
              ▼                  ▼                  ▼
   ┌────────────────┐ ┌────────────────┐ ┌────────────────┐
   │ Agent (EC2, on   │ │ Agent (EC2, on   │ │ Agent (ECS/Fargate│
   │ ASG, per-language │ │ ASG, per-language │ │ ephemeral, spins   │
   │ build capacity)   │ │ build capacity)   │ │ up per build)      │
   └────────────────┘ └────────────────┘ └────────────────┘

Security Groups:
- ALB SG: 443 inbound from corp CIDR
- Controller SG: 8080 inbound from ALB SG only; outbound to agents (JNLP port) and internet via NAT (plugin updates, dependency downloads)
- Agent SG: inbound from Controller SG only (JNLP), outbound to internet for build dependency resolution
```

### Component Notes
The Jenkins **controller** is deliberately kept stateful and singular (JENKINS_HOME holds job configs, build history, credentials store) — this is exactly why HA/DR for Jenkins is architecturally harder than for a stateless app, and why it's treated as its own dedicated section below. **Agents**, by contrast, are designed to be ephemeral/disposable — using EC2 Auto Scaling or Fargate-based dynamic agents (via the Amazon ECS or EC2 Fleet plugin) so build capacity scales with load and agents never accumulate drift (tying back to the immutable-infra ticket).

---

## C3. Software Configuration — Ansible Role (Documentation Only, POC via Manual Setup)

### Ansible Role Documentation (for implementation next sprint)

**Role name:** `jenkins_controller_setup`

**Intended structure:**
```
roles/jenkins_controller_setup/
├── tasks/
│   ├── main.yml
│   ├── prerequisites.yml   # Java version install, repo key/package source setup
│   ├── install.yml         # package install of jenkins
│   ├── plugins.yml         # installs pinned plugin list via jenkins-plugin-cli
│   ├── configure.yml       # Configuration as Code (JCasC) yaml templated in
│   └── service.yml
├── templates/
│   └── jenkins.yaml.j2     # JCasC config: security realm (SAML), authorization strategy, tool installations
├── defaults/main.yml       # plugin version pins, Java version
└── handlers/main.yml       # restart jenkins handler
```

**Key documented requirements before implementation:**
- **Plugin versions must be pinned**, not "latest" — unpinned plugin upgrades are one of the most common real-world sources of a Jenkins instance breaking unexpectedly after a routine playbook run.
- **JCasC (Jenkins Configuration as Code)** is the mechanism by which the role avoids manual click-ops for SSO/authorization setup — the entire security realm and authorization strategy is expressed as YAML and loaded at startup, making Jenkins config itself version-controlled and reproducible (this is what allows fast DR rebuild, addressed below).
- Idempotency: check installed plugin versions before reinstalling; only restart Jenkins when config actually changed (avoid unnecessary downtime on every playbook run).

### POC (this sprint)
Manual installation on an EC2 instance: install Java + Jenkins package, unlock via initial admin password, install a baseline plugin set (Git, Pipeline, Credentials Binding, SonarQube Scanner, Slack Notification), configure a sample pipeline job, confirm a webhook-triggered build executes successfully end-to-end.

---

## C4. High Availability (HA)

### Introduction
Jenkins HA is harder than typical stateless-app HA because the **controller is inherently stateful** — job definitions, build history, and (critically) the credentials store all live in `JENKINS_HOME`.

### Methods Compared
| Method | Description | Trade-off |
|---|---|---|
| **Active-Passive with shared EFS-backed JENKINS_HOME** | Standby controller mounts the same EFS volume; on failure, standby is promoted and starts against the same data | Simpler, cheaper; brief downtime during failover/promotion |
| **CloudBees CI (commercial)** | Purpose-built HA/clustering for Jenkins with proper multi-controller support | True HA but requires commercial licensing |
| **Backup + fast redeploy via ASG (near-HA)** | Single controller, ASG min=1 auto-replaces on health-check failure, `JENKINS_HOME` restored from the latest EFS/EBS snapshot | Not true zero-downtime, but automated and low-cost |

### Recommendation
**Active-Passive via EFS-backed `JENKINS_HOME`** combined with ASG-driven auto-replacement of the controller instance on failure — a pragmatic middle ground given Jenkins' architectural constraints and (implied) budget, with commercial CloudBees HA flagged as a future option if uptime requirements tighten.

---

## C5. Disaster Recovery (DR)

### Backup & Recovery
- **`JENKINS_HOME` backup:** scheduled EFS-to-S3 sync (or EBS snapshot if not using EFS) capturing job configs, build history, plugins, and the credentials store (encrypted).
- **Configuration as Code:** since JCasC (from the Ansible role) defines security/authorization config declaratively, a full Jenkins rebuild in a DR region needs only: restore `JENKINS_HOME` from the latest backup + re-run the Ansible role — this dramatically reduces DR complexity versus a hand-configured Jenkins instance.
- **Job definitions as code:** Jenkinsfiles live in each application repo (not in Jenkins' own config) — meaning even a total `JENKINS_HOME` loss only truly costs build *history*, not the pipeline logic itself, since that's recoverable straight from Git.

### DR Methods Compared
Same framework as SonarQube (cold backup/restore vs pilot light vs warm standby) — recommendation here is similarly **pilot light**: JCasC + Ansible role ready to deploy in a DR region on demand, with `JENKINS_HOME` backups replicated cross-region, rather than paying for an always-on warm-standby controller.

### MTTR
Because job logic lives in Git (not Jenkins config) and security config lives in JCasC (versioned YAML), MTTR for a full Jenkins loss is dominated by "spin up new controller + restore JENKINS_HOME from S3," typically well under an hour — a key argument for why the pilot-light approach is sufficient rather than paying for full warm standby.

---

## C6. Monitoring Metrics

### Metrics to Monitor
| Category | Specific Metrics |
|---|---|
| Executor/Agent health | Executor utilization %, queue length (jobs waiting for a free executor), number of offline/disconnected agents |
| JVM health | Heap usage, GC pause time (Jenkins controller is a long-running JVM process — heap leaks from plugin bugs are a real, common failure mode) |
| Build performance | Average build duration trend, build queue wait time |
| Disk | `JENKINS_HOME` disk usage (build artifacts/logs accumulate — a very common real-world outage cause is the controller disk filling up) |
| Plugin health | Failed plugin updates, plugin compatibility warnings on startup |

### How Monitored
Prometheus plugin for Jenkins exposes a `/prometheus` metrics endpoint scraped by Prometheus, visualized in Grafana; CloudWatch agent for OS-level disk/CPU/memory; alerts via CloudWatch Alarms/Prometheus Alertmanager → Slack/PagerDuty.

### Thresholds & Alerts
- Build queue length > 5 sustained for 10+ minutes → warning (capacity/agent scaling issue).
- Controller disk usage > 80% → warning; > 90% → critical.
- JVM heap > 85% sustained → critical.
- Any agent offline for > 15 minutes unexpectedly → warning (investigate connectivity/capacity).

---

## C7. Log Files

### Log Files Identified
| Log | Purpose |
|---|---|
| `jenkins.log` (or systemd journal via `journalctl -u jenkins`) | Main controller log — startup, plugin loading errors, general exceptions |
| Per-build console log (`$JENKINS_HOME/jobs/<job>/builds/<n>/log`) | The actual pipeline execution output developers look at for build failures |
| `$JENKINS_HOME/logs/` (custom log recorders, if configured) | Fine-grained logging for specific components (e.g., SAML plugin debug logs) when troubleshooting auth issues specifically |
| Agent-side logs (via JNLP/SSH launch logs) | Connectivity/launch failures between controller and a specific agent |

### Best Practice
Centralize both the controller log and (critically, often overlooked) per-build console logs into a log aggregator with retention policy — build logs are frequently needed weeks later for audit/incident postmortems, and local disk retention alone risks both loss and the disk-fill-up failure mode mentioned above.

---

# Reviewer & Interview Questions — Section C/B/A

**Reviewer questions:**
1. Why is SonarQube Community Edition unable to provide true active-active HA, and what does that mean practically for this project?
2. Walk through exactly what `waitForQualityGate` is polling and why it isn't a simple synchronous check.
3. Why is the Jenkins controller architecturally harder to make HA than the agents?
4. Why JCasC matters specifically for Jenkins DR — what would DR look like without it?
5. Why did we choose Jenkins over GitLab CI given GitLab has built-in security scanning?
6. What's the actual difference between a Quality Gate condition on "new code" vs the whole codebase, and why does that distinction exist?
7. Which specific log file would you check first if a SonarQube analysis silently never completes?
8. Why must Ansible plugin versions be pinned rather than always installing "latest"?
9. What's the security group boundary that protects the SonarQube/Jenkins RDS/DB tier specifically?
10. Why is Buildpiper not the right choice at this project's current scale, even though it simplifies developer experience?

**Scenario questions:**
1. "Jenkins controller's EBS volume just filled to 100% at 2am — walk me through your response." → identify via monitoring alert (disk usage metric), SSH in (or use SSM), find largest consumers (typically old build artifacts/logs under JENKINS_HOME/jobs), apply a retention/log-rotation policy going forward, restore service, postmortem to prevent recurrence (this is exactly why "workspace cleanup" and artifact retention policies matter operationally, not just theoretically).
2. "SonarQube's Elasticsearch status shows red — what do you check?" → `es.log` first, then `vm.max_map_count` (the single most common real-world cause), then disk space on the data volume, then JVM heap allocated to the embedded ES process.

---

*End of Part 3. Part 4 will cover: OWASP ZAP deep dive, Python CI specifics (Poetry/Gunicorn/Liquibase/Attendance API), Cloud Infra Design (30k-feet + Dev environment diagrams), and Cost Optimization (tags, SCPs, budget alerts, Cost Explorer). Let me know if you want that next.*
