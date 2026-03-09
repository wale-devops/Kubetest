pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins
  containers:
    - name: kaniko
      image: gcr.io/kaniko-project/executor:latest
      command:
        - /busybox/cat
      tty: true
      volumeMounts:
        - name: docker-config
          mountPath: /kaniko/.docker
    - name: kubectl
      image: bitnami/kubectl:latest
      command:
        - cat
      tty: true
  volumes:
    - name: docker-config
      secret:
        secretName: dockerhub-secret
"""
        }
    }

    triggers {
        githubPush()
    }

    environment {
        DOCKER_IMAGE = 'olawaledevops/static-app'
        DOCKER_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Push Image') {
            steps {
                container('kaniko') {
                    sh '''
                        /kaniko/executor \
                          --dockerfile=./webapp/Dockerfile \
                          --context=${WORKSPACE}/webapp \
                          --destination=${DOCKER_IMAGE}:${DOCKER_TAG} \
                          --destination=${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                container('kubectl') {
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
    }

    post {
        success {
            echo "Pipeline completed successfully: ${DOCKER_IMAGE}:${DOCKER_TAG}"
        }
        failure {
            echo "Pipeline failed. Check console output."
        }
    }
}
