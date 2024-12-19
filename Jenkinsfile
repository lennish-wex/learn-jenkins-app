pipeline {
    agent any

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
                    npm
                    npm run build
                    ls -la
                '''
            }
        }
    }
}
