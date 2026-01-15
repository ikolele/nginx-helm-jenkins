pipeline {
    agent any

    environment {
        KUBECONFIG = "/var/jenkins_home/.kube/config"
        IMAGE_NAME = "nginx-demo"
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh '''
                  docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Load Image into kind') {
            steps {
                sh '''
                  kind load docker-image ${IMAGE_NAME}:${IMAGE_TAG} --name dev
                '''
            }
        }

        stage('Deploy with Helm') {
            steps {
                sh '''
                    kubectl config use-context kind-dev

                    helm upgrade --install nginx charts \
                      --set image.repository=${IMAGE_NAME} \
                      --set image.tag=${IMAGE_TAG} \
                      --namespace default
                '''
            }
        }
    }
}
