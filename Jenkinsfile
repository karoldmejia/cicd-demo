pipeline {
    agent any

    environment {
        IMAGE_NAME = 'cicd-demo:latest'
    }

    stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/karoldmejia/cicd-demo.git'
            }
        }

        stage('Build & Test') {
            agent {
                docker {
                    image 'maven:3.9.9-eclipse-temurin-17'
                }
            }
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Run Container (Test Local)') {
            steps {
                sh 'docker run -d -p 8081:8080 $IMAGE_NAME || true'
            }
        }
    }

    post {
        always {
            echo 'Limpiando workspace...'
            cleanWs()
        }
        failure {
            echo 'El pipeline falló'
        }
    }
}