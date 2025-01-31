pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.${BUILD_ID}"
        AWS_DEFAULT_REGION = 'eu-central-1'
    }

    stages {

        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                    reuseNode true
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'jenkins-user', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        aws ecs register-task-definition --cli-input-json file://aws/task-definition.json
                        aws ecs update-service --cluster LearnJenkinsApp-Prod --service learn-jenkins-app-service --task-definition LearnJenkinsApp-TaskDefinition-Prod:2
                    '''
                }
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
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                '''
            }
        }

        
    }
}
