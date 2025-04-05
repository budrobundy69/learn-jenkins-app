pipeline {
    agent any

    stages {
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
