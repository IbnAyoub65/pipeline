pipeline {
    agent any
    environment {
        // Définir l'image Docker et le registre
        DOCKER_IMAGE = "spring-docker-pipeline"
        DOCKER_TAG = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                // Récupérer le code source depuis le dépôt Git
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Construire l'image Docker
                    sh 'docker build -t $DOCKER_IMAGE:$DOCKER_TAG .'
                }
            }
        }

        stage('Run Tests in Docker') {
            steps {
                script {
                    // Tester l'application en exécutant le conteneur Docker
                    sh 'docker run --rm $DOCKER_IMAGE:$DOCKER_TAG java -jar /app/pipeline-0.0.1-SNAPSHOT.jar'
                }
            }
        }

        stage('Push Docker Image') {
            when {
                branch 'main' // Pousser uniquement sur la branche principale
            }
            steps {
                script {
                    // Authentification et poussée de l'image Docker vers un registre (Docker Hub)
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                            docker tag $DOCKER_IMAGE:$DOCKER_TAG \$DOCKER_USER/$DOCKER_IMAGE:$DOCKER_TAG
                            docker push \$DOCKER_USER/$DOCKER_IMAGE:$DOCKER_TAG
                        """
                    }
                }
            }
        }

        stage('Deploy to Docker') {
            steps {
                script {
                    // Supprimer le conteneur existant et déployer le nouveau
                    sh """
                        docker rm -f mon-app || true
                        docker run -d -p 8080:8080 --name mon-app $DOCKER_IMAGE:$DOCKER_TAG
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline terminé avec succès.'
        }
        failure {
            echo 'Le pipeline a échoué.'
        }
    }
}
