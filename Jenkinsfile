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
		Docker_Image="sathik2074751/cicd-image:${BUILD_NUMBER}"
		REGISTRY_CREDENTIALS = credentials('github')
          }
            steps {
	      script{
                sh 'docker build -t ${Docker_Image} .'
		def dockerImage = docker.image("$Docker_Image")
		docker.withRegistry('https://index.docker.io/v1/', "docker-hub"){
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

             sed -i "s/latest/${BUILD_NUMBER}/g" deployment.yml

             git add deployment.yml

             if ! git diff --cached --quiet; then
             git commit -m "Update deployment image to version ${BUILD_NUMBER}"
              git push https://${GITHUB_TOKEN}@github.com/sathik-git/k8s-project.git HEAD:main
            else
             echo "No changes in deployment.yml. Skipping commit and push."
             fi
             '''
            }
        }
    }
    stage('Deploy') {
         steps {
        sh '''
            kubectl apply -f deployment.yml
			kubectl apply -f service.yml
            kubectl rollout status deployment/project-cicd
            kubectl get pods
            kubectl get services
        '''
    }
        }
    }
}
