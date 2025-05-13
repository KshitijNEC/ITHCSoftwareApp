pipeline {
    agent any

    environment {
        DEPLOY_DIR = "C:\\Users\\kshitij.waikar\\deploy_folder"
        VM_USER = "kshitij-necsws"
        VM_HOST = "10.102.192.172"
        SSH_KEY_CREDENTIALS_ID = 'jenkins_vm_ssh_key'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', credentialsId: 'jenkins_github_cred', url: 'https://github.com/KshitijNEC/ITHCSoftwareApp.git'
            }
        }

      stage('Setup Backend Dependencies') {
    steps {
        dir('backend') {
            bat '''
                python -m venv venv
                call venv\\Scripts\\activate
                pip install -r requirements.txt
            '''
        }
    }
}


        stage('Setup Frontend Dependencies') {
            steps {
                bat '''
                cd frontend
                call npm install
                '''
            }
        }

        stage('Run Backend Tests') {
            steps {
                bat '''
                cd backend
                venv\\Scripts\\activate && pytest
                '''
            }
        }

        stage('Run Frontend Tests') {
            steps {
                bat '''
                cd frontend
                call npm test
                '''
            }
        }

        stage('Deploy to VM') {
            steps {
                sshagent (credentials: ["${env.SSH_KEY_CREDENTIALS_ID}"]) {
                    bat """
                    powershell -Command "scp -r backend ${VM_USER}@${VM_HOST}:/home/${VM_USER}/Desktop/deploy_folder/"
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
