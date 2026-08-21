pipeline {

    agent any

    environment {
        APP_IMAGE = 'food-delivery-app'
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
                    sh 'npm run build'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {

                withSonarQubeEnv('SonarQube') {

                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.projectName=${SONAR_PROJECT_NAME} \
                        -Dsonar.sources=. \
                        -Dsonar.exclusions=**/node_modules/**,**/build/**,**/dist/**,**/coverage/**
                    '''
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

                sh """
                    docker build \
                    -t ${APP_IMAGE}:${IMAGE_TAG} \
                    -t ${APP_IMAGE}:latest \
                    .
                """
            }
        }

        stage('Deploy MongoDB + Application') {
            steps {

                sh """
                    IMAGE_TAG=${IMAGE_TAG} docker compose up -d
                """
            }
        }

        stage('Verify Deployment') {
            steps {

                sh '''
                    docker compose ps
                    echo "--------------------------------"
                    docker ps
                '''
            }
        }
    }

    post {

        success {
            echo '========================================'
            echo 'BUILD SUCCESSFUL'
            echo 'SONARQUBE QUALITY GATE PASSED'
            echo 'DOCKER DEPLOYMENT SUCCESSFUL'
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
