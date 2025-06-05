pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID='9ed2dbab-b7bb-4182-be91-7fdabb58894c'
        NETLIFY_AUTH_TOKEN=credentials('netlify-token')
        REACT_APP_VERSION = '1.2.3'
    }
    stages {
        /*
         * This is the Build Stage
         */
        stage('Build') {
            // Configuriomg a Docker based agent
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps { 
                sh '''
                    echo "Small change ;)"
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                ''' 
            }
        } 
        stage('Run Tests') {
            parallel {
                stage('Unittests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            ls -la
                            node --version
                            npm --version
                            test build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.52.0-noble'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            ls -la
                            node --version
                            npm --version
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 5
                            npx playwright test --reporter=html
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

        stage('Deploy Staging') {
            environment {
                CI_ENVIRONMENT_URL="Not initialized"
            }     
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.52.0-noble'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    node --version
                    npm --version
                    echo "Deyploying..."
                    npm install netlify-cli@20.1.1 node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploying to STAGING site Id: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > deploy-staging.json
                    CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' deploy-staging.json)
                    echo "Deyployed to $CI_ENVIRONMENT_URL."

                    echo "Testing against $CI_ENVIRONMENT_URL ..."
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        /*
        stage('Approval') {
            steps {
                timeout(time: 600, unit: 'SECONDS') {
                    input cancel: 'Do not deploy', message: 'Ready to deploy?', ok: 'Yes, I want to deploy'
                }
            }
        }
        */

        stage('Deploy Prod') {
            environment {
                CI_ENVIRONMENT_URL="https://heroic-belekoy-7c11cd.netlify.app"
            } 
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.52.0-noble'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    
                    echo "Deploying..."
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    echo "Deploying to PROD site Id: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                    
                    echo "Running E2E tests..."
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}
