pipeline {
    agent any

    environment {
        IMAGE_NAME = 'cicd-demo:latest'
        SONAR_HOST_URL = 'http://sonarqube:9000'
        SONAR_TOKEN = credentials('sonar-token')
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

        stage('Static Analysis (SonarQube)') {
            agent {
                docker {
                    image 'maven:3.9.9-eclipse-temurin-17'
                }
            }
            steps {
                withSonarQubeEnv('sonar') {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=cicd-demo'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
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
                sh """
                    docker run --rm \
                        -v /var/run/docker.sock:/var/run/docker.sock \
                        aquasec/trivy:latest \
                        image --severity CRITICAL --exit-code 1 --no-progress ${IMAGE_NAME}
                """
            }
        }

        stage('Deploy') {
            when { branch 'main' }
            steps {
                sh '''
                    docker stop mi-app 2>/dev/null || true
                    docker rm mi-app 2>/dev/null || true
                    docker run -d -p 8080:8080 --name mi-app ${IMAGE_NAME}
                '''
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