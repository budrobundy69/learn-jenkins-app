pipeline {
    agent any

    stages {
        stage('Install Dependencies') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    echo "Installing dependencies..."
                    node --version
                    npm --version

                    npm ci
                '''
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    echo "Building the project..."
                    node --version
                    npm --version

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
                    echo "Testing the project..."
                    test -f 'build/index.html'
                    npm test
                '''
            }
        }
        stage('E2E Tests') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.51.1-noble'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    echo "Running E2E tests..."
                    npm install serve
                    node_modules/.bin/serve -s build &
                    sleep 5
                    echo "Running Playwright tests..."
                    npx playwright install
                    npx playwright test --reporter=html
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'build/**/*', fingerprint: true
            junit 'jest-results/**/*.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
        success {
            echo "Build and test completed successfully."
        }
        failure {
            echo "Build or test failed."
        }
    }
}
