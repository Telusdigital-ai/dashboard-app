pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // SCM checkout step (preferred)
                git url: 'https://github.com/Telusdigital-ai/dashboard-app', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                sh 'ls -la'
                sh 'git status'
            }
        }
    }
}
