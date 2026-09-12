# Jenkins Interview Questions — L2/L3 Focused

> One-day revision guide based on the supplied Jenkins notes and selected L2/L3 interview topics.
>
> **Excluded:** Prometheus/Grafana monitoring, AWS IAM, SSM deployments, artifact immutability/promotion, Terraform state locking/approval workflows, Kubernetes dynamic agents, and detailed Shared Libraries.

## How to Use

- **Must Know:** Read first.
- **L2:** Practical questions.
- **L3 Scenario:** Troubleshooting/design.
- Learn the reason and investigation flow; do not memorize every sentence.

---

# 1. Jenkins Fundamentals

### Q1. What is Jenkins? — Must Know

Jenkins is an open-source automation server used to automate build, test, integration, and delivery activities. Jenkins orchestrates tools such as Git, Maven, Gradle, npm, Python, scanners, and deployment commands; it does not compile or test code by itself.

### Q2. CI vs Continuous Delivery vs Continuous Deployment

| Term | Meaning |
|---|---|
| CI | Frequently integrate code and run automated validation |
| Continuous Delivery | Keep software deployable; production release may require approval |
| Continuous Deployment | Automatically deploy every validated change |

### Q3. Job vs Build vs Workspace

| Term | Meaning |
|---|---|
| Job | Configuration defining what Jenkins executes |
| Build | One execution of a job |
| Workspace | Directory where the build runs |
| Build number | Sequence number for a build |
| Console log | Build output |

### Q4. Why Pipeline over Freestyle?

Pipeline is stored as code, can be reviewed in Git, supports stages/conditions/parallelism/retries, is easier to reproduce, and is better for complex workflows.

### Q5. What is a Jenkins Pipeline?

A Pipeline is a CI/CD process defined as code.

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
    }
}
```

---

# 2. Jenkins Architecture

### Q6. Explain controller-agent architecture. — Must Know

```text
Developer -> Git -> Jenkins Controller -> Build Queue -> Jenkins Agents
                                      |              |       |
                                      |              |       +-- Java build
                                      |              +---------- Node build
                                      +------------------------- Test/scan
```

The controller schedules and coordinates work. Agents execute build commands.

### Q7. What does the controller do?

- Schedules builds
- Manages jobs and Pipelines
- Provides UI/API
- Coordinates agents
- Stores configuration and build metadata
- Persists Pipeline state

### Q8. What is an agent?

A machine or runtime where Jenkins executes build steps. Agents may be static, dynamic, or ephemeral.

### Q9. Node vs Agent vs Executor

| Item | Meaning |
|---|---|
| Node | Machine/runtime registered with Jenkins |
| Agent | Build execution environment |
| Executor | Capacity slot that runs one task |

### Q10. Why avoid builds on the controller?

Builds can consume CPU, memory, disk, and network resources; execute untrusted code; and affect Jenkins scheduling/UI. A common design keeps controller executors at zero and runs builds on agents.

### Q11. What is a label?

A label identifies nodes with a capability or purpose.

```groovy
pipeline {
    agent { label 'java' }
    stages {
        stage('Build') {
            steps { sh 'mvn package' }
        }
    }
}
```

### Q12. Why is a build queued even when an agent is online?

- Label does not match
- All matching executors are busy
- Node is disabled
- Job is restricted to another node
- Lock/throttle is active
- Dynamic provisioning is delayed or failing

### Q13. Executor sizing best practice

Start conservatively. One executor per node is often safest. Increase only after checking CPU, memory, disk I/O, and workload behavior. More executors do not automatically mean better performance.

---

# 3. Generic Cloud Agents — Short Revision

### Q14. What is a cloud agent?

An agent provisioned dynamically when Jenkins needs additional capacity.

### Q15. Static vs dynamic cloud agent

| Static | Dynamic |
|---|---|
| Fixed capacity | Elastic capacity |
| May remain idle | Created when needed |
| Manual maintenance | Requires provisioning logic |

### Q16. Cloud agent is provisioned but never connects. What do you check?

Provisioning logs, runtime/JVM, network, connection settings, work-directory permissions, labels, and controller-agent compatibility.

---

# 4. Jobs, Workspaces, and Artifacts

### Q17. What is a workspace?

The directory where Jenkins checks out code and runs commands. It may contain source, reports, temporary files, and build outputs. It is not permanent storage.

### Q18. Why can two builds interfere with each other?

They may use the same workspace and overwrite source, build directories, reports, or temporary files.

```groovy
options {
    disableConcurrentBuilds()
}
```

Other solutions: isolated workspaces, separate agents, and unique temporary directories.

### Q19. `stash` vs `unstash` vs `archiveArtifacts`

| Feature | Purpose |
|---|---|
| `stash` | Temporarily save files inside the same Pipeline |
| `unstash` | Retrieve stashed files |
| `archiveArtifacts` | Store files with the build record |

### Q20. How do you clean a workspace?

```groovy
steps {
    deleteDir()
    checkout scm
}
```

With the Workspace Cleanup plugin:

```groovy
steps {
    cleanWs()
}
```

### Q21. What is build retention?

Removing old builds and logs according to configured rules.

```groovy
options {
    buildDiscarder(logRotator(numToKeepStr: '10'))
}
```

---

# 5. Jenkins Job Configuration and Build Triggers

## Q. What are the main components of a Jenkins job?

A Jenkins job commonly contains:

1. **General** — description, parameters, log rotation, concurrency settings.
2. **Source Code Management** — Git repository, branch, credentials, checkout behavior.
3. **Build Triggers** — when the job should start.
4. **Build Environment** — tools, environment variables, cleanup, timestamps.
5. **Build Steps / Pipeline** — commands or Jenkinsfile execution.
6. **Post-build Actions** — archive reports, publish test results, notify users, trigger another job.
7. **Permissions** — who can view, build, configure, or administer the job.

## Q. Explain Jenkins build triggers.

| Trigger | Use case | Important point |
|---|---|---|
| Manual | Developer or operator starts a build | Useful for controlled releases or troubleshooting |
| Webhook | Git provider notifies Jenkins after a push or PR event | Usually better than frequent polling |
| Poll SCM | Jenkins checks Git periodically | Consumes resources and may delay detection |
| Build periodically | Run on a schedule regardless of code changes | Uses cron syntax |
| Upstream project | Start after another job completes | Useful for chained workflows |
| Remote/API trigger | Start using an authenticated HTTP/API request | Protect the endpoint |
| Parameterized trigger | Pass values from one job to another | Validate parameters before use |

### Q22. Example cron expressions

```text
H/15 * * * *     Run approximately every 15 minutes
H 2 * * *        Run once daily around 2 AM
H H * * 1-5      Run once on weekdays
```

`H` spreads jobs across different times to avoid a thundering herd at exactly the same minute.

## Q. How do you prevent duplicate builds?

- Enable **Disable concurrent builds** when only one build should run at a time.
- Use a queue/throttling strategy for shared environments.
- Avoid configuring both webhook and aggressive polling without understanding the behavior.
- Use a unique build or deployment lock for shared target environments.
- Cancel obsolete builds for fast-moving branches where appropriate.

---

# 6. Pipeline Syntax — Must Know

### Q23. Main Declarative blocks

| Block | Purpose |
|---|---|
| `pipeline` | Root block |
| `agent` | Execution location |
| `stages` | Collection of stages |
| `stage` | Logical phase |
| `steps` | Commands |
| `post` | Actions after execution |
| `environment` | Environment variables |
| `options` | Pipeline behavior |
| `parameters` | User inputs |
| `when` | Conditional execution |
| `triggers` | Automatic triggers |
| `tools` | Tool configuration |

### Q24. Stage vs step

A stage is a logical phase such as Build or Test. A step is an executable operation such as `sh`, `echo`, or `checkout`.

### Q25. What is `agent none`?

It avoids allocating one global agent and lets individual stages choose their own agents.

```groovy
pipeline {
    agent none
    stages {
        stage('Build') {
            agent { label 'java' }
            steps { sh 'mvn package' }
        }
        stage('Test') {
            agent { label 'test' }
            steps { sh 'mvn test' }
        }
    }
}
```

### Q26. What is `post`?

It runs actions after execution. Common conditions are `always`, `success`, `failure`, `unstable`, `aborted`, `changed`, and `cleanup`.

### Q27. What is `when`?

It controls whether a stage runs.

```groovy
stage('Deploy') {
    when { branch 'main' }
    steps { sh './deploy.sh' }
}
```

### Q28. Timeout and retry

```groovy
options {
    timeout(time: 30, unit: 'MINUTES')
}
```

```groovy
steps {
    retry(3) {
        sh './temporary-command.sh'
    }
}
```

Retry only operations safe to repeat.

### Q29. `error` vs `catchError` vs `returnStatus`

| Feature | Behavior |
|---|---|
| `error` | Explicitly fails Pipeline |
| `catchError` | Catches failure and controls result |
| `returnStatus` | Returns exit code instead of failing automatically |

### Q30. Parallel stages

```groovy
stage('Parallel Tests') {
    parallel {
        stage('Unit Tests') {
            steps { sh './run-unit-tests.sh' }
        }
        stage('Lint') {
            steps { sh './run-lint.sh' }
        }
    }
}
```

---

# 7. Declarative vs Scripted Pipeline

### Q31. Declarative vs Scripted

| Feature | Declarative | Scripted |
|---|---|---|
| Structure | Defined syntax | Groovy-driven |
| Validation | Stronger early validation | More runtime-oriented |
| Readability | Usually easier | Depends on code |
| Flexibility | Moderate | High |
| Best use | Standard CI/CD | Complex custom logic |

### Q32. What is `script {}`?

It permits Scripted Pipeline/Groovy logic inside Declarative Pipeline.

```groovy
steps {
    script {
        def status = sh(script: './check.sh', returnStatus: true)
        if (status == 0) {
            echo 'Passed'
        }
    }
}
```

---

# 8. Pipeline Execution Internals

### Q33. What happens when a Pipeline starts?

```text
Trigger -> Schedule -> Load Jenkinsfile -> Allocate Agent
       -> Execute Stages/Steps -> Persist Results -> post actions
