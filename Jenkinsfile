pipeline {
    agent any

    options {
        timestamps()
    }
    
    environment {
        DOCKER_IMAGE = 'flask-dashboard'
        DOCKER_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Prepare') {
            steps {
                sh 'chmod -R 777 .'
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Test') {
            steps {
                script {
                    try {
                        // Build Docker image
                        sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."

                        // Run tests inside a container
                        sh """
                            docker run --rm \
                                -v \$(pwd)/tests:/app/tests \
                                ${DOCKER_IMAGE}:${DOCKER_TAG} \
                                python3 -m pytest tests/
                        """
                    } catch (Exception e) {
                        error "Build or test failed: ${e.getMessage()}"
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    try {
                        // Stop existing container if running
                        sh '''
                            container_id=$(docker ps -q --filter "name=${DOCKER_IMAGE}")
                            if [ ! -z "$container_id" ]; then
                                docker stop $container_id
                                docker rm $container_id
                            fi
                        '''
                        
                        // Run new container
                        sh """
                            docker run -d \
                                --name ${DOCKER_IMAGE} \
                                -p 5000:5000 \
                                --restart unless-stopped \
                                ${DOCKER_IMAGE}:${DOCKER_TAG}
                        """

                        // Verify container is running
                        sh '''
                            sleep 5
                            if ! docker ps | grep -q ${DOCKER_IMAGE}; then
                                echo "Container failed to start"
                                exit 1
                            fi
                        '''
                    } catch (Exception e) {
                        error "Deployment failed: ${e.getMessage()}"
                    }
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
            script {
                // Cleanup on failure
                sh '''
                    container_id=$(docker ps -q --filter "name=${DOCKER_IMAGE}")
                    if [ ! -z "$container_id" ]; then
                        docker stop $container_id
                        docker rm $container_id
                    fi
                '''
            }
        }
        always {
            script {
                try {
                    // Clean up old images
                    sh 'docker image prune -f'
                } catch (Exception e) {
                    echo "Warning: Image cleanup failed: ${e.getMessage()}"
                }
            }
        }
    }
}
