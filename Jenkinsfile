pipeline {
    agent any

    environment {
        IMAGE_NAME = "incident-app"
        CONTAINER_NAME = "incident-container"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Pulling latest code from GitHub...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t ${IMAGE_NAME} .'
            }
        }

        stage('Test') {
            steps {
                echo 'Running import check...'
                sh 'docker run --rm ${IMAGE_NAME} python -c "import app";'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying new container...'
                sh '''
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p 5000:5000 \
                        --env-file /home/ubuntu/devops-project/.env \
                        ${IMAGE_NAME}
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment successful! App is live on port 5000.'
        }
        failure {
            echo 'Pipeline failed! Check the stage logs above.'
        }
    }
}
