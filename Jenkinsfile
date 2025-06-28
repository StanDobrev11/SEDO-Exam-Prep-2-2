pipeline {
    agent none

    options {
        skipStagesAfterUnstable()
    }

    environment {
        DOTNET_VERSION = "6.0.417"
        DOTNET_INSTALL_DIR = "${HOME}/dotnet"
        DOTNET_ROOT = "${HOME}/dotnet"
        PATH = "${HOME}/dotnet:${PATH}"
        BUILD_NODE = ''
    }

    triggers {
        pollSCM('H/5 * * * *')  // Optional: remove if using webhooks
    }

    stages {
        stage('Detect Branch & Agent') {
            agent { label 'master' } // Run detection stage anywhere (safely)
            steps {
                script {
                    // Detect current branch
                    def branch = env.BRANCH_NAME ?: sh(
                        returnStdout: true,
                        script: 'git rev-parse --abbrev-ref HEAD'
                    ).trim()

                    echo "Detected branch: ${branch}"

                    if (branch != 'main') {
                        echo "Skipping pipeline: not on 'main' branch."
                        currentBuild.result = 'SUCCESS'
                        error("Exiting early.")
                    }

                    // Check if 'TestAgent' node is online
                    def available = Jenkins.instance.slaves.find {
                        it.name == 'TestAgent' && it.computer.online
                    }

                    env.BUILD_NODE = available ? 'TestAgent' : 'master'
                    echo "Selected node: ${env.BUILD_NODE}"
                }
            }
        }

        stage('Checkout') {
            agent { label "${env.BUILD_NODE}" }
            steps {
                checkout scm
            }
        }

        stage('Setup .NET SDK') {
            agent { label "${env.BUILD_NODE}" }
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
            agent { label "${env.BUILD_NODE}" }
            steps {
                sh 'dotnet restore'
            }
        }

        stage('Build') {
            agent { label "${env.BUILD_NODE}" }
            steps {
                sh 'dotnet build --no-restore'
            }
        }

        stage('Test') {
            agent { label "${env.BUILD_NODE}" }
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
