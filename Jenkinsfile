pipeline {
    agent any

    environment {
        APP_NAME = 'ithcapp'
        DEPLOY_DIR = '/home/kshitij-necsws/Desktop/deploy_folder'
        VM_USER = 'kshitij-necsws'
        VM_HOST = '10.102.192.172'
        TAR_FILE = 'app_package.tar.gz'
        SSH_KEY = 'C:\\Users\\kshitij.waikar\\.ssh\\id_rsa'
        REMOTE_TAR_PATH = '/home/kshitij-necsws/Desktop/test_deploy/app_package.tar.gz'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/KshitijNEC/ITHCSoftwareApp.git'
            }
        }

        stage('Install Frontend Dependencies') {
            steps {
                bat '''
                    cd frontend
                    call npm install
                '''
                sleep(time: 10, unit: 'SECONDS')
            }
        }

        stage('Pre-Zip Cleanup') {
            steps {
                powershell '''
                    if (Test-Path "backend/uploads") {
                        Remove-Item -Path "backend/uploads/*" -Force -Recurse -ErrorAction SilentlyContinue
                    }
                '''
            }
        }

        stage('Tar Project') {
            steps {
                powershell '''
                    if (Test-Path "app_package.tar.gz") {
                        Remove-Item "app_package.tar.gz" -Force
                    }

                    # Tar backend and frontend, excluding node_modules and .git
                    tar --exclude="frontend/node_modules" --exclude="**/.git" -czf app_package.tar.gz backend frontend
                '''
            }
        }

        stage('Transfer to VM') {
            steps {
                bat """
                    scp "%SSH_KEY%" -o StrictHostKeyChecking=no "%WORKSPACE%\\%TAR_FILE%" %VM_USER%@%VM_HOST%:%REMOTE_TAR_PATH%
                """
            }
        }

        stage('Setup and Run Flask on VM') {
            steps {
                bat """
                    ssh -i "%SSH_KEY%" -o StrictHostKeyChecking=no %VM_USER%@%VM_HOST% ^
                    "rm -rf ${DEPLOY_DIR} && mkdir -p ${DEPLOY_DIR} && ^
                     tar -xzf ${REMOTE_TAR_PATH} -C ${DEPLOY_DIR} && rm ${REMOTE_TAR_PATH} && ^
                     python3 -m venv ${DEPLOY_DIR}/venv && source ${DEPLOY_DIR}/venv/bin/activate && ^
                     cd ${DEPLOY_DIR}/backend && pip install --upgrade pip && pip install -r requirements.txt && ^
                     export FLASK_APP=app.py && flask db upgrade && ^
                     nohup flask run --host=0.0.0.0 --port=8000 &"
                """
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo '✅ Deployment succeeded!'
        }
        failure {
            echo '❌ Deployment failed!'
        }
    }
}
