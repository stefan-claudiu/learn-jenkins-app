pipeline {
    agent {
       docker {
        image 'node:18-alpine'
        reuseNode true
       }
    }

    stages {
        stage('Build') {
            steps {
                sh '''
                    ls -la
                    npm --version
                    node --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
    }
}