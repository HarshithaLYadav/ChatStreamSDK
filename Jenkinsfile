
pipeline {

    agent any

    environment {

        // ==============================
        // Android SDK
        // ==============================
        ANDROID_HOME = '/home/azureuser/Android/Sdk'
        ANDROID_SDK_ROOT = '/home/azureuser/Android/Sdk'

        // ==============================
        // Android / System PATH
        // ==============================
        PATH = "/home/azureuser/Android/Sdk/platform-tools:/home/azureuser/Android/Sdk/cmdline-tools/latest/bin:/home/azureuser/Android/Sdk/build-tools/35.0.0:${env.PATH}"
    }

    stages {

        // =========================================================
        // 1. Environment Check
        // =========================================================
        stage('Environment Check') {
            steps {
                sh '''
                    echo "=========================================="
                    echo "        ENVIRONMENT CHECK"
                    echo "=========================================="

                    echo "User:"
                    whoami

                    echo "Workspace:"
                    pwd

                    echo "Node:"
                    node --version

                    echo "Yarn:"
                    yarn --version

                    echo "Java:"
                    java -version

                    echo "Git:"
                    git --version

                    echo "=========================================="
                '''
            }
        }


        // =========================================================
        // 2. Android Environment Check
        // =========================================================
        stage('Android Environment Check') {
            steps {
                sh '''
                    echo "=========================================="
                    echo "      ANDROID ENVIRONMENT CHECK"
                    echo "=========================================="

                    echo "ANDROID_HOME=$ANDROID_HOME"
                    echo "ANDROID_SDK_ROOT=$ANDROID_SDK_ROOT"

                    echo "PATH:"
                    echo "$PATH"

                    echo "=========================================="
                    echo "Checking Android SDK..."
                    echo "=========================================="

                    test -d "$ANDROID_HOME"

                    echo "Android SDK:"
                    ls -ld "$ANDROID_HOME"

                    echo "=========================================="
                    echo "Checking ADB..."
                    echo "=========================================="

                    test -f "$ANDROID_HOME/platform-tools/adb"
                    adb version

                    echo "=========================================="
                    echo "Checking SDK Manager..."
                    echo "=========================================="

                    test -f "$ANDROID_HOME/cmdline-tools/latest/bin/sdkmanager"
                    sdkmanager --version

                    echo "=========================================="
                    echo "Android environment is ready."
                    echo "=========================================="
                '''
            }
        }


        // =========================================================
        // 3. Clean Dependencies
        // =========================================================
        stage('Clean Dependencies') {
            steps {
                sh '''
                    echo "=========================================="
                    echo "        CLEANING NODE MODULES"
                    echo "=========================================="

                    rm -rf package/node_modules
                    rm -rf package/native-package/node_modules
                    rm -rf examples/SampleApp/node_modules

                    echo "Node modules cleaned."
                '''
            }
        }


        // =========================================================
        // 4. Install Package Dependencies
        // =========================================================
        stage('Install Package Dependencies') {
            steps {
                dir('package') {
                    sh '''
                        echo "=========================================="
                        echo "   INSTALLING PACKAGE DEPENDENCIES"
                        echo "=========================================="

                        yarn install --frozen-lockfile

                        echo "Package dependencies installed."
                    '''
                }
            }
        }


        // =========================================================
        // 5. Install Native Package Dependencies
        // =========================================================
        stage('Install Native Package Dependencies') {
            steps {
                dir('package/native-package') {
                    sh '''
                        echo "=========================================="
                        echo " INSTALLING NATIVE-PACKAGE DEPENDENCIES"
                        echo "=========================================="

                        yarn install --frozen-lockfile

                        echo "Checking mime..."

                        test -d node_modules/mime

                        echo "mime dependency exists."
                        echo "Native package dependencies installed."
                    '''
                }
            }
        }


        // =========================================================
        // 6. Build Package
        // =========================================================
        stage('Build Package') {
            steps {
                dir('package') {
                    sh '''
                        echo "=========================================="
                        echo "          BUILDING PACKAGE"
                        echo "=========================================="

                        yarn build

                        echo "Package build completed."
                    '''
                }
            }
        }


        // =========================================================
        // 7. Install SampleApp Dependencies
        // =========================================================
        stage('Install SampleApp Dependencies') {
            steps {
                dir('examples/SampleApp') {
                    sh '''
                        echo "=========================================="
                        echo " INSTALLING SAMPLEAPP DEPENDENCIES"
                        echo "=========================================="

                        yarn install --frozen-lockfile

                        echo "SampleApp dependencies installed."
                    '''
                }
            }
        }


        // =========================================================
        // 8. Verify React Native Dependencies
        // =========================================================
        stage('Verify React Native Dependencies') {
            steps {
                sh '''
                    echo "=========================================="
                    echo " VERIFYING REACT NATIVE DEPENDENCIES"
                    echo "=========================================="

                    echo "Checking SampleApp node_modules..."
                    test -d examples/SampleApp/node_modules

                    echo "Checking React Native..."
                    test -d examples/SampleApp/node_modules/react-native

                    echo "Checking React Native Gradle Plugin..."
                    test -d examples/SampleApp/node_modules/@react-native/gradle-plugin

                    echo "Checking linked native package..."
                    test -d package/native-package/node_modules

                    echo "Checking mime..."
                    test -d package/native-package/node_modules/mime

                    echo "All required dependencies are present."
                '''
            }
        }


        // =========================================================
        // 9. Lint
        // =========================================================
        stage('Lint') {
            steps {
                dir('package') {
                    sh '''
                        echo "=========================================="
                        echo "              LINT"
                        echo "=========================================="

                        yarn lint

                        echo "Lint completed successfully."
                    '''
                }
            }
        }


        // =========================================================
        // 10. Unit Tests
        // =========================================================
        stage('Unit Tests') {
            steps {
                dir('package') {
                    sh '''
                        echo "=========================================="
                        echo "           UNIT TESTS"
                        echo "=========================================="

                        yarn test:unit --passWithNoTests

                        echo "Unit tests completed."
                    '''
                }
            }
        }


        // =========================================================
        // 11. Android Release Build
        // =========================================================
        stage('Android Build') {
            steps {
                dir('examples/SampleApp/android') {
                    sh '''
                        echo "=========================================="
                        echo "         ANDROID RELEASE BUILD"
                        echo "=========================================="

                        echo "ANDROID_HOME=$ANDROID_HOME"
                        echo "ANDROID_SDK_ROOT=$ANDROID_SDK_ROOT"

                        echo "Checking SDK..."
                        test -d "$ANDROID_HOME"

                        echo "Checking ADB..."
                        adb version

                        echo "Checking SDK Manager..."
                        sdkmanager --version

                        echo "Making Gradle wrapper executable..."
                        chmod +x gradlew

                        echo "Starting Gradle release build..."

                        ./gradlew assembleRelease --no-daemon

                        echo "=========================================="
                        echo "       ANDROID BUILD COMPLETED"
                        echo "=========================================="

                        echo "APK files:"
                        find app/build/outputs/apk -type f -name "*.apk" -print
                    '''
                }
            }
        }


        // =========================================================
        // 12. Archive APK
        // =========================================================
        stage('Archive APK') {
            steps {
                echo "=========================================="
                echo "          ARCHIVING APK"
                echo "=========================================="

                archiveArtifacts artifacts: 'examples/SampleApp/android/app/build/outputs/apk/**/*.apk',
                                 fingerprint: true

                echo "APK archived successfully."
            }
        }
    }


    // =============================================================
    // POST ACTIONS
    // =============================================================
    post {

        success {
            echo "=========================================="
            echo "          PIPELINE SUCCESS"
            echo "=========================================="
            echo "CI/CD pipeline completed successfully."
            echo "APK was built and archived."
        }

        failure {
            echo "=========================================="
            echo "          PIPELINE FAILED"
            echo "=========================================="
            echo "Check the failed stage above."
        }

        always {
            echo "=========================================="
            echo "          PIPELINE COMPLETED"
            echo "=========================================="
        }
    }
}

