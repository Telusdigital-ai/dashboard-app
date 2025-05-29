pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', 
                    branches: [[name: '*/main']], 
                    userRemoteConfigs: [[url: 'https://github.com/Telusdigital-ai/dashboard-app']]])
            }
        }
        stage('Hello') {
            steps {
                echo 'Hello from Jenkins!'
            }
        }
    }
}
