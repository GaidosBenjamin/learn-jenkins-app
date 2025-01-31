pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.${BUILD_ID}"
        APP_NAME = "my-jenkins-app"
        AWS_DEFAULT_REGION = 'eu-central-1'
        AWS_ECS_CLUSTER = 'LearnJenkinsApp-Prod'
        AWS_ECS_SERVICE = 'learn-jenkins-app-service'
        AWS_ECS_TASK_DEFINITION = 'LearnJenkinsApp-TaskDefinition-Prod'
        AWS_ECR = "471112858623.dkr.ecr.eu-central-1.amazonaws.com"
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
                '''
            }
        }

        stage('Build Docker Image') {
            agent {
                docker {
                    image 'my-aws-cli'
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                    reuseNode true
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'jenkins-user', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        docker build -t ${AWS_ECR}/${APP_NAME}:${REACT_APP_VERSION} .
                        ECR_PASSWORD=$(aws ecr get-login-password)
                        docker login --username AWS --password ${ECR_PASSWORD} ${AWC_ECR}
                        docker push ${AWS_ECR}/${APP_NAME}:${REACT_APP_VERSION}
                    '''
                }
            }
        }

        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'my-aws-cli'
                    args "--entrypoint=''"
                    reuseNode true
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'jenkins-user', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition.json | jq ".taskDefinition.revision")
                        echo ${LATEST_TD_REVISION}
                        aws ecs update-service --cluster ${AWS_ECS_CLUSTER} --service ${AWS_ECS_SERVICE} --task-definition ${AWS_ECS_TASK_DEFINITION}:${LATEST_TD_REVISION}
                        aws ecs wait services-stable --cluster ${AWS_ECS_CLUSTER} --services ${AWS_ECS_SERVICE}
                    '''
                }
            }
        }
        
    }
}
