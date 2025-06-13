pipeline {
    agent any

    environment {
        APP_PORT = '5000'
        VENV_DIR = 'venv'
    }

    stages {
        stage('Checkout Code') {
            steps {                
                git url: 'git@github.com:yunnppa/Jenkins.git', branch: 'master'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install -r requirements.txt
                '''
            }
        }
        stage('Run Tests') {
            steps {
                sh '''#!/bin/bash
                source venv/bin/activate
                pytest
                '''
            }
        }        

        stage('Deploy App Locally') {
            steps {
                // Stop any existing gunicorn process if running on port 5000
                sh '''
                    pkill -f "gunicorn" || true
                '''
                // Start the app with gunicorn in the background, binding to port 5000
                sh '''
                    . ${VENV_DIR}/bin/activate
                    nohup gunicorn --bind 0.0.0.0:5000 app:app > app.log 2>&1 &
                '''
            }
        }
    }

    post {
        failure {
            echo 'Deployment failed.'
        }
        success {
            echo 'Deployed successfully.'
        }
    }
}

stages {
        stage('Checkout Code') {
            steps {
                git url: 'git@github.com:xxxxxxxxxxxxxxxxxxxxx.git', branch: 'main' // Your SSH link
            }
        }

        stage('Setup Environment') {
            steps {
                sh 'python3 -m venv venv'
                sh '. venv/bin/activate && pip install -r requirements.txt'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''#!/bin/bash
                source venv/bin/activate
                pytest
                '''
            }
        }

        stage('Deploy App Locally') {
            steps {
                sh '''
                    pkill -f "gunicorn"  true
                '''
                sh '''
                    . ${VENV_DIR}/bin/activate
                    nohup gunicorn --bind 0.0.0.0:5000 app:app > app.log 2>&1 &
                '''
            }
        }
        stage('SCA Scan (Dependency-Check)') {
            steps {
                sh '''
                /opt/dependency-check/bin/dependency-check.sh \
                --scan . \
                --format HTML \
                --project MovieRecommender \
                --out . \
                --failOnCVSS 8
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'dependency-check-report.html', fingerprint: true
                }
                failure {
                    echo 'Dependency-Check scan failed or found vulnerabilities!'
                }
            }
        }
