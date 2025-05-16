pipeline {
    agent any

    environment {
        DEPLOY_DIR = '/home/kshitij-necsws/Desktop/test_deploy'  // Your target local path
        REPO_URL = 'https://github.com/KshitijNEC/ITHCSoftwareApp.git'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: "${REPO_URL}"
            }
        }

        stage('Deploy Locally') {
            steps {
                sh """
                    mkdir -p "${DEPLOY_DIR}"
                    rm -rf "${DEPLOY_DIR:?}/"*
                    cp -r * "${DEPLOY_DIR}"

                    cd "${DEPLOY_DIR}/backend"
                    python3 -m venv venv
                    source venv/bin/activate
                    pip install --prefer-binary numpy
                    pip install --prefer-binary -r requirements.txt

                    cd "${DEPLOY_DIR}/frontend"
                    npm install
                """
            }
        }
    }
}
