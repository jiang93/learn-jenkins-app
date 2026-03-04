
pipeline {
    agent any 
    stages {
        stage ('build') {
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