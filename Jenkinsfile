pipeline {
    agent any

    environment {
        IMAGE_NAME = 'adnan72746/nodeapp'
        IMAGE_TAG  = "${IMAGE_NAME}:latest"
    }

    stages {

        stage('Checkout') {
            steps {
                git url: 'https://github.com/adnankhan-eng/nodeapp', branch: 'main'
                sh 'ls -ltr'
            }
        }

        stage('Build Docker Image for Minikube') {
            steps {
                script {
                    // Get Minikube Docker environment variables
                    def dockerEnv = sh(script: "minikube -p minikube docker-env --shell bash", returnStdout: true).trim()

                    // Build Docker image inside Minikube Docker environment
                    sh """
                        ${dockerEnv} 
                        docker build -t ${IMAGE_TAG} .
                    """

                    echo "✅ Docker image built successfully for Minikube"
                    sh """
                        ${dockerEnv}
                        docker images | grep ${IMAGE_NAME}
                    """
                }
            }
        }

        stage('Deploy to Local Kubernetes (Minikube)') {
            steps {
                script {
                    sh "kubectl apply -f k8s-deployment.yaml"
                    sh "kubectl apply -f k8s-service.yaml"

                    sh "kubectl set image deployment/node-app-deployment node-app=${IMAGE_TAG} --record || echo 'Deployment may already be using latest image'"

                    sh "kubectl rollout status deployment/node-app-deployment --timeout=180s"

                    echo "✅ App deployed successfully to Minikube"
                }
            }
        }
    }
}