```

### Q34. What is CPS?

Jenkins uses continuation-passing style transformation so Pipeline execution can pause and resume across steps, input, agent allocation, and supported restart situations.

### Q35. What is `@NonCPS`?

It prevents CPS transformation for a method. Use it only for suitable pure Groovy data processing.

Incorrect:

```groovy
@NonCPS
def invalidMethod() {
    sh 'echo Hello'
}
```

Pipeline steps must not be called from an `@NonCPS` method.

### Q36. Why can a Pipeline resume after restart?

Jenkins persists Pipeline execution state. However, external processes may not resume, agents may be lost, and non-durable operations may behave differently.

### Q37. Why avoid heavy computation in Pipeline Groovy?

It can create controller memory pressure, serialization overhead, slow execution, and CPS problems. Move heavy work to scripts or application code.

---

# 9. Parameters, Environment, and Credentials

### Q38. How do you define parameters?

```groovy
parameters {
    choice(name: 'ENVIRONMENT', choices: ['dev', 'qa', 'prod'])
    booleanParam(name: 'RUN_TESTS', defaultValue: true)
}
```

Use `params.ENVIRONMENT`.

### Q39. `params` vs `env`

| `params` | `env` |
|---|---|
| Build parameters | Environment variables |
| Usually user input | Runtime/configuration values |
| `params.NAME` | `env.NAME` |

### Q40. How do you define environment variables?

```groovy
environment {
    APP_NAME = 'otms'
    LOG_LEVEL = 'INFO'
}
```

### Q41. How should credentials be used?

Store them in Jenkins Credentials and reference them by ID.

```groovy
withCredentials([
    usernamePassword(
        credentialsId: 'repo-creds',
        usernameVariable: 'USER',
        passwordVariable: 'PASS'
    )
]) {
    sh '''
        set +x
        ./login.sh "$USER" "$PASS"
    '''
}
```

Never hardcode passwords, tokens, or private keys.

### Q42. Is secret masking complete security?

No. Secrets can leak through debug output, shell quoting, process arguments, workspace files, or malicious build scripts. Use least privilege.

---

# 10. Git, Webhooks, and Multibranch

### Q43. Poll SCM vs webhook

| Poll SCM | Webhook |
|---|---|
| Jenkins checks periodically | Git provider sends an event |
| More repeated traffic | Event-driven |
| May detect changes late | Usually faster |

### Q44. Webhook flow

```text
Developer Push -> Git Provider -> Webhook -> Jenkins Trigger -> Job/Branch -> Pipeline
```

### Q45. Webhook received but build does not start. What do you check?

Webhook delivery, endpoint URL, trigger configuration, repository/branch matching, credentials/token, multibranch indexing, permissions, event filters, and Jenkins logs.

### Q46. What is a Multibranch Pipeline?

It discovers branches containing a Jenkinsfile and creates a Pipeline job for each discovered branch.

### Q47. What is branch indexing?

Scanning a repository for new/deleted branches, changed Jenkinsfiles, pull requests, and branch metadata.

### Q48. Why can a webhook trigger duplicate builds?

Webhook plus polling, multiple endpoints, duplicate triggers, branch and pull-request events both matching, or Git-provider retries.

---

# 11. Plugins

### Q49. Why does Jenkins need plugins?

Plugins provide SCM integrations, Pipeline steps, credential types, agent integrations, test reporting, security features, and UI features.

### Q50. What causes `No such DSL method`?

- Plugin missing or failed to load
- Incorrect step name
- Plugin/version mismatch
- Missing dependency
- Incorrect Pipeline syntax
- Missing custom functionality

### Q51. How do you troubleshoot a plugin failure?

Identify the failing step, check installed/versioned plugins, inspect dependency warnings and startup logs, reproduce with a minimal Pipeline, and review recent upgrades.

---

# 12. JCasC and Job DSL — Important Only

### Q52. What is JCasC?

Jenkins Configuration as Code manages Jenkins system configuration using YAML.

### Q53. What is Job DSL?

Job DSL creates and manages Jenkins jobs using Groovy definitions.

### Q54. JCasC vs Job DSL

| JCasC | Job DSL |
|---|---|
| Jenkins system configuration | Job creation/configuration |
| Usually YAML | Groovy DSL |
| System-level | Job-level |

---

# 13. Jenkins Security

### Q55. Authentication vs authorization

| Authentication | Authorization |
|---|---|
| Who are you? | What can you do? |

### Q56. What is least privilege?

Give users, jobs, agents, and credentials only the permissions required. Avoid unnecessary administrator access and restrict production credentials.

### Q57. What is the Groovy sandbox?

It restricts potentially dangerous Groovy operations. Some operations require administrator approval.

### Q58. Why are agents a security boundary?

Build scripts may access workspace files, environment variables, exposed credentials, and network resources. A compromised agent should not compromise the controller.

---

# 14. Advanced Administration and Pipeline Topics

## Jenkins Agent Connectivity: JNLP vs SSM

### Q82. What is JNLP in Jenkins?

JNLP is the older term for a Jenkins **inbound agent** connection. The agent initiates the connection to the controller, which is useful behind NAT, firewalls, or in private networks.

```text
Jenkins Agent  ──initiates connection──>  Jenkins Controller
```

### Q83. Is JNLP required when using AWS SSM?

**No.** SSM executes commands on EC2; JNLP connects a Jenkins build agent to the controller.

```text
Jenkins → SSM → EC2 → Deployment
```

If Jenkins only deploys through SSM, no Jenkins agent or JNLP is required. JNLP is needed only if the EC2 instance runs Jenkins build steps as an agent.

### Q84. Can SSM and JNLP be used together?

Yes. JNLP may run builds on one EC2 instance while SSM deploys to another. They are independent mechanisms.

## Jenkins Security Lockout

### Q85. What causes a Jenkins security lockout?

Incorrect LDAP/AD, SAML/OIDC, Security Realm, matrix authorization, role mapping, or administrator permissions can prevent login or leave the user without administration rights.

### Q86. How do you recover Jenkins after security misconfiguration?

1. Back up the configuration:

```bash
sudo cp /var/lib/jenkins/config.xml /var/lib/jenkins/config.xml.backup
```

2. Stop Jenkins:

```bash
sudo systemctl stop jenkins
```

3. Edit `/var/lib/jenkins/config.xml` and temporarily change:

```xml
<useSecurity>true</useSecurity>
```

to:

```xml
<useSecurity>false</useSecurity>
```

4. Start Jenkins, repair the Security Realm and Authorization Strategy, re-enable security, and restart Jenkins.

Do not delete the complete `securityRealm` section unless it is specifically required. Identify whether the issue is authentication or authorization first.

## LDAP Authentication

### Q87. What is LDAP?

LDAP means **Lightweight Directory Access Protocol**. It is used to access directory data such as users, groups, departments, email addresses, and organizational units.

### Q88. How does LDAP work with Jenkins?

```text
User → Jenkins → LDAP verifies identity → Jenkins authorization grants permissions
```

LDAP authenticates the user. Jenkins decides whether the user can read, build, configure, or administer jobs.

### Q89. LDAP vs Active Directory

| LDAP | Active Directory |
|---|---|
| Protocol | Microsoft's directory service |
| Defines access to directory data | Stores users, groups, computers, and policies |
| Used by many directory systems | Supports LDAP and other protocols |

### Q90. Important LDAP terms

| Term | Example |
|---|---|
| Base DN | `dc=company,dc=com` |
| User DN | `uid=mukesh,ou=users,dc=company,dc=com` |
| OU | `ou=developers` |
| Group | `cn=jenkins-admins,ou=groups,dc=company,dc=com` |

Common LDAP failures include wrong URL, Base DN, search filter, bind credentials, DNS, TLS certificate, username attribute, group mapping, or system time.

## Jenkins SSO

### Q91. What is SSO?

SSO, or **Single Sign-On**, allows users to authenticate through a central Identity Provider such as Entra ID, Okta, Keycloak, or Google Workspace.

```text
User → Jenkins → Identity Provider → SAML response/OIDC token → Jenkins authorization
```

### Q92. General SAML/OIDC configuration steps

1. Install an approved SAML or OIDC Jenkins plugin.
2. Register Jenkins as an application in the Identity Provider.
3. Configure the exact Entity ID and ACS/Reply or Redirect URL.
4. Configure the issuer/SSO URL, certificate or client ID/secret.
5. Map username, email, and group claims.
6. Map IdP groups to Jenkins roles.
7. Test with a separate user and keep a recovery administrator.

### Q93. Most common SSO problem

The Jenkins callback URL and IdP redirect/Reply URL do not match exactly. Check protocol, hostname, port, path, trailing slash, issuer, certificate, audience, group claims, reverse-proxy headers, and system time.

**SSO authenticates; Jenkins authorization decides permissions.** SSO does not automatically make a user an administrator.

## Jenkins Backup Strategies

### Q94. What should be backed up?

The main directory is:

```bash
$JENKINS_HOME

