pipeline {
  agent any

  environment {
    IMAGE_NAME = "nginx-demo"
    IMAGE_TAG  = "${BUILD_NUMBER}"
    RELEASE    = "nginx"
    NAMESPACE  = "default"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Docker Image') {
      steps {
        sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
      }
    }

    stage('Deploy with Helm') {
      steps {
        sh "helm upgrade --install ${RELEASE} charts --set image.repository=${IMAGE_NAME} --set image.tag=${IMAGE_TAG} --namespace ${NAMESPACE}"
      }
    }

    stage('Verify Rollout') {
      steps {
        sh "kubectl rollout status deployment/${RELEASE} -n ${NAMESPACE}"
      }
    }
  }

  post {
    failure {
      sh "helm rollback ${RELEASE}"
    }
  }
}
