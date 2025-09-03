pipeline {
  agent {
    docker {
      image 'maven:3.9.6-eclipse-temurin-11'   // Maven + Java 11
      args  '-v $HOME/.m2:/root/.m2'          // cache dependencies (optional)
    }
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Verify tools') {
      steps { sh 'java -version && mvn -version' }
    }
    stage('Clean') {
      steps { sh 'mvn clean' }
    }
    stage('Build & Package') {
      steps { sh 'mvn package' }
    }
    stage('Test') {
      steps { sh 'mvn test' }
    }
    stage('Archive Artifact') {
      steps { archiveArtifacts artifacts: 'target/*.war', fingerprint: true }
    }
  }

  post {
    success { echo 'Build and tests completed successfully!' }
    failure { echo 'Build failed. Check the console output.' }
  }
}

