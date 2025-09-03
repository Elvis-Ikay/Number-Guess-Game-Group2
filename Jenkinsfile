pipeline {
    agent any

    // Use Jenkins-installed tools so mvn/java are available
    tools {
        jdk   'jdk11'       // Configure in Manage Jenkins → Tools
        maven 'maven39'     // Configure in Manage Jenkins → Tools
    }

    options {
        timestamps()
        ansiColor('xterm')
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code...'
                // In Multibranch Pipeline this checks out the correct branch (not hard-coded main)
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
                // verify = compile + test + package (fails build if tests fail)
                sh 'mvn -B -U verify'
            }
        }

        stage('Archive Artifact') {
            steps {
                echo 'Archiving build artifacts & test reports...'
                // publish JUnit results so Jenkins shows a Test report
                junit 'target/surefire-reports/*.xml'
                // archive either WAR or JAR (whichever your pom builds)
                archiveArtifacts artifacts: 'target/*.war, target/*.jar', fingerprint: true
            }
        }
    }

    post {
        success { echo 'Build and tests completed successfully!' }
        failure { echo 'Build failed. Check Console Output and Test Result.' }
    }
}

