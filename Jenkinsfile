pipeline {
    agent any

    environment {
        AWS_REGION      = 'us-east-1'
        ECR_BACKEND     = '795644302799.dkr.ecr.us-east-1.amazonaws.com/ecs-fargate-cicd-backend'
        ECR_FRONTEND    = '795644302799.dkr.ecr.us-east-1.amazonaws.com/ecs-fargate-cicd-frontend'
        ECS_CLUSTER     = 'ecs-fargate-cicd-cluster'
        BACKEND_SERVICE = 'ecs-fargate-cicd-backend-service'
        FRONTEND_SERVICE= 'ecs-fargate-cicd-frontend-service'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Images') {
            steps {
                sh 'docker build -t $ECR_BACKEND:latest ./backend'
                sh 'docker build -t $ECR_FRONTEND:latest ./frontend'
            }
        }

        stage('Push to ECR') {
            steps {
                withAWS(credentials: 'aws-credentials', region: "${AWS_REGION}") {
                    sh '''
                        aws ecr get-login-password --region $AWS_REGION | \
                        docker login --username AWS --password-stdin \
                        795644302799.dkr.ecr.us-east-1.amazonaws.com
                        docker push $ECR_BACKEND:latest
                        docker push $ECR_FRONTEND:latest
                    '''
                }
            }
        }

        stage('Deploy to ECS') {
            steps {
                withAWS(credentials: 'aws-credentials', region: "${AWS_REGION}") {
                    sh '''
                        aws ecs update-service \
                            --cluster $ECS_CLUSTER \
                            --service $BACKEND_SERVICE \
                            --force-new-deployment \
                            --region $AWS_REGION

                        aws ecs update-service \
                            --cluster $ECS_CLUSTER \
                            --service $FRONTEND_SERVICE \
                            --force-new-deployment \
                            --region $AWS_REGION
                    '''
                }
            }
        }

        stage('Validate Deployment') {
            steps {
                withAWS(credentials: 'aws-credentials', region: "${AWS_REGION}") {
                    sh '''
                        echo "Waiting for services to stabilize..."
                        sleep 60
                        STATUS_B=$(aws ecs describe-services \
                            --cluster $ECS_CLUSTER \
                            --services $BACKEND_SERVICE \
                            --region $AWS_REGION \
                            --query 'services[0].deployments[0].rolloutState' \
                            --output text)
                        STATUS_F=$(aws ecs describe-services \
                            --cluster $ECS_CLUSTER \
                            --services $FRONTEND_SERVICE \
                            --region $AWS_REGION \
                            --query 'services[0].deployments[0].rolloutState' \
                            --output text)
                        echo "Backend status: $STATUS_B"
                        echo "Frontend status: $STATUS_F"
                        echo "Deployment complete!"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully! App is deployed.'
        }
        failure {
            echo 'Pipeline failed! Check the logs above.'
        }
    }
}
