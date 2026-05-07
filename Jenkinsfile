pipeline {
    agent any

    environment {
        IMAGE_NAME = 'cicd-demo:latest'
        SONAR_HOST_URL = 'http://172.17.0.1:9000'
        SONAR_TOKEN = credentials('sonar-token')
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
                        sh 'mvn clean package -DskipTests'
                    }
                }
            }
        }

        stage('Static Analysis (SonarQube)') {
            steps {
                script {
                    docker.image('maven:3.9.9-eclipse-temurin-17').inside {
                        withSonarQubeEnv('sonar') {
                            sh 'mvn sonar:sonar -Dsonar.projectKey=cicd-demo'
                        }
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    docker.build(IMAGE_NAME, ".")
                }
            }
        }

        stage('Container Security Scan (Trivy)') {
            steps {
                script {
                    sh """
                        docker run --rm \
                            -v /var/run/docker.sock:/var/run/docker.sock \
                            aquasec/trivy:latest \
                            image --severity CRITICAL --exit-code 0 --no-progress ${IMAGE_NAME}
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh """
                        docker stop mi-app 2>/dev/null || true
                        docker rm mi-app 2>/dev/null || true
                        docker run -d -p 8081:8080 --name mi-app ${IMAGE_NAME}
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        failure {
            echo 'Pipeline falló'
        }
        success {
            echo 'Pipeline exitoso'
        }
    }
}