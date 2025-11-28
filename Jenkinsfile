pipeline {
    agent any
    environment {
	SSH_OPTS = '-o StrictHostKeyChecking=no -o PubkeyAuthentication=no -o PreferredAuthentications=password -o IdentitiesOnly=yes'
        DOCKER_HUB_USERNAME = 'prithv33' // Your Docker Hub Username
        VM_HOST = '3.109.157.120'       // Public IP of your Ubuntu VM
        VM_USER = 'ubuntu'           // SSH user for your VM

        DOCKER_HUB_CREDENTIALS = 'dockerhub-creds'
        SSH_CREDENTIALS = 'vm-ssh-creds'


        BACKEND_IMAGE = "${DOCKER_HUB_USERNAME}/mean-backend"
        FRONTEND_IMAGE = "${DOCKER_HUB_USERNAME}/mean-frontend"
        REMOTE_APP_DIR = "/home/${VM_USER}/mean-app"
    }
    stages {
        stage('Build Backend Image') {
            steps {
                script {
                    sh "/usr/bin/docker build -t ${BACKEND_IMAGE}:${BUILD_NUMBER} ./backend"
                }
            }
        }
        stage('Build Frontend Image') {
            steps {
                script {
                    sh "/usr/bin/docker build -t ${FRONTEND_IMAGE}:${BUILD_NUMBER} ./frontend"
                }
            }
        }
        stage('Tag and Push Images to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKER_HUB_CREDENTIALS, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USER')]) {
                    sh "/usr/bin/docker login -u ${DOCKER_USER} -p ${DOCKER_PASSWORD}"
                    
                    // Tag with 'latest' and build number
                    sh "/usr/bin/docker tag ${BACKEND_IMAGE}:${BUILD_NUMBER} ${BACKEND_IMAGE}:latest"
                    sh "/usr/bin/docker tag ${FRONTEND_IMAGE}:${BUILD_NUMBER} ${FRONTEND_IMAGE}:latest"
                    
                    // Push all tags
                    sh "/usr/bin/docker push ${BACKEND_IMAGE}:${BUILD_NUMBER}"
                    sh "/usr/bin/docker push ${BACKEND_IMAGE}:latest"
                    sh "/usr/bin/docker push ${FRONTEND_IMAGE}:${BUILD_NUMBER}"
                    sh "/usr/bin/docker push ${FRONTEND_IMAGE}:latest"
                }
            }
        }
        stage('Deploy to VM') {
            steps {
                
                withCredentials([usernamePassword(credentialsId: 'vm-password-creds', usernameVariable: 'VM_USER', passwordVariable: 'VM_PASSWORD')]) { // <--- NEW STRUCTURE

                    // 1. Create directory on remote VM
                    sh "sshpass -p ${VM_PASSWORD} ssh ${SSH_OPTS} ${VM_USER}@${VM_HOST} 'mkdir -p ${REMOTE_APP_DIR}'"

                    // 2. Copy docker-compose.yml to the remote VM
                    sh "sshpass -p ${VM_PASSWORD} scp ${SSH_OPTS} docker-compose.yml ${VM_USER}@${VM_HOST}:${REMOTE_APP_DIR}/docker-compose.yml"
                    
                    // 3. Log in to Docker Hub on the VM
                    sh "sshpass -p ${VM_PASSWORD} ssh ${SSH_OPTS} ${VM_USER}@${VM_HOST} 'docker login -u ${DOCKER_HUB_USERNAME} -p \$(echo ${DOCKER_PASSWORD})'" 

                    // 4. Pull latest images and restart containers
                    sh "sshpass -p ${VM_PASSWORD} ssh ${SSH_OPTS} ${VM_USER}@${VM_HOST} 'cd ${REMOTE_APP_DIR} && docker compose pull && docker compose up -d'"

                    
                    echo "Deployment completed. Application is accessible at http://${VM_HOST}"
                } 
            }
        }
    }
}
