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
                // Vérifier la version de Git installée
                bat '''
                    "C:\\Program Files\\Git\\bin\\git.exe" --version
                '''
                // Vérifier si le dossier 'pipeline' existe déjà
                bat '''
                    IF EXIST pipeline (
                        echo "Directory 'pipeline' exists. Pulling the latest changes..."
                        cd pipeline
                        git pull
                    ) ELSE (
                        echo "Cloning the repository..."
                        git clone https://github.com/IbnAyoub65/pipeline.git
                    )
                '''
                dir('pipeline') {
                    bat 'git checkout main'
                }
            }
        }

        stage('Build Docker Image') {
           steps {
               echo 'Building the Docker image...'

               // Vérifier si le Dockerfile existe
               bat '''
                   if exist Dockerfile (
                       echo "Dockerfile found. Proceeding with build."
                   ) else (
                       echo "Dockerfile not found!"
                       exit /b 1
                   )
               '''
               // Exécuter la commande de build Docker
               bat 'docker build -f DockerFile -t spring-docker-pipeline:latest .'
           }
        }

      stage('Run Tests') {
          steps {
              echo 'Running tests inside Docker container with Maven...'
                   bat '''
                       docker run --rm --name spring-test-container ^
                       -v "c:/users/yaya sambou/desktop/first_pipeline/pipeline:/app" ^
                       -w /app maven:3.9.5-eclipse-temurin-17 mvn test
                   '''
             }
      }

        stage('Push Docker Image') {
            steps {
                echo 'Pushing Docker image to registry...'
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
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
