pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        GIT_REPO = 'https://github.com/KshitijNEC/ITHCSoftwareApp.git'
        DEPLOY_FOLDER = '/home/kshitij-necsws/Desktop/deploy_folder'
        VM_USER = 'kshitij-necsws'
        VM_HOST = '10.102.192.172'
        VM_PATH = '/home/kshitij-necsws/Desktop/deploy_folder'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git url: "${GIT_REPO}", branch: 'main'
            }
        }

        stage('Setup Backend Dependencies') {
            steps {
                sh '''
                    python3 -m venv venv
                    source venv/bin/activate
                    cd backend
                    pip install -r requirements.txt
                    flask db upgrade
                '''
            }
        }

        stage('Setup Frontend Dependencies') {
            steps {
                sh '''
                    cd frontend
                    npm install
                '''
            }
        }

        stage('Run Backend Tests') {
            steps {
                sh '''
                    source venv/bin/activate
                    cd backend
                    pytest --cov=.
                '''
            }
        }

        stage('Run Frontend Tests') {
            steps {
                sh '''
                    cd frontend
                    npm test -- --coverage
                '''
            }
        }

        stage('Deploy to VM') {
            steps {
                sshagent (credentials: ['vm-ssh-cred']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${VM_USER}@${VM_HOST} 'rm -rf ${VM_PATH}/*'
                        scp -r * ${VM_USER}@${VM_HOST}:${VM_PATH}
                        ssh ${VM_USER}@${VM_HOST} << EOF
                            cd ${VM_PATH}/backend
                            source ../venv/bin/activate
                            nohup python3 -m flask run --host=0.0.0.0 --port=5000 &
                        EOF
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
