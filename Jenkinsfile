pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "dikshith118/voise-hospital-predictor"
        CONTAINER_NAME = "voise-hospital-container"
    }

    stages {

        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Git Version Check') {
            steps {
                bat 'git --version'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'python -m pip install -r requirements.txt'
            }
        }

        stage('Code Quality - PyLint') {
            steps {
                bat 'python -m pip install pylint && python -m pylint app.py logic.py --exit-zero'
            }
        }

        stage('Vulnerability Scan - Trivy') {
            steps {
                bat 'trivy fs --severity HIGH,CRITICAL --exit-code 0 .'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %DOCKER_IMAGE%:latest ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat """
                        echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                        docker push %DOCKER_IMAGE%:latest
                    """
                }
            }
        }

        stage('Deploy Container') {
            steps {
                bat """
                    docker stop %CONTAINER_NAME% || exit 0
                    docker rm %CONTAINER_NAME% || exit 0
                    docker run -d --name %CONTAINER_NAME% -p 5000:5000 %DOCKER_IMAGE%:latest
                """
            }
        }
    }

    post {
        success {
            echo 'Pipeline 1 completed successfully!'
        }
        failure {
            echo 'Pipeline 1 failed!'
        }
        always {
            echo 'Post actions done.'
        }
    }
}