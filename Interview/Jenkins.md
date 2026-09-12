# Jenkins L2/L3 Interview Questions and Detailed Answers

> A structured, interview-focused guide for DevOps engineers working with Jenkins, CI/CD, Git, SonarQube, credentials, distributed builds, pipelines, security, troubleshooting, and production operations.

---

## Table of Contents

1. Jenkins Fundamentals
2. Jenkins Architecture
3. Jobs, Builds, Workspaces, Artifacts, and Executors
4. Upstream and Downstream Jobs
5. Build Triggers
6. Freestyle Jobs vs Pipeline Jobs
7. Declarative and Scripted Pipelines
8. Pipeline Syntax and Execution
9. Parameters, Environment Variables, and Credentials
10. Git, Webhooks, Polling, and Multibranch Pipelines
11. Shared Libraries
12. Plugins, JCasC, and Job DSL
13. Jenkins Security
14. SonarQube and Quality Gates
15. CI/CD Pipeline Design
16. Notifications and Post-Build Actions
17. Artifacts and Artifact Repositories
18. Jenkins Backup and Disaster Recovery
19. Performance, Scaling, and Reliability
20. Troubleshooting Scenarios
21. L2/L3 Design and Scenario Questions
22. Common Interview Traps
23. Full Forms and Quick Revision Tables

---

# 1. Jenkins Fundamentals

## Q1. What is Jenkins?

Jenkins is an open-source automation server used to automate software delivery activities such as:

- Source-code checkout
- Compilation and packaging
- Unit and integration testing
- Static code analysis
- Security scanning
- Docker image building
- Artifact publishing
- Deployment to virtual machines, Kubernetes, or cloud platforms
- Notifications and approval workflows

Jenkins is commonly used as a CI/CD orchestrator.

- **Continuous Integration (CI):** Frequently integrate code changes and automatically validate them.
- **Continuous Delivery:** Keep software in a releasable state and deploy through a controlled process.
- **Continuous Deployment:** Automatically deploy validated changes to production.

Jenkins does not itself replace Git, Maven, Gradle, SonarQube, Terraform, Ansible, or Kubernetes. It coordinates them through jobs and pipelines.

## Q2. Why is Jenkins used in DevOps?

Jenkins provides:

1. Automation of repetitive tasks.
2. Early detection of defects.
3. Consistent build and deployment processes.
4. Integration with source-control systems.
5. Plugin-based extensibility.
6. Pipeline-as-code using a `Jenkinsfile`.
7. Parallel execution.
8. Approval and promotion stages.
9. Auditability through build history and logs.
10. Integration with cloud, security, monitoring, and deployment tools.

## Q3. What is the difference between CI, Continuous Delivery, and Continuous Deployment?

| Concept | Meaning | Example |
|---|---|---|
| CI | Build and test every code change | Git commit triggers unit tests |
| Continuous Delivery | Code is always ready for release, but production may require approval | Pipeline deploys to staging and waits for approval |
| Continuous Deployment | Every validated change is automatically released | Successful main-branch build deploys to production |

## Q4. What are the main components of Jenkins?

- Jenkins controller
- Jenkins agents/nodes
- Executors
- Jobs/projects
- Builds
- Workspaces
- Pipelines
- Plugins
- Credentials
- Artifacts
- Build queue
- Logs and reports
- Jenkins home directory

## Q5. What is Jenkins Home?

`JENKINS_HOME` is the directory where Jenkins stores its operational data, including:

- Job configurations
- Build history
- Pipeline metadata
- Credentials configuration
- Plugin data
- User data
- Nodes and agents configuration
- Fingerprints
- Workspace references

Typical locations:

```bash
/var/lib/jenkins
```

or a custom path configured through the service definition.

**Important:** Backing up only job files is not enough. A proper backup must include credentials, plugin information, security configuration, and relevant build metadata.

---

# 2. Jenkins Architecture

## Q6. What is the Jenkins controller?

The Jenkins controller is the central Jenkins service responsible for:

- Serving the web UI and API
- Managing jobs and pipeline definitions
- Maintaining the build queue
- Scheduling builds
- Managing credentials and security
- Coordinating agents
- Storing build metadata
- Displaying logs and reports

Modern Jenkins terminology prefers **controller** instead of the older term **master**.

## Q7. What is a Jenkins agent?

A Jenkins agent is a machine or execution environment where Jenkins runs build steps.

An agent may be:

- A virtual machine
- A physical server
- A cloud instance
- A Kubernetes pod
- A container-based worker
- A dynamically provisioned cloud node

Example:

```text
Controller
   |
   +-- Linux Agent: Java, Maven, Git
   +-- Docker Agent: Docker build tools
   +-- Security Agent: Trivy, ZAP
   +-- Deployment Agent: Ansible, Terraform
```

## Q8. What is the difference between controller, node, and agent?

- **Controller:** Central Jenkins service.
- **Node:** A machine registered with Jenkins. The controller itself can technically be a node, although builds on the controller are generally discouraged.
- **Agent:** The process that connects a worker machine to Jenkins and executes tasks.

In common usage, “node” and “agent” are often used interchangeably, but technically a node is the configured machine and an agent is the running Jenkins worker process.

## Q9. What is an executor?

An executor is a slot that can run one build or task at a time.

If an agent has four executors, it can execute up to four independent tasks concurrently, subject to CPU, memory, I/O, and pipeline constraints.

Example:

```text
Agent CPU: 8 vCPU
Executors: 4

Possible workload:
- Executor 1: Maven build
- Executor 2: Unit tests
- Executor 3: Terraform plan
- Executor 4: Security scan
```

More executors do not automatically mean better performance. Too many executors can cause CPU and memory contention.

## Q10. Why should heavy builds not run on the controller?

Running builds on the controller can cause:

- Slow Jenkins UI
- Delayed scheduling
- Memory pressure
- Security exposure
- Plugin instability affecting the entire Jenkins service
- Reduced availability of the control plane

Best practice:

- Keep the controller dedicated to orchestration.
- Run builds on agents.
- Use labels to select suitable agents.
- Restrict controller executors where practical.

## Q11. What is a Jenkins workspace?

A workspace is the directory on an agent where Jenkins checks out source code and performs build operations.

Example:

```text
/var/lib/jenkins/workspace/my-pipeline
```

A workspace may contain:

- Source code
- Build files
- Temporary files
- Test reports
- Generated packages
- Dependency caches