`JENKINS_HOME` is commonly `/var/lib/jenkins` on Linux installations.
```

Back up `config.xml`, `jobs/`, `credentials.xml`, `secrets/`, `plugins/`, `users/`, `nodes/`, `builds/`, `fingerprints/`, and `userContent/`.

**Important:** Restore `credentials.xml` together with `secrets/`; otherwise Jenkins may not decrypt credentials.

### Q95. Main backup strategies

1. **File-system backup** — tar the Jenkins home directory.
2. **Thin Backup plugin** — useful for Jenkins configuration.
3. **Disk/volume snapshots** — useful for full-server recovery; ensure consistency.
4. **External storage** — copy backups to S3, Blob Storage, NFS, or enterprise backup storage.
5. **Configuration as Code** — store JCasC, Jenkinsfiles, plugin lists, Job DSL, and scripts in Git.

Example:

```bash
sudo systemctl stop jenkins
sudo tar -czf /backup/jenkins-$(date +%F).tar.gz -C /var/lib jenkins
sudo systemctl start jenkins
```

Workspaces usually do not need backup because they are reproducible. Use encrypted off-site backups and the 3-2-1 rule: three copies, two storage types, one off-site copy.

### Q96. Jenkins restore process

1. Provision a compatible Jenkins server.
2. Stop Jenkins.
3. Restore `$JENKINS_HOME`, including `credentials.xml` and `secrets/`.
4. Restore compatible plugins.
5. Fix ownership:

```bash
sudo chown -R jenkins:jenkins /var/lib/jenkins
```

6. Start Jenkins and validate jobs, credentials, agents, plugins, webhooks, and pipelines.

### Q97. RPO vs RTO

| Term | Meaning |
|---|---|
| RPO | Maximum acceptable data loss |
| RTO | Maximum acceptable recovery time |

## Scripted Pipeline vs Jenkinsfile vs Declarative Pipeline

### Q98. Is Scripted Pipeline the same as a Jenkinsfile?

No. A **Jenkinsfile is the file** containing Pipeline code. Its contents can be Declarative or Scripted.

### Q99. Scripted Pipeline

Scripted Pipeline is a Groovy-based, programmatic DSL:

```groovy
node('linux') {
    stage('Build') {
        sh 'mvn clean package'
    }

    if (env.BRANCH_NAME == 'main') {
        stage('Deploy') {
            sh './deploy.sh'
        }
    }
}
```

It supports flexible loops, functions, closures, dynamic stages, complex branching, and `try/catch` handling.

### Q100. Declarative Pipeline

Declarative Pipeline is a structured DSL:

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh './deploy.sh'
            }
        }
    }
}
```

