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

        stage('Build and push Docker Image') {
	  environment{
		Docker_Image="cicd-image:${BUILD_NUMBER}"
		REGISTRY_CREDENTIALS = credentials('github')
          }
            steps {
	      script{
                sh 'docker build -t ${Docker_Image} .'
		def dockerImage = docker.image("$Docker_Image")
		docker.withRegistry('https://hub.docker.com/repository/docker/sathik2074751/cicd-image/', "docker-cred"){
		dockerImage.push()
	}
		}
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
