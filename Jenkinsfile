
pipeline {
    agent any 
    stages {
        stage('prepare') {
            steps {
                cleanWs()
            }
        }
        stage('build') {
            agent {
                docker {
                    image 'node:20-alpine'
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
    }
}