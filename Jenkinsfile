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

        stage('Build and Test') {
            steps {
                script {
                    // Build the Docker image
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")

                    // Run tests inside a container
                    docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").inside {
                        sh 'python -m pytest tests/'
                    }
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
        always {
            // Clean up old images
            sh 'docker image prune -f'
        }
    }
}
