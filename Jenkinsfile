pipeline {
    agent any
    
    environment {
        // Change these values accordingly
        DOCKER_REGISTRY = 'docker.io' // Replace with your container registry
        DOCKER_IMAGE_NAME = 'showmikb/weather-app' // Replace with your Docker image name
    }

    stages {
        stage('Clone Git Repository') {
            steps {
                git 'https://github.com/showmikb/Weather-App.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE_NAME .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withDockerRegistry([url: "https://${DOCKER_REGISTRY}", credentialsId: 'docker-creds']) {
                    sh "docker push $DOCKER_IMAGE_NAME"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh "kubectl apply -f deployment.yaml"  // Replace 'kubernetes/deployment.yaml' with the path to your Kubernetes deployment file
            }
        }
    }
}
