# Jenkins L2/L3 Interview Questions — Clear Definitions and Detailed Answers

> Every question follows the same pattern: **Definition → Explanation → Example → Interview point**.

## Table of Contents

1. Jenkins Fundamentals
2. Jenkins Architecture
3. Jobs, Builds, Agents, and Executors
4. Triggers and Job Dependencies
5. Pipelines and Jenkinsfile
6. Credentials, Parameters, and Environment
7. Git and Multibranch Pipelines
8. Shared Libraries and Plugins
9. Security
10. SonarQube and Quality Gates
11. Artifacts and Deployments
12. Reliability, Scaling, Backup, and Troubleshooting
13. L2/L3 Scenario Questions
14. Full Forms and Revision Tables

---

# 1. Jenkins Fundamentals

## Q1. What is Jenkins?

### Clear definition
**Jenkins is an open-source automation server that orchestrates software build, test, analysis, packaging, and deployment activities.** It is primarily used to implement CI/CD workflows.

### Explanation
Jenkins does not compile Java, scan code, create cloud infrastructure, or deploy applications by itself. It invokes tools such as Git, Maven, Gradle, SonarQube, Terraform, Ansible, Kubernetes, and cloud CLIs in a controlled sequence.

A Jenkins pipeline normally performs these activities:

1. Fetch source code from a source-control system.
2. Compile or package the application.
3. Execute unit and integration tests.
4. Run quality and security checks.
5. Publish artifacts.
6. Deploy to an environment.
7. Notify the team and preserve logs.

### Example
```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Build') {
            steps { sh 'mvn clean package' }
        }
        stage('Test') {
            steps { sh 'mvn test' }
        }
    }
}
```

### Interview point
Jenkins is an **orchestrator**, not a replacement for Git, Maven, SonarQube, Terraform, Ansible, Docker, or Kubernetes.

## Q2. What is Continuous Integration, Continuous Delivery, and Continuous Deployment?

### Clear definition
- **Continuous Integration (CI):** Developers frequently merge code, and every change is automatically built and tested.
- **Continuous Delivery:** The application is always kept in a releasable state, but production deployment may require approval.
- **Continuous Deployment:** Every change that passes the required checks is automatically deployed to production.

### Explanation
CI detects defects early. Continuous Delivery adds release readiness and controlled promotion. Continuous Deployment removes the manual production approval step.

### Example
```text
Developer commit
      ↓
Build + Unit Test
      ↓
Static Analysis + Security Scan
      ↓
Deploy to Staging
      ↓
Manual Approval       → Continuous Delivery
      ↓
Production
```

If the approval is removed and production deployment is automatic, it becomes Continuous Deployment.

### Interview point
CI is about **integration and validation**. Delivery is about **release readiness**. Deployment is about **automatic release execution**.

## Q3. What is the difference between a Jenkins job, build, and pipeline?

### Clear definition
- **Job:** A configured unit of work in Jenkins.
- **Build:** One execution or run of a job.
- **Pipeline:** A code-defined workflow containing stages and steps.

### Explanation
A job is the reusable configuration. Each time the job runs, Jenkins creates a build number such as `#101`. A Pipeline is one way to define the job logic using a `Jenkinsfile`.

### Example
```text
Job: employee-api-ci
Builds: #1, #2, #3, #4
Pipeline stages: Checkout → Build → Test → Publish
```

### Interview point
A failed build does not mean the job is deleted or broken permanently. It means one execution of the job failed.

## Q4. What is a Jenkins controller?

### Clear definition
**The Jenkins controller is the central Jenkins process responsible for storing configuration, scheduling work, managing plugins, maintaining build metadata, and coordinating agents.**

### Explanation
The controller normally handles:

- Job and pipeline configuration.
- Build queue management.
- Scheduling and executor allocation.
- Credentials and global configuration.
- Plugin execution and orchestration.
- Build history and metadata.
- Communication with agents.

Heavy compilation and testing should normally run on agents rather than consuming controller resources.

### Interview point
Modern Jenkins terminology prefers **controller** instead of the older term **master**.

## Q5. What is a Jenkins agent or node?

### Clear definition
A **Jenkins agent** is a machine or execution environment where Jenkins runs build steps. A **node** is a machine registered with Jenkins; an agent process usually connects that node to the controller.

### Explanation
An agent may be:

- A physical server.
- A virtual machine.
- A cloud instance.
- A Kubernetes pod.
- A container-based execution environment.

The controller schedules work, while the agent performs commands such as `mvn package`, `npm build`, or `terraform plan`.

