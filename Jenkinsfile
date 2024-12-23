pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'd20f3d35-6bb3-4a31-8865-d78882b8f837'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:current-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm install
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Run Tests') {
            parallel {
                stage('Unit Tests') {
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
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }
                stage('Prod E2E Tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.49.1-jammy'
                            reuseNode true
                        }
                    }

                    environment {
                        CI_ENVIRONMENT_URL = 'https://soft-blancmange-cb773c.netlify.app'
                    }
                    steps {
                        sh '''
                            echo "E2E Test stage"
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML(
                                [
                                    allowMissing: false,
                                    alwaysLinkToLastBuild: false,
                                    keepAll: false,
                                    reportDir: 'playwright-report',
                                    reportFiles: 'index.html',
                                    reportName: 'Playwright Local Report',
                                    reportTitles: '',
                                    useWrapperFileDirectly: true
                                ]
                            )
                        }
                    }
                }
            }
        }
        stage('Deploy') {
            agent {
                docker {
                    image 'node:current-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli  # --save-dev
                    node_modules/netlify-cli/bin/run.js --version
                    echo "Deploying to Netlify site ID: $NETLIFY_SITE_ID"
                    node_modules/netlify-cli/bin/run.js status
                    node_modules/netlify-cli/bin/run.js deploy --dir=build --prod
                '''
            }
        }

   }
    post {
        always {
            junit 'jest-results/junit.xml'
            publishHTML(
                [
                    allowMissing: false,
                    alwaysLinkToLastBuild: false,
                    keepAll: false,
                    reportDir: 'playwright-report',
                    reportFiles: 'index.html',
                    reportName: 'Playwright E2e Report',
                    reportTitles: '',
                    useWrapperFileDirectly: true
                ]
            )
        }
    }
}
