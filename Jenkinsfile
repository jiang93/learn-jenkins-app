pipeline {
    agent any 
    stages {
        stage('build') {
            agent {
                docker {
                    image 'node:20-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version 
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        stage('test') {
            agent {
                docker {
                    image 'node:20-alpine'
                    reuseNode true  
                }
            }
            steps {
                sh '''
                    echo "Test stage"
                    ls build/index.html
                    npm run test
                '''
            }
        }
    }
}