pipeline {
    agent any

    environment {
        IMAGE_NAME = "olawaledevops/nginx-trixie"  // Your Docker Hub username
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        KUBE_MANIFEST_DIR = "Kube1"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/wale-devops/Kubetest.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image from app/Dockerfile
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} app/"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Login to Docker Hub using Jenkins credentials
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Update deployment manifest with new image tag
                    sh "sed -i 's|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|' ${KUBE_MANIFEST_DIR}/app-deployment.yaml"

                    // Deploy using in-cluster credentials
                    sh "kubectl apply -f ${KUBE_MANIFEST_DIR}/app-deployment.yaml"
                    sh "kubectl apply -f ${KUBE_MANIFEST_DIR}/app-service.yaml"
                }
            }
        }

    }

    post {
        success {
            echo "✅ Full CI/CD pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs for details."
        }
    }
}
