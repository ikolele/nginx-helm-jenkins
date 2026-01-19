pipeline {
    agent any

    environment {
        AWS_REGION    = "us-east-1"
        AWS_ACCOUNT_ID = "692859926752"   // comes from Jenkins job env
        ECR_REPO      = "nginx-demo"

        IMAGE_NAME    = "nginx-demo"
        CLUSTER_NAME  = "dev"
        RELEASE_NAME  = "nginx"
        CHART_PATH    = "charts"
        KUBECONFIG    = "/var/jenkins_home/.kube/config"
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

        stage('Login to AWS ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-ecr-creds'
                ]]) {
                    sh '''
                      aws ecr get-login-password --region ${AWS_REGION} \
                      | docker login --username AWS --password-stdin \
                        ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    '''
                }
            }
        }

        stage('Tag & Push Image to ECR') {
            steps {
                sh '''
                  set -e
                  docker tag ${IMAGE_NAME}:${BUILD_NUMBER} \
                    ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${BUILD_NUMBER}

                  docker push \
                    ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Load Image into kind') {
            steps {
                sh '''
                  set -e
                  docker pull \
                    ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${BUILD_NUMBER}

                  kind load docker-image \
                    ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${BUILD_NUMBER} \
                    --name ${CLUSTER_NAME}
                '''
            }
        }

        stage('Deploy with Helm') {
            steps {
                sh '''
                  set -e
                  helm upgrade --install ${RELEASE_NAME} ${CHART_PATH} \
                    --set image.repository=${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO} \
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
        success {
            echo "Deployment completed successfully 🚀"
        }
        failure {
            echo "Deployment failed – rolling back"
            sh 'helm rollback ${RELEASE_NAME} || true'
        }
    }
}