### Q101. Technical comparison

| Area | Scripted | Declarative |
|---|---|---|
| Top-level form | Usually `node {}` | `pipeline {}` |
| Model | Programmatic Groovy | Structured DSL |
| Flexibility | Very high | Intentionally restricted |
| Validation | More runtime-oriented | Pipeline model validation |
| Dynamic stages | Natural | Less natural |
| Error handling | `try/catch/finally` | `post`, `retry`, `catchError`, or `script` |
| Agent allocation | `node()` | `agent` |
| Standard directives | Manually composed | `environment`, `options`, `parameters`, `when`, `post` |
| Best use | Dynamic orchestration | Standard CI/CD |

### Q102. Why is `script {}` used?

Declarative restricts arbitrary Groovy inside `steps`. Use `script {}` for imperative logic:

```groovy
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                script {
                    def value = 10
                    if (value > 5) {
                        sh 'echo greater'
                    }
                }
            }
        }
    }
}
```

`script {}` does not allocate an agent. Excessive use reduces Declarative readability and validation benefits.

### Q103. CPS and `@NonCPS`

Both pipeline styles use the Jenkins Pipeline CPS execution engine, which allows pausing and resuming at steps such as `input`. Avoid carrying non-serializable objects across suspension points.

Use `@NonCPS` only for pure Groovy methods that do not call Pipeline steps:

```groovy
@NonCPS
def calculateTotal(numbers) {
    return numbers.sum()
}
```

### Q104. Which style should be preferred?

Use Declarative by default for standard CI/CD, governance, and maintainability. Use Scripted when the workflow is highly dynamic or requires complex runtime orchestration.

---

---

# 15. SonarQube and Quality Gates

## Q. What is the difference between SonarQube and SonarScanner?

- **SonarQube** is the server that stores and analyzes code-quality results and displays dashboards.
- **SonarScanner** is the client-side analysis tool that scans source code and sends the report to SonarQube.
- Jenkins orchestrates the scan; SonarQube evaluates the result against the configured Quality Gate.

## Q. What is a Quality Profile?

A **Quality Profile** is the set of rules used during analysis. It defines which bugs, vulnerabilities, code smells, and language-specific issues are checked.

## Q. What is a Quality Gate?

A **Quality Gate** is the pass/fail policy applied to analysis results. Typical conditions include:

- New bugs or vulnerabilities must be zero.
- New code coverage must be above a threshold.
- Duplicated lines must remain below a threshold.
- Reliability, security, and maintainability ratings must meet the policy.

A Quality Profile defines **what is checked**. A Quality Gate defines **whether the result is acceptable**.

## Q. Typical Jenkins SonarQube flow

```text
Checkout
   ↓
Build and unit tests
   ↓
Generate coverage report
   ↓
SonarScanner analysis
   ↓
SonarQube processes the report
   ↓
Jenkins waits for Quality Gate
   ↓
Continue or fail the pipeline
```

### Q59. Declarative Pipeline example

```groovy
stage('SonarQube Analysis') {
    steps {
        withSonarQubeEnv('sonarqube-server') {
            sh 'mvn clean verify sonar:sonar'
        }
    }
}

stage('Quality Gate') {
    steps {
        timeout(time: 10, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
        }
    }
}
```

## Q. Why is the SonarQube webhook required?

`waitForQualityGate` does not continuously perform expensive polling. SonarQube sends the completed analysis status to Jenkins through a webhook. The webhook must point to the Jenkins SonarQube webhook endpoint, commonly:

```text
https://<jenkins-url>/sonarqube-webhook/
```

Common failures:

- Webhook URL is incorrect.
- Jenkins is not reachable from SonarQube.
- The SonarQube server name in `withSonarQubeEnv` does not match Jenkins configuration.
- The scanner runs but no report task is created.
- The pipeline waits forever because the webhook or task status is broken.

## Q. Should a failed Quality Gate fail the build?

For production-quality CI, normally yes. Use:

```groovy
waitForQualityGate abortPipeline: true
```

For an advisory phase, the result may be recorded without blocking the pipeline, but this should be an explicit policy rather than an accidental configuration.

---

# 16. Generic CI Pipeline Design

## Q. Design a standard CI pipeline for a microservices repository.

A practical sequence is:

```text
Checkout source
   ↓
Validate branch / parameters
   ↓
Compile or install dependencies
   ↓
Unit tests
   ↓
Coverage report
   ↓
Static analysis / SonarQube
   ↓
Dependency and vulnerability scan
   ↓
Credential/secrets scan
   ↓
Package or build image
   ↓
Optional DAST / ZAP scan
   ↓
Publish reports and artifacts
   ↓
Notify result
```

The exact tools depend on the language:

| Application | Build/test tools | Common checks |
|---|---|---|
| Java | Maven or Gradle, JUnit | Checkstyle, PMD, JaCoCo, SonarQube |
| React/Node.js | npm/pnpm/yarn | ESLint, Jest, coverage, npm audit/SCA |
| Python | pytest, pip/poetry | pylint, flake8, bandit, coverage |
| Go | go test, go build | go vet, golangci-lint, coverage |

## Q. Where should security scans be placed?

| Scan | Stage | Purpose |
|---|---|---|
| GitLeaks or secret scan | Early, before build | Detect committed credentials and tokens |
| SCA/dependency scan | After dependency resolution | Detect vulnerable third-party packages |
| SAST/SonarQube | After source checkout/build metadata | Detect code-quality and source-level issues |
| Container scan | After image build | Detect vulnerable packages in the image |
| DAST/ZAP | Against a running test environment | Detect runtime web vulnerabilities |
| License scan | During dependency validation | Detect policy violations in licenses |

Do not treat one scan as a replacement for all others. They inspect different layers.

## Q. What should happen when a security scan fails?

Define policy explicitly:

- Block on critical/high vulnerabilities when required.
- Allow warnings for low-risk findings if the organization permits it.
- Publish the report even when the build fails.
- Record the dependency, source revision, scanner version, and policy used.
- Provide an exception process with owner and expiry date rather than permanently ignoring findings.

