pipeline {
  agent any  // Or `any`; ensure Docker is available

tools {
    maven 'Maven 3.9.9'   // name must match Jenkins tool config
  }

  environment {
    APP_NAME = 'spring-boot-demo'
    IMAGE_TAG = 'latest'
    REGISTRY = '' // leave empty for Minikube docker-env; else 'localhost:5000'
    KUBECONFIG = "${WORKSPACE}/.kube/config"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Set kubeconfig') {
      steps {
        sh '''
          mkdir -p ${WORKSPACE}/.kube
          # Copy kubeconfig from Jenkins home (mounted from host)
          cp ~/.kube/config ${WORKSPACE}/.kube/config
        '''
      }
    }

    stage('Build') {
      steps {
        sh 'mvn -B -q -DskipTests clean package'
      }
    }

    stage('Docker Build') {
      steps {
        script {
          def imageName = REGISTRY ? "${REGISTRY}/${APP_NAME}:${IMAGE_TAG}" : "${APP_NAME}:${IMAGE_TAG}"
          sh "docker build -t ${imageName} ."
        }
      }
    }

    stage('Docker Push') {
      when { expression { return env.REGISTRY?.trim() } }
      steps {
        sh "docker push ${REGISTRY}/${APP_NAME}:${IMAGE_TAG}"
      }
    }

    stage('Deploy to Minikube') {
      steps {
        script {
          def imageName = REGISTRY ? "${REGISTRY}/${APP_NAME}:${IMAGE_TAG}" : "${APP_NAME}:${IMAGE_TAG}"
          // Update image in deployment if using registry
          sh """
            kubectl --kubeconfig=${KUBECONFIG} apply -f deployment.yaml
            kubectl --kubeconfig=${KUBECONFIG} apply -f service.yaml
            kubectl --kubeconfig=${KUBECONFIG} set image deployment/${APP_NAME} ${APP_NAME}=${imageName} --record || true
          """
        }
      }
    }

    stage('Verify rollout') {
      steps {
        sh "kubectl --kubeconfig=${KUBECONFIG} rollout status deployment/${APP_NAME} --timeout=60s"
      }
    }
  }

  post {
    success {
      echo 'Deployment succeeded.'
    }
    failure {
      echo 'Deployment failed.'
      sh "kubectl --kubeconfig=${KUBECONFIG} describe deployment/${APP_NAME} || true"
      sh "kubectl --kubeconfig=${KUBECONFIG} logs deployment/${APP_NAME} || true"
    }
  }
}