pipeline {
    agent any

<<<<<<< HEAD
    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        NODE_ENV = 'development'
        SONAR_SCANNER_HOME = tool 'SonarScanner'
    }

    triggers {
        githubPush()
    }

    stages {

=======
    stages {
>>>>>>> ea179b5f5 (Jenkinsfile)
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
<<<<<<< HEAD

        stage('Environment Info') {
            steps {
                sh '''
                    echo "========== Environment =========="
                    node -v
                    npm -v
                    yarn -v
                    git --version
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    yarn install --frozen-lockfile
                '''
            }
        }

        stage('Lint') {
            steps {
                sh 'yarn lint'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'yarn test:unit'
            }
        }

        stage('Build') {
            steps {
                sh 'yarn build'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    withCredentials([
                        string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')
                    ]) {
                        sh '''
                        ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=ChatStreamSDK \
                        -Dsonar.projectName=ChatStreamSDK \
                        -Dsonar.sources=. \
                        -Dsonar.sourceEncoding=UTF-8 \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.token=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: '**/lib/**, **/dist/**, **/*.tgz', fingerprint: true
            }
        }
    }

    post {
        success {
            echo 'CI Pipeline completed successfully.'
        }

        failure {
            echo 'CI Pipeline failed.'
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
