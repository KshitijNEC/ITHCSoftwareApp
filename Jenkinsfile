pipeline {
    agent any

    environment {
        APP_NAME = 'ithcapp'
        DEPLOY_DIR = '/home/kshitij-necsws/Desktop/deploy_folder'
        VM_USER = 'kshitij-necsws'
        VM_HOST = '10.102.192.172'
        ZIP_FILE = 'app_package.zip'
        SSH_KEY = 'C:\\Users\\kshitij.waikar\\.ssh\\id_rsa.ppk'
        ZIP_REMOTE_PATH = '/home/kshitij-necsws/Desktop/test_deploy/app_package.zip'
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

        stage('Zip Project') {
            steps {
                powershell '''
                    $zipSuccess = $false
                    $attempts = 0

                    while (-not $zipSuccess -and $attempts -lt 3) {
                        try {
                            if (Test-Path "app_package.zip") {
                                Remove-Item "app_package.zip"
                            }

                            $backendFiles = Get-ChildItem -Path "backend" -Recurse -File | Select-Object -ExpandProperty FullName
                            $frontendFiles = Get-ChildItem -Path "frontend" -Recurse -File | Where-Object { $_.FullName -notmatch "node_modules" } | Select-Object -ExpandProperty FullName

                            $allFiles = $backendFiles + $frontendFiles

                            Compress-Archive -Path $allFiles -DestinationPath "app_package.zip"

                            $zipSuccess = $true
                        } catch {
                            Write-Host "Compress-Archive failed, retrying in 5 seconds..."
                            Start-Sleep -Seconds 5
                            $attempts++
                        }
                    }

                    if (-not $zipSuccess) {
                        throw "Compress-Archive failed after multiple attempts."
                    }
                '''
            }
        }

        stage('Transfer to VM') {
            steps {
                bat """
                    pscp -i "%SSH_KEY%" -q -C "C:\\ProgramData\\Jenkins\\.jenkins\\workspace\\deployment\\app_package.zip" %VM_USER%@%VM_HOST%:%ZIP_REMOTE_PATH%
                """
            }
        }

        stage('Setup and Run Flask on VM (plink)') {
            steps {
                bat """
                    plink -batch -i "%SSH_KEY%" %VM_USER%@%VM_HOST% ^
                    "rm -rf ${DEPLOY_DIR} && mkdir -p ${DEPLOY_DIR} && ^
                     unzip ${ZIP_REMOTE_PATH} -d ${DEPLOY_DIR} && rm ${ZIP_REMOTE_PATH} && ^
                     python3 -m venv ${DEPLOY_DIR}/venv && source ${DEPLOY_DIR}/venv/bin/activate && ^
                     cd ${DEPLOY_DIR}/backend && pip install --upgrade pip && pip install -r requirements.txt && ^
                     export FLASK_APP=app.py && flask db upgrade && ^
                     nohup flask run --host=0.0.0.0 --port=8000 &"
                """
            }
        }
    }

    post {
        success {
            echo '✅ Deployment succeeded!'
        }
        failure {
            echo '❌ Deployment failed!'
        }
    }
}
