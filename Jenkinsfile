pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'flask-dashboard'
        DOCKER_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', 
                    branches: [[name: 'refs/heads/main']], // Explicitly specify main branch
                    userRemoteConfigs: [[
                        url: 'https://github.com/Telusdigital-ai/dashboard-app',
                        
                    ]]
                ])
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    try {
                        sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    } catch (Exception e) {
                        error "Failed to build Docker image: ${e.getMessage()}"
                    }
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    try {
                        sh """
                            docker run --rm \
                                -v \$(pwd)/tests:/app/tests \
                                ${DOCKER_IMAGE}:${DOCKER_TAG} \
                                python3 -m pytest tests/
                        """
                    } catch (Exception e) {
                        error "Tests failed: ${e.getMessage()}"
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
                        
                        // Deploy new container
                        sh """
                            docker run -d \
                                --name ${DOCKER_IMAGE} \
                                -p 5000:5000 \
                                --restart unless-stopped \
                                ${DOCKER_IMAGE}:${DOCKER_TAG}
                        """
                        
                        // Verify deployment
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
            echo 'Access your application at http://localhost:5000'
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
                // Cleanup old images
                sh 'docker image prune -f'
            }
        }
    }
}
