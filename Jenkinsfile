stages {

    stage('Checkout') {
        steps {
            checkout scm
        }
    }

    stage('Environment Check') {
        steps {
            sh '''
                echo "===== Environment ====="
                whoami
                echo "Workspace: $WORKSPACE"

                echo "===== Node ====="
                node --version

                echo "===== Yarn ====="
                yarn --version

                echo "===== Java ====="
                java -version

                echo "===== Git ====="
                git --version
            '''
        }
    }

    stage('Install Dependencies') {
        steps {
            sh '''
                cd package
                yarn install --frozen-lockfile
            '''
        }
    }

    stage('Lint') {
        steps {
            sh '''
                cd package
                yarn lint
            '''
        }
    }

    stage('Unit Tests') {
        steps {
            sh '''
                cd package
                yarn test:unit --runInBand
            '''
        }
    }
}
