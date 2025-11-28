pipeline {
    agent any
    environment {
        // VM_HOST is kept for the final success message and is not used for connection
        DOCKER_HUB_USERNAME = 'prithv33' 
        VM_HOST = '3.109.157.120' 
        VM_USER = 'ubuntu' // Used for setting the correct deployment path

        // Only Docker Hub credential needed now
        DOCKER_HUB_CREDENTIALS = 'dockerhub-creds' 
        
        BACKEND_IMAGE = "${DOCKER_HUB_USERNAME}/mean-backend"
        FRONTEND_IMAGE = "${DOCKER_HUB_USERNAME}/mean-frontend"
        // This path is local on the VM where Jenkins is running
        REMOTE_APP_DIR = "/home/${VM_USER}/mean-app" 
    }
    stages {
        stage('Setup Node.js Tool') {
            // Must match the name set in Manage Jenkins > Global Tool Configuration
            tools {
                nodejs 'NodeJS-18' 
            }
            steps {
                echo 'Node.js environment loaded.'
            }
        }
        stage('Build Backend Image') {
            steps {
                // Running Docker directly on the local VM's daemon
                sh "docker build -t ${BACKEND_IMAGE}:${BUILD_NUMBER} ./backend"
            }
        }
        stage('Build Frontend Image') {
            steps {
                sh "docker build -t ${FRONTEND_IMAGE}:${BUILD_NUMBER} ./frontend"
            }
        }
        stage('Tag and Push Images to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKER_HUB_CREDENTIALS, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USER')]) {
                    sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASSWORD}"

                    sh "docker tag ${BACKEND_IMAGE}:${BUILD_NUMBER} ${BACKEND_IMAGE}:latest"
                    sh "docker tag ${FRONTEND_IMAGE}:${BUILD_NUMBER} ${FRONTEND_IMAGE}:latest"

                    sh "docker push ${BACKEND_IMAGE}:${BUILD_NUMBER}"
                    sh "docker push ${BACKEND_IMAGE}:latest"
                    sh "docker push ${FRONTEND_IMAGE}:${BUILD_NUMBER}"
                    sh "docker push ${FRONTEND_IMAGE}:latest"
                }
            }
        }
        stage('Deploy Locally on VM') {
            steps {
                // Copy the necessary file and run Docker Compose in the target directory
                sh """
                    
                    mkdir -p ${REMOTE_APP_DIR}
                    
                    
                    cp docker-compose.yml ${REMOTE_APP_DIR}/docker-compose.yml

                    cd ${REMOTE_APP_DIR}
                    
                    docker compose pull
                    docker compose up -d

                    echo "Deployment successful on VM."
                """
                echo "Application is accessible at http://${VM_HOST}"
            }
        }
    }
}
