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

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:current-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            #test -f build/index.html
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
                            image 'mcr.microsoft.com/playwright:v1.49.1-jammy'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            npm install serve
                            node_modules/serve/build/main.js -s build &
                            sleep 10
                            npx playwright test  --reporter=html
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
                                    reportName: 'Playwright Local',
                                    reportTitles: '',
                                    useWrapperFileDirectly: true
                                ]
                            )
                        }
                    }
                }
            }
        }

        stage('Deploy Stage') {
            agent {
                docker {
                    image 'node:current-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli node-jq
                    node_modules/netlify-cli/bin/run.js --version
                    echo "Deploying to Netlify site ID: $NETLIFY_SITE_ID"
                    node_modules/netlify-cli/bin/run.js status
                    node_modules/netlify-cli/bin/run.js deploy --dir=build --json > deploy-output.json
                    node_modules/node-jq/bin/jq -r '.deploy_url' deploy-output.json
                '''

                script {
                    env.STAGING_URL = sh(script: "node_modules/node-jq/bin/jq -r '.deploy_url' deploy-output.json", returnStdout: true)
                }

                sh '''
                    echo "Staging URL: $env.STAGING_URL"
                '''
            }
        }

        stage('Staging E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.49.1-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "{$env.STAGING_URL}"
            }

            steps {
                sh '''
                    npx playwright test  --reporter=html
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
                            reportName: 'Staging E2E',
                            reportTitles: '',
                            useWrapperFileDirectly: true
                        ]
                    )
                }
            }
        }

        stage('Approval') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message:  'Do you wish to deploy to production?',
                    ok: 'Yes, I am sure!'
                }
            }
        }

        stage('Deploy Prod') {
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

        stage('Prod E2E') {
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
                    npx playwright test  --reporter=html
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
                            reportName: 'Prod E2E',
                            reportTitles: '',
                            useWrapperFileDirectly: true
                        ]
                    )
                }
            }
        }
    }
}
