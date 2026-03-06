pipeline {
    agent any

    environment {
        IMAGE = "nginx:trixie"
        KUBE_MANIFEST_DIR = "Kube1"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/wale-devops/Kubetest.git'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh "kubectl apply -f ${KUBE_MANIFEST_DIR}/app-deployment.yaml"
                sh "kubectl apply -f ${KUBE_MANIFEST_DIR}/app-service.yaml"
            }
        }

    }

    post {
        success {
            echo "Deployment to Kubernetes completed successfully!"
        }
        failure {
            echo "Deployment failed. Check logs."
        }
    }
}
