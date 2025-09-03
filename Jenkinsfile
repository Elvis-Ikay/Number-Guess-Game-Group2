pipeline {
  agent any
  tools {
    jdk 'jdk11'
    maven 'maven39'
  }
  stages {
    stage('Checkout'){ steps { checkout scm } }
    stage('Verify tools'){ steps { sh 'java -version && mvn -version' } }
    stage('Clean'){ steps { sh 'mvn -B clean' } }
    stage('Build & Test'){ steps { sh 'mvn -B verify' } } // runs tests too
    stage('Archive'){
      steps {
        archiveArtifacts artifacts: 'target/*.war, target/*.jar', fingerprint: true
        junit 'target/surefire-reports/*.xml'
      }
    }
  }
  post {
    success { echo 'Build & tests OK' }
    failure { echo 'Build failed - check Console Output & Test Result' }
  }
}
