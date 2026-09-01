pipeline {

    agent any

    environment {
        SONAR_PROJECT_KEY = 'nikhil-website'
        SONAR_PROJECT_NAME = 'nikhil-website'

        DOCKER_IMAGE = 'manual-sonarqube:1.0'
        CONTAINER_NAME = 'manual-sonarqube-container'

        SONAR_TOKEN = credentials('sonar-token')
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'

                checkout scm
            }
        }

        stage('Verify Environment') {
            steps {
                sh '''
                    echo "Java version:"
                    java -version

                    echo "Node version:"
                    node -v

                    echo "NPM version:"
                    npm -v

                    echo "Docker version:"
                    docker --version

                    echo "Trivy version:"
                    trivy --version
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {

                withSonarQubeEnv('SonarQube') {

                    sh '''
                        sonar-scanner \
                          -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                          -Dsonar.projectName=${SONAR_PROJECT_NAME} \
                          -Dsonar.sources=.
                    '''
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

                sh '''
                    echo "================================"
                    echo "Building Docker Image"
                    echo "================================"

                    docker build -t ${DOCKER_IMAGE} .

                    echo "Docker image built successfully"
                    docker images | grep manual-sonarqube
                '''
            }
        }

        stage('Trivy Security Scan') {
            steps {

                sh '''
                    echo "================================"
                    echo "Running Trivy Security Scan"
                    echo "================================"

                    trivy image \
                      --severity HIGH,CRITICAL \
                      --exit-code 1 \
                      ${DOCKER_IMAGE}

                    echo "Trivy scan passed!"
                '''
            }
        }

        stage('Deploy') {
            steps {

                sh '''
                    echo "================================"
                    echo "Deploying Application"
                    echo "================================"

                    echo "Stopping old container..."

                    docker rm -f ${CONTAINER_NAME} 2>/dev/null || true

                    echo "Starting new container..."

                    docker run -d \
                      --name ${CONTAINER_NAME} \
                      -p 8081:80 \
                      ${DOCKER_IMAGE}

                    echo "Container started successfully!"

                    docker ps
                '''
            }
        }
    }

    post {

        success {
            echo '================================'
            echo 'PIPELINE SUCCESS'
            echo '================================'
        }

        failure {
            echo '================================'
            echo 'PIPELINE FAILED'
            echo '================================'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}
