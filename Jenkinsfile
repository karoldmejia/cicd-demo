pipeline {
    agent any

    environment {
        IMAGE_NAME = 'mi-app:latest'
    }

    stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/karoldmejia/cicd-demo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Run Container (Test Local)') {
            steps {
                sh 'docker run -d -p 8081:8080 $IMAGE_NAME'
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