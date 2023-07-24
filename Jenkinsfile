pipeline {
    agent {
        kubernetes {
            // cloud 'kubernetes'
            yaml """
kind: Pod
metadata:
  name: kaniko
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug-539ddefcae3fd6b411a95982a830d987f4214251
    imagePullPolicy: Always
    command:
    - cat
    tty: true
    volumeMounts:
      - name: docker-config
        mountPath: /kaniko/.docker
  volumes:
    - name: docker-config
      configMap:
        name: docker-config
"""
        }
    }

    environment {
        // Change these values accordingly
        DOCKER_REGISTRY = '956408752768.dkr.ecr.us-east-1.amazonaws.com' // Replace with your ECR registry
        DOCKER_IMAGE_NAME = 'weather-app' // Replace with your Docker image name (without the registry part)
    }

    stages {
        stage('Install Tools') {
            steps {
                sh '''
                    curl -L -o awscliv2.zip https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip
                    unzip -q awscliv2.zip
                    sudo ./aws/install --install-dir /usr/local/aws-cli --bin-dir /usr/local/bin
                    python -m ensurepip
                    python -m pip install --upgrade pip
                    pip install awscli --upgrade
                '''
                sh 'curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"'
                sh 'chmod +x kubectl'
                sh 'sudo mv kubectl /usr/local/bin/'
            }
        }

        stage('Clone Git Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/showmikb/Weather-App.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'echo $DOCKER_REGISTRY'
                sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $DOCKER_REGISTRY'
                sh 'docker build -t $DOCKER_REGISTRY/$DOCKER_IMAGE_NAME .'
            }
        }

        stage('Push Docker Image') {
            steps {
                sh "docker push $DOCKER_REGISTRY/$DOCKER_IMAGE_NAME"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh "kubectl apply -f deployment.yaml"  // Replace 'deployment.yaml' with the path to your Kubernetes deployment file
            }
        }
    }
}
