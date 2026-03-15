pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub' // Jenkins Docker Hub credentials ID
        DOCKER_IMAGE = 'adnan72746/nodeapp:latest' // Docker image name
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/adnankhan-eng/nodeapp.git'
            }
        }

        stage('Install & Build') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm run test || echo "No tests or tests failed, continuing..."'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh "kubectl apply -f k8s-deployment.yaml"
                sh "kubectl apply -f k8s-service.yaml"
                sh "kubectl rollout status deployment/node-app-deployment"
            }
        }
    }
}