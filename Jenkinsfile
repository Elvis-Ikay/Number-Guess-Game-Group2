pipeline {
    agent any

    // Use Jenkins-installed tools matching your configuration
    tools {
        jdk   'JDK11'       // Matches your configured name
        maven 'Maven'       // Matches your configured name
    }

    options {
        timestamps()
        ansiColor('xterm')
    }

    stages {
        stage('Verify Tools') {
            steps {
                sh 'java -version'
                sh 'mvn --version'
                sh 'ls -la'
            }
        }

        stage('Checkout') {
            steps {
                echo 'Checking out code...'
                checkout scm
            }
        }

        stage('Clean') {
            steps {
                echo 'Cleaning previous builds...'
                sh 'mvn -B clean'
            }
        }

        stage('Build & Test') {
            steps {
                echo 'Building and running tests...'
                sh 'mvn -B -U verify'
            }
        }

        stage('Archive Artifact') {
            steps {
                echo 'Archiving build artifacts & test reports...'
                junit 'target/surefire-reports/*.xml'
                archiveArtifacts artifacts: 'target/*.war', fingerprint: true
            }
        }
    }

    post {
        success { echo 'Build and tests completed successfully!' }
        failure { echo 'Build failed. Check Console Output and Test Result.' }
    }
}