A workspace is not a permanent artifact repository. It can be deleted or recreated.

## Q12. What is the difference between workspace and artifact?

| Workspace | Artifact |
|---|---|
| Temporary build directory | Output intended for reuse or distribution |
| Exists on an agent | Should be stored in durable storage |
| Can be deleted after build | Must be retained according to policy |
| Contains source and intermediate files | Contains packages, binaries, images, reports |

Examples of artifacts:

- `.jar`
- `.war`
- `.zip`
- `.deb`
- `.rpm`
- Test reports
- Terraform plan files
- Container image references

---

# 3. Jobs, Builds, Workspaces, Artifacts, and Executors

## Q13. What is a Jenkins job?

A job is a configured unit of automation. It defines what Jenkins should execute and under which conditions.

Examples:

- Freestyle project
- Pipeline job
- Multibranch Pipeline
- Maven project
- Organization Folder
- External job
- Matrix project

## Q14. What is a build?

A build is one execution of a job.

A build has:

- Build number
- Start and end time
- Console log
- Result
- Workspace
- Parameters
- Artifacts
- Test reports
- Cause of execution

Possible results include:

- `SUCCESS`
- `FAILURE`
- `UNSTABLE`
- `ABORTED`
- `NOT_BUILT`

## Q15. What is the build queue?

The build queue contains tasks waiting for an available executor or required resource.

A build may remain queued because:

- No executor is available.
- No agent matches the label.
- Required agent is offline.
- A lockable resource is busy.
- Throttling rules limit concurrency.
- The job is waiting for an upstream dependency.

## Q16. What is `cleanWs()` and why is it useful?

`cleanWs()` is provided by the Workspace Cleanup plugin and removes workspace contents.

Example:

```groovy
post {
    always {
        cleanWs()
    }
}
```

Benefits:

- Prevents stale files from affecting later builds.
- Reduces disk usage.
- Avoids accidental reuse of old artifacts.
- Improves build reproducibility.

Use it carefully when builds depend on caches. Prefer dedicated dependency caches instead of preserving the entire workspace.

---

# 4. Upstream and Downstream Jobs

## Q17. What is an upstream job?

An upstream job is a job that executes before another job and triggers or influences it.

Example:

```text
Compile Job
    |
    v
Test Job
    |
    v
Deploy Job
```

Here:

- Compile is upstream of Test.
- Test is downstream of Compile.
- Test is upstream of Deploy.

## Q18. What is a downstream job?

A downstream job is a job triggered by another job or dependent on another job’s result.

For example, a deployment job may be downstream of a successful CI job.

## Q19. How can one Jenkins job trigger another job?

Common methods:

1. **Build after other projects are built** in Freestyle jobs.
2. `build` step in a Pipeline.
3. Parameterized Trigger plugin.
4. REST API call.
5. Remote trigger token.
6. Event-based integration through another automation system.

Pipeline example:

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Trigger Tests') {
            steps {
                build job: 'test-job', wait: true, propagate: true
            }
        }
    }
}
```

## Q20. What do `wait` and `propagate` mean in the `build` step?

```groovy
build job: 'downstream-job', wait: true, propagate: true
```

- `wait: true` means the upstream job waits for the downstream job to finish.
- `wait: false` means the upstream job triggers the downstream job and continues without waiting.
- `propagate: true` means the downstream result affects the upstream build result.
- `propagate: false` means the upstream job does not automatically fail because the downstream job failed.

Example:

```groovy
def result = build job: 'security-scan', wait: true, propagate: false

echo "Security scan result: ${result.result}"

if (result.result != 'SUCCESS') {
    error 'Security scan failed'
}
```

This allows custom handling of downstream results.

## Q21. What is the difference between chaining jobs and using one Pipeline?

| Job chaining | Single Pipeline |
|---|---|
| Logic is distributed across jobs | Logic is centralized in Jenkinsfile |
| Can be useful for independent teams | Easier to view end-to-end flow |
| More configuration overhead | Better version control |
| Debugging may require multiple jobs | One build contains all stages |
| Useful for independent lifecycle boundaries | Useful for application CI/CD |

Use separate jobs when there is a strong boundary, such as a reusable security scan or independent deployment process. Use one Pipeline when stages form one logical delivery workflow.

---

# 5. Build Triggers

## Q22. What is a build trigger?

A build trigger is an event or schedule that starts a Jenkins job.

Common triggers:

- Manual trigger
- Git webhook
- Poll SCM
- Cron schedule
- Upstream job completion
- Remote API trigger
- Scheduled parameterized trigger
- Plugin-specific event

## Q23. What is the difference between webhook and polling?

| Webhook | Polling |
|---|---|
| Git provider sends an event to Jenkins | Jenkins periodically checks Git |
| Near real-time | Delayed until next poll |
| More efficient | Creates repeated Git checks |
| Requires network access to Jenkins endpoint | Does not require inbound webhook if Jenkins can reach Git |
| Preferred when available | Useful when webhook is not possible |

Webhook flow:

```text
Git Push
   -> Git Provider Webhook
   -> Jenkins Endpoint
   -> Job Trigger
   -> Checkout and Build
```

Polling example:

```groovy
triggers {
    pollSCM('H/5 * * * *')
}
```

This checks approximately every five minutes, with Jenkins distributing the load using `H`.

## Q24. Explain Jenkins cron syntax.

Jenkins uses five fields:

```text
MINUTE HOUR DAY-OF-MONTH MONTH DAY-OF-WEEK
```

Examples:

```text
H/5 * * * *       Every approximately 5 minutes
H * * * *         Once every hour
H 2 * * *         Once daily around 2 AM
H 2 * * 1-5       Weekdays around 2 AM
H H * * 0         Every Sunday
```

`H` means hash-based distribution. It avoids every job starting at exactly the same minute.

## Q25. What is the difference between manual trigger and automated trigger?

- **Manual:** A user clicks “Build Now” or “Build with Parameters.”
- **Automated:** A webhook, schedule, upstream job, API call, or plugin starts the job.

Production deployments often use automated CI but controlled approval for production release.

---

# 6. Freestyle Jobs vs Pipeline Jobs

## Q26. What is a Freestyle job?

A Freestyle project is configured mainly through the Jenkins UI.

It can contain:

- Source-code management configuration
- Build triggers
- Build environment options
- Build steps
- Post-build actions

Advantages:

- Easy for beginners.
- Useful for simple tasks.
- Fast to configure for small jobs.

Limitations:

- Configuration is not naturally versioned.
- Complex workflows become difficult to maintain.
- Reuse is limited.
- Changes may not be reviewed through Git.

## Q27. What is a Pipeline job?

A Pipeline job defines automation using Pipeline syntax, usually stored in a `Jenkinsfile`.

Advantages:

- Pipeline-as-code
- Version control
- Code review
- Stages and parallelism
- Error handling
- Approvals
- Reusable shared libraries
- Better auditability

Example:

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'make build'
            }
        }

        stage('Test') {
            steps {
                sh 'make test'
            }
        }
    }
}
```

