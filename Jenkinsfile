pipeline {
    agent any

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/OT-MICROSERVICES/employee-api.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'go mod tidy'
            }
        }

        stage('GO Compilation Check') {
            steps {
                sh 'go build -o employee-api'
            }
        }

    }

    post {

        success {
            echo 'GO Build Successful'
        }

        failure {
            echo 'GO Build Failed'
        }

    }
}
