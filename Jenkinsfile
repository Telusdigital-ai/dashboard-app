pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'dashboard-app'
        DOCKER_TAG = 'latest'
        CONTAINER_NAME = 'dashboard-container'
        APP_PORT = '5003'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh '''
                    python -m pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }
        
        stage('Run Tests') {
            steps {
                sh '''
                    python -m pytest tests/ || true
                '''
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    // Stop and remove existing container if it exists
                    sh '''
                        docker stop ${CONTAINER_NAME} || true
                        docker rm ${CONTAINER_NAME} || true
                    '''
                    
                    // Run new container
                    sh '''
                        docker run -d \
                            --name ${CONTAINER_NAME} \
                            --network jenkins-network \
                            -p ${APP_PORT}:5003 \
                            -e AUTH0_CLIENT_ID=${AUTH0_CLIENT_ID} \
                            -e AUTH0_CLIENT_SECRET=${AUTH0_CLIENT_SECRET} \
                            -e AUTH0_DOMAIN=${AUTH0_DOMAIN} \
                            ${DOCKER_IMAGE}:${DOCKER_TAG}
                    '''
                }
            }
        }
    }
    
    post {
        failure {
            echo 'Pipeline failed! Check the logs for details.'
        }
        success {
            echo 'Pipeline succeeded! Application deployed successfully.'
        }
    }
}
