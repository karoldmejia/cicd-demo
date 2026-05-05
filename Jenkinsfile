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
            steps {
                script {
                    docker.image('maven:3.9.9-eclipse-temurin-17').inside {
                        sh 'mvn clean package'
                    }
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    def customImage = docker.build(IMAGE_NAME, ".")
                    echo "Image built: ${customImage.id}"
                }
            }
        }

        stage('Run Container (Test Local)') {
            steps {
                script {
                    // Clean up if exists
                    try {
                        docker.container('cicd-demo-test').stop()
                        docker.container('cicd-demo-test').remove()
                    } catch(err) {}
                    
                    // Run container
                    def container = docker.image(IMAGE_NAME).run('-p 8081:8080 -d --name cicd-demo-test')
                    echo "Container running on port 8081"
                }
            }
        }
    }

    post {
        always {
            script {
                try {
                    docker.container('cicd-demo-test').stop()
                    docker.container('cicd-demo-test').remove()
                } catch(err) {}
            }
            echo 'Limpiando workspace...'
            cleanWs()
        }
        failure {
            echo 'El pipeline falló'
        }
    }
}