## Q28. What is a Multibranch Pipeline?

A Multibranch Pipeline automatically discovers branches containing a `Jenkinsfile` and creates separate branch jobs.

Example:

```text
Repository
├── main       -> main job
├── develop    -> develop job
├── feature/a  -> feature-a job
└── feature/b  -> feature-b job
```

Benefits:

- Automatic branch discovery
- Branch-specific Jenkinsfiles
- Pull-request integration
- Isolated build history per branch
- Reduced manual job creation

---

# 7. Declarative and Scripted Pipelines

## Q29. What is Declarative Pipeline?

Declarative Pipeline is a structured Pipeline syntax with a defined format.

Example:

```groovy
pipeline {
    agent any

    options {
        timeout(time: 30, unit: 'MINUTES')
        timestamps()
    }

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

    post {
        always {
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
        }
    }
}
```

Advantages:

- Easier to read.
- Built-in validation.
- Standard structure.
- Good for most CI/CD workflows.
- Supports `post`, `when`, `options`, `parameters`, and `environment`.

## Q30. What is Scripted Pipeline?

Scripted Pipeline is Groovy-based and gives greater programming flexibility.

Example:

```groovy
node('linux') {
    try {
        stage('Build') {
            sh 'mvn clean package'
        }

        stage('Test') {
            sh 'mvn test'
        }
    } catch (err) {
        currentBuild.result = 'FAILURE'
        throw err
    } finally {
        cleanWs()
    }
}
```

Use Scripted Pipeline when dynamic logic is complex and difficult to express declaratively.

## Q31. Declarative vs Scripted Pipeline: which should be preferred?

Prefer Declarative Pipeline for most application CI/CD workflows because it provides structure and readability.

Use Scripted Pipeline when:

- Dynamic stage generation is required.
- Complex branching logic is unavoidable.
- Advanced Groovy control is needed.
- A shared library requires scripted implementation.

Do not use complex Groovy merely because it is possible. Keep the Jenkinsfile simple and move reusable logic into shared libraries or scripts.

---

# 8. Pipeline Syntax and Execution

## Q32. What are the main sections of a Declarative Pipeline?

- `pipeline`
- `agent`
- `options`
- `parameters`
- `environment`
- `triggers`
- `stages`
- `stage`
- `steps`
- `post`
- `when`
- `input`
- `tools`

## Q33. Difference between stage and step?

- **Stage:** Logical phase of the workflow.
- **Step:** Individual action executed inside a stage.

Example:

```groovy
stage('Build') {
    steps {
        sh 'mvn clean package'
        archiveArtifacts artifacts: 'target/*.jar'
    }
}
```

`Build` is the stage. `sh` and `archiveArtifacts` are steps.

## Q34. What is the purpose of `agent`?

`agent` defines where the Pipeline or stage executes.

Examples:

```groovy
agent any
```

```groovy
agent none
```

```groovy
agent { label 'linux' }
```

```groovy
agent {
    docker { image 'maven:3.9-eclipse-temurin-17' }
}
```

With `agent none`, each stage must define its own agent if it executes commands.

## Q35. What is `agent none` useful for?

It prevents Jenkins from reserving an executor for the entire Pipeline.

Example:

```groovy
pipeline {
    agent none

    stages {
        stage('Build') {
            agent { label 'java' }
            steps {
                sh 'mvn package'
            }
        }

        stage('Security Scan') {
            agent { label 'security' }
            steps {
                sh 'trivy fs .'
            }
        }
    }
}
```

## Q36. What is the purpose of `post`?

`post` defines actions after a stage or Pipeline completes.

Conditions include:

- `always`
- `success`
- `failure`
- `unstable`
- `aborted`
- `changed`
- `fixed`
- `regression`

Example:

```groovy
post {
    always {
        junit testResults: '**/target/surefire-reports/*.xml', allowEmptyResults: true
        cleanWs()
    }
    success {
        echo 'Pipeline completed successfully'
    }
    failure {
        echo 'Pipeline failed'
    }
}
```

## Q37. What is `when`?

`when` conditionally executes a stage.

Examples:

```groovy
when {
    branch 'main'
}
```

```groovy
when {
    expression {
        return params.DEPLOY == true
    }
}
```

```groovy
when {
    anyOf {
        branch 'main'
        branch 'release/*'
    }
}
```

## Q38. What are `timeout`, `retry`, and `timestamps`?

```groovy
options {
    timeout(time: 45, unit: 'MINUTES')
    retry(2)
    timestamps()
}
```

- `timeout`: Stops a build that exceeds the allowed time.
- `retry`: Repeats a block after failure.
- `timestamps`: Adds timestamps to console output.

Use `retry` only for transient operations. Retrying a deterministic compilation error usually wastes time.

## Q39. What is the `input` step?

`input` pauses a Pipeline and waits for human approval or input.

Example:

```groovy
stage('Production Approval') {
    steps {
        input message: 'Deploy to production?', ok: 'Deploy'
    }
}
```

For stronger governance, configure:

- Authorized approvers
- Submission timeout
- Audit requirements
- Separation of duties
- Deployment change records

## Q40. What is parallel execution?

Parallel execution runs independent branches simultaneously.

Example:

```groovy
stage('Parallel Checks') {
    parallel {
        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Lint') {
            steps {
                sh 'make lint'
            }
        }
        stage('Security Scan') {
            steps {
                sh 'trivy fs .'
            }
        }
    }
}
```

Parallelism reduces duration but increases resource usage and requires independent tasks.

---

# 9. Parameters, Environment Variables, and Credentials

## Q41. What are build parameters?

Parameters allow users or upstream jobs to provide values at runtime.

Example:

```groovy
parameters {
    choice(name: 'ENVIRONMENT', choices: ['dev', 'qa', 'prod'])
    booleanParam(name: 'RUN_SECURITY_SCAN', defaultValue: true)
    string(name: 'VERSION', defaultValue: '1.0.0')
}
```

