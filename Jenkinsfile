pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Branch Filter') {
            steps {
                script {
                    def branch = env.BRANCH_NAME ?: sh(
                        returnStdout: true,
                        script: 'git name-rev --name-only HEAD'
                    ).trim()

                    // Normalize branch name
                    branch = branch.replaceAll(/^remotes\/origin\//, '')
                                   .replaceAll(/^origin\//, '')
                                   .replaceAll(/~.*/, '')

                    echo "Detected branch: ${branch}"

                    if (branch != 'main') {
                        echo "Skipping pipeline: not on 'main' branch."
                        currentBuild.result = 'SUCCESS'
                        error("Exiting early because branch is not 'main'.")
                    }
                }
            }
        }

        stage('Restore Dependencies') {
            steps {
                sh 'dotnet restore'
            }
        }

        stage('Build') {
            steps {
                sh 'dotnet build --no-restore'
            }
        }

        stage('Test') {
            steps {
                sh 'dotnet test --no-build'
            }
        }
    }

    post {
        always {
            echo "Completing..."
        }
    }
}
