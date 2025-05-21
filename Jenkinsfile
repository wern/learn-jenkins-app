pipeline {
    agent any

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
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                ''' 
            }
        } 

         stage('Test') {
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
                    npx playwright test
                '''
            }
        }
    }

    post {
        always {
            junit 'jest-results/junit.xml'
        }
    }
}
