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
                    git branch: 'main', url: 'https://github.com/Elvis-Ikay/Number-Guess-Game-Group2'
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
        stage('code-analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    script {
                        sh """
                            ${SCANNER_HOME}/bin/sonar-scanner \
                                -Dsonar.projectKey=webapp \
                                -Dsonar.projectName=webapp\
                                -Dsonar.host.url=http://54.160.188.226:9000\
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
                        nexusUrl: '44.220.131.181:8081/',
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
                        url: 'http://18.208.172.154:8080/manager/text'
                    )], war: '**/*.war'
                }
            }
        }
    }
}
