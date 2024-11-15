pipeline {
    agent any
    environment {
        // Définir l'image Docker (vous pouvez également configurer une variable pour le nom de votre registre Docker)
        DOCKER_IMAGE = "spring-docker-pipeline"
        DOCKER_REGISTRY = "docker.io" // Remplacez par votre registre Docker si nécessaire (par ex. Docker Hub)
        DOCKER_TAG = "latest"
    }

    tools {
        // Utilisez Docker installé sur Jenkins
        dockerTool 'Docker'
    }

    stages {
        stage('Checkout') {
            steps {
                // Récupérer le code source du dépôt Git
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
                    // Exécuter les tests dans le conteneur Docker
                    // Ici, vous pouvez ajouter des tests ou simplement vérifier si le conteneur démarre correctement
                    sh 'docker run --rm $DOCKER_IMAGE:$DOCKER_TAG java -jar /app/pipeline-0.0.1-SNAPSHOT.jar'
                }
            }
        }

        stage('Push Docker Image') {
            when {
                branch 'main' // Push uniquement sur la branche principale
            }
            steps {
                script {
                    // Pousser l'image Docker vers le registre Docker (Docker Hub ou autre)
                    // Connectez-vous à Docker si nécessaire
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                            docker tag $DOCKER_IMAGE:$DOCKER_TAG $DOCKER_REGISTRY/$DOCKER_USER/$DOCKER_IMAGE:$DOCKER_TAG
                            docker push $DOCKER_REGISTRY/$DOCKER_USER/$DOCKER_IMAGE:$DOCKER_TAG
                        """
                    }
                }
            }
        }

        stage('Deploy to Docker') {
            steps {
                script {
                    // Déployer l'image Docker sur un serveur (en utilisant un conteneur Docker pour le déploiement)
                    // Cela suppose que vous avez accès à un serveur Docker et que vous avez configuré un hôte Docker accessible
                    sh """
                        docker run -d -p 8080:8080 --name mon-app $DOCKER_IMAGE:$DOCKER_TAG
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
