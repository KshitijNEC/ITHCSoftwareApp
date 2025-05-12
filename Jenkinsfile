pipeline {
    agent { label 'linux' }

    environment {
        VENV_PATH = '.venv'
        REPORT_PATH = 'backend/test-report.html'
        DEPLOY_USER = 'kshitij-necsws'              // CHANGE THIS
        DEPLOY_HOST = 'kshitij-necsws'        // CHANGE THIS
        DEPLOY_DIR = '/opt/ithcapp'
        RECIPIENTS = 'kshitj.waikar@necsws.com' // CHANGE THIS
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Setup Python Environment') {
            steps {
                sh """
                    python3 -m venv ${VENV_PATH}
                    source ${VENV_PATH}/bin/activate
                    pip install --upgrade pip
                    pip install -r backend/requirements.txt
                    pip install pytest pytest-html
                """
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh """
                    source ${VENV_PATH}/bin/activate
                    cd backend
                    pytest --html=test-report.html || exit 1
                """
            }
            post {
                always {
                    mail to: "${RECIPIENTS}",
                         subject: "Test Report: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                         body: "Find attached the test report for ${env.JOB_NAME}.",
                         attachLog: true,
                         attachmentsPattern: "**/backend/test-report.html"
                }
            }
        }

        stage('Deploy to Ubuntu VM') {
            when {
                expression {
                    return currentBuild.result == null || currentBuild.result == 'SUCCESS'
                }
            }
            steps {
                sh """
                    ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_HOST} << 'ENDSSH'
                        set -e
                        mkdir -p ${DEPLOY_DIR}
                        cd ${DEPLOY_DIR}

                        # Clone or pull latest
                        if [ ! -d ".git" ]; then
                            git clone https://github.com/your-user/ITHCSoftwareApp.git .
                        else
                            git pull origin main
                        fi

                        python3 -m venv venv
                        source venv/bin/activate
                        pip install --upgrade pip
                        pip install -r backend/requirements.txt

                        # Kill old app and start new
                        pkill -f backend/app.py || true
                        nohup python3 backend/app.py > app.log 2>&1 &
                    ENDSSH
                """
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo '✅ Deployment succeeded.'
        }
        failure {
            echo '❌ Build failed.'
        }
    }
}
