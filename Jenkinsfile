#!groovy
pipeline {
    agent any
    environment {
        IMAGE = "myapp:${env.BUILD_NUMBER}"
    }
    options {
        timestamps()
    }
    // triggers { pollSCM('* * * * *') }
    stages {
        stage('Lint') {
            steps {
                echo 'Running linter...'
                sh 'pip install flake8'
                sh 'flake8 .'
                sleep 2
            }
        }
        stage('Test') {
            steps 
                echo 'Running steps'
                sleep 3
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