Usage:

```groovy
echo "Deploying ${params.VERSION} to ${params.ENVIRONMENT}"
```

Do not allow arbitrary production deployment parameters without validation.

## Q42. What is the difference between parameters and environment variables?

| Parameters | Environment variables |
|---|---|
| Input supplied to a build | Values available to processes |
| Usually selected by user/upstream job | Can be static or dynamically assigned |
| Accessed using `params.NAME` | Accessed using `env.NAME` or shell `$NAME` |
| Used for build decisions | Used for configuration and command execution |

## Q43. How do you define environment variables?

Pipeline-level:

```groovy
environment {
    APP_NAME = 'attendance-api'
    BUILD_MODE = 'release'
}
```

Stage-level:

```groovy
stage('Deploy') {
    environment {
        TARGET_ENV = 'qa'
    }
    steps {
        sh 'echo $TARGET_ENV'
    }
}
```

## Q44. How should Jenkins credentials be managed?

Use Jenkins Credentials rather than hardcoding secrets in:

- Jenkinsfiles
- Shell scripts
- Git repositories
- Dockerfiles
- Terraform variables committed to Git
- Console output

Example:

```groovy
withCredentials([
    usernamePassword(
        credentialsId: 'registry-creds',
        usernameVariable: 'REG_USER',
        passwordVariable: 'REG_PASS'
    )
]) {
    sh 'echo "$REG_PASS" | docker login --username "$REG_USER" --password-stdin'
}
```

Recommended practices:

- Use least privilege.
- Rotate secrets.
- Restrict credential scope.
- Avoid printing secrets.
- Prefer cloud identity or workload roles where possible.
- Use external secret managers for highly sensitive environments.

## Q45. What is the difference between secret text, username/password, SSH key, and certificate credentials?

- **Secret text:** API token, password, access token.
- **Username/password:** Registry, database, or application login.
- **SSH username with private key:** Git over SSH or remote server access.
- **Certificate:** Mutual TLS or certificate-based authentication.

Choose the credential type based on the target system’s authentication method.

## Q46. How can credentials leak in Jenkins?

Common causes:

- `echo $PASSWORD`
- Shell tracing using `set -x`
- Passing secrets in command-line arguments
- Writing secrets to artifacts
- Unsafe Groovy interpolation
- Printing environment variables
- Uploading logs to public storage
- Malicious or untrusted Pipeline code

Safer example:

```groovy
withCredentials([string(credentialsId: 'token', variable: 'TOKEN')]) {
    sh '''
        set +x
        curl -H "Authorization: Bearer $TOKEN" https://example.internal/api
    '''
}
```

---

# 10. Git, Webhooks, Polling, and Multibranch Pipelines

## Q47. What happens when a Git webhook triggers Jenkins?

Typical flow:

1. Developer pushes code.
2. Git provider receives the push.
3. Git provider sends an HTTP request to Jenkins.
4. Jenkins validates the event.
5. Jenkins identifies the matching job.
6. Jenkins schedules a build.
7. Agent checks out the correct commit.
8. Pipeline executes.
9. Jenkins reports status back to the Git provider.

## Q48. Why can a webhook succeed but the Jenkins build not start?

Possible causes:

- Incorrect webhook URL.
- Jenkins endpoint not reachable.
- Reverse proxy misconfiguration.
- CSRF or authentication issue.
- Wrong repository/job mapping.
- Branch filtering.
- Missing plugin integration.
- Webhook event type not supported.
- Jenkins job disabled.
- Duplicate or ignored events.

Check:

- Git provider webhook delivery logs.
- Jenkins system log.
- Job trigger configuration.
- Plugin status.
- Reverse proxy logs.
- Network security rules.

## Q49. What is a Git checkout in Jenkins?

Checkout retrieves source code into the workspace.

Example:

```groovy
checkout scm
```

or:

```groovy
git branch: 'main',
    credentialsId: 'git-creds',
    url: 'https://git.example.com/team/app.git'
```

For Multibranch Pipelines, `checkout scm` is usually preferred because Jenkins already knows the branch and revision.

## Q50. What is a shallow clone?

A shallow clone retrieves limited Git history.

Example:

```bash
git clone --depth 1 https://git.example.com/team/app.git
```

Benefits:

- Faster checkout.
- Lower network usage.
- Less disk space.

Limitations:

- Some versioning commands require full history.
- Changelog generation may be incomplete.
- Git merge-base operations may fail.

---

# 11. Shared Libraries

## Q51. What is a Jenkins Shared Library?

A Shared Library stores reusable Pipeline code in a separate Git repository.

Typical structure:

```text
jenkins-shared-library/
├── vars/
│   └── deployApplication.groovy
├── src/
│   └── com/company/Utils.groovy
└── resources/
```

Usage:

```groovy
@Library('company-shared-library') _

deployApplication(environment: 'qa')
```

Benefits:

- Reuse
- Standardization
- Centralized maintenance
- Reduced Jenkinsfile size
- Consistent security and deployment patterns

## Q52. What belongs in `vars` and `src`?

- `vars`: Global Pipeline steps or DSL-like functions.
- `src`: Groovy classes and reusable application logic.
- `resources`: Static files loaded by the library.

Example `vars/buildJava.groovy`:

```groovy
def call(Map config = [:]) {
    sh "${config.maven ?: 'mvn'} clean package"
}
```

## Q53. What are risks of Shared Libraries?

- A central change can affect many pipelines.
- Unsafe code may access credentials or agents.
- Poor versioning can break old jobs.
- Unreviewed changes can create supply-chain risk.

Best practices:

- Version libraries using tags or branches.
- Review changes.
- Test library functions.
- Use trusted libraries carefully.
- Avoid unnecessary global privileges.

---

# 12. Plugins, JCasC, and Job DSL

## Q54. What is a Jenkins plugin?

A plugin extends Jenkins functionality.

Examples:

- Git plugin
- Pipeline plugin
- Credentials plugin
- SSH Build Agents plugin
- Workspace Cleanup plugin
- SonarQube Scanner plugin
- Docker Pipeline plugin
- Kubernetes plugin
- Email Extension plugin

Plugin risks:

- Compatibility issues
- Security vulnerabilities
- Dependency conflicts
- Unexpected behavior after upgrades

## Q55. How should Jenkins plugins be managed?

