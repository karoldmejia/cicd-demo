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
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    docker.build(IMAGE_NAME)
                }
            }
        }

        stage('Run Container (Test Local)') {
            steps {
                script {
                    docker.image(IMAGE_NAME).run('-p 8081:8080')
                }
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