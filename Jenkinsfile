pipeline {

    agent any

    environment {
        APP_IMAGE = 'food-delivery-app'
        FRONTEND_IMAGE = 'food-delivery-frontend'
        IMAGE_TAG = "${BUILD_NUMBER}"

        SONAR_PROJECT_KEY = 'Food_Delivery_Application'
        SONAR_PROJECT_NAME = 'Food_Delivery_Application'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Satoo36/Food_Delivery_Application.git'
            }
        }

        stage('Check Tools') {
            steps {
                sh '''
                    echo "=============================="
                    echo "Checking Installed Tools"
                    echo "=============================="

                    echo "Java:"
                    java -version

                    echo "Node:"
                    node --version

                    echo "NPM:"
                    npm --version

                    echo "Docker:"
                    docker --version

                    echo "Docker Compose:"
                    docker compose version

                    echo "=============================="
                '''
            }
        }

        stage('Install Backend Dependencies') {
            steps {
                dir('backend') {
                    sh 'npm ci'
                }
            }
        }

        stage('Install Frontend Dependencies') {
            steps {
                dir('frontend') {
                    sh 'npm ci --legacy-peer-deps'
                }
            }
        }

        stage('Frontend Build') {
            steps {
                dir('frontend') {
                    sh 'CI=false npm run build'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {

                    def scannerHome = tool 'SonarScanner'

                    withSonarQubeEnv('SonarQube') {

                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.projectName=${SONAR_PROJECT_NAME} \
                            -Dsonar.sources=. \
                            -Dsonar.exclusions=**/node_modules/**,**/build/**,**/dist/**,**/coverage/**
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {

                timeout(time: 10, unit: 'MINUTES') {

                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    echo "=============================="
                    echo "Building Backend Docker Image"
                    echo "=============================="

                    docker build \
                        -t ${APP_IMAGE}:${IMAGE_TAG} \
                        -t ${APP_IMAGE}:latest \
                        .

                    echo "=============================="
                    echo "Building Frontend Docker Image"
                    echo "=============================="

                    docker build \
                        -t ${FRONTEND_IMAGE}:${IMAGE_TAG} \
                        -t ${FRONTEND_IMAGE}:latest \
                        ./frontend

                    echo "=============================="
                    echo "Docker Images"
                    echo "=============================="

                    docker images | grep -E "food-delivery-app|food-delivery-frontend"
                '''
            }
        }

        stage('Deploy MongoDB + Application') {
            steps {
                sh '''
                    echo "=============================="
                    echo "Stopping Previous Deployment"
                    echo "=============================="

                    IMAGE_TAG=${IMAGE_TAG} docker compose down

                    echo "=============================="
                    echo "Starting MongoDB + Backend + Frontend"
                    echo "=============================="

                    IMAGE_TAG=${IMAGE_TAG} docker compose up -d

                    echo "=============================="
                    echo "Deployment Started"
                    echo "=============================="
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "=============================="
                    echo "Docker Compose Status"
                    echo "=============================="

                    docker compose ps

                    echo "=============================="
                    echo "Running Containers"
                    echo "=============================="

                    docker ps

                    echo "=============================="
                    echo "Application URLs"
                    echo "=============================="

                    echo "Frontend:"
                    echo "http://44.198.52.77:3000"

                    echo "Backend:"
                    echo "http://44.198.52.77:5000"

                    echo "SonarQube:"
                    echo "http://44.198.52.77:9000"

                    echo "Jenkins:"
                    echo "http://44.198.52.77:8080"

                    echo "=============================="
                '''
            }
        }
    }

    post {

        success {
            echo '========================================'
            echo 'BUILD SUCCESSFUL'
            echo '========================================'
            echo 'SONARQUBE QUALITY GATE PASSED'
            echo 'DOCKER IMAGES BUILT SUCCESSFULLY'
            echo 'MONGODB DEPLOYED'
            echo 'BACKEND DEPLOYED'
            echo 'FRONTEND DEPLOYED'
            echo '========================================'
            echo 'Food Delivery Application:'
            echo 'http://44.198.52.77:3000'
            echo '========================================'
        }

        failure {
            echo '========================================'
            echo 'BUILD FAILED'
            echo 'CHECK JENKINS CONSOLE OUTPUT'
            echo '========================================'
        }
    }
}
