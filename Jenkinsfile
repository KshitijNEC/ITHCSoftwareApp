pipeline {
    agent any

    environment {
        DEPLOY_DIR = "/home/kshitij-necsws/Desktop/test_deploy"
        VENV_DIR = "${DEPLOY_DIR}/venv"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Clean Deployment Folder') {
            steps {
                sh "rm -rf ${DEPLOY_DIR}/*"
            }
        }

        stage('Copy Project Files') {
            steps {
                sh "cp -r * ${DEPLOY_DIR}"
            }
        }

        stage('Setup Python Environment') {
            steps {
                sh """
                    python3 -m venv ${VENV_DIR}
                    source ${VENV_DIR}/bin/activate
                    pip install --upgrade pip
                    pip install -r ${DEPLOY_DIR}/backend/requirements.txt
                """
            }
        }

        stage('Run Flask App') {
            steps {
                sh """
                    source ${VENV_DIR}/bin/activate
                    nohup python3 ${DEPLOY_DIR}/backend/app.py > ${DEPLOY_DIR}/flask.log 2>&1 &
                """
            }
        }
    }
}
