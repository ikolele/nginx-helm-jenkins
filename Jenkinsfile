pipeline {
    agent any

    environment {
        KUBECONFIG   = "/var/jenkins_home/.kube/config"
        IMAGE_NAME   = "nginx-demo"
        CLUSTER_NAME = "dev"
        RELEASE_NAME = "nginx"
        CHART_PATH   = "charts"
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
                  echo "Building Docker image..."
                  docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Load Image into kind') {
            steps {
                sh '''
                  set -e
                  echo "Loading image into kind cluster..."
                  kind load docker-image ${IMAGE_NAME}:${BUILD_NUMBER} --name ${CLUSTER_NAME}
                '''
            }
        }

        stage('Helm Lint') {
            steps {
                sh '''
                  set -e
                  echo "Linting Helm chart..."
                  helm lint ${CHART_PATH}
                '''
            }
        }

        stage('Deploy with Helm') {
            steps {
                sh '''
                  set -e
                  echo "Deploying application with Helm..."
                  helm upgrade --install ${RELEASE_NAME} ${CHART_PATH} \
                    --set image.repository=${IMAGE_NAME} \
                    --set image.tag=${BUILD_NUMBER}
                '''
            }
        }

        stage('Verify Kubernetes Resources') {
            steps {
                sh '''
                  echo "Checking pods..."
                  kubectl get pods -o wide

                  echo "Checking services..."
                  kubectl get svc
                '''
            }
        }

        stage('Wait for Rollout') {
            steps {
                sh '''
                  set -e
                  echo "Waiting for deployment rollout..."
                  kubectl rollout status deployment \
                    -l app.kubernetes.io/instance=${RELEASE_NAME} \
                    --timeout=120s
                '''
            }
        }
    }

    post {
        success {
            echo "Deployment completed successfully 🎉"
        }

        failure {
            echo "Deployment failed ❌ – rolling back Helm release"
            sh '''
              helm rollback ${RELEASE_NAME} || true
            '''
        }

        always {
            echo "Pipeline finished."
        }
    }
}
