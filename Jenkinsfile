pipeline {
    agent any

    environment {
        DOCKER_USERNAME = 'smya13'
        BACKEND_IMAGE   = 'spider-backend'
        FRONTEND_IMAGE  = 'spider-frontend'
        IMAGE_TAG       = 'latest'
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Images') {
            steps {
                sh '''
                  docker build -t $DOCKER_USERNAME/$BACKEND_IMAGE:$IMAGE_TAG ./Backend
                  docker build -t $DOCKER_USERNAME/$FRONTEND_IMAGE:$IMAGE_TAG ./Frontend
                '''
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                withCredentials([
                    string(credentialsId: 'dockerhub-password', variable: 'DOCKER_PASSWORD')
                ]) {
                    sh '''
                      echo "$DOCKER_PASSWORD" | docker login -u $DOCKER_USERNAME --password-stdin
                      docker push $DOCKER_USERNAME/$BACKEND_IMAGE:$IMAGE_TAG
                      docker push $DOCKER_USERNAME/$FRONTEND_IMAGE:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy to AWS EC2') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'ec2-ssh-key',
                        keyFileVariable: 'SSH_KEY'
                    )
                ]) {
                    sh """
ssh -o StrictHostKeyChecking=no -i \$SSH_KEY ubuntu@3.109.1.112 << EOF
cd ~/Spider-Devops-Induction
docker-compose pull
docker-compose down
docker-compose up -d
EOF
"""
                }
            }
        }
    }
}

