pipeline {
    agent any

    triggers {
        // Trigger when GitHub webhook fires
        githubPush()   // if using GitHub
    }

    tools {
        maven 'maven'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('git-checkout') {
            steps {
                script {
                    git branch: 'develop', url: 'https://github.com/Elvis-Ikay/Number-Guess-Game-Group2'
                }
            }
        }
        stage('build') {
            steps {
                script {
                    sh 'mvn clean install'
                    sh "cp target/NumberGuessGame-1.0-SNAPSHOT.war target/NumberGuessGame-${BUILD_NUMBER}-SNAPSHOT.war"
                }
            }
        }
        stage('Info') {
            steps {
                echo ">>> Building and Deploying Branch: ${env.GIT_BRANCH ?: env.BRANCH_NAME}"
                echo ">>> Commit: ${env.GIT_COMMIT}"
                currentBuild.displayName = "#${BUILD_NUMBER} - ${env.GIT_BRANCH ?: env.BRANCH_NAME}"
            }
        }

        stage('code-analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    script {
                        sh """
                            ${SCANNER_HOME}/bin/sonar-scanner \
                                -Dsonar.projectKey=webapp \
                                -Dsonar.projectName=webapp\
                                -Dsonar.host.url=http://54.82.13.156:9000\
                                -Dsonar.java.binaries=target/classes
                        """
                    }
                }
            }
        }
        stage('nexus-uploader') {
            steps {
                script {

                    def warFile = findFiles(glob: 'target/*.war')[0].path

                    nexusArtifactUploader(
                        artifacts: [[
                            artifactId: 'web',
                            classifier: '',
                            file: warFile,
                            type: 'war'
                        ]],
                        credentialsId: 'nexus-creds',
                        groupId: 'webapp',
                        nexusUrl: '52.91.130.98:8081/',
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        repository: 'number-guessing-game-artifacts',
                        version: "${BUILD_NUMBER}"
                    )
                }
            }
        }
        stage('deploy-to-tomcat') {
            steps {
                script {
                    deploy adapters: [tomcat9(
                        alternativeDeploymentContext: '',
                        credentialsId: 'nexusandtomcat',
                        path: '',
                        url: 'http://98.86.241.223:8080/manager/text'
                    )], war: '**/*.war'
                }
            }
        }
    }
}
