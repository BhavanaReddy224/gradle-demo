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
                        unzip gradle.zip
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

        stage('Clean Build') {
            steps {
                // Build inside the app module
                sh 'cd app && gradle clean build'
            }
        }

        stage('Clean Test') {
            steps {
                // Run tests after build
                sh 'cd app && gradle clean test'
            }
        }

        stage('Archive Artifact') {
            steps {
                // Explicit path to JAR
                archiveArtifacts artifacts: 'app/build/libs/*.jar', fingerprint: true
            }
        }
    }
}
