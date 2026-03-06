pipeline {
    agent any

    environment {
        IMAGE = "nginx:trixie"
        KUBE_MANIFEST_DIR = "Kube1"
        KUBECONFIG = "/var/jenkins_home/.kube/config" // adjust path if needed
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/wale-devops/Kubetest.git'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Apply deployment and service manifests
                    sh "kubectl --kubeconfig=${KUBECONFIG} apply -f ${KUBE_MANIFEST_DIR}/app-deployment.yaml"
                    sh "kubectl --kubeconfig=${KUBECONFIG} apply -f ${KUBE_MANIFEST_DIR}/app-service.yaml"
                }
            }
        }

    }

    post {
        success {
            echo "✅ Deployment to Kubernetes completed successfully!"
        }
        failure {
            echo "❌ Deployment failed. Check logs."
        }
    }
}
