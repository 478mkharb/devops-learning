# Jenkins Shared Library — Interview Questions & Answers

This README contains practical Jenkins Shared Library interview questions covering architecture, `vars/`, `src/`, `resources/`, custom steps, library loading, versioning, security, testing, credentials, Jenkins plugins, and enterprise CI/CD design.

---

## 1. What is a Jenkins Shared Library?

A Jenkins Shared Library is a reusable collection of Groovy code, Pipeline steps, and supporting resources that can be shared across multiple Jenkins pipelines.

Instead of repeating the same Jenkinsfile logic in every repository, common functionality is implemented once and reused.

### Example

```groovy
@Library('company-ci@v1.0.0') _

pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                buildJava()
            }
        }
    }
}
```

The main benefits are **reuse, standardization, centralized maintenance, and smaller Jenkinsfiles**.

---

## 2. Why do we use Jenkins Shared Libraries?

Shared Libraries avoid duplicating pipeline code across projects.

They provide:

- Reusable CI/CD logic
- Standardized pipelines
- Centralized maintenance
- Smaller Jenkinsfiles
- Consistent security and quality checks
- Easier rollout of common pipeline changes
- Reusable custom steps

For example, if 50 microservices use:

```text
Checkout → Build → Test → SonarQube → Security Scan → Publish → Deploy
```

the common workflow can be implemented once in a Shared Library.

---

## 3. What are the main components of a Jenkins Shared Library?

A common Shared Library structure is:

| Directory | Purpose |
|---|---|
| `vars/` | Global variables and custom Pipeline steps |
| `src/` | Reusable Groovy classes |
| `resources/` | Static resources and templates |
| `test/` | Tests, depending on the testing framework |
| `Jenkinsfile` | Optional pipeline for testing/building the library itself |

Example:

```text
jenkins-shared-library/
├── vars/
│   ├── buildJava.groovy
│   ├── securityScan.groovy
│   └── notifyBuild.groovy
├── src/
│   └── com/company/pipeline/
│       ├── MavenUtils.groovy
│       └── AwsUtils.groovy
├── resources/
│   └── com/company/templates/
│       └── deployment.yaml
└── Jenkinsfile
```

---

## 4. What is the purpose of the `vars/` directory?

The `vars/` directory contains scripts exposed as global variables or custom Pipeline steps.

Example:

```text
vars/buildJava.groovy
```

```groovy
def call() {
    sh 'mvn clean package'
}
```

The Jenkinsfile can then call:

```groovy
buildJava()
```

`vars/` is especially useful for simple, reusable Pipeline APIs.

---

## 5. How does a file in `vars/` become a Pipeline step?

The filename becomes the global variable name.

For:

```text
vars/buildJava.groovy
```

you can call:

```groovy
buildJava()
```

If the file contains:

```groovy
def call() {
    sh 'mvn clean package'
}
```

Jenkins executes the `call()` method when `buildJava()` is invoked.

### Parameter example

```groovy
def call(String command = 'mvn clean package') {
    sh command
}
```

Usage:

```groovy
buildJava('mvn clean verify')
```

---

## 6. What is the purpose of `src/`?

The `src/` directory contains reusable Groovy classes.

It is better suited to complex reusable logic than a large `vars/` script.

Example:

```text
src/com/company/pipeline/MavenUtils.groovy
```

```groovy
package com.company.pipeline

class MavenUtils implements Serializable {

    String getVersion() {
        return "1.0"
    }
}
```

This keeps reusable application logic organized into classes.

---

## 7. What is the difference between `vars/` and `src/`?

| Feature | `vars/` | `src/` |
|---|---|---|
| Main purpose | Global Pipeline steps | Reusable Groovy classes |
| Typical usage | `buildJava()` | `new MavenUtils(...)` |
| Pipeline-oriented | Yes | Less directly |
| Best for | Simple reusable Pipeline APIs | Complex reusable logic |
| Structure | Filename becomes global variable | Package/class structure |

A common design is to expose a simple API through `vars/` and keep complex implementation logic in `src/`.

---

## 8. What is the purpose of `resources/`?

