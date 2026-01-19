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

        stage('Test AWS Auth') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-ecr-creds'
                ]]) {
                    sh 'aws sts get-caller-identity'
                }
            }
        }

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

        stage('Helm Deploy') {
            steps {
                sh '''
                  set -e
                  helm upgrade --install ${RELEASE_NAME} ${CHART_PATH} \
                    --set image.repository=${IMAGE_NAME} \
                    --set image.tag=${BUILD_NUMBER}
                '''
            }
        }

        stage('Wait for Rollout') {
            steps {
                sh '''
                  kubectl rollout status deployment \
                    -l app.kubernetes.io/instance=${RELEASE_NAME} \
                    --timeout=120s
                '''
            }
        }
    }

    post {
        failure {
            echo "Pipeline failed – rolling back"
            sh 'helm rollback ${RELEASE_NAME} || true'
        }
    }
}
