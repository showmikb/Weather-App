pipeline {
    //  agent {
    //     kubernetes {
    //         label 'jenkins-agent'
    //         containerTemplate {
    //             name 'docker'
    //             image 'docker:20.10.8'  // Use the appropriate Docker version here
    //             command 'cat' // This keeps the container running indefinitely
    //             ttyEnabled true
    //         }
    //     }
    // }
    agent any
    environment {
        // Change these values accordingly
        DOCKER_REGISTRY = 'docker.io' // Replace with your container registry
        DOCKER_IMAGE_NAME = 'showmikb/weather-app' // Replace with your Docker image name
    }

    stages {
        stage('Install Tools') {
             steps {
                  sh '''curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"'''
                  sh '''unzip awscliv2.zip -y'''
                  sh '''./aws/install'''
                  sh '''curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"'''
                  sh '''install -o root -g root -m 0755 kubectl /usr/bin/kubectl'''
             }
        }
         
        stage('Clone Git Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/showmikb/Weather-App.git'
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
