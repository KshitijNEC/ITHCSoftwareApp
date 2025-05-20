pipeline {
    agent any

    environment {
        APP_NAME = 'ithcapp'
        DEPLOY_DIR = '/home/kshitij-necsws/deploy_folder'
        VM_USER = 'kshitij-necsws'
        VM_HOST = '10.102.192.172'
        ZIP_FILE = 'app_package.zip'
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
                sleep(time: 5, unit: 'SECONDS') // short delay to release file locks
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

        stage('Zip Project') {
            steps {
                powershell '''
                    if (Test-Path "app_package.zip") {
                        Remove-Item "app_package.zip"
                    }

                    $backendFiles = Get-ChildItem -Path "backend" -Recurse
                    $frontendFiles = Get-ChildItem -Path "frontend" -Recurse | Where-Object { $_.FullName -notmatch "node_modules" }

                    Compress-Archive -Path $backendFiles.FullName + $frontendFiles.FullName -DestinationPath "app_package.zip"
                '''
            }
        }

        stage('Transfer to VM') {
            steps {
                bat """
                    pscp app_package.zip %VM_USER%@%VM_HOST%:/tmp/%ZIP_FILE%
                """
            }
        }

        stage('Setup and Run Flask on VM') {
            steps {
                bat """
                    plink -batch %VM_USER%@%VM_HOST% ^
                    "rm -rf ${DEPLOY_DIR} && mkdir -p ${DEPLOY_DIR} && ^
                     unzip /tmp/${ZIP_FILE} -d ${DEPLOY_DIR} && rm /tmp/${ZIP_FILE} && ^
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
