pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID='9ed2dbab-b7bb-4182-be91-7fdabb58894c'
        NETLIFY_AUTH_TOKEN=credentials('netlify-token')
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
        stage('Staging Deploy') {
            // Configuriomg a Docker based agent
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps { 
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    echo "Deploying to STAGING site Id: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build 
                ''' 
            }
        } 
        stage('Staging E2E') {
            environment {
                CI_ENVIRONMENT_URL='https://heroic-belekoy-7c11cd.netlify.app'
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
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
        stage('Approval') {
            steps {
                timeout(time: 30, unit: 'SECONDS') {
                    input cancel: 'Do not deploy', message: 'Ready to deploy?', ok: 'Yes, I want to deploy'
                }
            }
        }
        stage('Prod Deploy') {
            // Configuriomg a Docker based agent
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps { 
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    echo "Deploying to PROD site Id: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                ''' 
            }
        } 

        stage('Prod E2E') {
            environment {
                CI_ENVIRONMENT_URL='https://heroic-belekoy-7c11cd.netlify.app'
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