### Example
```groovy
pipeline {
    agent { label 'linux-java' }
    stages {
        stage('Build') {
            steps { sh 'mvn clean package' }
        }
    }
}
```

### Interview point
A label selects an eligible execution environment; it does not itself create a machine.

## Q6. What is an executor?

### Clear definition
**An executor is a slot on a Jenkins node that can run one build or task at a time.**

### Explanation
If a node has four executors, it can run up to four executor-consuming tasks concurrently, assuming CPU, memory, labels, and other restrictions allow it.

### Example
```text
Node: build-agent-1
Executors: 2
Running builds: 2
Third build: waits in queue
```

Increasing executors increases concurrency but can reduce performance if the machine lacks CPU or memory.

### Interview point
Executor count is not the same as CPU core count. It is a scheduling setting, not a guarantee of performance.

## Q7. What is a Jenkins workspace?

### Clear definition
A **workspace is the directory on an agent where Jenkins checks out source code and executes build commands for a job.**

### Explanation
The workspace may contain source files, compiled output, temporary files, test reports, and downloaded dependencies. It should not be treated as permanent storage.

### Example
```groovy
post {
    always {
        cleanWs()
    }
}
```

### Interview point
Workspace data can become stale and cause false failures. Use clean workspaces or isolated workspaces for reliable builds.

## Q8. What is the difference between an artifact and a workspace file?

### Clear definition
- **Workspace file:** A file created during a build inside the agent workspace.
- **Artifact:** A build output intentionally preserved or published for later use.

### Example
```groovy
post {
    success {
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
    }
}
```

The JAR is created in the workspace, then archived as a build artifact. For long-term storage, publish it to an artifact repository such as Nexus, Artifactory, or an object-storage bucket.

### Interview point
A workspace is temporary execution storage. An artifact repository is durable distribution storage.

---

# 2. Jenkins Architecture

## Q9. Explain the Jenkins controller-agent architecture.

### Clear definition
Jenkins uses a distributed architecture in which the controller schedules and coordinates work, while agents execute the actual build steps.

### Flow
```text
Developer
   ↓
Git Repository
   ↓ webhook / polling
Jenkins Controller
   ↓ queue + label matching
Jenkins Agent
   ↓
Build / Test / Scan / Package
   ↓
Artifact Repository / Deployment Target
```

### Explanation
The controller should remain lightweight. Agents should contain the required tools, such as Java, Maven, Node.js, Python, Terraform, or cloud CLIs.

### Interview point
If the controller becomes overloaded, move builds to agents, reduce unnecessary executors, clean old build records, and review plugin behavior.

## Q10. What is the Jenkins build queue?

### Clear definition
The **build queue is the list of tasks waiting for an eligible executor.**

### Explanation
A build enters the queue when it is triggered but cannot start immediately. Common reasons include:

- No free executor.
- No online agent.
- Label mismatch.
- Node temporarily offline.
- Throttling or concurrency restrictions.
- Quiet period.
- Upstream dependency not completed.

### Troubleshooting
Check:

1. Queue item reason.
2. Agent online status.
3. Label requirements.
4. Executor availability.
5. Node disk and memory.
6. Job throttling or lock configuration.

## Q11. What is the difference between built-in node and external agent?

### Clear definition
The built-in node is the controller’s own execution environment. An external agent is a separate machine or environment connected to Jenkins.

### Explanation
Running builds on the controller is acceptable for small labs but is generally discouraged for production because builds can consume CPU, memory, disk, and network resources needed by Jenkins itself.

### Interview point
Use dedicated agents for isolation, scalability, security, and predictable tool versions.

## Q12. What are labels in Jenkins?

### Clear definition
A **label is a logical name assigned to nodes so that jobs can request a suitable execution environment.**

### Example
```groovy
agent { label 'terraform-linux' }
```

A node may have labels such as:

```text
linux
ubuntu
terraform-linux
java17
```

A job requiring `terraform-linux` runs only on nodes carrying that label.

---

# 3. Jobs, Triggers, and Dependencies

## Q13. What is an upstream job?

### Clear definition
An **upstream job is a job that runs before another job and can trigger it or provide an input to it.**

## Q14. What is a downstream job?

### Clear definition
A **downstream job is a job triggered by or dependent on the result of another job.**

### Example
```text
employee-api-build  →  employee-api-test  →  employee-api-deploy
      upstream              downstream
```

From the perspective of `employee-api-test`, `employee-api-build` is upstream. From the perspective of `employee-api-build`, `employee-api-test` is downstream.

### Freestyle example
Configure Job A with:

```text
Post-build Actions
→ Build other projects
→ employee-api-test
```