`resources/` stores static files used by the Shared Library, such as:

- YAML templates
- JSON files
- Configuration templates
- Other non-Groovy resources

Example:

```text
resources/
└── com/company/templates/deployment.yaml
```

A library can load a resource with:

```groovy
def template = libraryResource('com/company/templates/deployment.yaml')
```

---

## 9. What is the standard Jenkins Shared Library repository structure?

A practical structure is:

```text
shared-library/
├── vars/
│   ├── buildJava.groovy
│   ├── runTests.groovy
│   └── deployApp.groovy
├── src/
│   └── com/company/pipeline/
│       ├── MavenUtils.groovy
│       └── AwsUtils.groovy
├── resources/
│   └── com/company/templates/
│       └── app.yaml
└── Jenkinsfile
```

The exact structure can vary according to the library's design and testing setup.

---

## 10. How do you load a Jenkins Shared Library?

A library can be loaded explicitly using `@Library`:

```groovy
@Library('company-ci') _
```

A specific version can be selected:

```groovy
@Library('company-ci@v2.0.0') _
```

The Jenkinsfile can then use the library's reusable steps and classes.

---

## 11. What does `@Library('company-ci') _` mean?

Consider:

```groovy
@Library('company-ci') _
```

- `company-ci` is the Shared Library name configured in Jenkins.
- `@Library` tells Jenkins to load the library.
- `_` is commonly used to load the library into the Pipeline script without assigning it to a variable.

Example:

```groovy
@Library('company-ci@main') _

buildJava()
```

---

## 12. What is the difference between Global and Folder-level Shared Libraries?

| Scope | Availability |
|---|---|
| Global | Available broadly to Jenkins jobs according to the configured library settings |
| Folder-level | Available to jobs within the relevant Jenkins folder hierarchy |

Folder-level libraries are useful when different teams need different reusable pipeline functionality or governance boundaries.

---

## 13. What is a Global Shared Library?

A Global Shared Library is configured in Jenkins and made available broadly to Jenkins jobs.

A typical configuration is under:

```text
Manage Jenkins
→ System
→ Global Trusted Pipeline Libraries
```

The exact UI wording can vary by Jenkins version and installed plugins.

A library configuration can define:

```text
Name: company-ci
Default version: main
Retrieval method: Modern SCM
SCM: Git
Repository: company shared-library repository
```

---

## 14. What is the difference between trusted and untrusted Shared Libraries?

This is an important Jenkins security concept.

| Type | Execution context | Security consideration |
|---|---|---|
| Trusted | Can execute with Jenkins-level privileges | High impact if compromised |
| Untrusted | Subject to Pipeline sandbox restrictions | More restricted |

Trusted libraries should be controlled like Jenkins administrator code.

If an attacker can modify trusted library code, the impact can be significantly greater than modifying an ordinary application pipeline.

---

## 15. What is the Pipeline Groovy sandbox?

The Jenkins Pipeline sandbox restricts Groovy operations that could perform unsafe actions.

Some Groovy methods or Java APIs may require administrator approval when executed from sandboxed Pipeline code.

This provides a security boundary for untrusted Pipeline scripts.

Trusted library code is not subject to the same sandbox restrictions, which is why trusted libraries require strong access control and code review.

---

## 16. What is a custom Pipeline step?

A custom Pipeline step is reusable Pipeline functionality exposed by a Shared Library.

Example:

```text
vars/notifySlack.groovy
```

```groovy
def call(String message) {
    slackSend(
        channel: '#jenkins-alert',
        message: message
    )
}
```

Jenkinsfile:

```groovy
notifySlack('Build completed successfully')
```

---

## 17. Can a Shared Library step accept parameters?

Yes.

Example:

```groovy
def call(String environment, String version) {
    echo "Deploying ${version} to ${environment}"
}
```

Usage:

```groovy
deployApp('production', '2.5.1')
```

For multiple options, a map is often cleaner:

```groovy
def call(Map config) {
    echo "Environment: ${config.environment}"
    echo "Version: ${config.version}"
}
```

Usage:

```groovy
deployApp(
    environment: 'production',
    version: '2.5.1'
)
```

