pipeline {
    agent any

    environment {
        DEPLOY_DIR = '/home/kshitij-necsws/Desktop/test_deploy'
        VENV_PATH = "${DEPLOY_DIR}/venv"
        BACKEND_DIR = "${DEPLOY_DIR}/backend"
        FRONTEND_DIR = "${DEPLOY_DIR}/frontend"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/KshitijNEC/ITHCSoftwareApp.git'
            }
        }

        stage('Deploy Locally (Linux)') {
            steps {
                sh """
                    mkdir -p "${DEPLOY_DIR}"
                    rm -rf "${DEPLOY_DIR:?}/"*
                    cp -r * "${DEPLOY_DIR}"
                """
            }
        }

        stage('Setup Environment & Install Dependencies') {
            steps {
                sh """
                    python3 -m venv "${VENV_PATH}"
                    source "${VENV_PATH}/bin/activate"
                    cd "${BACKEND_DIR}"
                    pip install -r requirements.txt
                    pip install pytest pytest-cov pytest-html
                    cd "${FRONTEND_DIR}"
                    npm install
                """
            }
        }

        stage('Run Backend Tests') {
            steps {
                sh """
                    source "${VENV_PATH}/bin/activate"
                    cd "${BACKEND_DIR}"
                    pytest --maxfail=1 --disable-warnings --html=report.html
                """
            }
        }

        stage('Start Flask App') {
            steps {
                sh """
                    source "${VENV_PATH}/bin/activate"
                    cd "${BACKEND_DIR}"
                    nohup python3 app.py > flask.log 2>&1 &
                """
            }
        }
    }
}
