pipeline {
  agent any
  // Use Jenkins tool installations (Manage Jenkins → Tools). Make sure these names exist.
  tools { jdk 'jdk17'; maven 'maven3' }
  options { skipDefaultCheckout(); timestamps() }

  stages {
    stage('Checkout') {
      steps {
        echo 'Checking out code from GitHub...'
        // Multibranch sets BRANCH_NAME; fallback to your feature branch if run standalone
        git url: 'https://github.com/Elvis-Ikay/Number-Guess-Game-Group2.git',
            branch: env.BRANCH_NAME ?: 'group2-subomi'
      }
    }

    // Do build + tests in one Maven invocation (verify = compile + unit tests + package)
    stage('Build & Test') {
      steps {
        sh 'mvn -B -q -U clean verify'
      }
      post {
        always {
          // Publish JUnit results so Jenkins shows the Tests tab
          junit '**/target/surefire-reports/*.xml'
        }
      }
    }

    // Only archive artifacts on develop to keep feature branches test-focused
    stage('Archive Artifact (develop only)') {
      when { branch 'develop' }
      steps {
        echo 'Archiving build artifacts...'
        archiveArtifacts artifacts: 'target/*.jar, target/*.war',
                          fingerprint: true, onlyIfSuccessful: true
      }
    }
  }

  post {
    success { echo 'Build and tests completed successfully!' }
    failure { echo 'Build failed. Check the console output.' }
  }
}
