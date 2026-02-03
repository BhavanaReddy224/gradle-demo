pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Build & Test') {
            steps { sh 'gradle clean test' }
        }
        stage('Archive Artifact') {
            steps { archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true }
        }
    }
}
