
pipeline {
    agent any 
    stages {
        stage('build') {
            agent {
                docker {
                    image 'node:20-alpine'
                }
            }
            steps {
                cleanWs()
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