---

## 18. Can a `vars/` script contain multiple methods?

Yes.

The main Pipeline-facing entry point is commonly `call()`.

Example:

```groovy
def call() {
    validate()
    build()
}

def validate() {
    echo 'Validating'
}

def build() {
    sh 'mvn package'
}
```

The Jenkinsfile can simply call:

```groovy
buildJava()
```

---

## 19. How do you call Jenkins Pipeline steps from a class in `src/`?

Pipeline steps such as `sh`, `echo`, and `checkout` belong to the Pipeline execution context.

A common pattern is to pass the Pipeline script/context into the class.

```groovy
package com.company.pipeline

class MavenUtils implements Serializable {

    private final def steps

    MavenUtils(def steps) {
        this.steps = steps
    }

    void build() {
        steps.sh 'mvn clean package'
    }
}
```

Then:

```groovy
def call() {
    def utils = new com.company.pipeline.MavenUtils(this)
    utils.build()
}
```

This separates reusable classes from the Pipeline DSL.

---

## 20. Why should complex logic be moved from `vars/` to `src/`?

Large `vars/` scripts can become difficult to maintain because they may mix:

- Pipeline orchestration
- Business logic
- API calls
- Configuration handling
- Error handling

A cleaner architecture is:

```text
Jenkinsfile
    ↓
vars/buildApplication.groovy
    ↓
src/com/company/pipeline/*.groovy
```

The `vars/` script becomes a small Pipeline-facing API while `src/` contains reusable classes.

---

## 21. What is a Declarative Pipeline and how does it work with Shared Libraries?

Declarative Pipeline uses the `pipeline {}` syntax.

Example:

```groovy
@Library('company-ci@v1.2.0') _

pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                buildJava()
            }
        }

        stage('Deploy') {
            steps {
                deployApp('dev')
            }
        }
    }
}
```

The Jenkinsfile defines the high-level workflow while the Shared Library supplies reusable functionality.

---

## 22. Can a Shared Library define an entire Declarative Pipeline?

Yes.

Example:

```text
vars/javaServicePipeline.groovy
```

```groovy
def call(Map config = [:]) {
    pipeline {
        agent any

        stages {
            stage('Build') {
                steps {
                    sh 'mvn clean verify'
                }
            }

            stage('Deploy') {
                steps {
                    echo "Deploying ${config.environment}"
                }
            }
        }
    }
}
```

Jenkinsfile:

```groovy
@Library('company-ci@v2.0.0') _

javaServicePipeline(
    environment: 'dev'
)
```

This can enforce a standardized pipeline structure.

---

## 23. What is the difference between a Shared Library step and a Jenkinsfile?

| Jenkinsfile | Shared Library |
|---|---|
| Defines a pipeline for a specific application | Provides reusable pipeline functionality |
| Usually stored with application source | Usually stored in a dedicated repository |
| Application-specific orchestration | Organization-wide reusable logic |
| Changes with the application | Can be maintained centrally |
| Calls library steps | Implements reusable steps |

A good design keeps application-specific configuration in the application repository and reusable behavior in the Shared Library.

---

## 24. How do you version a Jenkins Shared Library?

A Shared Library can be referenced using a Git branch, tag, or commit depending on the SCM configuration and loading method.

Example:

```groovy
@Library('company-ci@v2.1.0') _
```

Release tags are commonly useful for production because they provide an explicit, stable library version.

Using:

```groovy
@Library('company-ci@main') _
```

tracks ongoing changes and can introduce changes into consuming pipelines.

---

## 25. Why is library version pinning important?

Suppose 100 applications use:

```groovy
@Library('company-ci@main') _
```

A breaking change merged into `main` could affect many pipelines.

Using:

```groovy
@Library('company-ci@v2.3.0') _
```

allows a pipeline to remain on a known version until it is intentionally upgraded.

A typical release flow is:

```text
Feature branch
      ↓
Code review
      ↓
main
      ↓
Release tag v2.3.0
      ↓
Production pipelines
```

---

## 26. What happens if no version is specified?

If:

```groovy
@Library('company-ci') _
```

is used, Jenkins uses the library's configured default version.

