pipeline {
    agent any

    environment {
        KUBECONFIG   = "/var/jenkins_home/.kube/config"
        IMAGE_NAME   = "nginx-demo"
        CLUSTER_NAME = "dev"
        RELEASE_NAME = "nginx"
        CHART_PATH  = "charts"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                  set -e
                  docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Load Image into kind') {
            steps {
                sh '''
                  set -e
                  kind load docker-image ${IMAGE_NAME}:${BUILD_NUMBER} --name ${CLUSTER_NAME}
                '''
            }
        }

        stage('Deploy with Helm') {
            steps {
                sh '''
                  set -e
                  helm upgrade --install ${RELEASE_NAME} ${CHART_PATH} \
                    --set image.repository=${IMAGE_NAME} \
                    --set image.tag=${BUILD_NUMBER}
                '''
            }
        }

        stage('Verify Deployment Rollout') {
            steps {
                sh '''
                  set -e
                  kubectl rollout status deployment/${RELEASE_NAME} --timeout=60s
                  kubectl get pods -l app=${RELEASE_NAME}
                '''
            }
        }

    }
}

