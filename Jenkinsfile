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
		docker.withRegistry('https://index.docker.io/v1/', "docker-cred"){
		dockerImage.push()
	}
		}
            }
        }

    stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "k8s-project"
            GIT_USER_NAME = "sathik-git"
        }
        steps {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "sathikbasha.abdul@gmail.com"
                    git config user.name "sathik-git"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/latest/${BUILD_NUMBER}/g" deployment.yml
                    git add deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }
    stage('Deploy') {
         steps {
        sh '''
            kubectl apply -f deployment.yml
            kubectl rollout status deployment/cicd-app
            kubectl get pods
            kubectl get services
        '''
    }
        }
    }
}
