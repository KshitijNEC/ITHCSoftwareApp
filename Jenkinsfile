pipeline {
    agent any

    environment {
        APP_NAME = 'ithcapp'
        DEPLOY_DIR = '/application_deploy/deploy_folder'
        VENV_PATH = "${DEPLOY_DIR}/venv"
        VM_USER = 'kshitij-necsws'
        VM_HOST = '10.102.192.172'
        APP_PATH = '/home/kshitij-necsws/Desktop/deploy_folder'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/KshitijNEC/ITHCSoftwareApp.git'
            }
        }

        stage('Setup Environment') {
            steps {
                bat '''
                    python -m venv venv
                    call venv\\Scripts\\activate

                    cd backend

                    REM Install numpy first to help with pandas install
                    pip install --prefer-binary numpy

                    REM Install other dependencies with prefer-binary
                    pip install --prefer-binary -r requirements.txt

                    pip install pytest-cov pytest-html

                    cd ..\\frontend
                    npm install
                '''
            }
        }
    }
}
