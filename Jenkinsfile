pipeline {
    agent any

    environment {
        GRADLE_VERSION = "8.7"
        GRADLE_HOME = "${env.WORKSPACE}/gradle-${GRADLE_VERSION}"
        PATH = "${GRADLE_HOME}/bin:${env.PATH}"
        // Replace with your actual Sonar URL
        SONAR_URL = "http://your-sonarqube-server:9000" 
    }

    stages {
        stage('Install Gradle') {
            steps {
                sh '''
                    if [ ! -d "$GRADLE_HOME" ]; then
                        echo "Downloading Gradle $GRADLE_VERSION..."
                        curl -s -L https://services.gradle.org/distributions/gradle-${GRADLE_VERSION}-bin.zip -o gradle.zip
                        unzip -q gradle.zip
                        rm gradle.zip
                    fi
                '''
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh '''
                    cd app
                    gradle clean build
                    echo "=== Verifying JAR Production ==="
                    ls -lh build/libs/
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                /* We use withCredentials instead of withSonarQubeEnv 
                   to avoid the 'No such DSL' error. 
                   Create a 'Secret Text' credential in Jenkins with ID 'sonar-token'
                */
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh '''
                        cd app
                        gradle sonar \
                          -Dsonar.host.url=${SONAR_URL} \
                          -Dsonar.login=${SONAR_AUTH_TOKEN}
                    '''
                }
            }
        }

        stage('Archive Artifact') {
            steps {
                // This archives the JAR so it appears in the Jenkins UI
                archiveArtifacts artifacts: 'app/build/libs/*.jar', fingerprint: true
            }
        }
    }
}