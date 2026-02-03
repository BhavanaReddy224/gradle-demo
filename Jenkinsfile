pipeline {
    agent any

    environment {
        GRADLE_VERSION = "8.7"
        // Windows uses backslashes; Groovy needs them escaped as \\
        GRADLE_HOME = "${env.WORKSPACE}\\gradle-${GRADLE_VERSION}"
        PATH = "${GRADLE_HOME}\\bin;${env.PATH}"
        // UPDATE THIS with your SonarQube Private IP
        SONAR_URL = "http://localhost:9000" 
    }

    stages {
        stage('Install Gradle') {
            steps {
                // Use 'bat' for Windows. Using PowerShell to download.
                bat """
                    if not exist "%GRADLE_HOME%" (
                        echo Downloading Gradle...
                        powershell -Command "[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; Invoke-WebRequest -Uri https://services.gradle.org/distributions/gradle-${GRADLE_VERSION}-bin.zip -OutFile gradle.zip"
                        powershell -Command "Expand-Archive -Path gradle.zip -DestinationPath ."
                        del gradle.zip
                    )
                """
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                // Windows uses 'bat'
                bat "cd app && gradle clean build"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_AUTH_TOKEN')]) {
                    bat """
                        cd app
                        gradle sonar ^
                          -Dsonar.host.url=${SONAR_URL} ^
                          -Dsonar.login=%SONAR_AUTH_TOKEN% ^
                          -Dsonar.gradle.skipCompile=true ^
                          --no-configuration-cache
                    """
                }
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'app/build/libs/*.jar', fingerprint: true
            }
        }
    }
}