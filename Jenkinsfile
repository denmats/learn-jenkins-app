pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -al
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -al
                '''
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    cd build
                    test -f index.html
                '''
                sh '''
                    ls -al
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    npm test
                    ls -al
                '''
            }
        }
    }
}
