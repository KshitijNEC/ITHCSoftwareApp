pipeline {
    agent any

    environment {
        DEPLOY_DIR = 'C:\\opt\\ithcapp'
        VENV_PATH = "${DEPLOY_DIR}\\venv"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup Environment') {
            steps {
                bat """
                    python -m venv ${VENV_PATH}
                    call ${VENV_PATH}\\Scripts\\activate

                    cd backend
                    pip install -r requirements.txt
                    pip install pytest-cov pytest-html

                    cd ..\\frontend
                    npm install
                """
            }
        }

        stage('Run Tests') {
            parallel {
                stage('Backend Tests') {
                    steps {
                        bat """
                            call ${VENV_PATH}\\Scripts\\activate
                            cd backend
                            python -m pytest --cov=. --cov-report=html:coverage-report --html=test-report.html || exit /b 0
                        """
                    }
                    post {
                        always {
                            publishHTML([
                                allowMissing: true,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: 'backend',
                                reportFiles: 'test-report.html,coverage-report\\index.html',
                                reportName: 'Backend Test Report'
                            ])
                        }
                    }
                }

                stage('Frontend Tests') {
                    steps {
                        bat """
                            cd frontend
                            npm test -- --coverage --ci --reporters=default --reporters=jest-junit || exit /b 0
                        """
                    }
                    post {
                        always {
                            junit 'frontend\\junit.xml'
                            publishHTML([
                                allowMissing: true,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: 'frontend\\coverage',
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
                bat """
                    cd frontend
                    npm run build
                """
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
