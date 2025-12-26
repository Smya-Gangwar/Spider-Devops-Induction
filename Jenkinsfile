pipeline
{
    agent any

    environment
    {
        DOCKER_USERNAME = credentials('dockerhub-username')
        IMAGE_NAME = 'spider-devops-induction'
        IMAGE_TAG = 'latest'
    }

    stages
    {
        stage('Checkout Code')
        {
            steps
            {
                checkout scm
            }
        }

        stage('Build Docker Image')
        {
            steps
            {
                sh '''
                    docker build -t $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Push Image to Docker Hub')
        {
            steps
            {
                withCredentials([string(credentialsId: 'dockerhub-password', variable: 'DOCKER_PASSWORD')])
                {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u $DOCKER_USERNAME --password-stdin
                        docker push $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy Locally using Docker Compose')
        {
            steps
            {
                sh '''
                    docker-compose down
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