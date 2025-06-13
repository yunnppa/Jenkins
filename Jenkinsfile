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
                    pkill -f "gunicorn" || true
                '''
                sh '''
                    . ${VENV_DIR}/bin/activate
                    nohup gunicorn --bind 0.0.0.0:5000 app:app > app.log 2>&1 &
                '''
            }
        }

    //     stage('SCA Scan (Dependency-Check)') {
    //         steps {
    //             sh '''
    //             /opt/dependency-check/bin/dependency-check.sh \
    //                 --scan . \
    //                 --format HTML \
    //                 --project MovieRecommender \
    //                 --out . \
    //                 --failOnCVSS 8
    //             '''
    //         }
    //     }

    //         post {
    //             always {
    //                 archiveArtifacts artifacts: 'dependency-check-report.html', fingerprint: true
    //             }
    //             failure {
    //                 echo 'Dependency-Check scan failed or found vulnerabilities!'
    //             }
    //         }
    //     }
    // }

        stage('DAST Scan (OWASP ZAP)') {
            steps {
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
                    archiveArtifacts artifacts: '**/zap_report.html', fingerprint: true
                }
                failure {
                    echo 'ZAP DAST scan failed or found vulnerabilities!'
                }
            }
        }

        stage('Cleanup') {
            steps {
                sh 'rm -rf venv'
            }
        }
    }  // <-- Only ONE closing brace here for stages

} // <-- And this closes the pipeline block

   
