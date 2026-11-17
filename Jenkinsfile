pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "anilkumarbh/student-demo-app"
        DOCKER_CREDENTIALS_ID = "dockerhub-creds"
    }

    stages {
        stage('1. Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('2. Application Build & Test') {
            steps {
                echo "Verifying Python syntax..."
                bat 'python -m py_compile app.py'
            }
        }

        stage('3. Build & Tag Docker Image') {
            steps {
                echo "Building image tagged with Build #${env.BUILD_NUMBER} and latest..."
                bat "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} -t ${DOCKER_IMAGE}:latest ."
            }
        }

        stage('4. Push Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDENTIALS_ID}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat '''
                        echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                        docker push %DOCKER_IMAGE%:%BUILD_NUMBER%
                        docker push %DOCKER_IMAGE%:latest
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning up local images..."
            bat """
                docker logout
                docker rmi ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest 2>nul || exit /b 0
            """
        }
        success {
            echo "Pipeline succeeded! Image ${DOCKER_IMAGE}:${BUILD_NUMBER} is live on Docker Hub."
        }
    }
}
