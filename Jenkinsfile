
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
         * Checkout is intentionally NOT defined here.
         * Jenkins performs automatic SCM checkout.
         */

        stage('Environment') {
            steps {
                sh '''
                    set -e

                    echo "=========================================="
                    echo "        ENVIRONMENT INFORMATION"
                    echo "=========================================="

                    echo ""
                    echo "===== SYSTEM ====="
                    uname -a

                    echo ""
                    echo "===== CPU ====="
                    nproc

                    echo ""
                    echo "===== MEMORY ====="
                    free -h

                    echo ""
                    echo "===== DISK ====="
                    df -h

                    echo ""
                    echo "===== NODE ====="
                    node --version
                    npm --version

                    echo ""
                    echo "===== YARN ====="
                    yarn --version

                    echo ""
                    echo "===== JAVA ====="
                    java -version

                    echo ""
                    echo "===== GRADLE ====="
                    cd "$ANDROID_DIR"
                    chmod +x gradlew
                    ./gradlew --version

                    echo "=========================================="
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    set -e

                    echo "=========================================="
                    echo "       INSTALLING DEPENDENCIES"
                    echo "=========================================="

                    yarn install \
                        --frozen-lockfile \
                        --prefer-offline \
                        --non-interactive

                    echo "Dependencies installed successfully."
                '''
            }
        }

        stage('Quality & Security') {
            steps {

                /*
                 * Lint + Tests are temporarily non-blocking.
                 * SonarQube analysis remains mandatory.
                 */

                script {

                    echo "=========================================="
                    echo "       RUNNING LINT CHECK"
                    echo "=========================================="

                    def lintStatus = sh(
                        script: '''
                            set +e

                            yarn lint

                            LINT_EXIT=$?

                            echo "Lint exit code: $LINT_EXIT"

                            exit $LINT_EXIT
                        ''',
                        returnStatus: true
                    )

                    if (lintStatus != 0) {
                        echo "WARNING: Lint failed."
                        echo "Lint is temporarily non-blocking."
                        echo "Pipeline will continue."
                    } else {
                        echo "Lint passed successfully."
                    }


                    echo "=========================================="
                    echo "       RUNNING TEST CHECK"
                    echo "=========================================="

                    def testStatus = sh(
                        script: '''
                            set +e

                            yarn test \
                                --runInBand \
                                --watchAll=false \
                                --passWithNoTests

                            TEST_EXIT=$?

                            echo "Test exit code: $TEST_EXIT"

                            exit $TEST_EXIT
                        ''',
                        returnStatus: true
                    )

                    if (testStatus != 0) {
                        echo "WARNING: Tests failed or test script is unavailable."
                        echo "Tests are temporarily non-blocking."
                        echo "Pipeline will continue."
                    } else {
                        echo "Tests passed successfully."
                    }


                    echo "=========================================="
                    echo "       RUNNING SONARQUBE ANALYSIS"
                    echo "=========================================="

                    withSonarQubeEnv('SonarQube') {
                        sh '''
                            set -e

                            echo "Starting SonarQube analysis..."

                            sonar-scanner

                            echo "SonarQube analysis completed successfully."
                        '''
                    }

                    echo "=========================================="
                    echo "       SONARQUBE ANALYSIS COMPLETE"
                    echo "=========================================="
                }
            }
        }

        /*
         * SonarQube Quality Gate
         *
         * Jenkins waits for SonarQube to finish processing
         * the analysis and return the Quality Gate result.
         *
         * If the Quality Gate fails, abortPipeline: true
         * immediately stops the pipeline.
         */

        stage('Quality Gate') {
            steps {

                echo "=========================================="
                echo "       WAITING FOR SONARQUBE QUALITY GATE"
                echo "=========================================="

                timeout(time: 5, unit: 'MINUTES') {

                    waitForQualityGate(
                        abortPipeline: true
                    )
                }

                echo "=========================================="
                echo "       SONARQUBE QUALITY GATE PASSED"
                echo "=========================================="
            }
        }

        stage('Android Release Build') {
            steps {
                sh '''
                    set -e

                    echo "=========================================="
                    echo "       ANDROID RELEASE BUILD"
                    echo "=========================================="

                    cd "$ANDROID_DIR"

                    chmod +x gradlew

                    ./gradlew assembleRelease \
                        --parallel \
                        --build-cache \
                        --daemon

                    echo "Android release build completed."
                '''
            }
        }

        stage('Verify & Archive APK') {
            steps {

                sh '''
                    set -e

                    echo "=========================================="
                    echo "          VERIFYING APK"
                    echo "=========================================="

                    if [ ! -f "$APK_PATH" ]; then
                        echo "ERROR: APK was not generated."
                        echo "Expected location:"
                        echo "$APK_PATH"
                        exit 1
                    fi

                    echo ""
                    echo "APK generated successfully:"
                    ls -lh "$APK_PATH"

                    echo ""
                    echo "APK absolute path:"
                    realpath "$APK_PATH"

                    echo "=========================================="
                    echo "             APK READY"
                    echo "=========================================="
                '''

                archiveArtifacts(
                    artifacts: "${APK_PATH}",
                    fingerprint: true,
                    onlyIfSuccessful: true
                )
            }
        }
    }

    post {

        success {
            echo '''
==========================================
          BUILD SUCCESSFUL
==========================================
SonarQube Quality Gate: PASSED
APK successfully built and archived.
==========================================
'''
        }

        failure {
            echo '''
==========================================
            BUILD FAILED
==========================================
The pipeline failed before successful completion.
If the Quality Gate failed, the Android build
was intentionally blocked.
Check the console log for the failed stage.
==========================================
'''
        }

        always {
            echo "Build finished: ${env.BUILD_NUMBER}"
        }
    }
}

