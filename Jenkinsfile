#!groovy
pipeline {
    agent none
    environment {
        IMAGE = "myapp:${env.BUILD_NUMBER}"
    }
    options {
        timestamps()
    }
    stages {
        stage('Lint') {
            agent { label 'worker1' }
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
        stage('Build') {
            agent { label 'worker2' }
            steps {
                withCredentials([file(credentialsId: 'ENV_FILE', variable: 'APP_ENV_FILE')]) {
                    sh 'docker system prune -a --volumes -f'
                    sh 'docker compose up -d'
                    sh 'docker compose ps'
                    }
                sleep 4
            } 
        }
        stage('Test') {
            agent { label 'worker1' }
            steps {
                echo 'Running tests...'
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install -r requirements.txt
                    pytest --junitxml=report.xml
                '''
                sleep 3
            }
        }
        stage('Push') {
            agent { label 'worker2' }
            steps {
                echo 'Pushing image to regisrty...'
                sleep 2
            }
        }
        stage('Deploy') {
            agent { label 'worker2' }
            steps {
                echo 'Deploying to staging...'
                sleep 2
            }
        }
    }
    post {
        always {
            script {
                node ('worker2'){
                    echo 'Cleanup: removing local image...'
                    withCredentials([file(credentialsId: 'ENV_FILE', variable: 'APP_ENV_FILE')]) {
                        sh 'docker compose down --remove-orphans -v'
                        sh 'docker compose ps'
                        sleep 1
                    }
                }
            }
        }
        success {
            echo 'Pipeline finished successfully, build by commit'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}