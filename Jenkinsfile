pipeline {
    agent any

    environment {
        IMAGE_NAME = "iniz/bliss_docker:latest"
        CONTAINER_NAME = "ai-model-api"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-token',
                    url: 'https://github.com/inainz/aiDocker.git'
            }
        }

        stage('Docker Build') {
            steps {
                sh '/usr/local/bin/docker build -t $IMAGE_NAME .'
            }
        }

        stage('Docker Hub Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-token',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh 'echo $DOCKER_PASS | /usr/local/bin/docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Docker Push') {
            steps {
                sh '/usr/local/bin/docker push $IMAGE_NAME'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                /usr/local/bin/docker rm -f $CONTAINER_NAME || true
                /usr/local/bin/docker pull $IMAGE_NAME
                /usr/local/bin/docker compose down || true
                /usr/local/bin/docker compose pull
                /usr/local/bin/docker compose up -d
                '''
            }
        }

        stage('Test API') {
            steps {
                sh '''
                sleep 5
                curl http://localhost:8008
                curl "http://localhost:8008/predict?value=1.5"
                '''
            }
        }
    }
}
