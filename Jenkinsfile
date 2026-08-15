pipeline {

    agent {
        label 'built-in'
    }

    environment {
        GRADLE_USER_HOME = "${WORKSPACE}/.gradle"
        YARN_CACHE_FOLDER = "${WORKSPACE}/.yarn-cache"

        ANDROID_DIR = "examples/SampleApp/android"
        APK_PATH = "examples/SampleApp/android/app/build/outputs/apk/release/app-release.apk"
    }

    options {
        timestamps()
        disableConcurrentBuilds()

        buildDiscarder(
            logRotator(
                numToKeepStr: '5',
                artifactNumToKeepStr: '5'
            )
        )

        timeout(time: 60, unit: 'MINUTES')
    }

    stages {

        /*
         * Jenkins automatically performs:
         * Declarative: Checkout SCM
         */

        stage('Environment') {
            steps {
                sh '''
                    set -e

                    echo "=========================================="
                    echo "          ENVIRONMENT INFORMATION"
                    echo "=========================================="

                    echo
                    echo "===== SYSTEM ====="
                    uname -a

                    echo
                    echo "===== CPU ====="
                    nproc

                    echo
                    echo "===== MEMORY ====="
                    free -h

                    echo
                    echo "===== DISK ====="
                    df -h

                    echo
                    echo "===== NODE ====="
                    node --version
                    npm --version

                    echo
                    echo "===== YARN ====="
                    yarn --version

                    echo
                    echo "===== JAVA ====="
                    java -version

                    echo
                    echo "===== GRADLE ====="
                    cd "$ANDROID_DIR"
                    chmod +x gradlew
                    ./gradlew --version

                    echo
                    echo "=========================================="
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    set -e

                    echo "Installing dependencies..."

                    yarn install \
                        --frozen-lockfile \
                        --prefer-offline \
                        --non-interactive
                '''
            }
        }

        stage('Quality Checks') {
            parallel {

                stage('Lint') {
                    steps {
                        sh '''
                            echo "Running lint..."

                            set +e

                            yarn lint

                            LINT_EXIT=$?

                            if [ $LINT_EXIT -ne 0 ]; then
                                echo ""
                                echo "WARNING: Lint errors detected."
                                echo "Lint is temporarily non-blocking."
                                echo "Pipeline will continue."
                                echo ""
                            else
                                echo "Lint passed successfully."
                            fi

                            exit 0
                        '''
                    }
                }

                stage('Tests') {
                    steps {
                        sh '''
                            set +e

                            echo "Running tests..."

                            yarn test \
                                --runInBand \
                                --watchAll=false \
                                --passWithNoTests

                            TEST_EXIT=$?

                            echo "Test exit code: $TEST_EXIT"

                            exit $TEST_EXIT
                        '''
                    }
                }

                stage('SonarQube Analysis') {
                    steps {
                        withSonarQubeEnv('SonarQube') {
                            sh '''
                                set -e

                                echo "Running SonarQube analysis..."

                                sonar-scanner
                            '''
                        }
                    }
                }
            }
        }

        stage('Android Release Build') {
            steps {
                sh '''
                    set -e

                    cd "$ANDROID_DIR"

                    echo "=========================================="
                    echo "       ANDROID RELEASE BUILD"
                    echo "=========================================="

                    chmod +x gradlew

                    ./gradlew assembleRelease \
                        --parallel \
                        --build-cache \
                        --daemon

                    echo
                    echo "Android release build completed."
                '''
            }
        }

        stage('Verify APK') {
            steps {
                sh '''
                    set -e

                    echo "=========================================="
                    echo "             VERIFY APK"
                    echo "=========================================="

                    echo "Checking APK..."

                    if [ ! -f "$APK_PATH" ]; then
                        echo ""
                        echo "ERROR: APK was not generated!"
                        echo "Expected location:"
                        echo "$APK_PATH"
                        exit 1
                    fi

                    echo ""
                    echo "APK generated successfully:"
                    ls -lh "$APK_PATH"

                    echo ""
                    echo "APK location:"
                    realpath "$APK_PATH"

                    echo ""
                    echo "APK verification completed successfully."
                '''
            }
        }

        stage('Archive APK') {
            steps {
                echo "Archiving APK..."

                archiveArtifacts(
                    artifacts: "${APK_PATH}",
                    fingerprint: true,
                    onlyIfSuccessful: true
                )

                echo "APK archived successfully."
            }
        }
    }

    post {

        success {
            echo '''
==========================================
        BUILD SUCCESSFUL
==========================================
APK has been built, verified and archived.
==========================================
'''
        }

        failure {
            echo '''
==========================================
          BUILD FAILED
==========================================
Check the console log for the failed stage.
==========================================
'''
        }

        always {
            echo "Build finished: ${env.BUILD_NUMBER}"
        }
    }
}
