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
        stage('e2e') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-noble'
                    reuseNode true  
                }
            }
            steps {
                sh '''
                    ls -la
                    npm install -g serve
                    which serve || echo "serve not in PATH"
                    serve --version
                    serve -s build &
                    sleep 10
                    npx playwright test
                '''
            }
        }
    }

    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}