---

# 17. Jenkins Post-build Actions and Notifications

## Q. What are common post-build actions?

- Publish JUnit test results.
- Publish coverage reports.
- Archive build artifacts.
- Publish static-analysis reports.
- Send email, Slack, Teams, or webhook notifications.
- Trigger downstream jobs.
- Clean the workspace.
- Update commit status in GitHub/GitLab/Bitbucket.

### Q60. Example

```groovy
post {
    always {
        junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
        archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
    }
    success {
        echo 'Build succeeded'
    }
    failure {
        echo 'Build failed'
    }
    cleanup {
        cleanWs()
    }
}
```

## Q. Why publish reports even when the build fails?

A failed test or scan often produces the most useful diagnostic evidence. Publishing reports in `always` allows developers and interviewers to inspect the failure instead of losing the evidence during cleanup.

---

# 18. Performance and Scaling

### Q61. Why does Jenkins become slow?

Builds on the controller, too many executors, large logs, heavy Pipeline Groovy, large build history, disk pressure, plugin overhead, too many concurrent jobs, slow SCM, or insufficient agents.

### Q62. How do you reduce controller load?

Run builds on agents, keep controller executors at zero where practical, move heavy logic to scripts, reduce logs, retain fewer builds, avoid excessive polling, and remove unused plugins.

### Q63. What is the danger of too many executors?

CPU contention, memory exhaustion, disk I/O contention, slower builds, and agent instability.

### Q64. How do you investigate a growing queue?

Read the queue reason, check labels, agent status, executors, locks/throttling, provisioning, input steps, and controller resource pressure.

---

# 19. Troubleshooting Scenarios

### Q65. Agent is offline. What do you check?

Agent logs, runtime/JVM, network, work directory, disk space, permissions, connection settings, and controller-agent compatibility.

### Q66. Agent is online but build does not run.

Check label mismatch, busy executors, node restrictions, disabled node, throttling, locks, input steps, and `when` conditions.

### Q67. Build works manually but fails in Jenkins.

Compare user account, PATH, HOME, tool versions, permissions, working directory, credentials, environment variables, network, and non-interactive shell behavior.

### Q68. Git checkout fails.

Check repository URL, branch, credentials, network/DNS, host-key verification, Git installation, workspace permissions, and repository access.

### Q69. Shell command fails with exit code 1.

Inspect the actual command and environment:

```groovy
sh '''
    set -eux
    whoami
    pwd
    env | sort
    ./build.sh
'''
```

Do not blindly ignore the exit code.

### Q70. Pipeline succeeds although a command failed.

Possible causes: `|| true`, `returnStatus`, `catchError`, ignored script errors, a successful final command, or a changed build result.

### Q71. Pipeline is waiting for a long time.

Check `input`, queue, agent allocation, external commands, network calls, locks, retries, sleeps, plugin steps, and dead processes. Use timeouts.

### Q72. Two builds corrupt each other.

Use `disableConcurrentBuilds()`, isolated workspaces, unique temporary directories, separate agents, and unique filenames.

### Q73. Plugin upgrade caused failures.

Identify the first failing step, review upgrade history/dependencies, check Jenkins core and Java compatibility, reproduce minimally, and roll back carefully if necessary.

---

# 20. L2/L3 Scenarios

### Q74. 100 builds are queued but agents are mostly idle. Explain.

Likely causes are incorrect labels, disabled nodes, reserved executors, locks/throttling, failed dynamic provisioning, or jobs requiring a special agent. Start with the queue reason.

### Q75. One agent works and another fails. What do you compare?

OS/architecture, Java/runtime, tools, PATH, user permissions, disk, workspace ownership, network/DNS, credentials, and labels.

### Q76. Controller CPU is high during builds. What do you do?

Confirm builds are not on the controller; inspect Pipeline Groovy loops, large logs, concurrency, plugins, recent changes, and build retention. Move heavy work to agents/scripts.

### Q77. Jenkinsfile is very large. What do you recommend?

Remove repetition, keep the Jenkinsfile readable, use the existing Shared Library solution for reusable logic, and move business logic to tested scripts/application code.

### Q78. Webhook works for one branch but not another.

Check branch indexing, Jenkinsfile existence, branch naming, multibranch configuration, event type, filters, permissions, and Pipeline syntax.

### Q79. Build fails after Jenkins restart.

Investigate Pipeline resumability, agent loss, external processes, plugin errors, persisted state, workspace availability, and credentials/network state.

### Q80. How would you design Jenkins for many teams?

Use folders, consistent permissions/naming, agents and labels, Pipeline-as-code, controlled plugin versions, configuration as code, build retention, workload limits, and tested backups.

### Q81. Pipeline is slow but not failed. How do you investigate?

Break time into queue wait, agent provisioning, checkout, dependency downloads, build, tests, archive/report steps, and post actions. Optimize the slowest stage.

---

# 21. Additional Troubleshooting Scenarios

## Scenario 1: Job does not start after a Git push

Investigation order:

1. Confirm the webhook was delivered by the Git provider.
2. Check the Jenkins endpoint and reverse proxy.
3. Confirm the job trigger is enabled.
4. Verify repository URL and credentials.
5. Check branch filters and PR rules.
6. Inspect Jenkins system log and job event logs.
7. Check whether the job is disabled or blocked by queue/concurrency rules.

## Scenario 2: Build is stuck in the queue

Check:

- No executor is available.
- The required label has no matching online agent.
- An agent is offline or has insufficient executors.
- A throttle or lock is blocking the job.
- The job is waiting for an input/approval step.
- The controller is overloaded.

## Scenario 3: SonarQube stage waits indefinitely

Check:

1. SonarScanner completed successfully.
2. A Sonar analysis task ID was generated.
3. `withSonarQubeEnv` references the correct configured server.
4. SonarQube can reach Jenkins.
5. The `/sonarqube-webhook/` endpoint is correct.
6. The Quality Gate task completed in SonarQube.
7. Jenkins logs show webhook reception or authentication errors.

## Scenario 4: Credentials are printed in console output

Immediate actions:

- Revoke or rotate the exposed credential.
- Remove it from logs and source code.
- Check whether it was stored in Git history or artifacts.
- Use Jenkins Credentials Binding or `withCredentials`.
- Avoid `echo` of secrets and unsafe shell interpolation.
- Review access to the build logs.

---

# 22. High-value Interview Traps

