pipeline {
    agent any

    environment {
        IMAGE_NAME = 'adnankhan7788/newnode'          // Docker Hub repository
        IMAGE_TAG  = "${IMAGE_NAME}:${BUILD_NUMBER}" // Unique image per build
    }

    stages {

        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/adnankhan-eng/nodeapp', branch: 'main'
                sh 'ls -ltr'
            }
        }

        stage('Docker Hub Login') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub',    // Jenkins credential ID
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_TAG} ."
                sh "docker tag ${IMAGE_TAG} ${IMAGE_NAME}:latest"
                echo "✅ Docker image built successfully"
                sh "docker images"
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                sh "docker push ${IMAGE_TAG}"
                sh "docker push ${IMAGE_NAME}:latest"
                echo "✅ Docker image pushed successfully"
            }
        }

        stage('Deploy to Local Minikube') {
            steps {
                script {
                    // Load Minikube Docker environment
                    sh "eval \$(minikube -p minikube docker-env)"

                    // Apply Kubernetes manifests
                    sh "kubectl apply -f k8s-deployment.yaml"
                    sh "kubectl apply -f k8s-service.yaml"

                    // Update deployment image (rolling update)
                    sh "kubectl set image deployment/node-app-deployment node-app=${IMAGE_TAG} --record"

                    // Wait for rollout to finish
                    sh "kubectl rollout status deployment/node-app-deployment --timeout=180s"

                    echo "✅ App deployed successfully to Minikube"
                }
            }
        }
    }
}