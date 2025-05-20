pipeline {
    agent any

    environment {
        APP_NAME = 'ithcapp'
        DEPLOY_DIR = '/home/kshitij-necsws/deploy_folder'
        VENV_PATH = "${DEPLOY_DIR}/venv"
        VM_USER = 'kshitij-necsws'
        VM_HOST = '10.102.192.172'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/KshitijNEC/ITHCSoftwareApp.git'
            }
        }

        stage('Build Frontend') {
            steps {
                bat '''
                    cd frontend
                    call npm install
                    call npm run build
                '''
            }
        }

        stage('Prepare Archive for Deployment') {
            steps {
                powershell '''
                    if (!(Test-Path build_artifacts)) {
                        New-Item -ItemType Directory -Path build_artifacts
                    }
                    Copy-Item -Recurse -Force backend build_artifacts/
                    Copy-Item -Recurse -Force frontend build_artifacts/
                '''
            }
        }

        stage('Deploy to VM') {
            steps {
                bat """
                    pscp -r build_artifacts\\* %VM_USER%@%VM_HOST%:/tmp/%APP_NAME%
                """
            }
        }

        stage('Setup on VM') {
            steps {
                bat """
                    plink %VM_USER%@%VM_HOST% ^
                    "rm -rf $DEPLOY_DIR && mkdir -p $DEPLOY_DIR && cp -r /tmp/$APP_NAME/* $DEPLOY_DIR && rm -rf /tmp/$APP_NAME && ^
                    cd $DEPLOY_DIR && python3 -m venv venv && source venv/bin/activate && ^
                    cd backend && pip install --upgrade pip && pip install -r requirements.txt && ^
                    export FLASK_APP=app.py && flask db upgrade && nohup flask run --host=0.0.0.0 --port=8000 &"
                """
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Deployment succeeded!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
