@Library('frontend-ci-checks') _

pipeline {
    agent any

    parameters {
        string(name: 'GIT_REPO',      defaultValue: 'https://github.com/OT-MICROSERVICES/frontend.git', description: 'Git Repository URL')
        string(name: 'GIT_BRANCH',    defaultValue: 'main',                                             description: 'Git Branch')
        string(name: 'TARGET_URL',    defaultValue: ' http://13.204.84.8/',                             description: 'Target Application URL to scan')
        string(name: 'EMAIL_TO',      defaultValue: 'jenkinsotms@gmail.com',                            description: 'Email Recipient')
        string(name: 'SLACK_CHANNEL', defaultValue: '#build-status',                                    description: 'Slack Channel')
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Repository') {
            steps {
                script {
                    frontendCI.checkout(params.GIT_REPO, params.GIT_BRANCH)
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    frontendCI.install()
                }
            }
        }

        stage('Build (Webpack)') {
            steps {
                script {
                    frontendCI.build()
                }
            }
        }

        stage('Unit Testing (Jest)') {
            steps {
                script {
                    frontendCI.test()
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    frontendCI.sonar("frontend-bhawna", "SonarQube")
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    frontendCI.qualityGate()
                }
            }
        }

        stage('Dependency Scan (Trivy)') {
            steps {
                script {
                    frontendCI.trivy()
                }
            }
        }

        stage('DAST Scan') {
            steps {
                script {
                    frontendCI.setupJava()
                    frontendCI.downloadZAP()
                    frontendCI.startZAP()
                    frontendCI.spiderScan(params.TARGET_URL)
                    frontendCI.activeScan(params.TARGET_URL)
                    frontendCI.generateReport('zap_report.html')
                }
            }
        }
    }

    post {
        always {
            script {
                // Shutdown ZAP daemon to release resources and cleanup
                frontendCI.stopZAP()

                // Archive reports
                archiveArtifacts artifacts: 'zap_report.html', allowEmptyArchive: true
            }
        }
        success {
            script {
                echo "Final Build Status: SUCCESS"
                frontendCI.notifications('SUCCESS', params.SLACK_CHANNEL, params.EMAIL_TO)
            }
        }
        failure {
            script {
                echo "Final Build Status: FAILED"
                frontendCI.notifications('FAILED', params.SLACK_CHANNEL, params.EMAIL_TO)
            }
        }
    }
}
