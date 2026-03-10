pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = '9cb184e8-3f05-44e0-920b-d5f27390fae5'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION= "1.0.$BUILD_ID"
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

        stage('docker') {
            steps {
                sh 'docker build -t my-playwright .'
            }
            
        }

        stage('test') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true 
                    args '-p 3000:3000'
                }
            }
            steps {
                sh '''
                echo "Test stage"
                ls build/index.html
                npm run test
                serve -s build --listen 3000 & 
                sleep 10
                npx playwright test --reporter=html --list    
                '''
            }

            post {
                always {
                    junit 'test-results/junit.xml'
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                }
            }

        }

        stage('deploy staging') {

            environment {
                DEPLOY_URL = ''
            }
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true 
                }
            }
            steps {
                sh '''
                    netlify --version
                    echo "Deploying to staging site $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir build --json > deploy-output.json
                    npx playwright test --reporter=html --list                    
                '''
                script {
                    env.DEPLOY_URL = sh(script: "node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
                }
                sh 'echo "website deploy url is $DEPLOY_URL"'

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
            environment {
                CI_ENVIRONMENT_URL = 'https://delicate-hui-b0901b.netlify.app' 
            }
            agent {
                    docker {
                        image 'my-playwright'
                        reuseNode true 
                    }
            }
            steps {
                sh '''
                    netlify --version
                    echo "Deploying to production site $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir build --prod
                    echo "website deploy url is ${CI_ENVIRONMENT_URL}"
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