pipeline {
    agent any

    triggers {
        githubPush()
    }

    tools {
        maven 'maven'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Branch Filter') {
            steps {
                script {
                    // Get the branch name from environment variables
                    def branchName = env.GIT_BRANCH ?: env.BRANCH_NAME ?: 'unknown'

                    // Remove 'origin/' prefix if present
                    if (branchName.startsWith('origin/')) {
                        branchName = branchName.replaceFirst('origin/', '')
                    }

                    echo "Triggered by: ${currentBuild.getBuildCauses()}"
                    echo "Branch detected: ${branchName}"

                    // Only proceed if this is master or main branch
                    if (branchName != 'master' && branchName != 'main') {
                        echo "⏭️ Skipping pipeline - not master/main branch (detected: ${branchName})"
                        currentBuild.result = 'ABORTED'
                        currentBuild.displayName = "#${BUILD_NUMBER} - SKIPPED (${branchName})"
                        error("Pipeline execution stopped - not targeting master/main branch")
                    }

                    echo "✅ Branch filter passed - proceeding with pipeline for ${branchName}"
                }
            }
        }

        stage('git-checkout') {
            steps {
                script {
                    git branch: 'main', url: 'https://github.com/Elvis-Ikay/Number-Guess-Game-Group2'
                    echo "✅ Checked out main branch"
                }
            }
        }

        stage('build') {
            steps {
                script {
                    sh 'mvn clean install'
                    sh "cp target/NumberGuessGame-1.0-SNAPSHOT.war target/NumberGuessGame-${BUILD_NUMBER}-SNAPSHOT.war"
                    echo "✅ Build completed successfully"
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
                                -Dsonar.projectName=guessing-game\
                                -Dsonar.host.url=http://100.24.36.13:9000\
                                -Dsonar.java.binaries=target/classes
                        """
                        echo "✅ Code analysis completed"
                    }
                }
            }
        }

        stage('nexus-uploader') {
            steps {
                script {
                    def warFile = findFiles(glob: 'target/*.war')[0].path
                    echo "📦 Uploading ${warFile} to Nexus..."

                    nexusArtifactUploader(
                        artifacts: [[
                            artifactId: 'web',
                            classifier: '',
                            file: warFile,
                            type: 'war'
                        ]],
                        credentialsId: 'nexus-creds',
                        groupId: 'webapp',
                        nexusUrl: '174.129.125.250:8081/',
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        repository: 'number-guessing-game-artifacts',
                        version: "${BUILD_NUMBER}"
                    )

                    echo "✅ Artifact uploaded to Nexus successfully"
                }
            }
        }

        stage('deploy-to-tomcat') {
            steps {
                script {
                    echo "🚀 Deploying to Tomcat server..."

                    deploy adapters: [tomcat9(
                        alternativeDeploymentContext: '',
                        credentialsId: 'nexusandtomcat',
                        path: '',
                        url: 'http://3.81.205.214:8080/manager/text'
                    )], war: '**/*.war'

                    echo "✅ Deployment to Tomcat completed"
                }
            }
        }

        stage('Deployment Verification') {
            steps {
                script {
                    echo "🔍 Verifying deployment..."

                    try {
                        def appUrl = "http://3.81.205.214:8080/NumberGuessGame-${BUILD_NUMBER}-SNAPSHOT/"

                        // Test if application is accessible
                        def response = sh(
                            script: "curl -s -o /dev/null -w '%{http_code}' '${appUrl}'",
                            returnStdout: true
                        ).trim()

                        if (response == '200') {
                            echo "✅ Application is responding correctly (HTTP ${response})"
                            echo "🌐 Application URL: ${appUrl}"
                        } else {
                            echo "⚠️ Application responded with HTTP ${response}"
                            echo "🌐 Application URL: ${appUrl}"
                        }
                    } catch (Exception e) {
                        echo "⚠️ Could not verify deployment: ${e.message}"
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                echo "Pipeline execution completed"
            }
        }

        success {
            script {
                def commitInfo = sh(
                    script: 'git log -1 --pretty="%h - %s (%an)"',
                    returnStdout: true
                ).trim()

                echo """
                    🎉 DEPLOYMENT SUCCESSFUL!

                    Build: #${BUILD_NUMBER}
                    Branch: master
                    Commit: ${commitInfo}
                    Approved by: ${env.APPROVER ?: 'System'}

                    🌐 Application URL: http://3.81.205.214:8080/NumberGuessGame-${BUILD_NUMBER}-SNAPSHOT/
                    📦 Nexus Artifact: webapp:web:${BUILD_NUMBER}
                """
            }
        }

        failure {
            echo """
                ❌ PIPELINE FAILED!

                Build: #${BUILD_NUMBER}
                Branch: master

                Please check the console output for error details.
            """
        }

        aborted {
            script {
                def branchName = env.GIT_BRANCH ?: env.BRANCH_NAME ?: 'unknown'
                if (branchName.startsWith('origin/')) {
                    branchName = branchName.replaceFirst('origin/', '')
                }

                echo """
                    ⏹️ PIPELINE ABORTED

                    Reason: Branch filter (detected: ${branchName})
                    Note: Pipeline only runs for master/main branch pushes
                """
            }
        }
    }
}