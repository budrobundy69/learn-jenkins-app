pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '78a80345-6b9d-4448-bca2-77a96f1e1dfe'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

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
        stage('Tests') {
            parallel {
                stage('Unit Tests') {
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
                    post {
                        always {
                            junit 'jest-results/**/*.xml'
                        }
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
                            echo "Running Local tests..."
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }

                }
            }
        }

        stage('Deploy to Staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                echo "Running Deployment ..."
                npm install sharp node-jq
                npm install netlify-cli
                node_modules/.bin/netlify --version
                echo "Deploying to Staging. Site ID: $NETLIFY_SITE_ID"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                node_modules/.bin/node-jq -r 'deploy_url' deploy-output.json
                '''
            }
        }

        stage('Approval') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    input message: 'Do you want to deploy to Production?', ok: 'Yes, I am sure!'
                }
            }
        }

        stage('Deploy to Production') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                echo "Running Deployment ..."
                npm install sharp
                npm install netlify-cli
                node_modules/.bin/netlify --version
                echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }

        stage('Prod E2E Tests') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.51.1-noble'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://adorable-ganache-bbe55d.netlify.app'
            }

            steps {
                sh '''
                    echo "Running E2E tests..."
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }

        }

    }
    post {
//         always {
//             archiveArtifacts artifacts: 'build/**/*', fingerprint: true
//         }
        success {
            echo "Build and test completed successfully."
        }
        failure {
            echo "Build or test failed."
        }
    }
}
