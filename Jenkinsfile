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
    }

    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}