For deterministic production pipelines, explicitly referencing a controlled release version can be preferable to relying on a moving default branch.

---

## 27. What is the difference between `@Library` and `library()`?

Both can load Shared Libraries, but they are used differently.

### `@Library`

```groovy
@Library('company-ci@v1.0.0') _
```

This is an annotation and is commonly used when library code is needed as part of Jenkinsfile compilation.

### `library()`

```groovy
library 'company-ci@v1.0.0'
```

This loads the library dynamically during Pipeline execution.

Dynamic loading is useful when the library/version needs to be determined at runtime, subject to Jenkins Pipeline constraints.

---

## 28. What is the difference between `@Library` and `libraryResource`?

| Feature | `@Library` | `libraryResource` |
|---|---|---|
| Purpose | Loads Shared Library code | Loads a static resource |
| Used for | Steps/classes | YAML, JSON, templates, files |
| Example | `@Library('company-ci') _` | `libraryResource('com/company/app.yaml')` |

Example:

```groovy
def template = libraryResource(
    'com/company/templates/app.yaml'
)

writeFile(
    file: 'app.yaml',
    text: template
)
```

---

## 29. How can Shared Libraries standardize CI/CD across teams?

A library can define common stages such as:

```text
Checkout
   ↓
Build
   ↓
Unit Test
   ↓
Static Analysis
   ↓
Security Scan
   ↓
Package
   ↓
Publish
   ↓
Deploy
   ↓
Notify
```

For example:

```groovy
standardJavaPipeline(
    sonarProject: 'employee-api',
    environment: 'dev'
)
```

The application repository provides configuration while the organization controls reusable implementation.

---

## 30. How do you implement a reusable Maven build step?

`vars/buildMaven.groovy`:

```groovy
def call(String command = 'mvn clean verify') {
    sh command
}
```

Jenkinsfile:

```groovy
@Library('company-ci@v1.0.0') _

pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                buildMaven()
            }
        }
    }
}
```

The command can be overridden:

```groovy
buildMaven('mvn clean package -DskipTests')
```

---

## 31. How would you create a reusable Terraform deployment step?

Example:

```text
vars/terraformDeploy.groovy
```

```groovy
def call(String directory) {
    dir(directory) {
        sh 'terraform init'
        sh 'terraform validate'
        sh 'terraform plan -out=tfplan'
        sh 'terraform apply tfplan'
    }
}
```

Jenkinsfile:

```groovy
terraformDeploy('infra')
```

In production, approval, credentials, backend configuration, error handling, and plan artifact handling should be designed explicitly.

---

## 32. How do you handle credentials in a Shared Library?

Credentials should be stored in Jenkins Credentials rather than hardcoded in the library repository.

Example:

```groovy
withCredentials([
    usernamePassword(
        credentialsId: 'aws-deploy-user',
        usernameVariable: 'AWS_ACCESS_KEY_ID',
        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
    )
]) {
    sh 'aws sts get-caller-identity'
}
```

Do not commit secrets such as passwords or access keys into the Shared Library repository.

For AWS workloads, IAM roles or short-lived credentials are generally preferable to long-lived access keys when supported by the Jenkins execution environment.

---

## 33. What is the security risk of putting secrets in a Shared Library?

A Shared Library can be consumed by many pipelines, so exposing secrets in source code creates a broad security risk.

Bad:

```groovy
def call() {
    def password = 'MySecretPassword'
}
```

Better:

```groovy
withCredentials([
    string(
        credentialsId: 'db-password',
        variable: 'DB_PASSWORD'
    )
]) {
    sh './deploy.sh'
}
```

Secrets should remain in Jenkins Credentials or an appropriate external secret-management system.

---

## 34. How do you prevent a Shared Library from becoming a single point of failure?

Use:

- Git repository availability and backup
- Versioned releases
- Backward-compatible APIs
- Automated testing
- Code review
- Branch protection
- Release tags
- Controlled rollout
- Clear ownership
- Documentation

Pinning production pipelines to known library versions also reduces the blast radius of a bad library release.

---

## 35. How do you test a Jenkins Shared Library?

