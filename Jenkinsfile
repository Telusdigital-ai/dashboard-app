pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'flask-dashboard'
        DOCKER_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'python -m pip install --upgrade pip'
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'python -m pytest tests/'
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
                    // Stop existing container if running
                    sh '''
                        docker ps -q --filter "name=${DOCKER_IMAGE}" | grep -q . && docker stop ${DOCKER_IMAGE} && docker rm ${DOCKER_IMAGE} || echo "No container running"
                    '''
                    
                    // Run new container
                    sh """
                        docker run -d \
                            --name ${DOCKER_IMAGE} \
                            -p 5000:5000 \
                            ${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded! Application deployed successfully.'
        }
        failure {
            echo 'Pipeline failed! Check the logs for details.'
        }
    }
}
