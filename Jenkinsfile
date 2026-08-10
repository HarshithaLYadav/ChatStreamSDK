
pipeline {
    agent any

    environment {
        HOME = '/var/lib/jenkins'

        ANDROID_HOME = '/home/azureuser/Android/Sdk'
        ANDROID_SDK_ROOT = '/home/azureuser/Android/Sdk'

        PATH = "/usr/local/bin:/usr/bin:/bin:${env.PATH}"
    }

    stages {

        stage('Environment Check') {
            steps {
                sh '''
                    echo "===== Environment ====="
                    echo "User: $(whoami)"
                    echo "Workspace: $WORKSPACE"

                    echo "===== Node ====="
                    node --version

                    echo "===== Yarn ====="
                    yarn --version

                    echo "===== Java ====="
                    java -version

                    echo "===== Git ====="
                    git --version
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    yarn install --frozen-lockfile
                '''
            }
        }

        stage('Lint') {
            steps {
                sh '''
                    yarn lint
                '''
            }
        }
    }

    post {
        success {
            echo 'CI pipeline completed successfully.'
        }

        failure {
            echo 'CI pipeline failed.'
        }
    }
}

