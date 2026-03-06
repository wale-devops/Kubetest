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
        
        stage('Build & Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                        // Build image from app directory
                        def appImage = docker.build("${DOCKER_IMAGE}:latest", 'app/')
                        
                        // Push both latest and versioned tag
                        appImage.push()
                        appImage.push("${DOCKER_TAG}")
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    echo "Deploying from kube1 directory..."
                    kubectl apply -f kube1/
                    kubectl rollout status deployment/static-app --timeout=60s || true
                    kubectl get pods
                '''
            }
        }
    }
    
    post {
        success {
            echo "✅ Pipeline completed! Image: ${DOCKER_IMAGE}:${DOCKER_TAG} deployed from ${GIT_BRANCH} branch"
        }
        failure {
            echo "❌ Pipeline failed! Check the logs for details."
        }
    }
}
