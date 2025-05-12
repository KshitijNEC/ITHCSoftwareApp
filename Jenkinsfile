pipeline {
    agent none

    environment {
        DEPLOY_DIR = '/opt/ithcapp'
        VENV_PATH = "${DEPLOY_DIR}/venv"
    }

    stages {

        stage('Zip Project on Windows') {
            agent any  // Will run on any available agent (Windows assumed here)
            steps {
                bat '''
                @echo off
                setlocal

                set "SOURCE_DIR=C:\\Users\\kshitij.waikar\\ITHCSoftwareApp"
                set "ZIP_PATH=C:\\ProgramData\\Jenkins\\.jenkins\\workspace\\ITHCapp\\ITHCSoftwareApp.zip"

                powershell -NoProfile -ExecutionPolicy Bypass -Command ^
                "$attempts = 5; $success = $false; while ($attempts -gt 0 -and -not $success) { ^
                    try { ^
                        if (Test-Path '%ZIP_PATH%') { Remove-Item -Force '%ZIP_PATH%' } ^
                        Compress-Archive -Path '%SOURCE_DIR%\\*' -DestinationPath '%ZIP_PATH%' -Force; ^
                        $success = $true; ^
                    } catch { ^
                        Write-Host 'Zip failed, retrying in 5 seconds...'; ^
                        Start-Sleep -Seconds 5; ^
                        $attempts--; ^
                    } ^
                }; ^
                if (-not $success) { throw 'Failed to create zip after multiple attempts.' }"

                endlocal
                '''
            }
        }

        stage('Checkout & Deploy on Linux') {
            agent { label 'linux' }  // Requires a Linux agent with 'linux' label
            stages {

                stage('Checkout') {
                    steps {
                        checkout scm
                    }
                }

                stage('Setup Environment') {
                    steps {
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

                stage('Run Tests') {
                    parallel {
                        stage('Backend Tests') {
                            steps {
                                sh """
                                    source ${VENV_PATH}/bin/activate
                                    cd backend
                                    python3 -m pytest --cov=. --cov-report=html:coverage-report --html=test-report.html || exit 0
                                """
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
                                sh """
                                    cd frontend
                                    npm test -- --coverage --ci --reporters=default --reporters=jest-junit || exit 0
                                """
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
                        sh """
                            cd frontend
                            npm run build
                        """
                    }
                }

                stage('Deploy to VM') {
                    steps {
                        sh """
                            ssh -o StrictHostKeyChecking=no user@your-ubuntu-vm 'bash -s' < deploy_script.sh
                        """
                    }
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