Shared Libraries can be tested with frameworks such as JenkinsPipelineUnit.

A conceptual test can verify that the expected Pipeline step is invoked:

```groovy
helper.registerAllowedMethod(
    'sh',
    [String],
    { command ->
        assert command == 'mvn clean verify'
    }
)
```

The exact test implementation depends on the testing framework and project setup.

---

## 36. What should you test in a Shared Library?

| Area | Example |
|---|---|
| Normal execution | Build succeeds |
| Parameters | Correct environment/version is passed |
| Failure handling | Build command fails correctly |
| Conditional logic | Production approval is required |
| Credentials | Correct credential binding is used |
| Notifications | Success/failure notification is triggered |
| External commands | Correct commands are generated |
| Regression | Existing library behavior remains intact |

The goal is to catch library defects before they affect multiple consuming pipelines.

---

## 37. How should a Shared Library handle errors?

A library should fail clearly and should not silently hide deployment failures.

Example:

```groovy
try {
    sh 'mvn clean verify'
} catch (Exception e) {
    echo "Build failed: ${e.message}"
    throw e
}
```

Re-throwing the exception preserves the failed build status.

`catchError` can be used when the desired Pipeline behavior is to record a failure while allowing subsequent stages or post-processing to continue.

---

## 38. How can you implement notifications in a Shared Library?

Example:

```text
vars/notifyBuild.groovy
```

```groovy
def call(String status) {
    slackSend(
        channel: '#jenkins-alert',
        message: "Build status: ${status}"
    )
}
```

Jenkinsfile:

```groovy
post {
    success {
        notifyBuild('SUCCESS')
    }

    failure {
        notifyBuild('FAILURE')
    }
}
```

This centralizes notification formatting and configuration.

---

## 39. Can Shared Libraries use Jenkins plugins?

Yes.

Shared Library code can call Pipeline steps exposed by installed Jenkins plugins.

Examples include integrations for:

- Slack
- SonarQube
- Kubernetes
- AWS
- Credentials Binding

The required plugin must be installed and configured in Jenkins.

A Shared Library does not automatically install the plugins it depends on.

---

## 40. What happens if a Shared Library calls a plugin step that is not installed?

The Pipeline can fail because Jenkins cannot resolve or execute the required step.

For example:

```groovy
slackSend(...)
```

requires the appropriate Slack integration/plugin and configuration.

Shared Libraries should therefore document their plugin dependencies and expected Jenkins configuration.

---

## 41. How do you design a good Jenkins Shared Library?

A practical design is:

```text
Jenkinsfile
    |
    v
Shared Library API
    |
    +-- vars/
    |     ├── buildJava.groovy
    |     ├── securityScan.groovy
    |     └── deployApp.groovy
    |
    v
src/
    ├── MavenUtils.groovy
    ├── AwsUtils.groovy
    └── DeploymentUtils.groovy
    |
    v
resources/
    └── templates/
```

Good principles:

1. Keep the Jenkinsfile application-specific.
2. Put reusable Pipeline APIs in `vars/`.
3. Put complex reusable classes in `src/`.
4. Keep static templates in `resources/`.
5. Keep secrets outside Git.
6. Version production releases.
7. Test library changes before rollout.
8. Avoid breaking existing library APIs.

---

## 42. How would you design a Shared Library for multiple application types?

A library can expose standardized entry points:

```groovy
javaPipeline(
    application: 'salary-api',
    environment: 'dev'
)
```

```groovy
pythonPipeline(
    application: 'attendance-api',
    environment: 'dev'
)
```

```groovy
terraformPipeline(
    directory: 'terraform/network'
)
```

Internally, common functionality can be shared:

```text
vars/
├── javaPipeline.groovy
├── pythonPipeline.groovy
├── terraformPipeline.groovy
└── notifyBuild.groovy

src/com/company/pipeline/
├── BuildUtils.groovy
├── SecurityUtils.groovy
└── DeploymentUtils.groovy
```

This gives teams a standardized interface without forcing every technology into one large Pipeline script.

---

## 43. What is the difference between a Shared Library and a Jenkins plugin?