### Pipeline example
```groovy
build job: 'employee-api-test',
      wait: true,
      propagate: true
```

- `wait: true` means the upstream pipeline waits for the downstream job.
- `propagate: true` means downstream failure causes the calling build to fail.
- `propagate: false` allows the upstream job to continue while the downstream result is handled separately.

### Interview point
Upstream/downstream describes **job dependency direction**. It is not the same as Git upstream/downstream branches.

## Q15. What is the difference between webhook and polling?

### Clear definition
- **Webhook:** The source-control system sends an event to Jenkins when a change occurs.
- **Polling:** Jenkins periodically checks the repository for changes.

### Explanation
Webhooks are generally faster and reduce unnecessary repository checks. Polling is useful when webhooks cannot be configured, but it creates periodic load and delay.

### Example
```text
Git push → GitHub webhook → Jenkins build
```

### Interview point
A webhook reaching Jenkins does not guarantee a build will start. Jenkins still evaluates branch filters, permissions, job configuration, and change conditions.

## Q16. Explain common Jenkins build triggers.

### Clear definition
A build trigger is an event or schedule that causes Jenkins to start a job.

### Common triggers

| Trigger | Meaning |
|---|---|
| Manual | User starts the build |
| Webhook | SCM sends a change event |
| Poll SCM | Jenkins checks SCM periodically |
| Cron | Jenkins starts according to a schedule |
| Upstream | Another job triggers this job |
| Remote/API | External system calls Jenkins |
| Timer | Scheduled execution independent of SCM |

### Cron example
```text
H/15 * * * *
```

This means approximately every 15 minutes, using Jenkins’ hashed distribution to avoid all jobs starting at the same second.

---

# 4. Pipelines and Jenkinsfile

## Q17. What is a Jenkinsfile?

### Clear definition
A **Jenkinsfile is a text file that defines Jenkins Pipeline logic as code.**

### Explanation
It can be stored in source control and reviewed like application code. This provides versioning, auditability, repeatability, and code review.

### Example
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
}
```

### Interview point
A Jenkinsfile should contain orchestration logic. Complex business logic should be placed in scripts or shared libraries when appropriate.

## Q18. What is a Declarative Pipeline?

### Clear definition
A **Declarative Pipeline is a structured Pipeline syntax with predefined sections and validation rules.**

### Main sections

- `pipeline`
- `agent`
- `options`
- `parameters`
- `environment`
- `stages`
- `post`
- `when`
- `triggers`

### Example
```groovy
pipeline {
    agent any
    options {
        timeout(time: 30, unit: 'MINUTES')
    }
    stages {
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
    }
    post {
        always {
            junit 'target/surefire-reports/*.xml'
        }
    }
}
```

### Interview point
Declarative Pipeline is usually preferred for maintainable production pipelines because its structure is easier to review and validate.

## Q19. What is a Scripted Pipeline?

### Clear definition
A **Scripted Pipeline is a Groovy-based Pipeline style that provides flexible programming control through a `node` block.**

### Example
```groovy
node('linux') {
    stage('Build') {
        sh 'mvn package'
    }
    if (env.BRANCH_NAME == 'main') {
        stage('Deploy') {
            sh './deploy.sh'
        }
    }
}
```

### Difference
Declarative Pipeline emphasizes a defined structure. Scripted Pipeline provides more programming freedom but can become difficult to maintain if overused.

## Q20. What is the difference between a stage and a step?

### Clear definition
- **Stage:** A logical phase of the delivery process.
- **Step:** An individual operation executed inside a stage.

### Example
```groovy
stage('Build') {
    steps {
        sh 'mvn clean package'
        sh 'ls -lh target/'
    }
}
```

`Build` is the stage. The two `sh` commands are steps.

### Interview point
Stages improve visibility and reporting. Steps perform the actual work.

## Q21. What is the `post` section?

### Clear definition
The **`post` section defines actions that Jenkins performs after stages or the entire pipeline finish.**

### Common conditions

```groovy
post {
    always  { /* always run */ }
    success { /* only after success */ }
    failure { /* only after failure */ }
    unstable { /* unstable result */ }
    aborted { /* manually aborted */ }
    cleanup { /* final cleanup */ }
}
```

### Example
```groovy
post {
    always {
        junit 'reports/*.xml'
        cleanWs()
    }
    failure {
        echo 'Notify the team'
    }
}
```

## Q22. Explain `timeout`, `retry`, and `catchError`.

### Clear definition
- **`timeout`:** Stops a block when it exceeds a time limit.
- **`retry`:** Re-executes a block when it fails.
- **`catchError`:** Captures an error and allows controlled build or stage result handling.

### Example
```groovy
stage('Deploy') {
    steps {
        timeout(time: 10, unit: 'MINUTES') {
            retry(2) {
                sh './deploy.sh'
            }
        }
    }
}
```

Use retry only for transient failures. Do not retry deterministic compilation or validation errors blindly.

## Q23. What is parallel execution?

### Clear definition
Parallel execution runs independent tasks at the same time to reduce total pipeline duration.

### Example
```groovy
stage('Parallel Checks') {
    parallel {
        UnitTests: {
            sh 'mvn test'
        }
        SecurityScan: {
            sh './security-scan.sh'
        }
        Lint: {
            sh './lint.sh'
        }
    }
}
```

### Interview point
Parallelism requires enough executors and must be used only when tasks are independent and resource contention is acceptable.

---

# 5. Parameters, Environment, and Credentials

## Q24. What are Jenkins parameters?

### Clear definition
Parameters are user-provided or externally supplied values that customize a build without changing the Jenkinsfile.

### Example
```groovy
parameters {
    choice(name: 'ENVIRONMENT', choices: ['dev', 'qa', 'prod'])
    booleanParam(name: 'RUN_SECURITY_SCAN', defaultValue: true)
}
```

Use parameters for controlled choices, not as a replacement for secure secret storage.

## Q25. What are environment variables?

### Clear definition
Environment variables are key-value settings made available to build processes.

### Example
```groovy
environment {
    APP_NAME = 'employee-api'
    DEPLOY_ENV = 'dev'
}

