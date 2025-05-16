pipeline {
    agent any

    environment {
        DEPLOY_DIR = 'C:\\Users\\kshitij-necsws\\Desktop\\test_deploy'  // Change this to your desired path
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
                bat """
                    if exist "${DEPLOY_DIR}" rmdir /s /q "${DEPLOY_DIR}"
                    mkdir "${DEPLOY_DIR}"
                    xcopy * "${DEPLOY_DIR}" /E /I /Y

                    cd "${DEPLOY_DIR}\\backend"
                    python -m venv venv
                    call venv\\Scripts\\activate
                    pip install --prefer-binary numpy
                    pip install --prefer-binary -r requirements.txt

                    cd ..\\frontend
                    npm install
                """
            }
        }
    }
}
