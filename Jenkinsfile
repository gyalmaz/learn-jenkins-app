pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.2.${BUILD_ID}"
        AWS_DEFAULT_REGION = "us-east-1"
        APP_NAME = "learnjenkinsapp"
        AWS_ECS_CLUSTER = "LearningJenkins-Prod"
        AWS_ECS_SERVICE = "learnjenkinsapp-taskdefinition-prod-service-rzm98kb4"
        AWS_ECS_TD = "learnjenkinsapp-taskdefinition-prod"
        AWS_ECR_NAME ="767903311120.dkr.ecr.us-east-1.amazonaws.com"
    }

    stages {  
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

         stage('Build Docker Image') {

            agent {
                docker{
                    image 'my-aws-cli'
                    reuseNode true
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                }
            }

            steps {
                 withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {

                    sh '''
                    
                        docker build -t $AWS_ECR_NAME/${APP_NAME}:${REACT_APP_VERSION} .
                        aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_ECR_NAME
                        docker push $AWS_ECR_NAME/${APP_NAME}:${REACT_APP_VERSION}
                    
                    '''
                
                }


            }
        }

        stage('Deploy to AWS') {
            agent {
                docker{
                    image 'my-aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://AWS/task-definition-prod.json | jq '.taskDefinition.revision')
                        echo ${LATEST_TD_REVISION}
                        aws ecs update-service --cluster ${AWS_ECS_CLUSTER} --service ${AWS_ECS_SERVICE} --task-definition ${AWS_ECS_TD}:${LATEST_TD_REVISION}
                        aws ecs wait services-stable --cluster ${AWS_ECS_CLUSTER} --services ${AWS_ECS_SERVICE}
                    '''
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
