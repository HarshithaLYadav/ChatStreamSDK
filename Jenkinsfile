pipeline {
    agent any

    environment {
        GRADLE_USER_HOME = "${WORKSPACE}/.gradle"
        YARN_CACHE_FOLDER = "${WORKSPACE}/.yarn-cache"

        ANDROID_DIR = "examples/SampleApp/android"
        APK_PATH = "examples/SampleApp/android/app/build/outputs/apk/release/app-release.apk"

        STORAGE_CONTAINER = "apk-artifacts"
    }

    options {
        timestamps()
        disableConcurrentBuilds()
        skipDefaultCheckout(false)

        buildDiscarder(
            logRotator(
                numToKeepStr: '5',
                artifactNumToKeepStr: '5'
            )
        )

        timeout(time: 60, unit: 'MINUTES')
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Environment') {
            steps {
                sh '''
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

                    echo "Cleaning/building Android application..."

                    chmod +x gradlew

                    ./gradlew assembleRelease \
                        --parallel \
                        --build-cache \
                        --daemon
                '''
            }
        }

        stage('Verify APK') {
            steps {
                sh '''
                    set -e

                    echo "Checking APK..."

                    if [ ! -f "$APK_PATH" ]; then
                        echo "ERROR: APK was not generated!"
                        exit 1
                    fi

                    echo
                    echo "APK generated successfully:"
                    ls -lh "$APK_PATH"

                    echo
                    echo "APK location:"
                    realpath "$APK_PATH"
                '''
            }
        }

        stage('Archive APK') {
            steps {

                archiveArtifacts artifacts: "${APK_PATH}",
                    fingerprint: true,
                    onlyIfSuccessful: true

                withAzureKeyvault(
                    keyVaultURLOverride: 'https://chatstream-kv.vault.azure.net/',
                    credentialIDOverride: 'chatstream-jenkins-sp',
                    azureKeyVaultSecrets: [
                        [
                            secretType: 'Secret',
                            name: 'storage-account-name',
                            envVariable: 'STORAGE_ACCOUNT'
                        ],
                        [
                            secretType: 'Secret',
                            name: 'storage-account-key',
                            envVariable: 'STORAGE_KEY'
                        ]
                    ]
                ) {
                    sh '''
                        set -e

                        echo "Uploading APK to Azure Blob Storage..."

                        az storage blob upload \
                            --account-name "$STORAGE_ACCOUNT" \
                            --account-key "$STORAGE_KEY" \
                            --container-name "$STORAGE_CONTAINER" \
                            --name "chatstream-sdk-${BUILD_NUMBER}.apk" \
                            --file "$APK_PATH" \
                            --overwrite true

                        echo "APK uploaded successfully."
                    '''
                }
            }
        }
    }

    post {

        success {
            echo '''
            ==========================================
                    BUILD SUCCESSFUL
            ==========================================
            APK has been archived by Jenkins.
            '''
        }

        failure {
            echo '''
            ==========================================
                    BUILD FAILED
            ==========================================
            Check the console log for the failed stage.
            '''
        }

        always {
            echo "Build finished: ${env.BUILD_NUMBER}"
        }
    }
}
