pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'olawaledevops/static-app'
        DOCKER_TAG = "${BUILD_NUMBER}"
        GIT_REPO = 'https://github.com/wale-devops/Kubetest.git'
        GIT_BRANCH = 'master'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${GIT_BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    appImage = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}", 'app/')
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                        appImage.push()
                        sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                        sh "docker push ${DOCKER_IMAGE}:latest"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    echo "Applying Kubernetes manifests..."
                    kubectl apply -f kube1/
                    kubectl set image deployment/static-app static-app=${DOCKER_IMAGE}:${DOCKER_TAG}
                    kubectl rollout status deployment/static-app --timeout=60s
                    kubectl get pods
                """
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed! Image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
        }
        failure {
            echo "❌ Pipeline failed. Check the console output."
        }
        always {
            sh 'docker logout || true'
        }
    }
}
