pipeline 
{
    agent any

    environment 
    {
        DOCKER_USERNAME = 'smya13'
        BACKEND_IMAGE   = 'spider-backend'
        FRONTEND_IMAGE  = 'spider-frontend'
        IMAGE_TAG       = "${env.BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') 
        {
            steps 
            {
                checkout scm
            }
        }

        stage('Build Images')
        {
            steps
            {
                sh '''
                  docker build -t $DOCKER_USERNAME/$BACKEND_IMAGE:$IMAGE_TAG ./Backend
                  docker build -t $DOCKER_USERNAME/$FRONTEND_IMAGE:$IMAGE_TAG ./Frontend
                '''
            }
        }

        stage('Push Images to Docker Hub') 
        {
            steps 
            {
                withCredentials([string(credentialsId: 'dockerhub-password', variable: 'DOCKER_PASSWORD')]) 
                {
                    sh '''
                      echo "$DOCKER_PASSWORD" | docker login -u $DOCKER_USERNAME --password-stdin
                      docker push $DOCKER_USERNAME/$BACKEND_IMAGE:$IMAGE_TAG
                      docker push $DOCKER_USERNAME/$FRONTEND_IMAGE:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy using Docker Compose') 
        {
            steps 
            {
                sh '''
                  docker-compose down -v --remove-orphans
                  docker-compose pull
                  docker-compose up -d
                '''
            }
        }
    }

    post
    {
        success
        {
            echo 'CI/CD Pipeline Completed Successfully!'
        }
        failure
        {
            echo 'Pipeline Failed.'
        }
    }
}
