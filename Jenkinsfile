#!groovy
pipeline {
    agent none
    environment {
        REPO_NAME = "imageforpipe"
        FULL_IMAGE = ""
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
        stage('Unit Tests') {
            agent { label 'worker1' }
            steps {
                echo 'Start unit tests...'
                sh '''
                    . venv/bin/activate
                    pip install pytest pytest-asyncio httpx fastapi
                    pytest test_unit.py --junitxml=unit_report.xml
                '''
            }
        }
        stage('Build') {
            agent { label 'worker2' }
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    script {
                        def tag = "${USER}/${REPO_NAME}:${env.BUILD_NUMBER}"
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh 'docker system prune -f'
                        sh "docker build -t ${tag} ."
                        sh "docker push ${tag}"
                        env.DEPLOY_TAG = tag
                    }
                }
            } 
        }
        stage('Deploy') {
            agent { label 'worker2' }
            steps {
                withCredentials([file(credentialsId: 'ENV_FILE', variable: 'SECRET_FILE_PATH')]) {
                    sh '''
                        IMAGE_NAME=${env.DEPLOY_TAG} \
                        docker compose --env-file ${SECRET_FILE_PATH} down --remove-orphans
                        IMAGE_NAME=${env.DEPLOY_TAG} \
                        docker compose --env-file ${SECRET_FILE_PATH} up -d
                    '''
                }
                sleep 5
            }
        }
        stage('Integration_tests') {
            agent { label 'worker1' }
            steps {
                echo 'Running tests...'
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install -r requirements.txt
                    pytest test_currency_app.py --junitxml=integration_report.xml
                '''
                sleep 3
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