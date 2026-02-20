pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.2.${BUILD_ID}"
        AWS_DEFAULT_REGION = "us-east-1"
    }

    stages {
        
        stage('Deploy to AWS') {
            agent {
                docker{
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        yum install jq -y
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://AWS/task-definition-prod.json | jq '.taskDefinition.revision')
                        echo ${LATEST_TD_REVISION}
                        aws ecs update-service --cluster LearningJenkins-Prod --service learnjenkinsapp-taskdefinition-prod-service-rzm98kb4 --task-definition learnjenkinsapp-taskdefinition-prod:${LATEST_TD_REVISION}
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
    }
    post {
        always {
            junit 'jest-results/junit.xml'
        }
    }
}