pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_REPO = 'vibha555/appointmentservice'
        DOCKER_CREDENTIALS = 'dockerhub_creds'
        IMAGE_TAG = "${BUILD_NUMBER}"
        LATEST_TAG = 'latest'
    }
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
        timestamps()
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                script {
                    echo "Checking out code from GitHub..."
                    checkout(
                        [
                            $class: 'GitSCM',
                            branches: [[name: '*/main']],
                            userRemoteConfigs: [[
                                url: 'https://github.com/vibha555/AppointmentService.git',
                                credentialsId: 'github-credentials' // Optional: use if private repo
                            ]]
                        ]
                    )
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${DOCKER_REPO}:${IMAGE_TAG}" 
                        sh '''
                            docker build \
                                -t ${DOCKER_REPO}:${IMAGE_TAG} \
                                -t ${DOCKER_REPO}:${LATEST_TAG} \
                                -f Dockerfile .
                        '''

                    echo "✓ Docker image built successfully"
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    echo "Pushing image to Docker Hub..."
                    withCredentials([usernamePassword(
                        credentialsId: "${DOCKER_CREDENTIALS}",
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )]) {
                        sh '''
                            echo "${DOCKER_PASSWORD}" | docker login -u "${DOCKER_USERNAME}" --password-stdin
                            docker push ${DOCKER_REPO}:${IMAGE_TAG}
                            docker push ${DOCKER_REPO}:${LATEST_TAG}
                            docker logout
                        '''
                    }
                    echo "✓ Image pushed successfully to Docker Hub"
                }
            }
        }
    }
    
    post {
        success {
            echo "✓ Pipeline completed successfully!"
            echo "Image available at: ${DOCKER_REPO}:${IMAGE_TAG}"
        }
        failure {
            echo "✗ Pipeline failed. Check logs above for details."
        }
        always {
            echo "Cleaning up workspace..."
            cleanWs()
        }
    }
}
