pipeline {
    agent any

    environment {
        DEPLOY_DIR = 'C:\\Users\\kshitij-necsws\\Desktop\\test_deploy'
        VENV_PATH = "${DEPLOY_DIR}\\venv"
        BACKEND_DIR = "${DEPLOY_DIR}\\backend"
        FRONTEND_DIR = "${DEPLOY_DIR}\\frontend"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/KshitijNEC/ITHCSoftwareApp.git'
            }
        }

        stage('Deploy Locally to Desktop') {
            steps {
                bat '''
                    REM Create deploy folder if it doesn't exist
                    if not exist "%DEPLOY_DIR%" mkdir "%DEPLOY_DIR%"

                    REM Clean deploy folder
                    del /q "%DEPLOY_DIR%\\*" 2>nul
                    for /d %%i in ("%DEPLOY_DIR%\\*") do rmdir /s /q "%%i"

                    REM Copy project files to deploy folder
                    xcopy /E /I /Y * "%DEPLOY_DIR%\\"
                '''
            }
        }

        stage('Setup Environment & Install Dependencies') {
            steps {
                bat '''
                    cd "%DEPLOY_DIR%"
                    python -m venv venv
                    call venv\\Scripts\\activate

                    cd backend
                    pip install -r requirements.txt
                    pip install pytest pytest-cov pytest-html

                    cd ..\\frontend
                    npm install
                '''
            }
        }

        stage('Run Backend Tests') {
            steps {
                bat '''
                    call "%VENV_PATH%\\Scripts\\activate"
                    cd "%BACKEND_DIR%"
                    pytest --maxfail=1 --disable-warnings --html=report.html
                '''
            }
        }

        stage('Start Flask App') {
            steps {
                bat '''
                    call "%VENV_PATH%\\Scripts\\activate"
                    cd "%BACKEND_DIR%"
                    start /B python app.py
                '''
            }
        }
    }
}
