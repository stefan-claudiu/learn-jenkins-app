pipeline {
agent any

environment {
BUILD_NUMBER = '12.3.5'

}

stages {
        


        stage('Docker') {
            steps {
                sh 'docker build -t my-playwright .'
            }
        }
// This is a comment
stage('Build') {
    agent {
        docker {
            image 'node:18-alpine'
            reuseNode true
        }
    }
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

stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "--entrypoint=''"
                }
            }
            environment {
                AWS_S3_BUCKET = 'learn-jenkins-20250129'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-credentials', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                    aws --version
                    aws s3 ls
                    aws s3 sync build s3://$AWS_S3_BUCKET
                '''
                }
                
            }
        }

stage('Run Tests') {
    parallel {
        stage('Unit Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
            test -f build/index.html
            npm test
        '''
            }

            post {
                always {
                    junit 'jest-results/junit.xml'
                }
            }
        }

        stage('E2E') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            steps {
                sh '''
                serve -s build &
            sleep 11
            npx playwright test --reporter=html
        '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to production."
                    node_modules/.bin/netlify status
                '''
            }
        }
    }
}
}
}
