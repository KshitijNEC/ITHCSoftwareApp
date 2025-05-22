pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/KshitijNEC/ITHCSoftwareApp.git'
        BRANCH = 'main'
        APP_NAME = 'ithcapp'
        DEPLOY_DIR = '/home/kshitij-necsws/Desktop/test_deploy'
        ZIP_FILE = 'app_package.zip'
        LOCAL_WORKSPACE = 'C:/ProgramData/Jenkins/.jenkins/workspace/deployment'
        VM_USER = 'kshitij-necsws'
        VM_HOST = '10.102.192.172'
        SSH_KEY = 'ubuntu_vm_access_key'  // ✅ Updated
    }

    stages {

        stage('Clone Repository') {
            steps {
                dir("${env.LOCAL_WORKSPACE}") {
                    git branch: "${env.BRANCH}", url: "${env.GIT_REPO}"
                }
            }
        }

        stage('Zip Application') {
            steps {
                powershell """
                    Compress-Archive -Path ${env.LOCAL_WORKSPACE}\\* -DestinationPath ${env.LOCAL_WORKSPACE}\\${env.ZIP_FILE} -Force
                """
            }
        }

        stage('Transfer ZIP to VM') {
    steps {
        script {
            sshagent(credentials: ["ubuntu_vm_access_key"]) {
                bat """scp ${LOCAL_WORKSPACE}\\${ZIP_FILE} ${VM_USER}@${VM_HOST}:${DEPLOY_DIR}/"""
            }
        }
    }
}


        stage('Remote Setup & Deploy on VM') {
            steps {
                sshagent(credentials: ["${env.SSH_KEY}"]) {
                    sh """
                        ssh ${env.VM_USER}@${env.VM_HOST} << 'ENDSSH'
                        set -e
                        cd ${DEPLOY_DIR}
                        unzip -o ${ZIP_FILE}

                        cd ITHCSoftwareApp

                        # Backend setup
                        cd backend
                        python3 -m venv venv
                        source venv/bin/activate
                        pip install -r requirements.txt
                        flask db upgrade
                        deactivate

                        # Frontend setup
                        cd ../frontend
                        npm install

                        echo "Deployment Completed. Start backend manually if needed:"
                        echo "source ${DEPLOY_DIR}/ITHCSoftwareApp/backend/venv/bin/activate && python -m flask run"
                        ENDSSH
                    """
                }
            }
        }
    }

    post {
        failure {
            echo "❌ Deployment failed."
        }
        success {
            echo "✅ Deployment successful."
        }
    }
}
