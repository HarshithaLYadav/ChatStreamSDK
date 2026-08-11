
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

        stage('Install Package Dependencies') {
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

        stage('Install SampleApp Dependencies') {
            steps {
                dir('examples/SampleApp') {
                    sh '''
                        echo "===== Installing SampleApp dependencies ====="
                        yarn install --frozen-lockfile
                    '''
                }
            }
        }

        stage('Android Build') {
            steps {
                dir('examples/SampleApp/android') {
                    sh '''
                        echo "===== Android Build ====="

                        chmod +x gradlew

                        echo "===== React Native Gradle Plugin ====="
                        ls -la ../node_modules/@react-native/gradle-plugin

                        echo "===== Gradle Version ====="
                        ./gradlew --version

                        echo "===== Building Release APK ====="
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
            echo '=========================================='
            echo 'CI/CD PIPELINE COMPLETED SUCCESSFULLY'
            echo '=========================================='
            echo 'Android APK has been built and archived.'
        }

        failure {
            echo '=========================================='
            echo 'CI/CD PIPELINE FAILED'
            echo '=========================================='
            echo 'Check the failed stage above.'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}
