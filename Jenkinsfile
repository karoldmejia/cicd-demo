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
                        sh 'mvn clean package'
                    }
                }
            }
        }

        stage('Static Analysis (SonarQube)') {
            steps {
                script {
                    docker.image('maven:3.9.9-eclipse-temurin-17').inside {
                        sh """
                            mvn sonar:sonar \
                                -Dsonar.projectKey=cicd-demo \
                                -Dsonar.projectName="CI/CD Demo App" \
                                -Dsonar.host.url=${SONAR_HOST_URL} \
                                -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        stage('Quality Gate (SonarQube)') {
            steps {
                script {
                    timeout(time: 5, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
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

        stage('Container Security Scan (Trivy)') {
            steps {
                script {
                    sh """
                        trivy image --severity CRITICAL --exit-code 1 --no-progress ${IMAGE_NAME}
                    """
                }
            }
        }

        stage('Deploy') {
            when { 
                branch 'main' 
            }
            steps {
                script {
                    sh '''
                        docker stop mi-app 2>/dev/null || true
                        docker rm mi-app 2>/dev/null || true
                        docker run -d -p 8080:8080 --name mi-app ${IMAGE_NAME}
                    '''
                }
            }
        }
    }

    post {
        always {
            script {
                echo "Limpiando entorno..."
                try {
                    docker.container('cicd-demo-test').stop()
                    docker.container('cicd-demo-test').remove()
                } catch(err) {
                    echo "No hay contenedor de prueba para limpiar"
                }
            }
            cleanWs()
        }
        success {
            echo "Pipeline exitoso - Todas las validaciones pasaron"
            echo "Aplicación disponible en: http://localhost:8080"
        }
        failure {
            echo "Pipeline falló "
        }
        unstable {
            echo "Pipeline inestable - Calidad del código por debajo del umbral"
        }
    }
}