pipeline {
    agent any

    environment {
        IMAGE_NAME = 'nodeapp'           // Local image name
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
                    // Switch Docker to Minikube's environment
                    sh "eval \$(minikube docker-env)"

                    // Build the Docker image inside Minikube's Docker daemon
                    sh "docker build -t ${IMAGE_TAG} ."

                    echo "✅ Docker image built successfully for Minikube"
                    sh "docker images | grep ${IMAGE_NAME}"
                }
            }
        }

        stage('Deploy to Local Kubernetes (Minikube)') {
            steps {
                script {
                    // Apply Kubernetes manifests
                    sh "kubectl apply -f k8s-deployment.yaml"
                    sh "kubectl apply -f k8s-service.yaml"

                    // Optional: force rolling update if needed
                    sh "kubectl set image deployment/node-app-deployment node-app=${IMAGE_TAG} --record || echo 'Deployment may already be using latest image'"

                    // Wait for rollout to finish
                    sh "kubectl rollout status deployment/node-app-deployment --timeout=180s"

                    echo "✅ App deployed successfully to Minikube"
                }
            }
        }
    }
}