- Maintain an approved plugin list.
- Remove unused plugins.
- Test upgrades in a non-production Jenkins instance.
- Review security advisories.
- Pin versions where required.
- Back up before upgrades.
- Monitor plugin dependencies.

## Q56. What is JCasC?

**JCasC** means **Jenkins Configuration as Code**.

It defines Jenkins configuration in YAML instead of manually configuring everything through the UI.

Example:

```yaml
jenkins:
  systemMessage: "Managed by Configuration as Code"
  numExecutors: 0
```

Benefits:

- Repeatability
- Version control
- Faster recovery
- Consistent environments
- Easier automation

Do not store secrets directly in a public YAML file. Use secret references or an external secret-management solution.

## Q57. What is Job DSL?

Job DSL is a Groovy-based method for generating Jenkins jobs programmatically.

Example concept:

```groovy
pipelineJob('application-ci') {
    definition {
        cpsScm {
            scm {
                git {
                    remote {
                        url('https://git.example.com/app.git')
                    }
                    branch('*/main')
                }
            }
            scriptPath('Jenkinsfile')
        }
    }
}
```

Difference:

- **JCasC:** Configures Jenkins itself.
- **Job DSL:** Creates and configures jobs.
- **Jenkinsfile:** Defines the execution workflow of a job.

---

# 13. Jenkins Security

## Q58. What is authentication?

Authentication verifies **who** a user or system is.

Examples:

- Username/password
- LDAP
- Active Directory
- SSO
- SAML
- OpenID Connect
- API token
- SSH key

## Q59. What is authorization?

Authorization determines **what** an authenticated identity is allowed to do.

Examples:

- Read jobs
- Build jobs
- Configure jobs
- Cancel builds
- Manage credentials
- Manage Jenkins

## Q60. Authentication vs authorization?

| Authentication | Authorization |
|---|---|
| Who are you? | What can you do? |
| Login verification | Permission evaluation |
| Happens before access | Controls allowed actions |
| Example: LDAP login | Example: build-only permission |

## Q61. What is Role-Based Access Control?

**RBAC** means **Role-Based Access Control**.

Permissions are assigned to roles rather than individually to every user.

Example roles:

- Developer: Read and build development jobs.
- Release Engineer: Deploy to staging.
- Production Approver: Approve production releases.
- Jenkins Administrator: Manage global configuration.

Use least privilege and separate production permissions from development permissions.

## Q62. What is the Groovy sandbox?

The Groovy sandbox restricts Pipeline scripts from executing potentially dangerous methods without approval.

It protects Jenkins from unsafe Pipeline code, especially when code is contributed by users who should not have unrestricted administrative access.

Do not approve every script-signature request blindly. Review what the method does and whether the library or script is trusted.

## Q63. How do you secure Jenkins?

- Enable authentication.
- Use authorization strategies.
- Enforce HTTPS.
- Restrict anonymous access.
- Use least-privilege service accounts.
- Protect credentials.
- Keep plugins updated.
- Restrict controller builds.
- Isolate agents.
- Use network segmentation.
- Enable audit logging.
- Back up securely.
- Restrict script approval.
- Avoid exposing Jenkins directly to the internet.
- Use CSRF protection and secure reverse-proxy configuration.

---

# 14. SonarQube and Quality Gates

## Q64. What is SonarQube?

SonarQube is a code-quality and static-analysis platform. It analyzes source code for issues such as:

- Bugs
- Vulnerabilities
- Code smells
- Duplicated code
- Maintainability problems
- Coverage information from test tools

## Q65. What is a Quality Gate?

A Quality Gate is a set of conditions that determines whether code meets an organization’s quality standards.

Example conditions:

- New bugs must be zero.
- New vulnerabilities must be zero.
- Coverage on new code must be above a threshold.
- Duplicated lines must remain below a threshold.
- Maintainability rating must meet the required level.

## Q66. How does Jenkins integrate with SonarQube?

Typical flow:

```text
Checkout
   -> Build
   -> Unit Tests
   -> SonarQube Analysis
   -> Wait for Quality Gate
   -> Continue or Fail
```

Example:

```groovy
stage('SonarQube Analysis') {
    steps {
        withSonarQubeEnv('sonarqube-server') {
            sh 'mvn sonar:sonar'
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

## Q67. Why should the Quality Gate have a timeout?

Without a timeout, Jenkins may wait indefinitely if:

- SonarQube is unavailable.
- The webhook is not delivered.
- The analysis remains pending.
- The task fails to complete.

A timeout prevents executor starvation and stuck pipelines.

## Q68. What is the difference between static analysis and dynamic testing?

- **Static analysis:** Examines code or binaries without running the application. Example: SonarQube, Checkstyle, PMD.
- **Dynamic testing:** Tests behavior while the application is running. Example: integration tests, DAST, API tests.

---

# 15. CI/CD Pipeline Design

## Q69. Design a production-grade Java CI pipeline.

A typical pipeline:

```text
Checkout
   -> Validate
   -> Compile
   -> Unit Test
   -> Code Coverage
   -> Static Analysis
   -> Quality Gate
   -> Dependency/SCA Scan
   -> Package
   -> Publish Artifact
   -> Notify