| Shared Library | Jenkins Plugin |
|---|---|
| Usually Groovy-based | Usually implemented as Jenkins plugin code |
| Stored in SCM | Packaged and installed in Jenkins |
| Reuses Pipeline logic | Extends Jenkins functionality |
| Easier to change through Git | More involved development/release process |
| Best for organization-specific pipeline logic | Best for adding Jenkins platform capabilities |

Use a Shared Library when the requirement is reusable CI/CD orchestration. Use a plugin when Jenkins itself needs new platform functionality.

---

## 44. What are common mistakes when implementing Jenkins Shared Libraries?

Common mistakes include:

- Putting secrets in Git
- Making every library trusted
- Keeping all logic in one huge `vars/` file
- Using moving library versions for production without a controlled release strategy
- Breaking existing function signatures
- Calling plugin steps without documenting dependencies
- Not testing shared code
- Coupling the library tightly to one application
- Hardcoding environment-specific values
- Making the library responsible for application-specific business logic

A Shared Library should provide a stable automation interface rather than becoming an unmaintainable replacement for every Jenkinsfile.

---

## 45. Explain a real-world Jenkins Shared Library architecture.

A practical enterprise architecture can look like:

```text
Application Repository
        |
        | Jenkinsfile
        v
+---------------------------+
| Jenkins                   |
|                           |
| @Library('company-ci')    |
+-------------+-------------+
              |
              v
+---------------------------+
| Jenkins Shared Library    |
|                           |
| vars/                     |
|  - buildJava              |
|  - securityScan           |
|  - deployApp              |
|  - notifyBuild            |
|                           |
| src/                      |
|  - AwsUtils               |
|  - MavenUtils             |
|  - DeploymentUtils        |
|                           |
| resources/                |
|  - templates              |
+-------------+-------------+
              |
       +------+------+
       |             |
       v             v
      AWS        External Tools
                 SonarQube
                 Trivy
                 Slack
```

### Example Jenkinsfile

```groovy
@Library('company-ci@v2.1.0') _

pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                buildJava('mvn clean verify')
            }
        }

        stage('Security Scan') {
            steps {
                securityScan()
            }
        }

        stage('Deploy') {
            steps {
                deployApp(
                    environment: 'dev',
                    application: 'salary-api'
                )
            }
        }
    }

    post {
        success {
            notifyBuild('SUCCESS')
        }

        failure {
            notifyBuild('FAILURE')
        }
    }
}
```

The Jenkinsfile stays small because reusable implementation is centralized in the Shared Library.

---

## 46. What is the most important advantage of Jenkins Shared Libraries in large organizations?

The major advantage is **standardized, reusable automation with centralized governance**.

For example:

```text
Every application pipeline
        |
        v
securityScan()
        |
        +-- SAST
        +-- Dependency Scan
        +-- Secret Scan
        +-- Container Scan
```

If the organization changes its security standard, the implementation can be updated centrally instead of manually modifying dozens or hundreds of Jenkinsfiles.

However, centralized change also creates risk. Library changes should therefore be versioned, tested, reviewed, and rolled out in a controlled manner.

---

# Quick Reference

## Shared Library Directory Structure

| Directory | Main Purpose | Typical Example |
|---|---|---|
| `vars/` | Global Pipeline steps | `buildJava()` |
| `src/` | Reusable Groovy classes | `MavenUtils` |
| `resources/` | Static resources/templates | `deployment.yaml` |
| `test/` | Tests, depending on test setup | `BuildJavaTest.groovy` |

## Library Loading

```groovy
@Library('company-ci@v2.1.0') _
```

or:

```groovy
library 'company-ci@v2.1.0'
```

## Typical Shared Library Flow

```text
Application Jenkinsfile
        ↓
Load Shared Library
        ↓
Call reusable Pipeline step
        ↓
vars/
        ↓
src/ classes
        ↓
Jenkins Pipeline steps
        ↓
Build / Test / Scan / Deploy / Notify
```

## Key Interview Mental Model

> **`vars/` exposes the reusable Pipeline API, `src/` contains reusable Groovy classes, and `resources/` stores static files used by the library.**
