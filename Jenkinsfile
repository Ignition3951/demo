pipeline {
    agent any

    tools {
        maven 'Maven3.9'
        jdk 'JDK21'
    }

    stages {

    stage('Initialize Docker') {
                steps {
                    script {
                        // 'docker-latest' must match the name configured in Global Tool Configuration
                        def dockerHome = tool name: 'docker-latest', type: 'dockerTool'
                        env.PATH = "${dockerHome}/bin:${env.PATH}"
                    }
                }
            }

        stage('Checkout') {
            steps {
                dir('/var/jenkins_home/workspace/local-repo') {
                    echo "Using local repo mounted into Jenkins"
                }
            }
        }

        stage('Build') {
            steps {
                dir('/var/jenkins_home/workspace/local-repo') {
                    sh 'mvn clean package'
                }
            }
        }

        stage('Docker Build') {
            steps {
                dir('/var/jenkins_home/workspace/local-repo') {
                    sh 'docker build -t spring-demo:latest .'
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                dir('/var/jenkins_home/workspace/local-repo') {
                    sh 'kubectl apply -f deployment.yaml'
                    sh 'kubectl apply -f service.yaml'
                }
            }
        }
    }
}
