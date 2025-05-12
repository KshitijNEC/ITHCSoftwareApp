pipeline {
    agent any

    environment {
        APP_NAME = 'ithcapp'
        DEPLOY_DIR = '/application_deploy/deploy_folder'
        VENV_PATH = "${DEPLOY_DIR}/venv"
        VM_USER = 'kshitij-necsws'
        VM_HOST = '10.102.192.172'
        APP_PATH = '/home/kshitij-necsws/Desktop/deploy_folder'
        TEST_REPORT_EMAIL = 'kshitij.waikar@necsws.com' // Update with the actual recipient's email
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Setup Virtual Environment & Application') {
            steps {
                sh '''
                    # Set up the virtual environment and install dependencies
                    python3 -m venv ${VENV_PATH}
                    . ${VENV_PATH}/bin/activate

                    # Install backend requirements
                    cd backend
                    pip install -r requirements.txt
                    pip install pytest-cov pytest-html
                    
                    # Install frontend dependencies
                    cd ../frontend
                    npm install
                '''
            }
        }

        stage('Run Unit Tests') {
            parallel {
                stage('Backend Tests') {
                    steps {
                        sh '''
                            # Activate the virtual environment
                            . ${VENV_PATH}/bin/activate
                            
                            # Run backend tests
                            cd backend
                            pytest --maxfail=1 --disable-warnings -q
                        '''
                    }
                    post {
                        always {
                            junit 'backend/test-report.xml'
                            publishHTML([ 
                                allowMissing: true,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: 'backend',
                                reportFiles: 'test-report.html',
                                reportName: 'Backend Test Report'
                            ])
                        }
                    }
                }

                stage('Frontend Tests') {
                    steps {
                        sh '''
                            cd frontend
                            npm test --max-warnings=0
                        '''
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
                                reportName: 'Frontend Test Report'
                            ])
                        }
                    }
                }
            }
        }

        stage('Send Test Report Email') {
            when {
                expression {
                    return currentBuild.result == 'SUCCESS'  // Only send if tests pass
                }
            }
            steps {
                mail to: TEST_REPORT_EMAIL,
                     subject: "Test Results for ${APP_NAME}",
                     body: "The unit tests for ${APP_NAME} have completed. Please check the test reports attached."
            }
        }

        stage('Deploy to Ubuntu Machine') {
            when {
                expression {
                    return currentBuild.result == 'SUCCESS'  // Only deploy if tests pass
                }
            }
            steps {
                sh '''
                    ssh $VM_USER@$VM_HOST << 'EOF'
                        # Deploy the application to the Ubuntu machine
                        sudo mkdir -p $DEPLOY_DIR
                        sudo rm -rf $DEPLOY_DIR/*
                        sudo cp -r $APP_PATH/* $DEPLOY_DIR/
                        sudo chown -R $USER:$USER $DEPLOY_DIR

                        cd $DEPLOY_DIR
                        python3 -m venv venv
                        source venv/bin/activate

                        cd backend
                        pip install -r requirements.txt
                        pip install gunicorn

                        export FLASK_APP=app.py
                        flask db upgrade

                        # Set up Gunicorn to run the Flask app
                        sudo tee /etc/systemd/system/$APP_NAME.service > /dev/null << SERVICE
[Unit]
Description=ITHC Software App
After=network.target

[Service]
User=$USER
WorkingDirectory=$DEPLOY_DIR/backend
Environment="PATH=$DEPLOY_DIR/venv/bin"
Environment="FLASK_ENV=production"
ExecStart=$DEPLOY_DIR/venv/bin/gunicorn -w 4 -b 127.0.0.1:8000 app:app

[Install]
WantedBy=multi-user.target
SERVICE

                        # Set up Nginx to serve the Flask app
                        sudo tee /etc/nginx/sites-available/$APP_NAME > /dev/null << NGINX
server {
    listen 80;
    server_name $VM_HOST;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
    }

    location /static/ {
        alias $DEPLOY_DIR/frontend/static/;
    }
}
NGINX

                        sudo ln -sf /etc/nginx/sites-available/$APP_NAME /etc/nginx/sites-enabled/
                        sudo nginx -t
                        sudo systemctl daemon-reload
                        sudo systemctl restart nginx
                        sudo systemctl restart $APP_NAME
                        sudo systemctl enable $APP_NAME
                    EOF
                '''
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
