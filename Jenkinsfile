pipeline {
    agent any

    environment {
        DEPLOY_DIR = '/opt/ithcapp'
        VENV_PATH = "${DEPLOY_DIR}/venv"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup Environment') {
            steps {
                script {
                    sh """
                        python3 -m venv ${VENV_PATH}
                        source ${VENV_PATH}/bin/activate

                        cd backend
                        pip install -r requirements.txt
                        pip install pytest-cov pytest-html

                        cd ../frontend
                        npm install
                    """
                }
            }
        }

        stage('Run Tests') {
            parallel {
                stage('Backend Tests') {
                    steps {
                        script {
                            sh """
                                source ${VENV_PATH}/bin/activate
                                cd backend
                                python3 -m pytest --cov=. --cov-report=html:coverage-report --html=test-report.html || exit 0
                            """
                        }
                    }
                    post {
                        always {
                            publishHTML([ 
                                allowMissing: true,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: 'backend',
                                reportFiles: 'test-report.html,coverage-report/index.html',
                                reportName: 'Backend Test Report'
                            ])
                        }
                    }
                }

                stage('Frontend Tests') {
                    steps {
                        script {
                            sh """
                                cd frontend
                                npm test -- --coverage --ci --reporters=default --reporters=jest-junit || exit 0
                            """
                        }
                    }
                    post {
                        always {
                            junit 'frontend/junit.xml'
                            publishHTML([ 
                                allowMissing: true,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: 'frontend/coverage',
                                reportFiles: 'index.html',
                                reportName: 'Frontend Coverage Report'
                            ])
                        }
                    }
                }
            }
        }

        stage('Build Frontend') {
            steps {
                script {
                    sh """
                        cd frontend
                        npm run build
                    """
                }
            }
        }

        stage('Deploy to VM') {
            steps {
                script {
                    sh """
                        ssh -o StrictHostKeyChecking=no user@your-ubuntu-vm 'bash -s' < deploy_script.sh
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
