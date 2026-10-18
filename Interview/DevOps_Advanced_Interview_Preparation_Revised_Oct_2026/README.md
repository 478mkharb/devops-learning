# Advanced DevOps Interview Preparation — Revised Edition

**500 questions · 10 topics · 50 questions per topic**

**Target:** Mid-level/L2 DevOps Engineer (3–5 years).
**Reference date:** 18 October 2026.

## What is included
- Separate folder and Markdown study guide for each of 10 topics.
- Exactly 50 questions per topic, with an interview-ready answer, technical explanation, production scenario, diagnostic commands/checks, common mistake, and follow-up.
- JSON question bank and CSV index for search, filtering, or later conversion to flashcards.
- Official documentation and release references by topic.
- Greater emphasis on specific production incidents, diagnosis, reversible mitigation, validation, and prevention.

## Topic index

- [Jenkins & CI/CD — 50 questions](./01_jenkins_ci_cd/README.md)
- [Terraform & Infrastructure as Code — 50 questions](./02_terraform_iac/README.md)
- [AWS (EC2, VPC, IAM, S3, EKS, etc.) — 50 questions](./03_aws/README.md)
- [Kubernetes & EKS — 50 questions](./04_kubernetes_eks/README.md)
- [Docker & Docker Compose — 50 questions](./05_docker_compose/README.md)
- [Ansible & Configuration Management — 50 questions](./06_ansible/README.md)
- [Linux & Shell Scripting — 50 questions](./07_linux_shell/README.md)
- [Prometheus, Grafana, OpenTelemetry & SRE — 50 questions](./08_observability_sre/README.md)
- [DevSecOps & Cloud Security — 50 questions](./09_devsecops_security/README.md)
- [Networking, DNS, Load Balancers & TLS — 50 questions](./10_networking_dns_tls/README.md)

## Recommended study order
1. Linux & Shell Scripting
2. Networking, DNS, Load Balancers & TLS
3. AWS
4. Docker & Docker Compose
5. Kubernetes & EKS
6. Jenkins & CI/CD
7. Terraform & Infrastructure as Code
8. Ansible & Configuration Management
9. Prometheus, Grafana, OpenTelemetry & SRE
10. DevSecOps & Cloud Security

## Current-practice references as of 18 October 2026
- Jenkins changelog: https://www.jenkins.io/changelog/
- Terraform releases: https://github.com/hashicorp/terraform/releases
- Kubernetes releases: https://kubernetes.io/releases/
- Amazon EKS version lifecycle: https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html
- Docker Compose releases: https://github.com/docker/compose/releases
- OWASP DevSecOps guidance: https://owasp.org/projects/devsecops-guideline
- OpenTelemetry documentation: https://opentelemetry.io/docs/

Time-sensitive release snapshots should be rechecked at study time. The latest weekly release is not automatically the right production release; distinguish stable, LTS, beta, and release-candidate versions and verify plugin/provider/add-on compatibility.

## Interview answer framework
Use **impact → evidence → mitigation → validation → prevention** for incidents. State assumptions, mention blast radius, explain why a command is useful, and describe what output would change your diagnosis.

## What still needs your own validation
- Rehearse commands in a disposable lab; some depend on tool versions, installed plugins, permissions, CNI, cloud region, and organization policy.
- Verify the current official version and support lifecycle before discussing a version-specific feature as current.
- Adapt answers to your real experience. Do not claim you operated a tool or performed a remediation unless you can explain what you actually did.
- Check every code/configuration example against your production standards before reuse. This is an interview study guide, not an executable production runbook.

## Files
- `question_bank.json`: structured question and answer data.
- `question_index.csv`: topic, number, difficulty, question, and follow-up index.
- Each topic folder contains its full Markdown guide.