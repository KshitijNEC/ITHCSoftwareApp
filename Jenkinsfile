pipeline {
    agent any

    environment {
        SOURCE_DIR = 'C:\\Users\\kshitij.waikar\\ITHCSoftwareApp'
        ZIP_PATH = 'C:\\ProgramData\\Jenkins\\.jenkins\\workspace\\ITHCapp\\ITHCSoftwareApp.zip'
        VM_USER = 'kshitij-necsws'
        VM_HOST = '10.102.192.172'
        REMOTE_DIR = '/application_deploy/zips'
        VM_PASSWORD = 'YOUR_PASSWORD' // replace this securely
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Zip Source Code') {
            steps {
                bat '''
                @echo off
                setlocal

                set "SOURCE_DIR=%SOURCE_DIR%"
                set "ZIP_PATH=%ZIP_PATH%"

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

        stage('Copy ZIP to Linux VM') {
            steps {
                bat '''
                @echo off
                setlocal

                set "ZIP_PATH=%ZIP_PATH%"
                set "VM_USER=%VM_USER%"
                set "VM_HOST=%VM_HOST%"
                set "REMOTE_DIR=%REMOTE_DIR%"
                set "VM_PASSWORD=%VM_PASSWORD%"

                REM Use pscp to copy ZIP to the Linux VM (you need pscp from PuTTY installed and in PATH)
                echo y | pscp -pw %VM_PASSWORD% "%ZIP_PATH%" %VM_USER%@%VM_HOST%:%REMOTE_DIR%/

                endlocal
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