1. **Jenkins is not the build tool.** It orchestrates tools.
2. **A Jenkinsfile is not a pipeline syntax.** It is the file containing the pipeline; the syntax may be Declarative or Scripted.
3. **Webhook is not the same as polling.** Webhook is event-driven; polling is scheduled checking.
4. **SonarQube is not SonarScanner.** One is the analysis server; the other sends analysis results.
5. **Quality Profile is not Quality Gate.** Rules versus pass/fail policy.
6. **An online agent is not necessarily available for every job.** Labels, executors, permissions, and tool requirements still matter.
7. **`post { always }` does not mean the build succeeds.** It means the actions run regardless of result.
8. **`cleanWs()` removes workspace files, not archived artifacts or external reports.**
9. **`@NonCPS` is not a performance switch.** It is for non-Pipeline Groovy logic that should not call Pipeline steps.
10. **Storing a secret in a Jenkinsfile is not safe just because the repository is private.** Use Jenkins Credentials.
11. **A successful shell command does not prove application correctness.** Tests, quality gates, security checks, and deployment verification are separate concerns.
12. **A failed downstream job must be handled intentionally.** Decide whether to fail, retry, mark unstable, or continue.

---

# 23. Quick Revision Tables

## Controller vs Agent

| Controller | Agent |
|---|---|
| Orchestrates | Executes |
| Schedules | Runs commands |
| Hosts UI/API | Provides build environment |
| Stores metadata | Uses workspace |

## Workspace vs Artifact

| Workspace | Artifact |
|---|---|
| Temporary build directory | Retained build output |
| Used during execution | Used after build |
| Can be cleaned | Has retention/access requirements |

## Retry vs Timeout

| Retry | Timeout |
|---|---|
| Repeats operation | Limits duration |
| For transient failures | Prevents hanging |
| Must be safe to repeat | Surrounds risky waits |

---

# 24. One-Day Revision Order

## First Priority

1. Controller vs agent
2. Node vs executor
3. Queue and labels
4. Workspace
5. Pipeline structure
6. Declarative vs Scripted
7. `agent`, `stage`, `steps`, `post`
8. Parameters and credentials
9. Webhook flow
10. Troubleshooting scenarios

## Second Priority

1. CPS and `@NonCPS`
2. Restart/resume
3. Parallel stages
4. Multibranch Pipeline
5. Plugin failures
6. JCasC vs Job DSL
7. Backup and restore
8. Performance and scaling

---

## Official Reading — Only If Needed

