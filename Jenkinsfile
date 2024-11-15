pipeline {
    agent any // Exécute sur n'importe quel agent disponible

    environment {
        DOCKER_IMAGE = "spring-docker-demo" // Nom de l'image Docker locale avec tag
        DOCKER_HUB_REPO = "sambouyaya/spring-docker-pipeline "// Répertoire Docker Hub
    }

    stages {
        stage('Checkout') {
            steps {
                // Récupérer le code depuis le dépôt GitHub
                git branch: 'main', url: 'https://github.com/IbnAyoub65/pipeline.git'
            }
        }

        stage('Build') {
            steps {
                // Construire le projet Java avec Maven!
                sh 'mvn clean install'
            }
        }

        stage('Test') {
            steps {
                // Lancer les tests unitaires
                sh 'mvn test'
            }
        }

        stage('Docker Build') {
            steps {
                // Construire une image Docker avec le Dockerfile
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Docker Push') {
            steps {
                // Pousser l'image Docker sur Docker Hub
                script {
                    docker.withRegistry('https://index.docker.io/v1/','dockerhub-credentials') {
                        docker.image(env.DOCKER_IMAGE).push()
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                // Déployer et démarrer l'image Docker sur le serveur
                sh 'docker run -d -p 80:8080 $DOCKER_IMAGE'
            }
        }
    }

    post {
        always {
            echo 'Pipeline terminé'
        }
    }
}
