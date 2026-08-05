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
    }

    triggers {
        githubPush()
    }

    stages {

<<<<<<< HEAD
=======
    stages {
>>>>>>> ea179b5f5 (Jenkinsfile)
=======
>>>>>>> fe7d0a0e2 (Updated Jenkinsfile)
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
<<<<<<< HEAD
<<<<<<< HEAD
=======
>>>>>>> fe7d0a0e2 (Updated Jenkinsfile)

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
            steps {
                withSonarQubeEnv('SonarQube') {
                    withCredentials([
                        string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')
                    ]) {
                        sh '''
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
<<<<<<< HEAD
=======

>>>>>>> fe7d0a0e2 (Updated Jenkinsfile)
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
