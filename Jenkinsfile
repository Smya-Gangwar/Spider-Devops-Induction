pipeline {
    agent any

    environment {
        DOCKER_USERNAME = 'smya13'
        BACKEND_IMAGE   = 'spider-backend'
        FRONTEND_IMAGE  = 'spider-frontend'
    }

    stages {

        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Init Docker Buildx') {
            steps {
                sh '''
                  docker buildx create --name multiarch-builder --use || true
                  docker buildx inspect --bootstrap
                '''
            }
        }

        stage('Build & Push Multi-Arch Images') {
            steps {
                withCredentials([
                    string(credentialsId: 'dockerhub-password', variable: 'DOCKER_PASSWORD')
                ]) {
                    sh '''
                      echo "$DOCKER_PASSWORD" | docker login -u $DOCKER_USERNAME --password-stdin

                      docker buildx build \
                        --platform linux/amd64,linux/arm64 \
                        -t $DOCKER_USERNAME/$BACKEND_IMAGE:latest \
                        --push ./Backend

                      docker buildx build \
                        --platform linux/amd64,linux/arm64 \
                        -t $DOCKER_USERNAME/$FRONTEND_IMAGE:latest \
                        --push ./Frontend
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

