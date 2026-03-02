# Jenkins Pipeline with Shared Library and Email Notifications

This repository contains a Jenkins declarative pipeline that uses a shared library to handle common CI/CD tasks such as Java setup, secrets scanning, OWASP dependency scanning, unit testing with coverage, packaging, artifact archiving, and email notifications for every build.

## Pipeline Overview

```groovy
@Library('jenkins-shared-library') _

pipeline {
    agent any

    environment {
        APP_NAME   = 'spring3hibernate'
        JAVA_HOME  = '/usr/lib/jvm/java-8-openjdk-amd64'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/opstree/spring3hibernate.git', branch: 'master'
            }
        }

        stage('Java Setup') {
            steps {
                javaSetup()
            }
        }

        stage('Secrets Scan - GitLeaks') {
            steps {
                gitLeaks()
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                owaspScan(
                    project: env.APP_NAME,
                    nvdApiKey: credentials('nvd-api-key'),
                    failOnCVSS: 7
                )
            }
        }

        stage('Unit Tests + Coverage') {
            steps {
                mavenCoberturaTest()
            }
        }

        stage('Package Artifact') {
            steps {
                mvnPackage()
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifact()
            }
        }
    }

    post {
        always {
            notifyEmail(currentBuild.currentResult)
            echo "Pipeline finished with status: ${currentBuild.currentResult}"
        }
    }
}
```

## Shared Library Vars

### 1. `javaSetup.groovy`

Sets up Java environment.

```groovy
def call() {
    echo "Setting up Java environment: ${env.JAVA_HOME}"
    env.PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
}
```

### 2. `gitLeaks.groovy`

Runs GitLeaks scan to detect secrets.

```groovy
def call() {
    sh 'gitleaks detect --source . --report-format json --report-path gitleaks.json || true'
}
```

### 3. `owaspScan.groovy`

Runs OWASP Dependency Check.

```groovy
def call(Map config) {
    if (!config.nvdApiKey) {
        echo "OWASP NVD API key not provided, skipping scan"
        return
    }
    def projectName = config.get('project', 'app')
    def failScore   = config.get('failOnCVSS', 7)
    def scanDir     = config.get('scanDir', '.')
    def outDir      = config.get('outDir', 'dependency-check-report')

    withEnv(["NVD_API_KEY=${config.nvdApiKey}"]) {
        sh """
        dependency-check.sh \
          --project ${projectName} \
          --scan ${scanDir} \
          --format XML \
          --out ${outDir} \
          --nvdApiKey \$NVD_API_KEY \
          --failOnCVSS ${failScore}
        """
    }
}
```

### 4. `mavenCoberturaTest.groovy`

Runs Maven tests and Cobertura coverage.

```groovy
def call() {
    sh 'mvn test cobertura:cobertura'
}
```

### 5. `mvnPackage.groovy`

Packages the project using Maven.

```groovy
def call(Map config = [:]) {
    def skipTests = config.get('skipTests', true)
    def mvnGoal = skipTests ? 'clean package -DskipTests' : 'clean package'
    stage('Package Artifact') {
        sh "mvn ${mvnGoal}"
    }
}
```

### 6. `archiveArtifact.groovy`

Archives generated artifacts.

```groovy
def call(String pattern = '**/target/*.jar') {
    stage('Archive Artifact') {
        archiveArtifacts artifacts: pattern, fingerprint: true
    }
}
```

### 7. `notifyEmail.groovy`

Sends email notification for each build.

```groovy
def call(String status = 'SUCCESS') {
    def subjectPrefix = status == 'SUCCESS' ? '✅ SUCCESS' : '❌ FAILURE'

    emailext(
        to: 'your-email@example.com',  // replace with actual recipient
        subject: "${subjectPrefix}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """
        <h3>Build ${status}</h3>
        <p><b>Job:</b> ${env.JOB_NAME}</p>
        <p><b>Build:</b> #${env.BUILD_NUMBER}</p>
        <p><b>Branch:</b> ${env.GIT_BRANCH ?: env.BRANCH_NAME ?: 'N/A'}</p>
        <p><b>Build URL:</b> <a href='${env.BUILD_URL}'>${env.BUILD_URL}</a></p>
        """,
        mimeType: 'text/html',
        attachLog: status != 'SUCCESS'
    )
}
```

## Notes

* This pipeline ensures every build, whether success or failure, triggers an email notification.
* Artifact archiving is handled via the shared library function `archiveArtifact`.
* All common CI/CD tasks are modularized into shared library vars for reusability.
* Update the email recipient in `notifyEmail.groovy` to your intended recipients.
