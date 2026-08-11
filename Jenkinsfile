
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
                    echo "User:"
                    whoami

                    echo "Workspace:"
                    echo "$WORKSPACE"

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
                dir('package') {
                    sh '''
                        yarn install --frozen-lockfile
                    '''
                }
            }
        }

        stage('Lint') {
            steps {
                dir('package') {
                    sh '''
                        yarn lint
                    '''
                }
            }
        }

        stage('Unit Tests') {
            steps {
                dir('package') {
                    sh '''
                        yarn test:unit --passWithNoTests
                    '''
                }
            }
        }

        stage('Android Build') {
            steps {
                dir('examples/SampleApp/android') {
                    sh '''
                        chmod +x gradlew
                        ./gradlew assembleRelease
                    '''
                }
            }
        }

        stage('Archive APK') {
            steps {
                archiveArtifacts artifacts: 'examples/SampleApp/android/app/build/outputs/apk/release/*.apk',
                                 fingerprint: true
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully.'
            echo 'Android APK has been built and archived.'
        }

        failure {
            echo 'CI/CD pipeline failed.'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}
