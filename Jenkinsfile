pipeline {
    agent any

    options {
        skipStagesAfterUnstable()
    }

    environment {
        DOTNET_VERSION = "6.0.417"
        DOTNET_INSTALL_DIR = "${HOME}/dotnet"
        DOTNET_ROOT = "${HOME}/dotnet"
        PATH = "${HOME}/dotnet:${PATH}"
    }

    triggers {
        pollSCM('H/5 * * * *')  // Optional: remove if using webhooks
    }

    stages {
        stage('Branch Filter') {
            steps {
                script {
                    def branch = env.BRANCH_NAME ?: sh(
                        returnStdout: true,
                        script: 'git rev-parse --abbrev-ref HEAD'
                    ).trim()

                    echo "Detected branch: ${branch}"

                    if (branch != 'main') {
                        echo "Skipping pipeline: not on 'main' branch."
                        currentBuild.result = 'SUCCESS'
                        error("Exiting early because branch is not 'main'.")
                    }
                }
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup .NET SDK') {
            steps {
                sh '''
                    echo "Installing .NET SDK $DOTNET_VERSION..."
                    wget https://dotnet.microsoft.com/download/dotnet/scripts/v1/dotnet-install.sh -O dotnet-install.sh
                    chmod +x dotnet-install.sh
                    ./dotnet-install.sh --version $DOTNET_VERSION --install-dir $DOTNET_INSTALL_DIR
                    dotnet --info
                '''
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
                sh 'dotnet test --no-build --framework net6.0'
            }
        }
    }

    post {
        always {
            echo 'Cleaning up...'
        }
    }
}
