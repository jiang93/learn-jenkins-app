
pipeline {
    agent any 
    stages {
        stage('build') {
            cleanWs()
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