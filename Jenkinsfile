pipeline {
    agent any

    stages {
        /*
        stage('Build') {
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
        */
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }


            }
            steps {
                sh '''
                   echo "### Test Stage ###"
                   echo "### Testing if build/index.html exists ###"
                   test -f build/index.html
                   echo "### Running npm tests ###"
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