```

Example:

```groovy
pipeline {
    agent { label 'java' }

    options {
        timestamps()
        timeout(time: 45, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Compile and Test') {
            steps {
                sh 'mvn clean verify'
            }
        }

        stage('Static Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh 'mvn sonar:sonar'
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

        stage('Publish') {
            when {
                branch 'main'
            }
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }

    post {
        always {
            junit testResults: '**/target/surefire-reports/*.xml', allowEmptyResults: true
            cleanWs()
        }
    }
}
```

## Q70. What makes a pipeline production-ready?

A production-ready pipeline should include:

- Version-controlled Jenkinsfile
- Clear stages
- Immutable artifacts
- Secure credentials
- Input validation
- Timeouts
- Retry only for transient operations
- Test reports
- Static analysis
- Security scanning
- Quality gates
- Manual approval where required
- Notifications
- Workspace cleanup
- Concurrency control
- Rollback strategy
- Auditability
- Monitoring and metrics

## Q71. Why should build and deployment be separated?

Separating build and deployment provides:

- Artifact promotion.
- Repeatable deployments.
- Independent approvals.
- Reduced rebuild risk.
- Better traceability.
- Easier rollback.

Preferred model:

```text
Commit
  -> Build once
  -> Publish immutable artifact
  -> Deploy same artifact to QA
  -> Approve
  -> Deploy same artifact to Production
```

Do not rebuild the application separately for each environment if the goal is to promote the exact same binary.

## Q72. How should failures be handled?

Classify failures:

1. **Code failure:** Fix source code.
2. **Test failure:** Investigate application or test logic.
3. **Infrastructure failure:** Check agent, network, disk, CPU, memory, or service availability.
4. **Credential failure:** Validate credential ID, permissions, and expiry.
5. **Tool failure:** Check Maven, Git, SonarQube, Terraform, or scanner versions.
6. **Pipeline logic failure:** Review conditions, variables, and error handling.

Avoid blindly retrying all failures.

---

# 16. Notifications and Post-Build Actions

## Q73. What are post-build actions?

Post-build actions run after a job completes.

Examples:

- Archive artifacts
- Publish JUnit results
- Publish HTML reports
- Send email
- Send Slack notification
- Trigger downstream job
- Clean workspace
- Upload reports
- Update Git commit status

## Q74. How do you send notifications only when a build fails?

Example:

```groovy
post {
    failure {
        echo 'Sending failure notification'
        // slackSend or emailext can be used here
    }
}
```

For production, include:

- Job name
- Build number
- Branch
- Commit ID
- Failed stage
- Console log link
- Responsible team
- Environment

## Q75. What is the difference between `always`, `success`, and `failure` in `post`?

- `always`: Runs regardless of result.
- `success`: Runs only after successful completion.
- `failure`: Runs only after failure.

Cleanup and report publishing often belong in `always`; deployment notifications belong in `success` or `failure`.

---

# 17. Artifacts and Artifact Repositories

## Q76. What is artifact archiving in Jenkins?

`archiveArtifacts` stores build outputs with the Jenkins build record.

Example:

```groovy
archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
```

Use it for small or moderate outputs and short-term retention.

## Q77. Why should large artifacts not be stored only in Jenkins?

Jenkins is primarily an automation server, not a full artifact-management platform.

For large-scale environments, use:

- Nexus
- JFrog Artifactory
- AWS S3
- Container registries
- Cloud package repositories

Benefits:

- Better retention management
- Versioning
- Access control
- Replication
- Scalability
- Artifact promotion

## Q78. What is artifact immutability?

An immutable artifact is not modified after publication.

Example:

```text
attendance-api:1.4.2
```

The same artifact should be promoted through environments instead of rebuilding different content under the same version.

---

# 18. Jenkins Backup and Disaster Recovery

## Q79. What should be backed up in Jenkins?

At minimum:

- `JENKINS_HOME`
- Job configurations
- Pipeline definitions
- Credentials configuration and required secret material
- Plugin list and versions
- Security configuration
- JCasC files
- Shared library references
- User and permission configuration
- Important build metadata
- External configuration and reverse-proxy settings

Do not assume encrypted credential values are useful without the Jenkins encryption keys and correct restoration environment.

## Q80. What is the difference between RPO and RTO?

- **RPO — Recovery Point Objective:** Maximum acceptable data loss measured in time.
- **RTO — Recovery Time Objective:** Maximum acceptable time to restore service.

Example:

```text
RPO: 15 minutes
RTO: 1 hour
```

This means the organization accepts losing at most approximately 15 minutes of Jenkins data and expects recovery within one hour.

## Q81. What is a good Jenkins backup strategy?

- Take scheduled backups.
- Store backups outside the Jenkins host.
- Encrypt backups.
- Retain multiple recovery points.
- Test restoration regularly.
- Document plugin versions.
- Keep JCasC and Jenkinsfiles in Git.
- Back up credentials securely.
- Monitor backup success and age.

A backup that has never been restored is not fully trusted.

---

# 19. Performance, Scaling, and Reliability

## Q82. Why is Jenkins slow?

Possible causes:

- Too many builds on the controller.
- Insufficient CPU or memory.
- Too many executors.
- Slow storage.
- Large workspaces.
- Excessive console logging.
- Plugin problems.
- Large build history.
- Slow Git checkout.
- Slow artifact uploads.
- Too many concurrent pipelines.
- Memory leaks.
- Slow external services.

## Q83. How can Jenkins performance be improved?

- Move builds to agents.
- Tune executor count.
- Use appropriate agent labels.
- Clean workspaces.
- Use shallow Git clones where suitable.
- Cache dependencies carefully.
- Archive only required artifacts.
- Limit build history.
- Avoid unnecessary polling.
- Use webhooks.
- Split heavy workloads across agents.
- Upgrade hardware where justified.
- Remove unused plugins.
- Use pipeline timeouts.
- Avoid excessive Groovy logic.

## Q84. What is `disableConcurrentBuilds()`?

It prevents multiple builds of the same job from running concurrently.

Example:

```groovy
options {
    disableConcurrentBuilds()
}
```

Use it when concurrent executions could cause conflicts, such as:

- Deploying the same environment.
- Modifying shared infrastructure.
- Using a shared database migration lock.
- Updating a single release branch.

## Q85. What is throttling?

Throttling limits the number of concurrent executions for a job or category of jobs.

It helps prevent:

- Resource exhaustion.
- Too many deployments at once.
- API rate-limit violations.
- Database overload.

## Q86. How do you scale Jenkins?

Possible approaches:

1. Add more static agents.
2. Use cloud-based dynamic agents.
3. Use Kubernetes-based ephemeral agents.
4. Separate agents by workload.
5. Increase controller resources.
6. Reduce controller build execution.
7. Use artifact repositories.
8. Optimize plugins and job design.
9. Apply queue and concurrency controls.

---

# 20. Troubleshooting Scenarios

## Q87. A Jenkins job is stuck in the queue. What do you check?

Check:

1. Is an executor available?
2. Is the required agent online?
3. Does the label exist?
4. Is the agent accepting tasks?
5. Is the job blocked by throttling?
6. Is a lockable resource occupied?
7. Is the node temporarily offline?
8. Is the controller overloaded?

Useful areas:

- Build queue
- Manage Nodes
- System Log
- Executor status
- Plugin logs

## Q88. Jenkins says “There are no nodes with the label.”

Possible reasons:

- Typo in label.
- Agent offline.
- Label removed.
- Agent is not accepting tasks.
- Cloud agent provisioning failed.
- Job requires a label that no longer exists.

Example:

```groovy
agent { label 'java-linux' }
```

Verify the exact label on the agent configuration.

## Q89. Git checkout fails with permission denied. What do you check?

- Credential ID.
- Credential type.
- Repository URL.
- SSH key permissions.
- Known hosts configuration.
- Git provider permissions.
- Branch existence.
- Network connectivity.
- Proxy configuration.
- Token expiry.

For SSH:

```bash
ssh -T git@github.com
```

For HTTPS, verify the token and repository access.

## Q90. Maven build fails because the Java version is wrong.

Check:

```bash
java -version
mvn -version
which java
which mvn
```

The Java runtime used by Maven may differ from the system default.

In Jenkins, configure the correct tool or agent image and verify the version inside the job itself.

Example:

```groovy
stage('Verify Toolchain') {
    steps {
        sh 'java -version'
        sh 'mvn -version'
    }
}
```

## Q91. SonarQube Quality Gate waits forever.

Check:

- SonarQube server availability.
- Scanner task completion.
- Jenkins SonarQube configuration.
- Webhook URL.
- Reverse proxy.
- Network connectivity.
- Authentication token.
- Correct project key.
- Timeout configuration.

Always use a timeout around `waitForQualityGate`.

## Q92. Jenkins credentials are not available in a Pipeline.

Check:

- Credential ID spelling.
- Credential scope.
- Folder-level permissions.
- Job authorization.
- Correct credential type.
- Plugin installation.
- Whether the step is inside `withCredentials`.
- Whether the credential is allowed on the selected agent.

## Q93. A Pipeline works manually but fails from a webhook.

Possible differences:

- Different branch.
- Different parameters.
- Different user permissions.
- Different environment variables.
- Different workspace state.
- Different credentials context.
- Webhook event does not contain expected data.
- Manual build used a different agent.

Compare the build causes, parameters, environment, branch, and console logs.

## Q94. A deployment runs twice. What could cause it?

- Duplicate webhook deliveries.
- Both webhook and polling enabled.
- Multiple jobs watching the same repository.
- Retry logic triggering deployment again.
- Upstream and downstream jobs both deploying.
- Multibranch job plus standalone job.
- User manually triggered a second build.

Mitigations:

- Use idempotent deployment logic.
- Disable unnecessary triggers.
- Use `disableConcurrentBuilds()`.
- Add deployment locks.
- Track commit SHA and release version.

## Q95. Jenkins disk usage is increasing rapidly.

Check:

```bash
du -sh /var/lib/jenkins/*
du -sh /var/lib/jenkins/workspace/*
df -h
```

Likely causes:

- Old builds.
- Large workspaces.
- Archived artifacts.
- Console logs.
- Test reports.
- Plugin caches.
- Docker images on agents.

Actions:

- Configure build discarder.
- Clean workspaces.
- Remove unused artifacts.
- Clean agent caches safely.
- Expand storage if required.
- Set retention policies.

Example:

```groovy
options {
    buildDiscarder(logRotator(numToKeepStr: '20'))
}
```

## Q96. A Pipeline is marked `UNSTABLE`. What does it mean?

`UNSTABLE` usually means the build completed but one or more quality conditions were not fully satisfied.

Examples:

- Test failures configured as non-fatal.
- Coverage below threshold.
- Warnings or analysis issues.
- Publisher marked the build unstable.

`UNSTABLE` is different from `FAILURE`: the build may have completed technically, but quality expectations were not met.

---

# 21. L2/L3 Design and Scenario Questions

## Q97. How would you design Jenkins for multiple teams?

Design:

- Separate folders by team or product.
- Use role-based permissions.
- Use shared libraries for common patterns.
- Use standardized agent labels.
- Use JCasC for controller configuration.
- Use Job DSL or Multibranch Pipelines for job creation.
- Use centralized credential policies.
- Use artifact repositories.
- Use monitoring and alerting.
- Define ownership and escalation paths.

Example:

```text
Jenkins
├── Platform
├── Payments
├── HR
├── Notifications
└── Shared Libraries
```

## Q98. How would you implement a secure production deployment?

Recommended flow:

```text
Pull Request
   -> CI validation
   -> Unit tests
   -> Security scans
   -> Quality Gate
   -> Build immutable artifact
   -> Publish artifact
   -> Deploy to QA
   -> Approval
   -> Deploy to production
   -> Smoke test
   -> Notify
```

Controls:

- Restricted production permissions.
- Separate deploy credentials.
- Approval gate.
- Audit trail.
- Artifact checksum/version.
- Rollback plan.
- Deployment timeout.
- Health checks.
- Post-deployment verification.

## Q99. How would you prevent one faulty build from affecting other teams?

- Use separate agents.
- Use folder-level permissions.
- Use resource quotas.
- Use job throttling.
- Isolate credentials.
- Avoid shared mutable workspaces.
- Use ephemeral agents.
- Use separate tool versions where required.
- Apply pipeline timeouts.
- Monitor resource consumption.

## Q100. How would you migrate UI-configured jobs to Pipeline-as-code?

Steps:

1. Inventory existing jobs.
2. Identify SCM, triggers, tools, credentials, and post-build actions.
3. Create a Jenkinsfile.
4. Reproduce the behavior in stages.
5. Add tests and validation.
6. Run old and new jobs in parallel temporarily.
7. Compare outputs.
8. Switch triggers.
9. Disable the old job after verification.
10. Document rollback.

## Q101. How would you handle a Jenkins controller failure?

Immediate actions:

1. Confirm service and host status.
2. Check CPU, memory, disk, and filesystem.
3. Review Jenkins logs.
4. Check recent plugin or configuration changes.
5. Restore service if safe.
6. Fail over or restore from backup if required.
7. Validate agents and credentials.
8. Run a smoke-test job.
9. Communicate impact and recovery status.
10. Perform root-cause analysis.

## Q102. How would you design rollback?

Rollback should be based on a known-good immutable version.

Examples:

- Deploy previous artifact version.
- Revert Kubernetes deployment image tag.
- Restore previous infrastructure state carefully.
- Use blue/green or canary deployment.
- Keep database migrations backward-compatible where possible.

A rollback plan must define:

- Trigger condition.
- Authorized person.
- Exact version.
- Commands or Pipeline stage.
- Validation steps.
- Communication process.

---

# 22. Common Interview Traps

## Q103. Is Jenkins itself a container?

No. Jenkins is an automation server. It can run:

- Directly on a VM.
- On a physical server.
- Inside a container.
- In Kubernetes.

A Jenkins agent can also run in a container, but Jenkins and a container are not the same concept.

## Q104. Is an agent the same as an executor?

No.

- Agent = execution machine/process.
- Executor = concurrency slot on that agent.

One agent can have multiple executors.

## Q105. Does Jenkins store source code permanently?

Not necessarily. Jenkins checks out source into a workspace. The workspace may be cleaned after the build. Git remains the source of truth.

## Q106. Does a successful build mean production deployment succeeded?

No. A build may only compile and test code. Deployment is a separate stage and may fail because of:

- Network issues.
- Credentials.
- Target health.
- Configuration.
- Capacity.
- Application startup.

## Q107. Is `wait: false` the same as a successful deployment?

No. `wait: false` only means the upstream job does not wait for the downstream job result. The downstream job may later fail.

## Q108. Should every failure be retried?

No. Retry only transient failures such as:

- Temporary network failure.
- Rate limiting.
- Short-lived service unavailability.

Do not blindly retry:

- Compilation errors.
- Failed tests caused by code.
- Invalid credentials.
- Syntax errors.
- Incorrect configuration.

## Q109. Is a Quality Gate the same as a build?

No. A build produces or validates software. A Quality Gate evaluates whether defined quality conditions are satisfied.

## Q110. Is Jenkins a replacement for Git?

No. Git stores source history. Jenkins automates actions performed on that source.

---

# 23. Full Forms and Quick Revision Tables

## Important Full Forms

| Short form | Full form |
|---|---|
| CI | Continuous Integration |
| CD | Continuous Delivery / Continuous Deployment |
| SCM | Source Code Management |
| VCS | Version Control System |
| UI | User Interface |
| API | Application Programming Interface |
| CLI | Command-Line Interface |
| DSL | Domain-Specific Language |
| JCasC | Jenkins Configuration as Code |
| JVM | Java Virtual Machine |
| LDAP | Lightweight Directory Access Protocol |
| AD | Active Directory |
| SSO | Single Sign-On |
| MFA | Multi-Factor Authentication |
| SAML | Security Assertion Markup Language |
| OIDC | OpenID Connect |
| OAuth | Open Authorization |
| RBAC | Role-Based Access Control |
| ACL | Access Control List |
| TLS | Transport Layer Security |
| SSL | Secure Sockets Layer |
| HTTP | Hypertext Transfer Protocol |
| HTTPS | Hypertext Transfer Protocol Secure |
| REST | Representational State Transfer |
| JSON | JavaScript Object Notation |
| YAML | YAML Ain’t Markup Language |
| RPO | Recovery Point Objective |
| RTO | Recovery Time Objective |
| SLA | Service Level Agreement |
| SLO | Service Level Objective |
| SLI | Service Level Indicator |
| MTTR | Mean Time to Recovery/Repair |
| MTBF | Mean Time Between Failures |
| SAST | Static Application Security Testing |
| DAST | Dynamic Application Security Testing |
| SCA | Software Composition Analysis |
| PR | Pull Request |
| SSH | Secure Shell |
| SMTP | Simple Mail Transfer Protocol |
| DNS | Domain Name System |
| NTP | Network Time Protocol |
| CPS | Continuation Passing Style |
| JNLP | Java Network Launch Protocol; historically used by Jenkins agents |

## Quick Comparison: Controller, Agent, Executor

| Term | Meaning |
|---|---|
| Controller | Coordinates Jenkins operations |
| Agent | Executes build work |
| Node | Registered machine/environment |
| Executor | One concurrent task slot |
| Workspace | Temporary build directory |
| Artifact | Build output retained for reuse |
| Job | Automation configuration |
| Build | One execution of a job |

## Quick Comparison: Pipeline Types

| Type | Best use |
|---|---|
| Freestyle | Simple UI-configured automation |
| Declarative Pipeline | Standard CI/CD workflows |
| Scripted Pipeline | Complex dynamic Groovy logic |
| Multibranch Pipeline | Branch and pull-request automation |
| Shared Library | Reusable Pipeline functions |

## Quick Comparison: Trigger Types

| Trigger | Best use |
|---|---|
| Manual | Controlled execution |
| Webhook | Near-real-time Git events |
| Poll SCM | Environments where webhook is unavailable |
| Cron | Scheduled jobs |
| Upstream trigger | Job dependency chain |
| Remote API | External automation |

## Quick Comparison: RPO, RTO, SLA, SLO, SLI

| Term | Meaning |
|---|---|
| RPO | How much data loss is acceptable |
| RTO | How quickly service must recover |
| SLA | Contractual service commitment |
| SLO | Internal target for reliability |
| SLI | Actual measured reliability indicator |

---

# Final Interview Preparation Checklist

Before an L2/L3 Jenkins interview, make sure you can explain and demonstrate:

- [ ] Jenkins controller and agent architecture
- [ ] Node versus agent versus executor
- [ ] Workspace versus artifact
- [ ] Job versus build
- [ ] Upstream and downstream jobs
- [ ] `wait` versus `propagate`
- [ ] Webhook versus polling
- [ ] Cron syntax and `H`
- [ ] Freestyle versus Pipeline
- [ ] Declarative versus Scripted Pipeline
- [ ] `agent`, `stage`, `steps`, `post`, `when`, and `input`
- [ ] Parameters and environment variables
- [ ] Credentials binding and secret masking
- [ ] Git checkout and branch handling
- [ ] Multibranch Pipeline
- [ ] Shared Libraries
- [ ] Plugins and plugin governance
- [ ] JCasC and Job DSL
- [ ] Authentication versus authorization
- [ ] RBAC and Groovy sandbox
- [ ] SonarQube and Quality Gates
- [ ] Artifact immutability
- [ ] Backup, RPO, and RTO
- [ ] Jenkins performance and scaling
- [ ] Queue troubleshooting
- [ ] Agent offline troubleshooting
- [ ] Git credential failures
- [ ] Java/Maven version mismatch
- [ ] Stuck Quality Gate
- [ ] Duplicate deployments
- [ ] Disk-space problems
- [ ] Production approval and rollback design

## Final Advice

For L2/L3 interviews, do not answer only with definitions. Explain:

1. **What the feature is.**
2. **Why it is used.**
3. **How it works internally.**
4. **A practical Jenkinsfile or command example.**
5. **What can fail.**
6. **How you would troubleshoot it.**
7. **What security or reliability concern exists.**

That structure demonstrates operational understanding rather than memorized theory.