- [Jenkins User Handbook](https://www.jenkins.io/doc/book/)
- [Pipeline](https://www.jenkins.io/doc/book/pipeline/)
- [Using Jenkins agents](https://www.jenkins.io/doc/book/using/using-agents/)
- [Architecting for Scale](https://www.jenkins.io/doc/book/scaling/architecting-for-scale/)
- [Backing up Jenkins](https://www.jenkins.io/doc/book/system-administration/backing-up/)

---

# 25. Final One-Day Revision Checklist

- [ ] Jenkins architecture: controller, agent, executor, workspace.
- [ ] Job, build, workspace, artifact, node, label.
- [ ] Freestyle vs Pipeline vs Multibranch Pipeline.
- [ ] Declarative vs Scripted Pipeline and Jenkinsfile.
- [ ] CPS, `script {}`, `@NonCPS`, serialization.
- [ ] Pipeline stages, `post`, `when`, `input`, `timeout`, `retry`, `parallel`.
- [ ] Git checkout, credentials, webhook, polling, cron, upstream triggers.
- [ ] Agent labels, executors, offline agents, JNLP/inbound concepts.
- [ ] Credentials, masking, rotation, least privilege, secret leakage.
- [ ] LDAP, SSO, authorization strategy, break-glass recovery.
- [ ] Backup and restore of `JENKINS_HOME`.
- [ ] SonarQube, SonarScanner, Quality Profile, Quality Gate, webhook.
- [ ] Unit tests, coverage, static analysis, SCA, secret scan, DAST.
- [ ] Test reports, artifacts, notifications, cleanup.
- [ ] Queue troubleshooting, failed builds, stuck stages, missing tools.
- [ ] Explain one complete CI pipeline from Git push to notification.

## Best final interview answer pattern

For any Jenkins troubleshooting or design question, answer in this order:

1. **State the concept.**
2. **Explain the expected flow.**
3. **Name the likely failure points.**
4. **Give the commands/logs/configuration you would inspect.**
5. **Explain the corrective action.**
6. **Mention security, reliability, and maintainability considerations.**

This demonstrates practical L2/L3 thinking instead of only memorized definitions.

---

# 24. Concept Clarity Reference — Read Before Interview Questions

This section gives the **meaning, purpose, and relationship** of the terms that are commonly confused in Jenkins interviews.

## 24.1 The Jenkins mental model

```text
Developer pushes code
        |
        v
Git repository / Pull Request
        |
        v
Webhook or polling triggers Jenkins
        |
        v
Jenkins Controller
  - reads job/Pipeline configuration
  - creates a build
  - places work in the queue
  - selects a suitable node/agent
        |
        v
Jenkins Agent
  - receives the work
  - uses an executor
  - creates/uses a workspace
  - runs shell commands and tools
        |
        v
Build result
  - logs
  - test reports
  - artifacts
  - notifications
  - downstream jobs
```

**Important:** Jenkins is the orchestrator. Maven, Gradle, npm, Python, Go, scanners, and deployment tools perform the actual application work.

## 24.2 Core terms in one table

| Term | Clear definition | Example / relationship |
|---|---|---|
| **Controller** | Central Jenkins service that manages configuration, scheduling, UI, plugins, queues, and Pipeline coordination. | Decides where a build should run. |
| **Node** | A machine or runtime registered with Jenkins. | A VM, physical server, or container runtime. |
| **Agent** | The Jenkins process/environment that executes work on a node. | Runs `mvn test` or `npm build`. |
| **Executor** | A concurrency slot on a node. | 2 executors can run up to 2 tasks concurrently, if resources permit. |
| **Label** | A capability or grouping name assigned to nodes. | `linux`, `java`, `docker`, `prod-deploy`. |
| **Job / Project** | A configured unit of work in Jenkins. | Freestyle job or Pipeline job. |
| **Build** | One execution of a job. | `my-app #125` is build number 125. |
| **Workspace** | Working directory used during a build. | Source checkout, temporary files, reports. |
| **Artifact** | Output produced by a build and retained for later use. | `.jar`, `.war`, package, report, ZIP. |
| **Pipeline** | The complete CI/CD workflow defined as code. | Build → Test → Scan → Package → Deploy. |
| **Stage** | A logical phase of a Pipeline. | `Build`, `Test`, `Deploy`. |
| **Step** | An individual action inside a stage. | `sh 'mvn test'`, `junit`, `archiveArtifacts`. |
| **Plugin** | Extension that adds Jenkins functionality or integrations. | Git, Pipeline, credentials, SonarQube plugins. |
| **Folder** | Organizational container for jobs and other folders. | `team-a/backend/service-1`. |
| **Item** | General Jenkins UI object such as a job, Pipeline, or folder. | A folder and a Pipeline are both items. |

### Easy relationship to remember

```text
Controller
  └── Node
       └── Agent process
            └── Executor slot
                 └── Build
                      └── Workspace
                           └── Steps inside stages
```

A **node is the machine**, an **agent is the Jenkins execution process/environment**, and an **executor is the slot that runs one task**.

## 24.3 Job vs Pipeline vs Build vs Stage vs Step

- **Job:** The configured definition of work.
- **Pipeline:** A job type/workflow that describes the complete delivery process as code.
- **Build:** One run of that job or Pipeline.
- **Stage:** A named logical phase within that Pipeline.
- **Step:** The actual command or Jenkins operation executed in a stage.

Example:

```text
Job: employee-api-ci
  Build #42
    Pipeline
      Stage: Build
        Step: mvn clean package
      Stage: Test
        Step: mvn test
      Stage: Scan
        Step: SonarScanner
```

## 24.4 Workspace vs artifact

| Workspace | Artifact |
|---|---|
| Temporary working area | Build output retained intentionally |
| Used while executing the build | Used after the build or by another team/job |
| Can be cleaned or deleted | Stored through artifact storage or another repository |
| Contains source, caches, temporary files | Contains deliverables such as JAR, WAR, ZIP, reports |

**Interview answer:** Never treat a workspace as permanent artifact storage.

---

# 25. Upstream and Downstream Jobs — Must Know

## 25.1 What is an upstream job?

An **upstream job** is a job that runs earlier in a job chain and can trigger another job after it finishes.

Example:

```text
Build Job  ───────►  Test Job  ───────►  Deploy Job
 upstream            downstream          downstream
```

In this example, **Build Job is upstream of Test Job**. The same Build Job is also upstream of Deploy Job if it directly triggers Deploy Job.

## 25.2 What is a downstream job?

A **downstream job** is a job triggered by another job and normally depends on the upstream job’s result or output.

Example:

- `app-build` compiles the application and creates a JAR.
- `app-test` is triggered after `app-build` succeeds.
- `app-deploy` is triggered after testing succeeds.

Here, `app-test` is downstream of `app-build`, and `app-deploy` is downstream of `app-test`.

## 25.3 How are upstream and downstream jobs configured?

### Freestyle job

- Upstream job: configure **Post-build Actions → Build other projects**.
- Downstream job: configure **Build Triggers → Build after other projects are built**.
- Select the required result condition, such as:
  - Trigger only when upstream is stable.
  - Trigger even if upstream is unstable.
  - Trigger regardless of result.

### Pipeline job

Use the `build` step:

```groovy
stage('Trigger Tests') {
    steps {
        build job: 'app-test',
              wait: true,
              propagate: true
    }
}
```

Meaning:

- `job`: downstream job name.
- `wait: true`: upstream Pipeline waits for the downstream job to finish.
- `propagate: true`: downstream failure causes the current Pipeline to fail.

To trigger without waiting:

```groovy
build job: 'app-test', wait: false
```

## 25.4 `wait` vs `propagate`

| Option | Meaning |
|---|---|
| `wait: true` | Wait for downstream job completion. |
| `wait: false` | Trigger downstream and continue immediately. |
| `propagate: true` | Propagate downstream failure to the current Pipeline. |
| `propagate: false` | Do not automatically fail the current Pipeline because of downstream result; inspect it yourself if required. |

## 25.5 Upstream/downstream jobs vs stages

| Upstream/downstream jobs | Stages in one Pipeline |
|---|---|
| Separate Jenkins jobs | Logical sections inside one Pipeline |
| Separate build records | Usually one Pipeline build record |
| Can have separate permissions and schedules | Share Pipeline context unless separated intentionally |
| Communicate through parameters, artifacts, APIs, or job results | Share variables/stashes according to Pipeline rules |
| Useful for independent ownership or legacy workflows | Usually simpler for one end-to-end workflow |

**Best practice:** Prefer one Pipeline with clear stages for a tightly coupled workflow. Use separate jobs when teams, permissions, lifecycles, or independent execution require separation.

## 25.6 Common interview scenario

**Question:** The upstream build succeeds, but the downstream test job does not start. What do you check?

1. Confirm the downstream trigger is configured correctly.
2. Check the exact upstream job name and folder path.
3. Check whether the upstream result satisfies the configured condition.
4. Check downstream job is enabled and not blocked by permissions.
5. Check parameters required by the downstream job.
6. Check queue, labels, executors, locks, and throttling.
7. Check whether a manual input or disabled trigger is blocking the chain.
8. Review the upstream console log and downstream build trigger log.

---

# 26. Jenkins and DevOps Full Forms

| Short form | Full form | Meaning in this context |
|---|---|---|
| **CI** | Continuous Integration | Frequently integrate code and automatically validate it. |
| **CD** | Continuous Delivery / Continuous Deployment | Keep software releasable / automatically release validated changes. |
| **SCM** | Source Code Management | System used to store and version source code, such as Git. |
| **VCS** | Version Control System | Tool that tracks source-code history and changes. |
| **UI** | User Interface | Jenkins web console. |
| **API** | Application Programming Interface | Programmatic way to interact with Jenkins or another service. |
| **CLI** | Command-Line Interface | Terminal-based interaction with Jenkins or tools. |
| **DSL** | Domain-Specific Language | Language/syntax designed for a particular purpose, such as Job DSL. |
| **JCasC** | Jenkins Configuration as Code | Defines Jenkins configuration in YAML/code. |
| **CPS** | Continuation-Passing Style | Execution transformation used by Jenkins Pipeline to pause and resume. |
| **JVM** | Java Virtual Machine | Runtime required by Jenkins and Java-based agents/tools. |
| **JNLP** | Java Network Launch Protocol | Older Jenkins term associated with inbound agent connections; now commonly called inbound agent protocol. |
| **LDAP** | Lightweight Directory Access Protocol | Directory protocol used for user/group lookup and authentication integration. |
| **AD** | Active Directory | Microsoft directory service that can expose LDAP and other identity services. |
| **SSO** | Single Sign-On | One identity-provider login used across multiple applications. |
| **MFA** | Multi-Factor Authentication | Authentication using two or more factors. |
| **SAML** | Security Assertion Markup Language | XML-based protocol commonly used for SSO. |
| **OIDC** | OpenID Connect | Identity layer built on OAuth 2.0. |
| **OAuth** | Open Authorization | Delegated authorization framework; it is not by itself an authentication protocol. |
| **RBAC** | Role-Based Access Control | Permissions assigned through roles. |
| **ACL** | Access Control List | List of identities and permissions for a resource. |
| **TLS** | Transport Layer Security | Encrypts network communication. |
| **SSL** | Secure Sockets Layer | Older predecessor of TLS; the term is still used informally. |
| **HTTP** | Hypertext Transfer Protocol | Web communication protocol. |
| **HTTPS** | Hypertext Transfer Protocol Secure | HTTP protected with TLS. |
| **REST** | Representational State Transfer | Common architectural style for APIs. |
| **JSON** | JavaScript Object Notation | Common data format for APIs and configuration. |
| **YAML** | YAML Ain’t Markup Language | Human-readable configuration format used by JCasC. |
| **RPO** | Recovery Point Objective | Maximum acceptable amount of data loss measured in time. |
| **RTO** | Recovery Time Objective | Target time to restore service after failure. |
| **SLA** | Service Level Agreement | Contractual commitment made to a customer or stakeholder. |
| **SLO** | Service Level Objective | Target reliability/performance objective, such as 99.9% availability. |
| **SLI** | Service Level Indicator | Actual measured metric used to evaluate an SLO. |
| **MTTR** | Mean Time To Recovery/Repair | Average time to restore service or repair a failure. |
| **MTBF** | Mean Time Between Failures | Average operating time between failures. |
| **SAST** | Static Application Security Testing | Finds security issues by analyzing source or compiled code without running the application. |
| **DAST** | Dynamic Application Security Testing | Tests a running application for security weaknesses. |
| **SCA** | Software Composition Analysis | Finds vulnerabilities and license issues in third-party dependencies. |
| **MFA** | Multi-Factor Authentication | Requires multiple independent authentication factors. |
| **PR** | Pull Request | Proposed code change submitted for review and integration. |
| **SSH** | Secure Shell | Secure remote command/login protocol. |
| **SMTP** | Simple Mail Transfer Protocol | Protocol used to send email notifications. |
| **DNS** | Domain Name System | Resolves names to IP addresses. |
| **NTP** | Network Time Protocol | Synchronizes system clocks; important for SSO certificates and tokens. |

## 26.1 SLA vs SLO vs SLI — clear concept

```text
SLI = What we measure
SLO = What target we want
SLA = What we promise contractually
```

Example:

- **SLI:** Measured application availability is 99.95% this month.
- **SLO:** Engineering target is 99.9% availability.
- **SLA:** Customer contract promises 99.5% availability, possibly with service credits.

A Jenkins pipeline may help enforce delivery quality, but **SLA, SLO, and SLI are service reliability concepts**, not Jenkins job types.

## 26.2 RPO vs RTO — clear concept

- **RPO:** How much recent data can be lost? Example: RPO of 15 minutes means backups/replication should limit data loss to approximately 15 minutes.
- **RTO:** How quickly must service return? Example: RTO of 1 hour means the service should be restored within one hour.

```text
RPO = acceptable data loss
RTO = acceptable recovery time
```

---

# 27. Missing Concept Explanations — Interview Ready

## 27.1 Authentication vs authorization

- **Authentication:** “Who are you?”
- **Authorization:** “What are you allowed to do?”

Example: LDAP/SSO may authenticate a user, while Jenkins authorization decides whether that user can read, build, configure, or administer a job.

## 27.2 Webhook vs Poll SCM

- **Webhook:** Git provider sends an event to Jenkins after a push or pull request. It is event-driven and usually faster.
- **Poll SCM:** Jenkins periodically asks the Git provider whether anything changed. It is schedule-driven and can create unnecessary traffic.

A webhook delivery being successful does **not** guarantee that a build starts. The Jenkins job trigger, branch filter, permissions, indexing, and event type must also match.

## 27.3 Quality Profile vs Quality Gate

- **Quality Profile:** Which rules should be applied during analysis?
- **Quality Gate:** Does the analyzed project pass the required conditions?

```text
Quality Profile = rules used during inspection
Quality Gate    = pass/fail decision after inspection
```

## 27.4 `stash` vs artifact repository

`stash` is temporary Pipeline-to-Pipeline-stage file transfer, generally for the same run. It is not a replacement for Nexus, Artifactory, an object store, or another long-term artifact repository.

## 27.5 `archiveArtifacts` vs artifact repository

`archiveArtifacts` stores files with Jenkins build records. An artifact repository is designed for long-term versioned distribution and consumption by many systems. For large production artifacts, prefer a dedicated artifact repository where appropriate.

## 27.6 `post { always }` vs successful build

`post { always }` means the post actions run regardless of whether the Pipeline succeeded or failed. It does **not** mean the Pipeline result is successful.

## 27.7 `retry` vs `timeout`

- **Retry:** Re-execute a failed operation a specified number of times.
- **Timeout:** Stop waiting/executing after a time limit.

They solve different problems and are often combined:

```groovy
timeout(time: 10, unit: 'MINUTES') {
    retry(2) {
        sh './integration-test.sh'
    }
}
```

Use retries only for transient failures. Do not hide deterministic test or compilation failures with blind retries.

## 27.8 Declarative Pipeline vs Jenkinsfile

A **Jenkinsfile is the file** that stores Pipeline code. **Declarative Pipeline and Scripted Pipeline are two Pipeline syntaxes** that can be stored in a Jenkinsfile.

```text
Jenkinsfile = storage/file name
Declarative  = structured Pipeline syntax
Scripted     = Groovy-based programmatic syntax
```

## 27.9 Controller restart vs Pipeline failure

A controller restart may interrupt a build, but a Pipeline can sometimes resume because Jenkins persists Pipeline execution state. Resumability depends on the step, plugin, durability settings, and whether the required external state still exists.

## 27.10 Build result vs stage result

A stage can be marked unstable or failed while post actions still execute. The final build result depends on the Pipeline logic and steps used, such as `error`, `catchError`, `returnStatus`, and `post` behavior.

---

# 28. Final Completeness Checklist

Before considering Jenkins interview preparation complete, make sure you can explain these without memorizing:

- [ ] Jenkins purpose and why it is an orchestrator.
- [ ] CI, Continuous Delivery, and Continuous Deployment.
- [ ] Controller, node, agent, executor, label, queue, and workspace.
- [ ] Job, build, Pipeline, stage, step, artifact, plugin, folder, and item.
- [ ] Freestyle vs Pipeline vs Multibranch Pipeline.
- [ ] Upstream and downstream jobs, including `wait` and `propagate`.
- [ ] Webhook, Poll SCM, scheduled trigger, manual trigger, and API trigger.
- [ ] Declarative vs Scripted Pipeline vs Jenkinsfile.
- [ ] `agent none`, `post`, `when`, `timeout`, `retry`, `parallel`, and `input`.
- [ ] `params` vs `env` and safe credential handling.
- [ ] CPS and `@NonCPS`.
- [ ] Git checkout and webhook troubleshooting.
- [ ] Plugin and `No such DSL method` troubleshooting.
- [ ] LDAP, SSO, authentication, authorization, RBAC, and security recovery.
- [ ] JCasC and Job DSL distinction.
- [ ] SonarQube, SonarScanner, Quality Profile, Quality Gate, and webhook.
- [ ] SAST, DAST, SCA, dependency scanning, and credential scanning.
- [ ] Jenkins backup, restore, RPO, and RTO.
- [ ] SLA, SLO, and SLI distinction.
- [ ] Queue, executor, controller-load, and Pipeline-performance diagnosis.

**Interview rule:** For every Jenkins term, answer in this order:

1. Definition.
2. Why it exists.
3. How it works.
4. Practical example.
5. Common failure or interview trap.
