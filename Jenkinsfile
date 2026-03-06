pipeline {
    agent {
        dockercontainer {
            image 'docker:24.0.2-cli'   // Docker CLI image
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        IMAGE_NAME = 'olawaledevops/nginx-trixie'
        IMAGE_TAG  = '3'
        DOCKER_CREDENTIALS = 'dockerhub-cred'
        KUBE_CONFIG = credentials('kubeconfig-cred') // optional if using kubectl
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/wale-devops/Kubetest.git', branch: 'master'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG app/'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "$DOCKER_CREDENTIALS",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f k8s/   # adjust path to your manifests
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs for details.'
        }
    }
}
