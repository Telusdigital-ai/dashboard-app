pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'flask-dashboard-app'
        DOCKER_TAG = "${BUILD_NUMBER}"
        // Add any environment variables your app needs
        // DATABASE_URL = credentials('database-url-credential-id')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }
        
        stage('Test') {
            steps {
                sh 'python -m pytest tests/'
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    // Stop existing container if running
                    sh 'docker ps -q --filter "name=flask-dashboard" | grep -q . && docker stop flask-dashboard && docker rm flask-dashboard || echo "No container running"'
                    
                    // Run new container
                    sh "docker run -d --name flask-dashboard -p 8080:8080 ${DOCKER_IMAGE}:${DOCKER_TAG}"
                }
            }
        }
    }
    
    post {
        failure {
            echo 'Pipeline failed!'
        }
    }
}
