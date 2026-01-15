pipeline {
    agent any

    environment {
        KUBECONFIG = "/home/jenkins/.kube/config"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t nginx-demo:13 .'
            }
        }

        stage('Deploy with Helm') {
            steps {
                sh '''
                    kubectl config get-contexts
                    kubectl config use-context kind-dev

                    helm upgrade --install nginx charts \
                      --set image.repository=nginx-demo \
                      --set image.tag=13 \
                      --namespace default
                '''
            }
        }
    }
}
