pipeline {
    agent any

    environment {
        GRADLE_VERSION = "8.7"
        // Use Windows paths (Backslashes need escaping in Groovy)
        GRADLE_HOME = "${env.WORKSPACE}\\gradle-${GRADLE_VERSION}"
        PATH = "${GRADLE_HOME}\\bin;${env.PATH}"
        // IMPORTANT: Replace with your SonarQube Private IP from AWS Console
        SONAR_URL = "http://localhost:9000" 
    }

    stages {
        stage('Install Gradle') {
            steps {
                // Using 'bat' for Windows
                bat """
                    if not exist "%GRADLE_HOME%" (
                        powershell -Command "Invoke-WebRequest -Uri https://services.gradle.org/distributions/gradle-${GRADLE_VERSION}-bin.zip -OutFile gradle.zip"
                        powershell -Command "Expand-Archive -Path gradle.zip -DestinationPath ."
                        del gradle.zip
                    )
                """
            }
        }

        stage('Build & Test') {
            steps {
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