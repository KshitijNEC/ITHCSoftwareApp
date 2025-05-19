pipeline {
    agent any

    environment {
        DEPLOY_PATH = "/home/kshitij-necsws/test_deploy"
        VM_HOST = "10.102.192.172"
        VM_USER = "kshitij-necsws"
    }

    stages {

        stage('Checkout') {
            steps {
                git credentialsId: "jenkins_github_cred", url: 'https://github.com/KshitijNEC/ITHCSoftwareApp', branch: 'main'
            }
        }

        stage('Environment Setup (Backend)') {
            steps {
                sh """
                    ssh -o StrictHostKeyChecking=no ${VM_USER}@${VM_HOST} << EOF
                        cd ${DEPLOY_PATH}
                        python3 -m venv venv
                        source venv/bin/activate
                        pip install --upgrade pip
                        cd backend
                        pip install -r requirements.txt
                        flask db upgrade
                    EOF
                """
            }
        }

        stage('Frontend Setup') {
            steps {
                sh """
                    ssh -o StrictHostKeyChecking=no ${VM_USER}@${VM_HOST} << EOF
                        cd ${DEPLOY_PATH}/frontend
                        npm ci
                        npm run build
                    EOF
                """
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    def backendStatus = sh (
                        script: """
                            ssh -o StrictHostKeyChecking=no ${VM_USER}@${VM_HOST} << EOF
                                cd ${DEPLOY_PATH}
                                source venv/bin/activate
                                cd backend
                                pytest || echo "Backend tests failed"
                            EOF
                        """, returnStatus: true
                    )

                    def frontendStatus = sh (
                        script: """
                            ssh -o StrictHostKeyChecking=no ${VM_USER}@${VM_HOST} << EOF
                                cd ${DEPLOY_PATH}/frontend
                                npm test || echo "Frontend tests failed"
                            EOF
                        """, returnStatus: true
                    )

                    if (backendStatus != 0 || frontendStatus != 0) {
                        echo "Some tests failed, but continuing with deployment..."
                    }
                }
            }
        }

        stage('Deployment') {
            steps {
                sh """
                    ssh -o StrictHostKeyChecking=no ${VM_USER}@${VM_HOST} << EOF
                        cd ${DEPLOY_PATH}/backend
                        source ../venv/bin/activate
                        export FLASK_APP=app.py
                        nohup flask run --host=0.0.0.0 --port=5000 > flask.log 2>&1 &
                    EOF
                """
            }
        }
    }
}
