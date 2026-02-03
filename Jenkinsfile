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
                sh '''
                    cd app
                    gradle clean build
                    echo "=== Listing JARs produced ==="
                    ls -lh build/libs/*.jar || echo "No JAR found!"
                '''
            }
        }

        stage('Clean Test') {
            steps {
                sh 'cd app && gradle clean test'
                sh 'pwd'
            }
        }

        stage('Archive Artifact') {
            steps {
                // Archive whatever JAR was logged in Clean Build
                archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true
            }
        }
    }
}
