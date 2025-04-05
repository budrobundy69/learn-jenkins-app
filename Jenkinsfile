pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                script {
                    echo "Checking out the repository..."
                    checkout scm
                }
            }
        }
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
    }

    post {
        always {
            archiveArtifacts artifacts: 'build/**/*', fingerprint: true
            junit 'test-results/**/*.xml'
        }
        success {
            echo "Build and test completed successfully."
        }
        failure {
            echo "Build or test failed."
        }
    }
}
