pipeline {
    agent any

    tools {
        nodejs 'Node20'
        jdk 'JDK17'
    }

    environment {
    GRADLE_USER_HOME = "${WORKSPACE}/.gradle"
    YARN_CACHE_FOLDER = "${WORKSPACE}/.yarn-cache"

    ANDROID_DIR = "examples/SampleApp/android"
    APK_PATH = "examples/SampleApp/android/app/build/outputs/apk/release/app-release.apk"

    KEYVAULT_NAME = "chatstream-key"
    STORAGE_CONTAINER = "chatstreamapk8055"
}

    options {
        timestamps()
        disableConcurrentBuilds()
        skipDefaultCheckout(false)

        // Keep only recent builds
        buildDiscarder(
            logRotator(
                numToKeepStr: '5',
                artifactNumToKeepStr: '5'
            )
        )

        // Stop a stuck build
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
            }
        }
        stage('Deploy APK to Azure Blob') {
    steps {

        withAzureKeyvault(
            keyVaultURLOverride: 'https://chatstream-key.vault.azure.net/',
            credentialIDOverride: 'azure-managed-identity',
            azureKeyVaultSecrets: [
                [
                    secretType: 'Secret',
                    name: 'chatstreamapk8055',
                    envVariable: 'STORAGE_ACCOUNT'
                ],
                [
                    secretType: 'Secret',
                    name: 'chatstream-key',
                    envVariable: 'STORAGE_KEY'
                ]
            ]
        ) {

            sh '''
                set -e

                echo "=========================================="
                echo "Uploading APK to Azure Blob Storage"
                echo "=========================================="

                if [ ! -f "$APK_PATH" ]; then
                    echo "ERROR: APK not found!"
                    exit 1
                fi

                BLOB_NAME="chatstream-sdk-${BUILD_NUMBER}.apk"

                echo "Storage Account: $STORAGE_ACCOUNT"
                echo "Container: $STORAGE_CONTAINER"
                echo "Blob: $BLOB_NAME"
                echo "File: $APK_PATH"

                az storage blob upload \
                    --account-name "$STORAGE_ACCOUNT" \
                    --account-key "$STORAGE_KEY" \
                    --container-name "$STORAGE_CONTAINER" \
                    --name "$BLOB_NAME" \
                    --file "$APK_PATH" \
                    --overwrite true \
                    --no-progress

                echo
                echo "=========================================="
                echo "APK uploaded successfully!"
                echo "=========================================="
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
