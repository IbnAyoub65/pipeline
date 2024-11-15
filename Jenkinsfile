pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out the code...'
                // Utilisation de Git pour récupérer le code source
                bat 'git checkout main'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building the Docker image...'
                bat '''
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
                // Lancement des tests dans un conteneur temporaire basé sur l'image Docker
                bat '''
                docker run --rm --name spring-test-container spring-docker-pipeline:latest \\
                bash -c "mvn test"
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                echo 'Pushing Docker image to registry...'
                // Authentification et envoi de l'image Docker au registre
                bat 'docker login -u sambouyaya -p passer@123'
                bat 'docker tag spring-docker-pipeline:latest sambouyaya/spring-docker-pipeline:latest'
                bat 'docker push sambouyaya/spring-docker-pipeline:latest'
            }
        }

        stage('Deploy Application') {
            steps {
                echo 'Deploying the application...'
                // Déploiement de l'application avec Docker
                bat '''
                docker stop spring-docker-app || echo "No running container to stop"
                docker rm spring-docker-app || echo "No container to remove"
                docker run -d --name spring-docker-app -p 8080:8080 sambouyaya/spring-docker-pipeline:latest
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
