pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t simple-cicd:latest .'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker rm -f simple-cicd || true

                    docker run -d \
                    --name simple-cicd \
                    -p 3000:3000 \
                    simple-cicd:latest
                '''
            }
        }
    }
}
