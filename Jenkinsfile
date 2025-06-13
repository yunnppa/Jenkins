pipeline {
    agent any

    environment {
        APP_PORT = '5000'
        VENV_DIR = 'venv'
        // Define APP_URL dynamically using the APP_PORT.
        // Assuming Jenkins agent and the application run on the same host.
        APP_URL = "http://localhost:${APP_PORT}"
        ZAP_REPORT_PATH = "zap_report.html" // Define a variable for the ZAP report path
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Added credentialsId back as it was present in the original log.
                git url: 'git@github.com:yunnppa/Jenkins.git', branch: 'master', credentialsId: 'yunnppa'
            }
        }

        stage('Setup Python Environment') { // Renamed for clarity and combined previous 'Setup Environment' and 'Install Dependencies'
            steps {
                sh 'python3 -m venv ${VENV_DIR}' // Use VENV_DIR variable
                sh '. ${VENV_DIR}/bin/activate && pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                sh '''#!/bin/bash
                source ${VENV_DIR}/bin/activate
                pytest
                '''
            }
        }

        stage('Deploy App Locally') {
            steps {
                // Kill any existing gunicorn process to ensure a clean start.
                sh 'pkill -f "gunicorn" || true'
                // Activate virtual environment and start gunicorn in the background.
                // Redirecting output to a log file and sending to background with &
                sh '. ${VENV_DIR}/bin/activate && nohup gunicorn --bind 0.0.0.0:${APP_PORT} app:app > app.log 2>&1 &'
                // Add a delay to ensure the application is fully started before ZAP scans it.
                script {
                    echo "Waiting 15 seconds for the application to start..."
                    sleep 15
                }
            }
        }

        stage('DAST Scan (OWASP ZAP)') {
            steps {
                // Execute OWASP ZAP scan.
                // APP_URL is now defined in the environment block.
                // ZAP_REPORT_PATH is also defined.
                sh """
                /opt/owasp-zap/zap.sh -cmd \\
                    -port 8090 -host 127.0.0.1 \\
                    -config api.disablekey=true \\
                    -newsession zap_scan \\
                    -url ${APP_URL} \\
                    -autorun \\
                    -htmlreport ${ZAP_REPORT_PATH}
                """
            }
            post {
                always {
                    // Archive the generated ZAP report.
                    archiveArtifacts artifacts: "${ZAP_REPORT_PATH}", fingerprint: true
                }
                failure {
                    echo 'ZAP DAST scan failed or found vulnerabilities!'
                }
            }
        }

        stage('Cleanup') {
            steps {
                // Stop the gunicorn process
                sh 'pkill -f "gunicorn" || true'
                // Remove the virtual environment
                sh 'rm -rf ${VENV_DIR}'
            }
        }
    }
}
