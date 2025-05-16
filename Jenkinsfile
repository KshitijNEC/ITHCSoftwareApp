pipeline {
    agent any

    environment {
        DEPLOY_DIR = "/home/kshitij-necsws/Desktop/test_deploy"
        VENV_DIR = "${DEPLOY_DIR}/venv"
    }

    stages {
        stage('Checkout Code') {
            steps {
                dir("${DEPLOY_DIR}") {
                    checkout scm
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                dir("${DEPLOY_DIR}") {
                    sh """
                        python3 -m venv venv || true
                        source venv/bin/activate
                        pip install --upgrade pip
                        pip install -r backend/requirements.txt
                    """
                }
            }
        }

        stage('Run Flask App') {
            steps {
                dir("${DEPLOY_DIR}") {
                    sh """
                        source venv/bin/activate
                        nohup python3 backend/app.py > flask.log 2>&1 &
                    """
                }
            }
        }
    }
}
