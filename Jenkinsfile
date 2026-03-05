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

            post {
                always {
                    junit 'test-results/junit.xml'
                }
            }
        }
        stage('e2e') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true 
                    args '-p 3000:3000'
                }
            }
            steps {
                sh '''
                    ls -la
                    npm install -g serve
                    serve -s build --listen 3000 & 
                    sleep 10
                    npx playwright test --reporter=html
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'archiveArtifacts artifacts: \'playwright-report\', followSymlinks: false', reportFiles: 'playwright_index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])              
                '''
            }
        }

    }
}