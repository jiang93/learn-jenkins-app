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
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        stage('deploy staging') {
            agent {
                docker {
                    image 'node:20-alpine'
                    reuseNode true
                }
            }
            steps {
                sh"""
                    npm install netlify-cli@20.1.1 -g node-jq
                    netlify --version
                    echo "Deploying to staging site $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir build --json > deploy-output.json
                    """
                script {
                    env.DEPLOY_URL = sh(script: "node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
                }

            }
        }

        stage('staging e2e') {
            environment {
                CI_ENVIRONMENT_URL = '$env.DEPLOY_URL' 
            }
            agent {
                    docker {
                        image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                        reuseNode true 
                    }
            }
            steps {
                sh '''
                    npx playwright test --reporter=html --list                    
                '''
            }
            
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('approval') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                }
            }
        }

        stage('deploy prod') {
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
                    echo "website deploy url is ${env.DEPLOY_URL}"
                """
            }
        }
        stage('prod e2e') {
            environment {
                CI_ENVIRONMENT_URL = 'https://delicate-hui-b0901b.netlify.app' 
            }
            agent {
                    docker {
                        image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                        reuseNode true 
                    }
            }
            steps {
                sh '''
                    npx playwright test --reporter=html --list                    
                '''
            }
            
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prodution E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}