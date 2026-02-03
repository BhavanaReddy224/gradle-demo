pipeline {
    agent any

    environment {
        GRADLE_VERSION = "8.7"
        GRADLE_HOME = "${env.WORKSPACE}/gradle-${GRADLE_VERSION}"
        PATH = "${GRADLE_HOME}/bin:${env.PATH}"
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
                    gradle -v
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
                    # Running 'build' automatically runs 'test' and 'jar'
                    gradle clean build
                    
                    echo "=== Verifying Build Output ==="
                    ls -lh build/libs/
                '''
            }
        }

        stage('Archive Artifact') {
            steps {
                // Ensure this path matches your repository structure.
                // If the 'app' folder is at the root, use 'app/build/libs/*.jar'
                archiveArtifacts artifacts: 'app/build/libs/*.jar', fingerprint: true
            }
        }
       
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonarQubeServer') {
                    sh 'cd app && gradle sonarqube'
                }
            }
        }

    }
}