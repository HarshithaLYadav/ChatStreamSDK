pipeline {

    agent any

    environment {
        HOME = '/var/lib/jenkins'

        ANDROID_HOME = '/home/azureuser/Android/Sdk'
        ANDROID_SDK_ROOT = '/home/azureuser/Android/Sdk'

        PATH = "/usr/local/bin:/usr/bin:/bin:${env.PATH}"
    }

    stages {

        // 1. Checkout source code
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // 2. Install JavaScript dependencies
        stage('Install Dependencies') {
            steps {
                sh '''
                    node --version
                    yarn --version
                    yarn install --frozen-lockfile
                '''
            }
        }

        // 3. SonarQube code analysis
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'sonar-scanner'
                }
            }
        }

        // 4. Run tests and lint
        stage('Tests') {
            steps {
                sh '''
                    yarn lint || true
                    yarn test --runInBand || true
                '''
            }
        }

        // 5. Build Android release APK
        stage('Android Release Build') {
            steps {
                dir('examples/SampleApp/android') {
                    sh '''
                        chmod +x gradlew
                        ./gradlew clean
                        ./gradlew assembleRelease
                    '''
                }
            }
        }

        // 6. Archive APK
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
        }

        failure {
            echo 'CI/CD pipeline failed. Check the stage logs.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}
