pipeline {
    agent any

    environment {
        APP_NAME = 'ithcapp'
        DEPLOY_DIR = '/home/kshitij-necsws/Desktop/test_deploy'
        VM_USER = 'kshitij-necsws'
        VM_HOST = '10.102.192.172'
        TAR_FILE = 'app_package.tar.gz'
        SSH_KEY = 'C:\\Users\\kshitij.waikar\\.ssh\\id_rsa'
        REMOTE_TAR_PATH = '/home/kshitij-necsws/Desktop/test_deploy/app_package.tar.gz'
        GIT_BASH = 'C:\\Users\\kshitij.waikar\\AppData\\Local\\Programs\\Git\\git-bash.exe'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/KshitijNEC/ITHCSoftwareApp.git'
            }
        }

        stage('Install Frontend Dependencies') {
            steps {
                bat '''
                    cd frontend
                    call npm install
                '''
                sleep(time: 10, unit: 'SECONDS')
            }
        }

        stage('Pre-Zip Cleanup') {
            steps {
                powershell '''
                    if (Test-Path "backend/uploads") {
                        Remove-Item -Path "backend/uploads/*" -Force -Recurse -ErrorAction Silent-lyContinue
                    }
                '''
            }
        }

        stage('Tar Project') {
            steps {
                powershell '''
                    if (Test-Path "app_package.tar.gz") {
                        Remove-Item "app_package.tar.gz" -Force
                    }

                    tar --exclude="frontend/node_modules" --exclude="**/.git" -czf app_package.tar.gz backend frontend
                '''
            }
        }

        stage('Transfer to VM') {
            steps {
                bat """
                    "%GIT_BASH%" -c "scp -i /c/Users/kshitij.waikar/.ssh/id_rsa -o StrictHostKey-Checking=no /c/ProgramData/Jenkins/.jenkins/workspace/deployment/app_package.tar.gz ${VM_USER}@${VM_HOST}:${REMOTE_TAR_PATH}"
                """
            }
        }

      stage('Setup, Test, and Prepare Flask on VM') {
    steps {
        bat """
            "%GIT_BASH%" -c "ssh -i /c/Users/kshitij.waikar/.ssh/id_rsa -o StrictHostKeyChecking=no ${VM_USER}@${VM_HOST} 'tar -xzf ${REMOTE_TAR_PATH} -C ${DEPLOY_DIR} && cd ${DEPLOY_DIR} && python3 -m venv venv && source venv/bin/activate && pip install --upgrade pip && pip install -r backend/requirements.txt && cd backend && flask db upgrade && python -m pytest --cov=. --cov-report=html:coverage-report && cd ../frontend && npm install && npm test -- --coverage'"
        """
    }
}

        }
         stage('Run Flask on VM') {
            steps {
                bat """
                    "%GIT_BASH%" -c "ssh -i /c/Users/kshitij.waikar/.ssh/id_rsa -o StrictHostKey-Checking=no ${VM_USER}@${VM_HOST} 'cd ${DEPLOY_DIR}/backend && python -m flask run --host=0.0.0.0 --port=5000'"
                """
            }
         }
    }

    post {
        // always {
        //     cleanWs()
        // }
        success {
            echo '✅ Deployment succeeded!'
        }
        failure {
            echo '❌ Deployment failed!'
        }
    }
}
