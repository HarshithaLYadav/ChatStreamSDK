
pipeline {

    agent any

    environment {
        CI = 'true'
        NODE_ENV = 'development'
        GRADLE_OPTS = '-Dorg.gradle.daemon=false'
    }

    stages {

        stage('Environment Check') {
            steps {
                sh '''
                    echo "=========================================="
                    echo "        ENVIRONMENT CHECK"
                    echo "=========================================="

                    echo "User:"
                    whoami

                    echo "Workspace:"
                    echo "$WORKSPACE"

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

        stage('Android Build') {
            steps {
                dir('examples/SampleApp/android') {
                    sh '''
                        echo "=========================================="
                        echo "         ANDROID RELEASE BUILD"
                        echo "=========================================="

                        chmod +x gradlew

                        ./gradlew assembleRelease --no-daemon

                        echo "=========================================="
                        echo "        ANDROID BUILD COMPLETED"
                        echo "=========================================="

                        echo "APK files:"
                        find app/build/outputs/apk/release -type f -name "*.apk" -print
                    '''
                }
            }
        }

        stage('Verify APK') {
            steps {
                sh '''
                    echo "=========================================="
                    echo "             VERIFY APK"
                    echo "=========================================="

                    APK=$(find examples/SampleApp/android/app/build/outputs/apk/release \
                        -type f -name "*.apk" | head -1)

                    if [ -z "$APK" ]; then
                        echo "ERROR: APK was not generated."
                        exit 1
                    fi

                    echo "APK generated:"
                    echo "$APK"

                    ls -lh "$APK"

                    echo "APK verification successful."
                '''
            }
        }

        stage('Archive APK') {
            steps {
                archiveArtifacts artifacts:
                    'examples/SampleApp/android/app/build/outputs/apk/release/*.apk',
                    fingerprint: true

                echo "APK archived successfully in Jenkins."
            }
        }

        stage('Create Backup') {
            steps {
                sh '''
                    echo "=========================================="
                    echo "            CREATE BACKUP"
                    echo "=========================================="

                    mkdir -p backup

                    cp examples/SampleApp/android/app/build/outputs/apk/release/*.apk backup/

                    echo "Backup contents:"
                    ls -lh backup/
                '''

                archiveArtifacts artifacts: 'backup/*.apk',
                    fingerprint: true
            }
        }
    }

    post {

        success {
            echo "=========================================="
            echo "       CI/CD PIPELINE SUCCESSFUL"
            echo "=========================================="

            echo "APK was successfully:"
            echo "1. Built"
            echo "2. Verified"
            echo "3. Archived"
            echo "4. Backed up"
        }

        failure {
            echo "=========================================="
            echo "          CI/CD PIPELINE FAILED"
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

