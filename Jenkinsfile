pipeline {
    agent any

    environment {
        SOURCE_DIR = 'C:\\Users\\kshitij.waikar\\ITHCSoftwareApp'
        ZIP_PATH = 'C:\\ProgramData\\Jenkins\\.jenkins\\workspace\\ITHCapp\\ITHCSoftwareApp.zip'
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

                REM Set the paths
                set "SOURCE_DIR=%SOURCE_DIR%"
                set "ZIP_PATH=%ZIP_PATH%"

                REM Use PowerShell to zip the folder with retries
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
    }
}
