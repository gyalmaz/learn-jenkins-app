pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '2512c5d5-ed71-41d3-be42-c1e7e867202e'
        NETLIFY_AUTH_TOKEN = credentials ('netlify-token')
        REACT_APP_VERSION = "1.2.${BUILD_ID}"
    }

    stages {
        stage('AWS') {
            agent {
                docker{
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                }
            }
            environment {
                MY_BUCKET_NAME="learn-jenkins-180220261121"
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        echo "Hello S3!" > index.html
                        aws s3 cp index.html s3://$"{MY_BUCKET_NAME}"/index.html

                    '''
                }
                
            }
        }
        
        stage('Build') {
            agent {
                docker {
                    image 'my-playwright'
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
                            image 'my-playwright'
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
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'my-playwright'
                            reuseNode true
                        }        
                    }
                    steps {
                        sh '''
                            
                            serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Local E2E', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        stage('Deploy Staging') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_SET'
            }   
            steps {
                sh '''
                    
                    netlify --version
                    echo "Deploying to staging. Site ID = $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deploy-output.json)
                    echo "REACT_APP_VERSION is: $REACT_APP_VERSION"
                    npx playwright test --reporter=html
                   
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
        stage('Deploy Prod') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'https://monumental-gaufre-4fae47.netlify.app'
                REACT_APP_VERSION = "1.2.${BUILD_ID}"
            }   
            steps {
                sh '''
                    node --version
                    netlify --version
                    echo "Deploying to staging. Site ID = $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod --json > deploy-output.json
                    echo "REACT_APP_VERSION is: $REACT_APP_VERSION"
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }

    post {
        always {
            junit 'jest-results/junit.xml'
        }
    }
}