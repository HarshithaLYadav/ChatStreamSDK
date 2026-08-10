pipeline {
    agent any

    stages {
        stage('Test Webhook') {
            steps {
                echo 'GitHub webhook successfully triggered Jenkins!'
                sh 'whoami'
                sh 'pwd'
                sh 'java -version'
                sh 'node --version'
                sh 'yarn --version'
            }
        }
    }
}
