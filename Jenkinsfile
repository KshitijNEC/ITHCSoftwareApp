pipeline {
    agent any

    environment {
        VM_USER = 'kshitij-necsws'
        VM_HOST = '10.102.192.172'
        DEPLOY_DIR = '/home/kshitij-necsws/Desktop/test_deploy'
        ZIP_FILE = 'app_package.zip'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/KshitijNEC/ITHCSoftwareApp.git'
            }
        }

        stage('Zip Entire Project') {
            steps {
                powershell '''
                    $zipPath = "app_package.zip"
                    if (Test-Path $zipPath) { Remove-Item $zipPath }

                    Compress-Archive -Path * -DestinationPath $zipPath -Force
                '''
            }
        }

        stage('Transfer to VM via SCP') {
            steps {
                powershell '''
                    $source = "C:/ProgramData/Jenkins/.jenkins/workspace/deployment/app_package.zip"
                    $target = "${env.VM_USER}@${env.VM_HOST}:${env.DEPLOY_DIR}/"
                    Write-Host "📤 Sending $source to $target..."
                    scp $source $target
                '''
            }
        }

        stage('Unzip and Run on VM') {
            steps {
                powershell '''
                    ssh ${env.VM_USER}@${env.VM_HOST} @"
                        cd ${env.DEPLOY_DIR}
                        unzip -o app_package.zip
                        rm app_package.zip
                        python3 -m venv venv
                        source venv/bin/activate
                        cd backend
                        pip install --upgrade pip
                        pip install -r requirements.txt
                        export FLASK_APP=app.py
                        flask db upgrade
                        nohup flask run --host=0.0.0.0 --port=8000 &
                    "@
                '''
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
