pipeline {
    agent any

    environment {
        AWS_REGION = 'eu-north-1'
        ECR_REPO = 'docker-react-frontend'
        EB_APP_NAME = 'docker-react-frontend'
        EB_ENV_NAME = 'docker-react-frontend-env'
    }

    stages {
        stage('Build Docker Image') {
            steps {
                bat '''
                    docker build -t %ECR_REPO%:latest .
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                bat '''
                    for /f "delims=" %%i in ('aws sts get-caller-identity --query Account --output text') do set AWS_ACCOUNT_ID=%%i
                    set ECR_URL=%AWS_ACCOUNT_ID%.dkr.ecr.%AWS_REGION%.amazonaws.com/%ECR_REPO%
                    
                    aws ecr get-login-password --region %AWS_REGION% | docker login --username AWS --password-stdin %AWS_ACCOUNT_ID%.dkr.ecr.%AWS_REGION%.amazonaws.com
                    docker tag %ECR_REPO%:latest %ECR_URL%:latest
                    docker push %ECR_URL%:latest
                '''
            }
        }

        stage('Deploy to Elastic Beanstalk') {
            steps {
                bat '''
                    echo Creating Dockerrun.aws.json...

                    echo { > Dockerrun.aws.json
                    echo   "AWSEBDockerrunVersion": 2, >> Dockerrun.aws.json
                    echo   "containerDefinitions": [ >> Dockerrun.aws.json
                    echo     { >> Dockerrun.aws.json
                    echo       "name": "reactapp", >> Dockerrun.aws.json
                    echo       "image": "%AWS_ACCOUNT_ID%.dkr.ecr.%AWS_REGION%.amazonaws.com/%ECR_REPO%:latest", >> Dockerrun.aws.json
                    echo       "essential": true, >> Dockerrun.aws.json
                    echo       "memory": 256, >> Dockerrun.aws.json
                    echo       "portMappings": [ { "containerPort": 80 } ] >> Dockerrun.aws.json
                    echo     } >> Dockerrun.aws.json
                    echo   ] >> Dockerrun.aws.json
                    echo } >> Dockerrun.aws.json

                    eb use %EB_ENV_NAME%
                    eb deploy
                '''
            }
        }
    }
}