steps {
    sh 'echo "$APP_NAME deployed to $DEPLOY_ENV"'
}
```

Common Jenkins variables include `BUILD_NUMBER`, `BUILD_URL`, `JOB_NAME`, `WORKSPACE`, `BRANCH_NAME`, and `GIT_COMMIT`.

## Q26. What are Jenkins credentials?

### Clear definition
Jenkins credentials are securely stored authentication materials used by jobs to access external systems.

### Types

- Username/password.
- SSH private key.
- Secret text/token.
- Secret file.
- Cloud provider credentials.
- Certificates.

### Example
```groovy
withCredentials([
    string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')
]) {
    sh 'sonar-scanner -Dsonar.token="$SONAR_TOKEN"'
}
```

### Interview point
Never hardcode passwords, tokens, private keys, or cloud access keys in a Jenkinsfile.

## Q27. What is the difference between authentication and authorization?

### Clear definition
- **Authentication:** Verifies who a user is.
- **Authorization:** Determines what that authenticated user is allowed to do.

### Example
```text
Login with LDAP → Authentication
Can configure jobs? → Authorization
Can delete credentials? → Authorization
```

---

# 6. Git and Multibranch Pipelines

## Q28. What is a Multibranch Pipeline?

### Clear definition
A **Multibranch Pipeline automatically discovers branches containing a Jenkinsfile and creates a separate Pipeline job for each branch.**

### Explanation
It is useful for GitFlow, feature branches, pull requests, and branch-specific CI.

### Example
```text
Repository
├── main       → main pipeline
├── develop    → develop pipeline
├── feature/a  → feature/a pipeline
└── feature/b  → feature/b pipeline
```

### Interview point
Branch discovery, webhook configuration, credentials, and branch indexing must all be correct.

## Q29. What is SCM polling?

### Clear definition
SCM polling is Jenkins’ periodic check of the source-control repository to determine whether a new build is required.

### Problems

- Extra Git requests.
- Delayed builds.
- Load on Jenkins and Git server.
- Possible duplicate checks.

Webhooks are generally preferred when reliable connectivity is available.

## Q30. How do you prevent a deployment from running on every branch?

### Clear definition
Use branch conditions to restrict deployment to approved branches or tags.

### Example
```groovy
stage('Deploy') {
    when {
        anyOf {
            branch 'main'
            tag pattern: 'v*', comparator: 'GLOB'
        }
    }
    steps {
        sh './deploy.sh'
    }
}
```

---

# 7. Shared Libraries, Plugins, JCasC, and Job DSL

## Q31. What is a Jenkins Shared Library?

### Clear definition
A **Shared Library is a versioned repository of reusable Pipeline code shared by multiple Jenkinsfiles.**

### Why use it?

- Avoid duplicated pipeline logic.
- Standardize security checks.
- Standardize notifications.
- Centralize deployment patterns.
- Make fixes available to many pipelines.

### Example structure
```text
vars/
  buildJavaApp.groovy
src/
  com/company/Deployment.groovy
