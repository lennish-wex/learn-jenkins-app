pipeline {
    agent any

    stages {
        // stage('Build') {
        //     agent {
        //         docker {
        //             image 'node:current-alpine'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         sh '''
        //             ls -la
        //             node --version
        //             npm --version
        //             npm install
        //             npm ci
        //             npm run build
        //             ls -la
        //         '''
        //     }
        // }
        stage('Test') {
            agent {
                docker {
                    image 'node:current-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Test stage"
                    test -f build/index.html
                    npm test
                '''
            }
        }
         stage('E2E Test') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.49.1-jammy'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "E2E Test stage"
                    npm install serve
                    node_modules/serve/build/main.js -s build &  # Start server in the background
                    sleep 10  # Give the server some time to start
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
