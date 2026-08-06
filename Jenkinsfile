pipeline {
    agent any

<<<<<<< HEAD
<<<<<<< HEAD
=======
>>>>>>> fe7d0a0e2 (Updated Jenkinsfile)
    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        NODE_ENV = 'development'
        SONAR_SCANNER_HOME = tool 'SonarScanner'
        STORAGE_CONTAINER = 'apk-artifacts'
    }

    triggers {
        githubPush()
    }

    stages {

<<<<<<< HEAD
<<<<<<< HEAD
=======
    stages {
>>>>>>> ea179b5f5 (Jenkinsfile)
=======
>>>>>>> fe7d0a0e2 (Updated Jenkinsfile)
        stage('Checkout') {
=======
        stage('1. Checkout') {
>>>>>>> 14e64bc26 (updated Jenkinsfile)
            steps {
                checkout scm
            }
        }
<<<<<<< HEAD
<<<<<<< HEAD
=======
>>>>>>> fe7d0a0e2 (Updated Jenkinsfile)

        stage('2. Install, Lint & Test') {
            steps {
                sh '''
                    echo "Installing Dependencies..."
                    yarn install --frozen-lockfile

<<<<<<< HEAD
        stage('Lint') {
            steps {
<<<<<<< HEAD
                sh 'yarn lint'
=======
                sh '''
                    yarn lint
                '''
>>>>>>> fe7d0a0e2 (Updated Jenkinsfile)
            }
        }

        stage('Unit Tests') {
            steps {
<<<<<<< HEAD
                sh 'yarn test:unit'
=======
                sh '''
                    yarn test:unit
                '''
>>>>>>> fe7d0a0e2 (Updated Jenkinsfile)
            }
        }

        stage('Build') {
            steps {
<<<<<<< HEAD
                sh 'yarn build'
=======
                sh '''
                    yarn build
                '''
>>>>>>> fe7d0a0e2 (Updated Jenkinsfile)
            }
        }

        stage('SonarQube Analysis') {
=======
                    echo "Running Lint..."
                    yarn lint

                    echo "Running Unit Tests..."
                    yarn test
                '''
            }
        }

        stage('3. SonarQube Analysis') {
>>>>>>> 6b703f456 (updated Jenkinsfile)
            steps {
                withSonarQubeEnv('SonarQube') {
                    withCredentials([
                        string(credentialsId: 'sonar-pat', variable: 'SONAR_TOKEN')
                    ]) {

                        sh """
                        ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
<<<<<<< HEAD
                        -Dsonar.projectKey=ChatStreamSDK \
                        -Dsonar.projectName=ChatStreamSDK \
                        -Dsonar.sources=. \
                        -Dsonar.sourceEncoding=UTF-8 \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.token=$SONAR_TOKEN
=======
                          -Dsonar.projectKey=ChatStreamSDK \
                          -Dsonar.projectName=ChatStreamSDK \
                          -Dsonar.sources=. \
                          -Dsonar.sourceEncoding=UTF-8 \
                          -Dsonar.host.url=$SONAR_HOST_URL \
                          -Dsonar.token=$SONAR_TOKEN
>>>>>>> fe7d0a0e2 (Updated Jenkinsfile)
                        '''
=======
                        -Dsonar.projectKey=ChatStream \
                        -Dsonar.projectName=ChatStream \
                        -Dsonar.sources=. \
                        -Dsonar.sourceEncoding=UTF-8 \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.token=${SONAR_TOKEN}
                        """
>>>>>>> 6b703f456 (updated Jenkinsfile)
                    }
                }

                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('4. Build APK') {
            steps {
                sh '''
                    echo "Building Release APK..."

                    cd examples/android

                    chmod +x gradlew

                    ./gradlew clean

                    ./gradlew assembleRelease
                '''
            }
        }

        stage('5. Archive & Upload APK') {
            steps {

                archiveArtifacts artifacts: 'examples/android/app/build/outputs/apk/release/*.apk',
                                 fingerprint: true

                withCredentials([
                    string(credentialsId: 'storage-account-name', variable: 'STORAGE_ACCOUNT'),
                    string(credentialsId: 'storage-account-key', variable: 'STORAGE_KEY')
                ]) {

                    sh '''
                    echo "Uploading APK to Azure Blob Storage..."

                    az storage blob upload \
                      --account-name $STORAGE_ACCOUNT \
                      --account-key $STORAGE_KEY \
                      --container-name $STORAGE_CONTAINER \
                      --file examples/android/app/build/outputs/apk/release/app-release.apk \
                      --name ChatStream-${BUILD_NUMBER}.apk \
                      --overwrite

                    echo "APK uploaded successfully."
                    '''
                }
            }
        }

        stage('6. Download & Verify APK') {
            steps {

                withCredentials([
                    string(credentialsId: 'storage-account-name', variable: 'STORAGE_ACCOUNT'),
                    string(credentialsId: 'storage-account-key', variable: 'STORAGE_KEY')
                ]) {

                    sh '''
                    mkdir -p downloaded

                    echo "Downloading APK from Azure Blob Storage..."

                    az storage blob download \
                      --account-name $STORAGE_ACCOUNT \
                      --account-key $STORAGE_KEY \
                      --container-name $STORAGE_CONTAINER \
                      --name ChatStream-${BUILD_NUMBER}.apk \
                      --file downloaded/ChatStream-${BUILD_NUMBER}.apk

                    echo "Downloaded Artifact:"

                    ls -lh downloaded/

                    echo "Artifact verification completed successfully."

                    echo "CD stage completed."
                    '''
                }
            }
        }
    }

    post {
<<<<<<< HEAD
=======

>>>>>>> fe7d0a0e2 (Updated Jenkinsfile)
        success {
            echo "React Native CI/CD Pipeline completed successfully."
        }

        failure {
            echo "Pipeline failed."
        }

        always {
            cleanWs()
        }
    }
}
=======
    }
}
>>>>>>> ea179b5f5 (Jenkinsfile)
