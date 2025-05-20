pipeline {
    agent any

    environment {
        APP_NAME = 'ithcapp'
        DEPLOY_DIR = '/home/kshitij-necsws/deploy_folder'
        VENV_PATH = "${DEPLOY_DIR}/venv"
        VM_USER = 'kshitij-necsws'
        VM_HOST = '10.102.192.172'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/KshitijNEC/ITHCSoftwareApp.git'
            }
        }

        stage('Build Frontend') {
            steps {
                sh '''
                    cd frontend
                    npm install
                    npm run build
                '''
            }
        }

        stage('Prepare Archive for Deployment') {
            steps {
                sh '''
                    mkdir -p build_artifacts
                    cp -r backend frontend build_artifacts/
                '''
            }
        }

        stage('Deploy to VM') {
            steps {
                sh '''
                    # Copy files to remote VM
                    scp -r build_artifacts/* $VM_USER@$VM_HOST:/tmp/$APP_NAME
                '''
            }
        }

        stage('Setup on VM') {
            steps {
                sh '''
                    ssh $VM_USER@$VM_HOST << 'EOF'
                        # Move deployment to proper directory
                        rm -rf $DEPLOY_DIR
                        mkdir -p $DEPLOY_DIR
                        cp -r /tmp/$APP_NAME/* $DEPLOY_DIR
                        rm -rf /tmp/$APP_NAME

                        # Setup Python environment
                        cd $DEPLOY_DIR
                        python3 -m venv venv
                        source venv/bin/activate

                        cd backend
                        pip install --upgrade pip
                        pip install -r requirements.txt

                        # Run DB migrations
                        export FLASK_APP=app.py
                        flask db upgrade

                        # Start app (development server, for now)
                        nohup flask run --host=0.0.0.0 --port=8000 &
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
            echo 'Deployment succeeded!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
