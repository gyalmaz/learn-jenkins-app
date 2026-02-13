pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '2512c5d5-ed71-41d3-be42-c1e7e867202e'
        NETLIFY_AUTH_TOKEN = credentials ('netlify-token')
    }

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
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        

        stage('Tests') {
            parallel {
                stage('Unit test') {
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

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.58.2-noble'
                            reuseNode true
                        }        
                    }
                         
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }  // ← This closes 'parallel'
        }      // ← This closes 'stage('Tests')'

        stage('Deploy Staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    apk add --no-cache bash
                    echo "Small Change"
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to staging. Side ID = $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build
                '''
            }
        } // This closes Deploy Stage 
        stage('Approval') {
            steps {
                echo 'Waiting for approval....'
                timeout(time: 1, unit: 'HOURS') {
                    input message: 'Do you wish to deploy to Production?', ok: 'Yes I am sure!'
                }
            }
        } // This closes Approval

        stage('Deploy Production') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    apk add --no-cache bash
                    echo "Small Change"
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Side ID = $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        } // This closes Deploy Prod

        stage('Prod E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.58.2-noble'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'https://monumental-gaufre-4fae47.netlify.app'
            }   
            steps {
                sh '''
                    npx playwright test --reporter=html
                '''
            }
        }
    }  // ← This closes 'stages'

    post {
        always {
            junit 'jest-results/junit.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}