resources/
```

### Interview point
Shared Libraries should be versioned, reviewed, tested, and designed with clear interfaces.

## Q32. What is a Jenkins plugin?

### Clear definition
A plugin extends Jenkins functionality by adding integrations, Pipeline steps, triggers, credentials types, UI features, or agents.

### Examples

- Git plugin.
- Pipeline plugin.
- Credentials Binding plugin.
- SonarQube Scanner plugin.
- Kubernetes plugin.
- Slack notification plugin.

### Interview point
Plugins are powerful but increase dependency and security risk. Maintain compatibility and remove unused plugins.

## Q33. What is JCasC?

### Clear definition
**Jenkins Configuration as Code (JCasC) defines Jenkins system configuration in YAML so it can be reproduced and version-controlled.**

### Example
```yaml
jenkins:
  systemMessage: "Managed by Configuration as Code"
  numExecutors: 0
```

JCasC is useful for repeatable controller provisioning, disaster recovery, and environment consistency.

## Q34. What is Job DSL?

### Clear definition
Job DSL is a Groovy-based mechanism for generating Jenkins jobs and folders as code.

### Difference

| JCasC | Job DSL |
|---|---|
| Configures Jenkins itself | Creates jobs and folders |
| System settings, security, tools | Job definitions and pipeline jobs |
| Usually YAML | Usually Groovy DSL |

---

# 8. Security

## Q35. How do you secure Jenkins?

### Clear definition
Jenkins security protects the controller, agents, credentials, jobs, plugins, and build outputs from unauthorized access or malicious execution.

### Controls

1. Enable authentication.
2. Use role-based authorization.
3. Apply least privilege.
4. Restrict anonymous access.
5. Protect credentials.
6. Keep Jenkins and plugins updated.
7. Use HTTPS.
8. Isolate the controller from builds.
9. Restrict agent permissions.
10. Audit administrative actions.
11. Review script approvals.
12. Limit who can configure jobs.

### Interview point
A Jenkinsfile is executable code. Anyone who can modify a trusted pipeline may potentially execute commands with the permissions available to that job.

## Q36. What is the Groovy sandbox?

### Clear definition
The Groovy sandbox restricts Pipeline Groovy operations to approved methods so untrusted Pipeline code cannot freely execute dangerous operations.

### Explanation
Some methods require administrator approval. Approving a script or method should be done only after understanding its security impact.

### Interview point
Do not blindly approve every pending script signature.

## Q37. Why should builds not run on the controller?

### Clear definition
Builds should not normally run on the controller because build commands are untrusted or resource-intensive and may affect Jenkins stability or security.

### Risks

- CPU and memory exhaustion.
- Disk exhaustion.
- Credential exposure.
- Malicious build commands.
- Controller outage.

Use dedicated agents with restricted permissions.

---

# 9. SonarQube and Quality Gates

## Q38. What is SonarQube?

### Clear definition
**SonarQube is a code-quality and code-security analysis platform that identifies bugs, vulnerabilities, code smells, duplication, and coverage-related issues.**

### Explanation
Jenkins executes the scanner and waits for the analysis result. SonarQube evaluates the project against configured quality rules and a quality gate.

### Example
```groovy
stage('SonarQube') {
    steps {
        withSonarQubeEnv('sonarqube-server') {
            sh 'mvn sonar:sonar'
        }
    }
}
```

## Q39. What is a Quality Gate?

### Clear definition
A **Quality Gate is a set of pass/fail conditions that determines whether analyzed code meets the organization’s quality standards.**

### Example conditions

- New bugs must be zero.
- New vulnerabilities must be zero.
- Coverage on new code must exceed 80%.
- Duplicated lines must remain below a threshold.
- Reliability rating must be acceptable.

### Pipeline example
```groovy
stage('Quality Gate') {
    steps {
        timeout(time: 10, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
        }
    }
}
```

### Interview point
Running a SonarQube scan and enforcing its result are two different actions.

---

# 10. Artifacts and Deployment

## Q40. Why should artifacts be immutable?

### Clear definition
An **immutable artifact is a build output that is not modified after publication.**

### Explanation
If the same version is overwritten, it becomes difficult to know exactly what was tested and deployed. Use unique versions such as:

```text
employee-api:1.4.0-build.152
employee-api-1.4.0+152.jar
```

### Recommended flow
```text
Build once → Test artifact → Publish artifact → Promote same artifact
```

Do not rebuild separately for staging and production because the outputs may differ.

## Q41. How do you design a safe deployment pipeline?

### Clear definition
A safe deployment pipeline validates the release, deploys using controlled permissions, verifies health, and provides rollback or recovery.

### Typical stages

1. Checkout.
2. Build.
3. Unit tests.
4. Static analysis.
5. Security scan.
6. Package.
7. Publish immutable artifact.
8. Deploy to dev.
9. Smoke test.
10. Approval or policy check.
11. Deploy to production.
12. Verify health.
13. Notify and record evidence.

## Q42. What is rollback?

### Clear definition
Rollback is the process of returning an environment to a previously known-good application version or configuration.

### Example
```text
Current: v1.5.0
Previous: v1.4.2
Failure detected → redeploy v1.4.2
```

Rollback is easier when artifacts, infrastructure versions, database migrations, and configuration are versioned.

---

# 11. Reliability, Scaling, and Backup

## Q43. How do you improve Jenkins performance?

### Clear definition
Jenkins performance improvement means reducing queue time, controller workload, build duration, and resource contention while preserving reliability.

### Actions

- Move builds to agents.
- Use appropriate executor counts.
- Run independent stages in parallel.
- Avoid unnecessary SCM polling.
- Archive only required artifacts.
- Clean old workspaces and builds.
- Use build retention policies.
- Avoid huge console logs.
- Review slow plugins.
- Cache dependencies carefully.
- Separate workloads using labels.
- Monitor CPU, memory, disk, queue length, and executor utilization.

## Q44. What is Jenkins backup?

### Clear definition
A Jenkins backup is a recoverable copy of Jenkins configuration, job definitions, credentials configuration, plugin information, and required build metadata.

### Important data

- `$JENKINS_HOME`.
- Job configuration files.
- Pipeline definitions if not stored in Git.
- Credentials configuration and secret material.
- Plugin list and versions.
- JCasC files.
- Build metadata where required.
- External artifact references.

### Interview point
Backing up Jenkins configuration does not automatically back up artifacts stored in Nexus, Artifactory, S3, or another external repository.

## Q45. Explain RPO and RTO.

### Clear definition
- **Recovery Point Objective (RPO):** Maximum acceptable amount of data loss measured in time.
- **Recovery Time Objective (RTO):** Maximum acceptable time to restore service.

### Example
```text
RPO = 15 minutes → backups must lose no more than 15 minutes of data
RTO = 1 hour      → Jenkins should be restored within 1 hour
```

## Q46. How do you scale Jenkins?

### Clear definition
Jenkins scaling means increasing its ability to execute more work without making the controller unstable.

### Methods

- Add more agents.
- Use cloud or Kubernetes-based agents.
- Separate agents by workload.
- Use labels and queues.
- Limit controller executors.
- Use folders and governance.
- Reduce plugin overhead.
- Use artifact repositories.
- Split very large pipelines where justified.

---

# 12. Troubleshooting Questions

## Q47. A Jenkins job is stuck in the queue. What do you check?

### Clear definition
A queued job has been accepted by Jenkins but cannot obtain a suitable executor or satisfy a scheduling condition.

### Checklist

1. Is an eligible agent online?
2. Does the label exist?
3. Are all executors busy?
4. Is the agent disconnected?
5. Is the node temporarily offline?
6. Is the job blocked by throttling or locks?
7. Is there a quiet period?
8. Is the job waiting for an upstream build?
9. Is the agent out of disk or memory?
10. Are cloud agents failing to provision?

## Q48. The pipeline says `command not found`. What is the reason?

### Clear definition
The command is unavailable in the agent execution environment or is not present in the process `PATH`.

### Troubleshooting

```bash
whoami
pwd
which java
java -version
which mvn
mvn -version
echo "$PATH"
```

Check:

- Correct agent label.
- Tool installation.
- PATH configuration.
- Shell differences.
- Jenkins tool configuration.
- Container image contents.
- File execute permissions.

## Q49. A build works manually but fails in Jenkins. Why?

### Clear definition
The Jenkins process usually runs with a different user, environment, working directory, permissions, PATH, network route, or credential context than the interactive shell.

### Common causes

- Different user.
- Missing environment variables.
- Different Java or Python version.
- Missing SSH key.
- Different home directory.
- Permission denied.
- Proxy or network restrictions.
- Non-interactive shell behavior.
- Workspace contamination.

### Fix
Capture the Jenkins runtime context explicitly instead of assuming it matches the administrator’s shell.

## Q50. Jenkins reports `No space left on device`. How do you resolve it?

### Clear definition
The filesystem used by Jenkins or its agent has exhausted available disk blocks or inodes.

### Commands
```bash
df -h
df -i
du -sh "$JENKINS_HOME"/*
du -sh "$WORKSPACE"/*
```

### Actions

- Delete obsolete workspaces.
- Apply build discarders.
- Remove old archived artifacts where policy permits.
- Clean package caches.
- Rotate logs.
- Expand the volume.
- Check inode exhaustion.
- Identify unusually large files.

## Q51. A downstream job fails, but the upstream job is green. Explain.

### Clear definition
The upstream job may have triggered the downstream job without waiting for its result or without propagating the downstream failure.

### Example
```groovy
build job: 'deploy-job', wait: false
```

The upstream build finishes successfully because it only submitted the downstream request.

To wait and propagate:

```groovy
build job: 'deploy-job', wait: true, propagate: true
```

## Q52. SonarQube analysis completes but the pipeline does not fail on a bad gate. Why?

### Clear definition
A scan only submits analysis. The pipeline must explicitly wait for and enforce the Quality Gate result.

### Correct pattern
```groovy
withSonarQubeEnv('sonarqube-server') {
    sh 'mvn sonar:sonar'
}
waitForQualityGate abortPipeline: true
```

Also verify webhook configuration from SonarQube to Jenkins and the correct project/server configuration.

## Q53. Jenkins cannot clone Git repository. What do you check?

### Clear definition
Git checkout failures usually involve URL, credentials, network connectivity, host-key validation, branch/ref configuration, or repository permissions.

### Checklist

```bash
git ls-remote <repository-url>
ssh -T git@github.com
nslookup <git-host>
curl -I https://<git-host>
```

Check the credential type: SSH credentials are not interchangeable with HTTPS username/token credentials.

## Q54. A pipeline is aborted but the process continues on the agent. Why?

### Clear definition
Aborting a Jenkins build requests interruption, but external processes may not terminate immediately if they are detached, ignore signals, run in the background, or are launched outside Jenkins’ process tree.

### Prevention

- Avoid unnecessary background processes.
- Use proper process groups.
- Clean up in `post { always { ... } }`.
- Use timeouts.
- Ensure scripts handle termination signals.
- Inspect orphaned processes on the agent.

---

# 13. L2/L3 Scenario Questions

## Q55. How would you design a production CI/CD pipeline for a Java microservice?

### Clear definition
A production CI/CD pipeline is an automated, auditable, repeatable workflow that validates code and promotes one immutable artifact through environments.

### Design

```text
Pull Request
   ↓
Checkout
   ↓
Compile
   ↓
Unit Tests
   ↓
Checkstyle / PMD / Coverage
   ↓
SonarQube Quality Gate
   ↓
SAST / SCA / Secret Scan
   ↓
Package JAR
   ↓
Publish Artifact
   ↓
Deploy Dev
   ↓
Smoke Tests
   ↓
Approval / Policy
   ↓
Deploy Production
   ↓
Health Check + Notification
```

### L3 considerations

- Separate credentials by environment.
- Restrict production deployment permissions.
- Use immutable versioning.
- Add rollback.
- Store test reports.
- Use shared libraries.
- Add timeout and retry only where justified.
- Record approvals and deployment evidence.
- Monitor pipeline duration and failure rate.

## Q56. How do you prevent two production deployments from running together?

### Clear definition
Deployment concurrency control ensures that only one deployment for a protected environment runs at a time.

### Methods

- Disable concurrent builds.
- Use a lockable resource.
- Use an external deployment system with locking.
- Serialize production stages.
- Apply environment-level policies.

### Example
```groovy
options {
    disableConcurrentBuilds()
}
```

For multiple jobs sharing production, a cross-job lock or deployment controller is more appropriate.

## Q57. How do you handle flaky tests?

### Clear definition
A flaky test is a test that passes and fails intermittently without a relevant code change.

### Approach

1. Identify and measure flaky tests.
2. Capture logs and test timing.
3. Check race conditions and shared state.
4. Remove order dependency.
5. Fix environment instability.
6. Quarantine temporarily with ownership.
7. Do not hide failures permanently using unlimited retries.

Retries can reduce noise temporarily, but the root cause must be fixed.

## Q58. How would you migrate Jenkins to a new server?

### Clear definition
Jenkins migration is the controlled transfer of Jenkins configuration, jobs, credentials, plugins, and required metadata to a new controller.

### Steps

1. Inventory plugins, jobs, agents, credentials, tools, and integrations.
2. Back up `$JENKINS_HOME`.
3. Record Jenkins and plugin versions.
4. Provision the new server.
5. Install the compatible Jenkins version.
6. Restore or recreate configuration.
7. Restore credentials securely.
8. Reconnect agents.
9. Test Git, webhooks, SonarQube, artifact repositories, and notifications.
10. Run representative pipelines.
11. Switch DNS or access endpoint.
12. Keep rollback available.

## Q59. How do you investigate a sudden increase in Jenkins build duration?

### Clear definition
Build-duration investigation identifies which pipeline stage, agent, dependency, external service, or Jenkins component introduced latency.

### Method

1. Compare current and previous build durations.
2. Inspect stage-level timing.
3. Check queue time separately from execution time.
4. Review agent CPU, memory, disk, and network.
5. Check Git clone duration.
6. Check dependency download time.
7. Check SonarQube and security scanner latency.
8. Check artifact repository response.
9. Review recent Jenkins/plugin changes.
10. Compare logs and environment versions.

### Interview point
Do not immediately add more executors. First determine whether the bottleneck is queueing, compute, network, I/O, or an external service.

## Q60. What makes a Jenkins pipeline production-ready?

### Clear definition
A production-ready pipeline is reliable, secure, repeatable, observable, recoverable, and governed.

### Checklist

- Pipeline stored in Git.
- Clear stages and naming.
- Versioned dependencies.
- Secure credentials.
- Least-privilege agents.
- Automated tests.
- Quality and security gates.
- Immutable artifacts.
- Environment approvals.
- Deployment verification.
- Rollback plan.
- Notifications.
- Build retention.
- Workspace cleanup.
- Timeout and failure handling.
- Audit trail.
- Backup and recovery plan.
- Monitoring of Jenkins itself.

---

# 14. Full Forms and Quick Revision

| Term | Full form | Meaning |
|---|---|---|
| CI | Continuous Integration | Frequent integration and validation of code |
| CD | Continuous Delivery / Deployment | Release readiness or automatic release |
| SCM | Source Code Management | System managing source code versions |
| VCS | Version Control System | Tracks file history and collaboration |
| API | Application Programming Interface | Interface used by software systems |
| CLI | Command-Line Interface | Tool operated through terminal commands |
| DSL | Domain-Specific Language | Language designed for a specific domain |
| JCasC | Jenkins Configuration as Code | Jenkins configuration defined as code |
| JVM | Java Virtual Machine | Runs Java bytecode |
| LDAP | Lightweight Directory Access Protocol | Directory authentication/access protocol |
| SSO | Single Sign-On | One login for multiple systems |
| MFA | Multi-Factor Authentication | Authentication using multiple factors |
| RBAC | Role-Based Access Control | Permissions assigned through roles |
| ACL | Access Control List | Explicit permission rules |
| TLS | Transport Layer Security | Encrypts network communication |
| HTTP | Hypertext Transfer Protocol | Web communication protocol |
| HTTPS | HTTP Secure | HTTP protected by TLS |
| REST | Representational State Transfer | Common style for web APIs |
| JSON | JavaScript Object Notation | Data interchange format |
| YAML | YAML Ain’t Markup Language | Human-readable configuration format |
| RPO | Recovery Point Objective | Maximum acceptable data loss |
| RTO | Recovery Time Objective | Maximum acceptable recovery time |
| SLA | Service Level Agreement | Formal service commitment |
| SLO | Service Level Objective | Target service reliability level |
| SLI | Service Level Indicator | Measured service metric |
| MTTR | Mean Time To Recovery/Repair | Average restoration or repair time |
| MTBF | Mean Time Between Failures | Average operating time between failures |
| SAST | Static Application Security Testing | Security analysis of source or bytecode |
| DAST | Dynamic Application Security Testing | Security testing of a running application |
| SCA | Software Composition Analysis | Analysis of third-party dependencies |
| PR | Pull Request | Proposed code change for review |
| SSH | Secure Shell | Secure remote access protocol |
| SMTP | Simple Mail Transfer Protocol | Email transfer protocol |
| DNS | Domain Name System | Resolves names to network addresses |
| NTP | Network Time Protocol | Synchronizes system clocks |

## Final interview revision order

1. Explain Jenkins, controller, agent, node, executor, workspace, and artifact.
2. Explain webhook, polling, cron, upstream, downstream, `wait`, and `propagate`.
3. Write a Declarative Pipeline from memory.
4. Explain credentials and why secrets must not be hardcoded.
5. Explain Git checkout and Multibranch Pipeline.
6. Explain Shared Libraries, JCasC, and Job DSL.
7. Explain SonarQube scan versus Quality Gate enforcement.
8. Explain artifact immutability and rollback.
9. Troubleshoot queue, agent, Git, disk, tool, and downstream failures.
10. Design a secure production CI/CD pipeline with approvals, monitoring, and recovery.
