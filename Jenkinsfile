#!groovy
pipeline {
    agent any
    environment {
        IMAGE = "myapp:${env.BUILD_NUMBER}"
    }
    options {
        timestamps()
    }
    stages {
        stage('Lint') {
            steps {
                echo 'Running linter...'
                withCredentials([string(credentialsId: 'SUDO_PASS', variable: 'SUDO_PASSWORD')]) {
                    sh "echo $SUDO_PASSWORD | sudo -S apt-get update && sudo -S apt-get install -y python3-venv"
                }
                sh 'python3 -m venv venv'
                sh './venv/bin/pip install flake8'
                sh './venv/bin/flake8 . --exclude=venv,.git,__pycache__,.pytest_cache'
                sleep 2
            }
        }
        stage('Test') {
            steps {
                echo 'Running tests...'
                sleep 3
            }
        }
        stage('Build') {
            steps {
                echo "Building Docker image ${IMAGE}..."
                sleep 4
            }
        }
        stage('Push') {
            steps {
                echo 'Pushing image to regisrty...'
                sleep 2
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying to staging...'
                sleep 2
            }
        }
    }
    post {
        always {
            echo 'Cleanup: removing local image...'
            sleep 1
        }
        success {
            echo 'Pipeline finished successfully, build by commit'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}