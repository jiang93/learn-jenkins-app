pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = '9cb184e8-3f05-44e0-920b-d5f27390fae5'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }
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

        stage('run test') {
            parallel {
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
                            npm install -g serve
                            serve -s build --listen 3000 & 
                            sleep 10
                            npx playwright test --reporter=html --list                    
                        '''
                    }
                    
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        stage('deploy') {
            agent {
                docker {
                    image 'node:20-alpine'
                    reuseNode true
                }
            }
            steps {
                sh"""
                    npm install netlify-cli@20.1.1 -g
                    netlify --version
                    echo "Deploying to production site $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir build --prod
                    echo "A small change here..."
                """
            }
        }
    }
}