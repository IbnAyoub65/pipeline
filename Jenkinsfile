pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = "sambouyaya"
        DOCKER_IMAGE = "spring-docker-pipeline"
        IMAGE_TAG = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Cloning the repository...'
                 bat '''
                       "C:\\Program Files\\Git\\bin\\git.exe" --version
                     '''
                bat 'git clone https://github.com/IbnAyoub65/pipeline.git'
                dir('pipeline') {
                    bat 'git checkout main'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building the Docker image...'
                bat '''
                echo "Listing files in the current directory:"
                dir
                if not exist Dockerfile (
                    echo "Dockerfile not found in the current directory!"
                    exit /b 1
                )
                docker build -f Dockerfile -t spring-docker-pipeline:latest .
                '''
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running tests inside Docker container...'
                bat '''
                docker run --rm --name spring-test-container spring-docker-pipeline:latest ^
                bash -c "mvn test"
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                echo 'Pushing Docker image to registry...'
                withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    bat "docker login -u %DOCKER_USERNAME% -p %DOCKER_PASSWORD%"
                }
                bat "docker tag spring-docker-pipeline:latest %DOCKER_REGISTRY%/%DOCKER_IMAGE%:%IMAGE_TAG%"
                bat "docker push %DOCKER_REGISTRY%/%DOCKER_IMAGE%:%IMAGE_TAG%"
            }
        }

        stage('Deploy Application') {
            steps {
                echo 'Deploying the application...'
                bat '''
                docker stop spring-docker-app || echo "No running container to stop"
                docker rm spring-docker-app || echo "No container to remove"
                docker run -d --name spring-docker-app -p 8080:8080 %DOCKER_REGISTRY%/%DOCKER_IMAGE%:%IMAGE_TAG%
                '''
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
