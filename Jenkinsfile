pipeline {
  agent any
  parameters {
    string(name: 'build_version', defaultValue: 'V1.0', description: 'Build version to use for Docker image')
  }
  stages {
    stage('Checkout') {
      steps {
        git branch: 'master', url: 'https://github.com/Ignition3951/demo.git'
      }
    }
    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        // build the project and create a JAR file
        sh 'mvn clean package'
      }
    }
    stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://localhost:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
    stage('Build and Push Docker Image') {
          environment {
            DOCKER_IMAGE = "utk1311/demo-java-app:${build_version}"
            REGISTRY_CREDENTIALS = credentials('docker')
          }
          steps {
            script {
                sh 'docker build -t ${DOCKER_IMAGE} .'
                def dockerImage = docker.image("${DOCKER_IMAGE}")
                docker.withRegistry('https://index.docker.io/v1/', "docker") {
                    dockerImage.push()
                }
            }
          }
        }
    stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "demo"
            GIT_USER_NAME = "Ignition3951"
        }
        steps {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "ignition3951@gmail.com"
                    git config user.name "Utkarsh Srivastava"
                    sed -i "s/tag: .*/tag: \"${build_version}\"/" helm/app/values.yaml
                    git add helm/app/values.yaml
                    git commit -m "Update deployment image to version ${build_version}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }
  }
}