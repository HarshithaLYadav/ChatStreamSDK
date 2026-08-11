pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Environment Check') {
            steps {
                sh '''
                    echo "===== Environment ====="
                    whoami
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
                    cd package
                    yarn install --frozen-lockfile
                '''
            }
        }

        stage('Lint') {
            steps {
                sh '''
                    cd package
                    yarn lint
                '''
            }
        }
    }

    post {
        success {
            echo 'CI validation completed successfully.'
        }

        failure {
            echo 'CI pipeline failed.'
        }
    }
}
