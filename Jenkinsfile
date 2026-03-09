pipeline {
    agent any

    triggers {
        githubPush()
    }

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
                sh 'docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} -t ${DOCKER_IMAGE}:latest ./webapp'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-cred',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_TOKEN'
                )]) {
                    sh '''
                        echo "$DOCKER_TOKEN" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker push ${DOCKER_IMAGE}:latest
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f kube1/app-deployment.yaml
                    kubectl apply -f kube1/app-service.yaml
                    kubectl set image deployment/webapp webapp=${DOCKER_IMAGE}:${DOCKER_TAG}
                    kubectl rollout status deployment/webapp --timeout=120s
                    kubectl get pods -o wide
                    kubectl get svc
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully: ${DOCKER_IMAGE}:${DOCKER_TAG}"
        }
        failure {
            echo "Pipeline failed. Check console output."
        }
        always {
            sh 'docker image prune -f || true'
        }
    }
}
